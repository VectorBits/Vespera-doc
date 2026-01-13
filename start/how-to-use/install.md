# 安装与准备

## 通过 curl 安装（推荐）

项目提供一键安装脚本，用于安装依赖并初始化本地配置与数据库：

```bash
curl -fsSL http://github.com/vectorbits/vespera/install.sh | bash
```

脚本会完成：
- 安装 Mode 2 工具链（Slither / crytic-compile / py-solc-x）
- 安装 solc（通过 solc-select）与 solcx 相关环境
- 安装 Foundry
- 交互式输入 MySQL 连接信息，并写入 `config/settings.yaml`
- 创建数据库并执行 `config/migrate_tables.sql`

## 从源码构建（Build from Source）

- Go：1.21+
- MySQL：8.0+（用于存储合约与扫描目标）
- Python：3.8+（Mode 2 使用 Slither 与相关工具链）
- solc (建议安装全部版本。其实这里也可以使用静态二进制的,但是后来觉得也没必要。如果有bug后面再修改)
- Slither：Mode 2 必需

### 获取代码与依赖

在项目根目录执行：

```bash
go mod download
```

### 初始化配置文件

首次运行会自动生成 `config/settings.yaml`


编辑 `config/settings.yaml` 填写：
- 数据库连接信息
- AI 提供商的 API Key / BaseURL / Model
- 链 RPC 与 Explorer API Key（下载流程会用到）

### 初始化数据库表

```bash
mysql -u root -p < config/migrate_tables.sql
```

### 安装 Mode 2 依赖
```bash
pip3 install slither-analyzer crytic-compile py-solc-x
```

### 安装 solc

推荐使用版本管理工具安装（更方便切换不同 Solidity 版本）：

```bash
pip3 install solc-select
solc-select install 0.8.23  
solc-select use 0.8.23
solc --version
```

### 安装 Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### 编译

```bash
go run src/main.go -o vespera
```
