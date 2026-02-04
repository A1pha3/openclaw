---
summary: "/think、/verbose 和 /reasoning 指令语法及其对模型推理的影响"
read_when:
  - 调整 thinking 或 verbose 指令解析或默认值
title: "思考级别"
---

# 思考级别（/think 指令）

## 功能说明

通过内联指令控制模型的推理深度：`/t <级别>`、`/think:<级别>` 或 `/thinking <级别>`

### 可用级别

| 级别 | 别名 | 说明 |
|------|------|------|
| `off` | - | 关闭扩展思考 |
| `minimal` | - | 基础思考（"think"） |
| `low` | - | 轻度思考（"think hard"） |
| `medium` | - | 中度思考（"think harder"） |
| `high` | `highest`、`max` | 深度思考（"ultrathink"，最大预算） |
| `xhigh` | - | 超深度思考（"ultrathink+"，仅 GPT-5.2 + Codex 模型） |

### 提供商说明

- **Z.AI (`zai/*`)**：仅支持二元思考（`on`/`off`）。任何非 `off` 级别都视为 `on`（映射到 `low`）

## 级别解析顺序

1. 消息内联指令（仅对该消息生效）
2. 会话覆盖（通过发送纯指令消息设置）
3. 全局默认值（配置中的 `agents.defaults.thinkingDefault`）
4. 回退值：支持推理的模型为 `low`；否则为 `off`

## 设置会话默认值

发送**仅包含**指令的消息（允许空白），例如 `/think:medium` 或 `/t high`：

- 设置后对当前会话生效（默认按发送者区分）
- 通过 `/think:off` 或会话空闲重置清除
- 收到确认回复：`Thinking level set to high.` / `Thinking disabled.`
- 如果级别无效（如 `/thinking big`），命令被拒绝并提示，会话状态不变
- 发送 `/think`（或 `/think:`）不带参数可查看当前思考级别

## Verbose 指令（/verbose 或 /v）

控制工具调用的详细程度。

### 可用级别

| 级别 | 说明 |
|------|------|
| `on` | 最小详细（显示工具调用摘要） |
| `full` | 完全详细（工具调用 + 输出） |
| `off` | 关闭（默认） |

### 行为说明

- 纯指令消息切换会话 verbose 并回复 `Verbose logging enabled.` / `Verbose logging disabled.`
- 无效级别返回提示但不改变状态
- `/verbose off` 存储显式会话覆盖；通过会话 UI 选择 `inherit` 清除
- 内联指令仅影响该消息；否则应用会话/全局默认值
- 发送 `/verbose`（或 `/verbose:`）不带参数可查看当前 verbose 级别

### 工具输出显示

当 verbose 开启时：
- 发出结构化工具结果的 agent 会将每个工具调用作为单独的元数据消息发送
- 格式：`<emoji> <工具名>: <参数>`（路径/命令，如果可用）
- 这些工具摘要在每个工具开始时发送（单独气泡），而非流式增量

当 verbose 为 `full` 时：
- 工具输出在完成后也会转发（单独气泡，截断到安全长度）
- 如果在运行中切换 `/verbose on|full|off`，后续工具气泡遵循新设置

## 推理可见性（/reasoning）

控制思考块是否在回复中显示。

### 可用级别

| 级别 | 说明 |
|------|------|
| `on` | 显示推理（作为单独消息，前缀 `Reasoning:`） |
| `off` | 隐藏推理 |
| `stream` | 仅 Telegram：流式推理到草稿气泡，最终答案不含推理 |

别名：`/reason`

发送 `/reasoning`（或 `/reasoning:`）不带参数可查看当前推理级别。

## 心跳探测

- 心跳探测内容是配置的心跳提示词（默认：`如果存在 HEARTBEAT.md 则读取（工作区上下文）。严格遵循它。不要从之前的聊天推断或重复旧任务。如果没有需要注意的事项，回复 HEARTBEAT_OK。`）
- 心跳消息中的内联指令照常应用（但避免从心跳更改会话默认值）
- 心跳传递默认仅最终负载。要同时发送单独的 `Reasoning:` 消息（如果可用），设置 `agents.defaults.heartbeat.includeReasoning: true` 或每 agent `agents.list[].heartbeat.includeReasoning: true`

## Web 聊天 UI

- Web 聊天思考选择器在页面加载时反映入站会话存储/配置中的会话存储级别
- 选择另一个级别仅对下一条消息生效（`thinkingOnce`）；发送后选择器恢复到存储的会话级别
- 要更改会话默认值，发送 `/think:<级别>` 指令；选择器将在下次重新加载后反映

## 相关文档

- 提权模式文档位于 [提权模式](/zh-cn/tools/elevated)
