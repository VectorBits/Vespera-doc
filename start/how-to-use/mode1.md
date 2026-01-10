# Mode 1：定向扫描

Mode 1 适合做“有明确目标的审计与验证”：通过模板（`.tmpl`）与可选的漏洞特征（`.toml`）组织分析输入，对合约进行审计分析，并输出结构化结果与可读报告。

## 执行方式

单地址扫描：

```bash
go run src/main.go -m mode1 -ai deepseek -t contract -addr 0xYourAddress
```

从 DB 扫描（最近 1000 个开源合约，或按区块范围过滤）：

```bash
go run src/main.go -m mode1 -ai deepseek -t db
go run src/main.go -m mode1 -ai deepseek -t db -range 20000000-20001000
```

从文件扫描：

```bash
go run src/main.go -m mode1 -ai deepseek -t file -file ./targets.yaml
```

## 输入文件 `-i`（可选）

Mode 1 支持用 TOML 文件描述漏洞特征（位于 `src/strategy/exp_libs/mode1/`）。

扫描单个特征文件：

```bash
go run src/main.go -m mode1 -ai deepseek -i hourglassvul.toml -t contract -addr 0xYourAddress
```

扫描目录下所有 `.toml`：

```bash
go run src/main.go -m mode1 -ai deepseek -i all -t db
```

## 模板 `-s`

`-s` 对应 `src/strategy/prompts/mode1/` 下的模板（不带扩展名）。

在未提供 `-i` 的情况下：
- 若 `-s` 为空或为 `default`，会强制使用 `generic_scan`
- `generic_scan` / `callgraph_enhanced` 这类模板可不依赖 TOML 直接运行

模板输出格式请参考 [模型输出 JSON 格式指南](ai-json-format.md)。

## 输出

- 报告默认在 `reports/`（可用 `-r` 指定）
- 详细日志在 `logs/`
