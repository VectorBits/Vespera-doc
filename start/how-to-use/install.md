# 安装与准备

## 环境要求

- Go：1.21+
- MySQL：8.0+（用于存储合约与扫描目标）
- Python：3.8+（Mode 2 使用 Slither 与相关工具链）
- solc（Solidity 编译器）：用于 AST/调用图相关能力（Mode 1/2 都可能用到）
- Slither：Mode 2 必需

如仅使用 Mode 1（定向扫描）且目标合约源码已在 DB 中，Python/Slither 可后置安装。

## 获取代码与依赖

在项目根目录执行：

```bash
go mod download
```

## 初始化配置文件

首次运行会自动生成 `config/settings.yaml`（从内置的 `src/config/settings.example.yaml` 拷贝）。

也可以手动创建：

```bash
mkdir -p config
cp src/config/settings.example.yaml config/settings.yaml
```

然后编辑 `config/settings.yaml` 填写：
- 数据库连接信息
- AI Provider 的 API Key / BaseURL / Model
- 链 RPC 与 Explorer API Key（下载流程会用到）

## 初始化数据库表

```bash
mysql -u root -p < src/config/migrate_tables.sql
```

## 安装 Mode 2 工具链


```bash
pip3 install slither-analyzer crytic-compile py-solc-x
```

## 安装 solc

推荐使用版本管理工具安装（更方便切换不同 Solidity 版本）：

```bash
pip3 install solc-select
solc-select install 0.8.23
solc-select use 0.8.23
solc --version
```

macOS 也可以用 Homebrew 安装（不便于多版本管理）：

```bash
brew install solidity
solc --version
```

还需要 Foundry（用于扁平化等辅助步骤），安装：

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```
