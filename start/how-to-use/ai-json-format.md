# 模型输出 JSON 格式指南

本指南描述 Vespera（solidity-Excavator）解析器所期望的 JSON 输出格式。编写 Prompt 模板时，应通过输出约束确保模型遵循该结构，以便报告生成器稳定解析并生成报告。

代码参考：解析逻辑位于 [parser.go](../../../src/internal/ai/parser/parser.go)。

## Prompt 模板规则（Mode 1）

Mode 1 的模板位于 `src/strategy/prompts/mode1/*.tmpl`，用于组织“输入上下文”与“输出约束”。模板由两部分组成：

1. **上下文输入**：合约信息、代码、可选的调用图上下文。
2. **输出要求**：要求模型输出严格 JSON，字段名使用 snake_case。

### 上下文输入结构

模板通常包含以下可选字段，实际是否渲染由条件控制：

- `{{if .EnableCallGraph}} ... {{end}}`：是否启用调用图增强上下文。
- `{{.CallGraphInfo}}`：调用图统计信息，用于概览调用关系。
- `{{.CallersCode}}`：调用者代码片段（谁调用了公开函数）。
- `{{.CalleesCode}}`：被调用者代码片段（公开函数调用了谁）。

Mode 1 的调用图增强模板示例可参考 [callgraph_enhanced.tmpl](../../../src/strategy/prompts/mode1/callgraph_enhanced.tmpl)。

### 输出要求结构

输出必须是“纯 JSON 文本”，不要包含 Markdown 代码块。字段名一律使用 snake_case。模板可输出额外字段（例如 `call_graph_analysis`），解析器会忽略未识别字段。

建议在模板的“输出要求”中固定以下约束片段，并按本页的字段说明输出：

```text
输出要求：
请仅输出一个合法的 JSON 对象，不要包含 Markdown 标记（如 ```json）。字段名必须使用 snake_case。建议始终返回 vulnerabilities（无漏洞返回空数组 []）。除下述字段外，可输出额外字段，解析器会忽略未识别字段。
```

### 提示词解析与变量注入

提示词模板使用 Go 的 `text/template` 执行，变量来源与注入流程如下：

- 模板加载：通过 [template_loader.go](../../../src/strategy/prompts/template_loader.go) 读取 `strategy/prompts/<mode>/<name>.tmpl`（支持从 `src/strategy/...` 目录读取）。
- 输入文件处理（Mode 1 可选）：`-i` 指定的 TOML/sol 文件会被 [template_loader.go](../../../src/strategy/prompts/template_loader.go) 预处理为纯文本，并写入 `InputFileContent` 变量。
- Mode 1 变量注入：扫描时构建 [PromptVariables](../../../src/strategy/prompts/builder.go)，并在 [mode1_targeted.go](../../../src/internal/handler/mode1_targeted.go) 中传入模板执行。
- Mode 2 变量注入：逐条验证时在 [mode2_fuzzy.go](../../../src/internal/handler/mode2_fuzzy.go) 以 `map` 方式注入探测器字段（`DetectorCheck/Impact/Confidence/Description` 与 `LineNumbers`）。

#### Mode 1 可用变量

- `ContractAddress`：目标合约地址
- `ContractCode`：目标合约源码（可能是清洗/裁剪后的版本）
- `Strategy`：当前模板/策略名
- `InputFileContent`：`-i` 输入文件处理后的内容
- `EnableCallGraph`：是否启用调用图上下文
- `CallGraphInfo`：调用图概要信息
- `CallersCode`：调用者代码片段
- `CalleesCode`：被调用者代码片段
- `EnrichedContext`：完整调用链上下文
- `TotalFunctions`：函数总数
- `PublicFunctions`：公开函数数量
- `InternalFunctions`：内部函数数量

#### 调用图上下文触发条件

模板中只要出现 `.EnableCallGraph` / `.CallGraphInfo` / `.CallersCode` / `.CalleesCode` / `.EnrichedContext` 任意一个字段，就会在预处理阶段构建调用图上下文（见 [mode1_targeted.go](../../../src/internal/handler/mode1_targeted.go)）。

#### 统一输出约束注入

模型请求发送前，解析器会在提示词末尾拼接固定的输出结构要求；如果模板未显式包含 Solidity 代码块，也会自动附加合约代码片段（见 [ai_manager.go](../../../src/internal/ai/ai_manager.go)）。

## Mode 1（定向扫描）输出格式：AnalysisResult

Mode 1 的 `.tmpl` 模板建议输出一个根对象 `AnalysisResult`，用于生成风险摘要与漏洞列表。

### 输出约束片段（用于 `.tmpl` 的“输出要求”部分）

```text
输出要求：
请仅输出一个合法的 JSON 对象，不要包含 Markdown 标记（如 ```json）。字段名必须使用 snake_case。建议始终返回 vulnerabilities（无漏洞返回空数组 []）。除下述字段外，可输出额外字段，解析器会忽略未识别字段。

{
  "contract_address": "0x...（可选）",
  "risk_score": 0,
  "vuln_probability": "High/Medium/Low 或 85%（可选）",
  "severity": "Critical/High/Medium/Low/None/Unknown（可选；Safe 也可用，会被当作 None）",
  "summary": "简明扼要的审计总结，概括主要发现（可选）",
  "recommendations": ["建议 1...", "建议 2...（可选）"],
  "vulnerabilities": [
    {
      "type": "漏洞类型（如 Reentrancy）",
      "severity": "Critical/High/Medium/Low",
      "description": "详细描述：缺陷原因、攻击路径、潜在危害",
      "location": "文件名:行号 或 地址/合约名（可选）",
      "line_numbers": [42, 45],
      "code_snippet": "关键代码片段（可选）",
      "impact": "漏洞影响（可选）",
      "remediation": "修复建议（可选）",
      "references": ["参考链接 1（可选）"],
      "swc_id": "SWC-XXX（可选）"
    }
  ]
}
```

### 字段说明（与解析器结构一致）

根对象（AnalysisResult）：

| JSON 字段名 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `contract_address` | `string` | 否 | 合约地址（如果 Prompt 上下文包含）。 |
| `risk_score` | `number` / `string` | 否 | 风险评分（0-100），解析器允许数字或字符串。 |
| `vuln_probability` | `string` | 否 | 漏洞概率（如 `High`、`85%`）。 |
| `severity` | `string` | 否 | 合约整体风险等级（推荐：`Critical/High/Medium/Low/None/Unknown`；`Safe` 会被当作 `None`）。 |
| `summary` | `string` | 否 | 审计总结。 |
| `recommendations` | `[]string` | 否 | 整体改进建议列表。 |
| `vulnerabilities` | `array` | 建议 | 漏洞列表；无漏洞建议返回空数组 `[]`。 |
| `parse_error` | `string` | 否 | 该字段由解析器写入；模板不需要输出。 |

漏洞对象（Vulnerability）：

| JSON 字段名 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `type` | `string` | 是 | 漏洞类型（如 `Reentrancy`、`AccessControl`）。 |
| `severity` | `string` | 是 | 严重程度（推荐：`Critical/High/Medium/Low`）。 |
| `description` | `string` | 是 | 漏洞详情描述（原因、路径、影响）。 |
| `location` | `string` | 否 | 位置描述（如 `Vault.sol:L42`）。 |
| `line_numbers` | `[]int` | 否 | 行号列表（纯数字数组）。 |
| `code_snippet` | `string` | 否 | 相关代码片段。 |
| `impact` | `string` | 否 | 漏洞影响。 |
| `remediation` | `string` | 否 | 修复方案。 |
| `references` | `[]string` | 否 | 参考链接/线索。 |
| `swc_id` | `string` | 否 | SWC ID（如 `SWC-107`）。 |

## Mode 2（混合扫描）输出格式：AIVerificationResult

Mode 2 会用 Slither 先产出 detectors，再对每条结果进行“逐条验证”。因此 Mode 2 的模板（`src/strategy/prompts/mode2/*.tmpl`）需要输出更轻量的验证对象。

```json
{
  "is_vulnerability": true,
  "severity": "Critical/High/Medium/Low/None/Unknown（Safe 会被当作 None）",
  "reason": "简要说明判断理由（建议 100 字以内）",
  "vuln_type": "保持与 detector 名称一致"
}
```

| JSON 字段名 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `is_vulnerability` | `bool` | 是 | 真实漏洞为 `true`，误报为 `false`。 |
| `severity` | `string` | 是 | 严重程度；若为误报请填 `None`。 |
| `reason` | `string` | 是 | 判断理由（尽量短且可复核）。 |
| `vuln_type` | `string` | 是 | 漏洞类型/检测器名称（建议原样输出）。 |

## 注意事项

1. JSON 必须是标准格式：建议输出“纯 JSON 文本”，不要加代码块；解析器会尝试清理部分 Markdown 包裹，但不保证覆盖所有异常格式。
2. 字段名区分大小写：必须使用 snake_case（例如 `risk_score`），不要用 `riskScore`。
3. 空值与缺省：非必填字段可省略；但建议输出 `vulnerabilities`（无漏洞返回 `[]`），避免“漏报/误报”的语义歧义。
4. 额外字段：Mode 1 的部分模板会输出额外字段（例如 `call_graph_analysis`），解析器会自动忽略，不影响报告生成。
