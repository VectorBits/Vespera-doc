# 快速开始

以下命令默认在项目根目录执行。首次运行会自动创建 `config/settings.yaml`（从内置的 `config/settings.example.yaml` 复制）。

## 1) 初始化配置

```bash
go run src/main.go -h
```

打开并填写：
- `config/settings.yaml` 的数据库连接信息
- 你要使用的 AI 提供商的 `api_key / base_url / model`

程序连接数据库后会自动创建数据库与表（按 `settings.yaml` 里的 `chains.*.table_name`）。

## 2)（可选）先把合约拉进数据库

按区块范围下载：

```bash
go run src/main.go -d -c eth -range 20000000-20001000
```

按地址文件下载（建议一行一个地址）：

```bash
go run src/main.go -d -c eth -file ./addresses.txt
```

## 3) 扫描

Mode 1：定向扫描（模板驱动）

```bash
go run src/main.go -m mode1 -ai deepseek -t 0xYourAddress -s dao_governance -i dao_governance
```

Mode 2：混合扫描（Slither + AI 验证）

```bash
go run src/main.go -m mode2 -ai deepseek
```

## 4) 查看输出

- 报告：默认 `reports/scan_report_<mode>_<timestamp>.md`（用 `-r` 修改目录）
- 日志：默认 `logs/scan_<yyyy-mm-dd_hh-mm-ss>.log`
