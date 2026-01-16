# Mode 2：混合扫描（Slither + 模型验证）

Mode 2 先用 Slither 做静态分析，再用模型对结果做验证与过滤，适合批量扫描与快速筛查，重点降低误报。

## 前置依赖

- Python 3.8+
- Slither（`pip3 install slither-analyzer --break-system-packages`）

如需更好地处理多文件工程与编译器版本问题，建议同时安装：

```bash
pip3 install crytic-compile py-solc-x --break-system-packages
```

## 执行方式

从 DB 扫描：

```bash
go run src/main.go -m mode2 -ai deepseek -t db
```

单地址扫描：

```bash
go run src/main.go -m mode2 -ai deepseek -t contract -addr 0xYourAddress
```

持续监控（新合约目标流式进入扫描队列）：

```bash
go run src/main.go -m mode2 -ai deepseek -t last
```

## 关键参数

- `-concurrency`：并发 worker 数
- `-timeout`：单次 AI 请求超时
- `-proxy`：下载与 AI 请求使用的 HTTP 代理
- `-range`：当 `-t db` 时，过滤区块范围内的合约
- `-s`：选择 Mode 2 的验证模板（为空或 `all` 回退到 `default`）

模板输出格式请参考 [模型输出 JSON 格式指南](ai-json-format.md)（Mode 2 使用验证输出格式）。

## 输出

- 报告默认在 `reports/`（可用 `-r` 指定）
- 详细日志在 `logs/`
