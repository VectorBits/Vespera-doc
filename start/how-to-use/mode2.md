# Mode 2：混合扫描（Slither + 模型验证）

Mode 2 先用 Slither 做静态分析，再用模型对结果做验证与过滤，适合批量扫描与快速筛查，重点降低误报。

## 什么时候用

- 你要批量快速筛查（DB 目标 / 流式新目标）
- 你更关心降低误报，而不是“审计式长文本分析”

## 前置依赖

- Python 3.8+
- Slither：

```bash
pip3 install slither-analyzer --break-system-packages
```

可选（更好处理多文件工程与编译器版本问题）：

```bash
pip3 install crytic-compile py-solc-x --break-system-packages
```

依赖与编译器相关安装建议见 [安装与准备](install.md)。

## 最常用命令

从 DB 扫描：

```bash
go run src/main.go -m mode2 -ai deepseek
```

单地址扫描：

```bash
go run src/main.go -m mode2 -ai deepseek -t 0xYourAddress
```

持续监控（新合约目标流式进入扫描队列）：

```bash
go run src/main.go -m mode2 -ai deepseek -t last
```

## 关键参数

- `-concurrency`：并发 worker 数
- `-timeout`：单次 AI 请求超时
- `-proxy`：下载与 AI 请求使用的 HTTP 代理
- `-range`：DB 目标下的区块过滤（例如 1000-2000）
- `-s`：选择 Mode 2 的验证模板（为空或 `all` 回退到 `default`）

完整参数请看 [CLI 参数](cli.md)。

模板输出格式请参考 [模型输出 JSON 格式指南](ai-json-format.md)（Mode 2 使用验证输出格式）。

## 输出

- 报告默认在 `reports/`（可用 `-r` 指定）
- 详细日志在 `logs/`
更多输出说明见 [报告与日志](report-and-logs.md)。
