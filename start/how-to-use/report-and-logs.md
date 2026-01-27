# 报告与日志

## 报告（Markdown）

- 默认输出目录：`reports/`
- 可用 `-r` 指定输出目录，例如：

```bash
go run src/main.go -m mode1 -ai deepseek -t contract -addr 0xYourAddress -r ./out/reports
```

报告内容通常包含：
- 扫描模式、策略与 AI 提供商
- 每个合约的扫描状态
- 命中的漏洞列表（类型/严重性/描述）
- 可选的摘要与原始响应片段

## 日志

- 日志目录：`logs/`
- 用于排查下载失败、RPC 切换、Slither 执行失败、模型输出解析失败等问题

## 常见排错方向

- 报告为空但日志正常：检查目标源是否为空（DB/文件/地址）
- Slither 失败：通常是编译器版本/依赖/多文件工程处理问题
- 输出解析失败：查看日志中的原始响应，检查模板与输出格式是否匹配
