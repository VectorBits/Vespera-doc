# CLI 参数

Vespera 的 CLI 以 `-t` 的“智能解析”为入口，并配合统一参数：`-range / -addr / -file`。旧参数仅用于兼容，不作为示例重点。

## 最常用的 4 条命令

扫描单个合约（Mode 1）：

```bash
go run src/main.go -m mode1 -ai deepseek -t 0xYourAddress -s dao_governance -i dao_governance
```

扫描数据库中的目标（Mode 1 / Mode 2）：

```bash
go run src/main.go -m mode1 -ai deepseek -s dao_governance -i dao_governance
go run src/main.go -m mode2 -ai deepseek
```

按区块范围扫描 DB 目标（`start-end` 会被识别为 DB 范围过滤）：

```bash
go run src/main.go -m mode1 -ai deepseek -t 20000000-20001000 -s dao_governance -i dao_governance
```

从地址文件扫描（文件路径会被自动识别）：

```bash
go run src/main.go -ai deepseek -m mode1 -t ./Contract-list/gvtest.txt -s dao_governance -i dao_governance -c eth -proxy http://127.0.0.1:7897
```

## 扫描参数

### `-m`（模式）
- `mode1`：定向扫描（模板驱动，适合“明确目标”）
- `mode2`：混合扫描（Slither + AI 验证，适合批量快速筛查）

### `-ai`（模型/提供商）
- `chatgpt5` / `openai` / `gpt4`：读取 `settings.yaml` 的 `ai.openai`
- `deepseek`：读取 `ai.deepseek`
- `gemini`：读取 `ai.gemini`
- `local-llm` / `ollama`：读取 `ai.local_llm`（不需要 `api_key`）

### `-s`（提示词模板）
- 模板位置：`strategy/prompts/<mode>/<name>.tmpl`（也支持从 `src/strategy/...` 运行）
- `mode1` 下 `-s default`（或空/`all`）会回退到 `generic_scan`
- `mode2` 下 `-s default`（或空）会回退到 `default`
- 模板输出 JSON 需符合 [模型输出 JSON 格式指南](ai-json-format.md)

### `-i`（输入特征文件，仅 Mode 1）
- 用于 Mode 1 的漏洞特征/实验库（TOML）
- 支持写文件名或相对路径：
  - `hourglassvul.toml`（会在 `strategy/exp_libs/mode1/` 或 `src/strategy/exp_libs/mode1/` 中查找）
  - `./path/to/xxx.toml`（按你给的路径读取）
- `-i all`：扫描 `strategy/exp_libs/mode1/*.toml`（找不到会回退尝试 `src/strategy/exp_libs/mode1/*.toml`）

### `-t`（目标来源 / 智能解析）
- 推荐用法：直接把“地址 / 文件 / 区块范围”传给 `-t`，程序会自动识别
- 也支持显式指定来源：`db | file | contract | last`（一般不需要）

常用写法（更短）：

```bash
# 地址 / 文件 / 范围会被自动识别
go run src/main.go -m mode1 -ai deepseek -t 0xYourAddress -s dao_governance -i dao_governance

go run src/main.go -m mode1 -ai deepseek -t ./targets.txt

go run src/main.go -m mode2 -ai deepseek -t 20000000-20001000
```

### `-addr`（单地址）
- 一般不需要显式填写：`-t 0x...` 会自动写入目标地址
- 仍支持显式指定（一般不需要）

### `-file`（目标文件）
- 一般不需要显式填写：`-t ./targets.txt` 会自动识别为目标文件
- `-d`：下载地址文件
- 文件内容：一行一个地址（最稳妥）；也支持 YAML 列表/包装结构（由读取器决定）

### `-range`（区块范围）
- 格式：`start-end`
- `-t` 传入 `start-end` 会自动按 DB 中的 `createblock` 过滤目标
- `-d`：按区块范围下载

### `-c`（链）  可以自定义配置更多链
- `eth | bsc | base`

### `-concurrency`
- 并发 worker 数（默认 4）

### `-timeout`
- 单次 AI 请求超时（Go `time.Duration`，如 `60s`、`2m`；默认 `120s`）

### `-proxy`
- HTTP 代理，例如 `http://127.0.0.1:7897`
- 同时影响下载/explorer/RPC 相关请求与 AI 请求

### `-r`
- 报告输出目录（默认 `reports`）

### `-v`
- 更详细的运行信息；并影响 AI 日志（Verbose 才记录完整 prompt/response，否则记录 hash）

## 下载参数

### `-d`
- 启动下载流程（写入 MySQL）
- 优先级：
  - `-file`：按地址列表下载
  - `-range`：按区块范围下载
  - 否则：从 DB 记录的最后区块继续

## Benchmark（实验性）

### `-b` / `--benchmark`
- 运行基准测试（本质上批量调用 mode1）

### `--database`
- 数据集路径（默认 `benchmark/dataset.json`）

## 已弃用参数（兼容用，不推荐）
旧参数仍可用，但不会在示例中使用：
- `-d-range` → `-range`
- `-t-block` → `-range`
- `-t-file` → `-file`
- `-t-address` → `-addr`
