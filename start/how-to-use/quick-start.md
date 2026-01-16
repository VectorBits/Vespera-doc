# 快速开始

以下命令默认在项目根目录执行。

## 1. 运行方式

首次使用可直接运行（会自动生成默认配置文件）：

```bash
go run src/main.go --help
```

此时会在 `config/settings.yaml` 生成默认配置。请编辑该文件填入你的数据库账号密码和 API Key。
**无需手动建表，程序连接数据库后会自动初始化。**

也可以先编译：

```bash
go build -o vespera src/main.go
./vespera --help
```

## 2. 下载合约到数据库（可选，推荐）

按区块范围下载：

```bash
go run src/main.go -d -c eth -range 20000000-20001000
```

按地址文件下载（每行一个地址）：

```bash
go run src/main.go -d -c eth -file ./addresses.txt
```

## 3. Mode 1（定向扫描）

扫描单个合约地址：

```bash
go run src/main.go -m mode1 -ai deepseek -t contract -addr 0xYourAddress
```

从数据库批量扫描（最近 1000 个开源合约，或带区块范围过滤）：

```bash
go run src/main.go -m mode1 -ai deepseek -t db
go run src/main.go -m mode1 -ai deepseek -t db -range 20000000-20001000
```

从文件扫描：

```bash
go run src/main.go -m mode1 -ai deepseek -t file -file ./targets.yaml
```

## 4. Mode 2（混合扫描：Slither + 模型验证）

从数据库批量扫描：

```bash
go run src/main.go -m mode2 -ai deepseek -t db
```

扫描单个地址：

```bash
go run src/main.go -m mode2 -ai deepseek -t contract -addr 0xYourAddress
```

## 5. 查看输出

- 扫描报告：默认输出到 `reports/`，可用 `-r` 指定目录
- 扫描日志：输出到 `logs/`
