# CLI 参数

推荐优先使用“统一参数”（`-range` / `-addr` / `-file`），避免使用已弃用的老参数。

## 扫描相关

### `-m`
- 扫描模式：`mode1` | `mode2` | `mode3`（mode3 目前未实现）

### `-ai`
- AI 提供商：`chatgpt5` | `deepseek` | `gemini` | `local-llm`

### `-s`
- 策略/提示词模板名称
- Mode 1 默认 `generic_scan`
- Mode 2 为空或 `all` 时会回退到 `default`
- 模板路径：`src/strategy/prompts/<mode>/<name>.tmpl`
- 模板输出 JSON 需符合 [模型输出 JSON 格式指南](ai-json-format.md)

### `-t`
- 目标来源：`db` | `file` | `contract` | `last`
- 支持写成 `type:value` 的形式（例如 `-t contract:0x...`、`-t file:./targets.txt`）

### `-range`
- 区块范围：`start-end`
- 对 `-t db` 生效（过滤 DB 中 createblock 范围）
- 对 `-d` 下载也生效

### `-addr`
- 目标合约地址
- 配合 `-t contract`

### `-file`
- 目标文件路径（当 `-t file` 时）或下载地址文件（当 `-d` 时）
- 文件内容支持：
  - 一行一个地址（txt）
  - YAML 列表（`- file...`）
  - YAML 包装结构：`targets:` 或 `addresses:`

### `-i`
- Mode 1 专用：指定漏洞特征 TOML 文件（位于 `src/strategy/exp_libs/mode1/`）
- 可选值：
  - 具体文件名（或路径）
  - `all`（扫描 `exp_libs/mode1/` 目录下全部 `.toml`）

### `-concurrency`
- 并发 worker 数，默认 4

### `-timeout`
- 单次 AI 请求超时（`time.Duration` 格式，例如 `60s`、`2m`），默认 `120s`

### `-r`
- 报告输出目录，默认 `reports`

### `-proxy`
- HTTP 代理，例如 `http://127.0.0.1:7897`
- 影响下载/请求 explorer 与 AI 请求

### `-c`
- 链名称：`eth` | `bsc` | `base`

### `-v`
- 打印更详细的配置与运行信息

## 下载相关

### `-d`
- 启动下载流程（写入 MySQL）
- 行为优先级：
  1) 如果提供 `-file`：按地址列表下载  
  2) 否则如果提供 `-range`：按区块范围下载  
  3) 否则：从数据库记录的最后区块继续下载

## Benchmark

### `-b` / `--benchmark`
- 运行基准测试（本质上批量调用 mode1）

### `--database`
- 基准数据集文件路径，默认 `benchmark/dataset_defihack.json`

## 已弃用参数（不推荐）

- `-d-range`：请用 `-range`
- `-t-block`：请用 `-range`
- `-t-file`：请用 `-file`
- `-t-address`：请用 `-addr`
