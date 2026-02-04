---
summary: "提权执行模式和 /elevated 指令"
read_when:
  - 调整提权模式默认值、允许列表或斜杠命令行为
title: "提权模式"
---

# 提权模式（/elevated 指令）

## 它做什么

- `/elevated on` 在网关主机上运行并保持执行批准（与 `/elevated ask` 相同）。
- `/elevated full` 在网关主机上运行**并且**自动批准执行（跳过执行批准）。
- `/elevated ask` 在网关主机上运行但保持执行批准（与 `/elevated on` 相同）。
- `on`/`ask` **不**强制 `exec.security=full`；配置的 security/ask 策略仍然适用。
- 仅在代理**沙箱化**时改变行为（否则执行已经在主机上运行）。
- 指令形式：`/elevated on|off|ask|full`、`/elev on|off|ask|full`。
- 仅接受 `on|off|ask|full`；任何其他内容返回提示并不改变状态。

## 它控制什么（和不控制什么）

- **可用性门禁**：`tools.elevated` 是全局基线。`agents.list[].tools.elevated` 可以进一步限制每个代理的提权（两者都必须允许）。
- **每个会话状态**：`/elevated on|off|ask|full` 为当前会话密钥设置提权级别。
- **内联指令**：消息中的 `/elevated on|ask|full` 仅适用于该消息。
- **群组**：在群组聊天中，提权指令仅在代理被提及时才被遵守。绕过提及要求的纯命令消息被视为已提及。
- **主机执行**：提权强制 `exec` 到网关主机；`full` 也设置 `security=full`。
- **批准**：`full` 跳过执行批准；`on`/`ask` 在 allowlist/ask 规则要求时遵守它们。
- **非沙箱化代理**：对位置无操作；仅影响门禁、日志和状态。
- **工具策略仍然适用**：如果 `exec` 被工具策略拒绝，提权不能使用。
- **与 `/exec` 分开**：`/exec` 调整授权发送者的每个会话默认值，不需要提权。

## 解析顺序

1. 消息上的内联指令（仅适用于该消息）。
2. 会话覆盖（通过发送仅指令消息设置）。
3. 全局默认值（配置中的 `agents.defaults.elevatedDefault`）。

## 设置会话默认值

- 发送**仅**指令的消息（允许空格），例如 `/elevated full`。
- 发送确认回复（`Elevated mode set to full...` / `Elevated mode disabled.`）。
- 如果提权访问被禁用或发送者不在批准的 allowlist 中，指令回复可操作错误并不改变会话状态。
- 发送不带参数的 `/elevated`（或 `/elevated:`）以查看当前提权级别。

## 可用性 + 允许列表

- 功能门禁：`tools.elevated.enabled`（即使代码支持，默认也可以通过配置关闭）。
- 发送者允许列表：`tools.elevated.allowFrom` 带有每个提供程序的 allowlist（例如 `discord`、`whatsapp`）。
- 每个代理门禁：`agents.list[].tools.elevated.enabled`（可选；只能进一步限制）。
- 每个代理允许列表：`agents.list[].tools.elevated.allowFrom`（可选；设置时，发送者必须匹配**全局** + **每个代理** allowlist）。
- Discord 回退：如果 `tools.elevated.allowFrom.discord` 省略，`channels.discord.dm.allowFrom` 列表用作回退。设置 `tools.elevated.allowFrom.discord`（即使是 `[]`）以覆盖。每个代理允许列表**不**使用回退。
- 所有门禁必须通过；否则提权被视为不可用。

## 日志 + 状态

- 提权执行调用在 info 级别记录。
- 会话状态包括提权模式（例如 `elevated=ask`、`elevated=full`）。
