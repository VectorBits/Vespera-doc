# Mode 1：定向扫描

Mode 1 面向“目标明确”的审计式扫描：用 Prompt 模板（`.tmpl`）驱动分析，可选叠加漏洞特征（`.toml`），最终产出结构化结果与 Markdown 报告。

## 什么时候用

- 你已经知道要扫哪些合约（单个地址 / 地址列表 / DB 目标）
- 你希望输出稳定的结构化结果（用于报告与后处理）

## 最常用命令

单地址：

```bash
go run src/main.go -m mode1 -ai deepseek -t contract -addr 0xYourAddress
```

从 DB 扫描（可选用 `-range start-end` 按 createblock 过滤）：

```bash
go run src/main.go -m mode1 -ai deepseek -t db
go run src/main.go -m mode1 -ai deepseek -t db -range 20000000-20001000
```

从文件扫描：

```bash
go run src/main.go -m mode1 -ai deepseek -t file -file ./targets.txt
```

完整参数与 `-t` 的智能解析规则请看 [CLI 参数](cli.md)。

## 两个可调入口

- 模板 `-s`：对应 `strategy/prompts/mode1/<name>.tmpl`（不带扩展名）
  - `-s` 为空或 `default`：回退为 `generic_scan`
  - 例如 `generic_scan` / `callgraph_enhanced` 可不依赖 TOML 直接运行
- 特征 `-i`（可选）：对应 `strategy/exp_libs/mode1/` 下的 `.toml`
  - 扫描单个特征：`-i hourglassvul.toml`
  - 扫描目录全部特征：`-i all`

```bash
go run src/main.go -m mode1 -ai deepseek -i hourglassvul.toml -t contract -addr 0xYourAddress
go run src/main.go -m mode1 -ai deepseek -i all -t db
```

模板输出约束请参考 [模型输出 JSON 格式指南](ai-json-format.md)（Mode 1 的 AnalysisResult）。

## 输出

- 报告默认在 `reports/`（可用 `-r` 指定）
- 详细日志在 `logs/`
更多输出说明见 [报告与日志](report-and-logs.md)。
