# 配置文件 settings.yaml

## 配置文件位置

程序会按以下路径顺序寻找 `settings.yaml`：

1. `config/settings.yaml`（推荐）
2. `settings.yaml`
3. `src/config/settings.yaml`
4. `../config/settings.yaml`

首次运行会自动在 `config/settings.yaml` 生成默认配置（来自 `src/config/settings.example.yaml`）。

## 配置结构

### 模型配置（AI）

```yaml
ai:
  openai:
    api_key: "sk-..."
    base_url: "https://api.openai.com/v1"
    model: "gpt-4-turbo"
  deepseek:
    api_key: "sk-..."
    base_url: "https://api.deepseek.com/v1"
    model: "deepseek-chat"
  gemini:
    api_key: "..."
    base_url: "https://generativelanguage.googleapis.com/v1beta"
    model: "gemini-1.5-pro"
  local_llm:
    base_url: "http://localhost:11434"
    model: "llama2"
```

### 链与 Explorer 配置

```yaml
chains:
  eth:
    rpc_urls:
      - "https://..."
    explorer:
      api_keys:
        - "etherscan-key-1"
        - "etherscan-key-2"
      base_url: "https://api.etherscan.io/v2"
    table_name: "ethereum"
```

说明：
- `rpc_urls` 支持多个，会用于轮询/故障切换
- `explorer.api_keys` 支持多个，适合分摊限速（下载流程会用到）
- `table_name` 是数据库表名映射（程序启动时会自动根据此名创建/迁移表）

### 数据库配置示例

```yaml
database:
  host: "localhost"
  port: "3306"
  user: "root"
  password: "123456"
  name: "solidity_excavator"
```

> **注意**：只需在配置中指定数据库名（默认为 `solidity_excavator`）。程序启动时，如果该数据库不存在，会自动创建；所有配置的 `table_name` 对应的表也会自动创建。无需手动运行 SQL 脚本。

## 常见配置组合

- 只做扫描：建议配置数据库，用于缓存合约源码与扫描目标
- 下载 + 批量扫描：同时配置 `chains.*.rpc_urls` 与 `chains.*.explorer.api_keys`
