---
summary: "心跳轮询消息和通知规则"
read_when:
  - "调整心跳节拍或消息"
  - "决定心跳和 cron 用于计划任务"
title: "心跳"
---

# 心跳（网关）

> **心跳 vs Cron？** 请参阅 [Cron vs Heartbeat](/automation/cron-vs-heartbeat) 了解何时使用每种的指导。

心跳在主会话中运行**定期代理回合**，以便模型可以呈现任何需要注意的事项，而不会向你发送垃圾信息。

## 快速开始（初学者）

1. 保持心跳启用（默认是 `30m`，或者对于 Anthropic OAuth/setup-token 是 `1h`），或设置你自己的节拍。
2. 在代理工作区创建一个小的 `HEARTBEAT.md` 检查清单（可选但推荐）。
3. 决定心跳消息应该发送到哪里（`target: "last"` 是默认）。
4. 可选：启用心跳推理传递以提高透明度。
5. 可选：将心跳限制在活动时间内（本地时间）。

示例配置：

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // 可选：也发送单独的 `Reasoning:` 消息
      },
    },
  },
}
```

## 默认值

- 间隔：`30m`（或者当检测到的认证模式是 Anthropic OAuth/setup-token 时为 `1h`）。设置 `agents.defaults.heartbeat.every` 或每个代理的 `agents.list[].heartbeat.every`；使用 `0m` 禁用。
- 提示主体（可通过 `agents.defaults.heartbeat.prompt` 配置）：
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- 心跳提示**原样**作为用户消息发送。系统提示包含"心跳"部分，并且运行在内部被标记。
- 活动时间（`heartbeat.activeHours`）在配置的时区中检查。在窗口外，心跳被跳过，直到窗口内的下一个节拍。

## 心跳提示的用途

默认提示是故意宽泛的：

- **后台任务**："考虑待处理任务"促使代理审查跟进（收件箱、日历、提醒、排队的工作）并呈现任何紧急事项。
- **人工签到**："白天有时检查你的人"促使偶尔轻量级的"你需要什么？"消息，但通过使用你配置的本地时区避免夜间垃圾信息（请参阅[/concepts/timezone](/concepts/timezone)）。

如果你想让心跳做非常具体的事情（例如"检查 Gmail PubSub 统计"或"验证网关健康"），设置 `agents.defaults.heartbeat.prompt`（或 `agents.list[].heartbeat.prompt`）为自定义主体（原样发送）。

## 响应契约

- 如果不需要注意，用 **`HEARTBEAT_OK`** 回复。
- 在心跳运行期间，OpenClaw 将 `HEARTBEAT_OK` 视为确认，当它出现在回复的**开头或结尾**时。令牌被剥离，如果剩余内容 **≤ `ackMaxChars`**（默认：300），则丢弃回复。
- 如果 `HEARTBEAT_OK` 出现在回复的**中间**，则不会特别对待。
- 对于警报，**不要**包含 `HEARTBEAT_OK`；只返回警报文本。

在心跳之外，消息开头/结尾的游离 `HEARTBEAT_OK` 被剥离并记录；只有 `HEARTBEAT_OK` 的消息被丢弃。

## 配置

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 默认：30m（0m 禁用）
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // 默认：false（可用时传递单独的 Reasoning: 消息）
        target: "last", // last | none | <channel id>（核心或插件，例如 "bluebubbles"）
        to: "+15551234567", // 可选的通道特定覆盖
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // HEARTBEAT_OK 之后允许的最大字符数
      },
    },
  },
}
```

### 范围和优先级

- `agents.defaults.heartbeat` 设置全局心跳行为。
- `agents.list[].heartbeat` 合并在上面；如果任何代理有 `heartbeat` 块，**只有那些代理**运行心跳。
- `channels.defaults.heartbeat` 为所有通道设置可见性默认值。
- `channels.<channel>.heartbeat` 覆盖通道默认值。
- `channels.<channel>.accounts.<id>.heartbeat`（多账户通道）覆盖每个通道设置。

### 每个代理的心跳

如果任何 `agents.list[]` 条目包含 `heartbeat` 块，**只有那些代理**运行心跳。每个代理块合并在 `agents.defaults.heartbeat` 上面（所以你可以设置一次共享默认值并覆盖每个代理）。

示例：两个代理，只有第二个代理运行心跳。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### 字段说明

- `every`：心跳间隔（持续时间字符串；默认单位 = 分钟）。
- `model`：心跳运行的可选模型覆盖（`provider/model`）。
- `includeReasoning`：启用时，也在可用时传递单独的 `Reasoning:` 消息（与 `/reasoning on` 相同形状）。
- `session`：心跳运行的可选会话键。
  - `main`（默认）：代理主会话。
  - 显式会话键（从 `openclaw sessions --json` 或 [sessions CLI](/cli/sessions) 复制）。
  - 会话键格式：请参阅 [Sessions](/concepts/session) 和 [Groups](/concepts/groups)。
- `target`：
  - `last`（默认）：传递到最后使用的外部通道。
  - 显式通道：`whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
  - `none`：运行心跳但**不**外部传递。
- `to`：可选的收件人覆盖（通道特定 ID，例如 WhatsApp 的 E.164 或 Telegram 聊天 ID）。
- `prompt`：覆盖默认提示主体（不合并）。
- `ackMaxChars`：`HEARTBEAT_OK` 之后允许的最大字符数。

## 传递行为

- 心跳默认在代理的主会话中运行（`agent:<id>:<mainKey>`），或者当 `session.scope = "global"` 时为 `global`。设置 `session` 覆盖为特定的通道会话（Discord/WhatsApp 等）。
- `session` 仅影响运行上下文；传递由 `target` 和 `to` 控制。
- 要传递到特定通道/收件人，设置 `target` + `to`。使用 `target: "last"` 时，传递使用该会话的最后外部通道。
- 如果主队列繁忙，心跳被跳过并稍后重试。
- 如果 `target` 解析为没有外部目的地，运行仍然发生，但不会发送出站消息。
- 仅心跳回复**不会**保持会话存活；恢复最后的 `updatedAt`，使空闲到期行为正常。

## 可见性控制

默认情况下，`HEARTBEAT_OK` 确认被抑制，而警报内容被传递。你可以按通道或按账户调整：

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # 隐藏 HEARTBEAT_OK（默认）
      showAlerts: true # 显示警报消息（默认）
      useIndicator: true # 发出指示器事件（默认）
  telegram:
    heartbeat:
      showOk: true # 在 Telegram 上显示 OK 确认
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # 禁止此账户的警报传递
```

优先级：每个账户 → 每个通道 → 通道默认值 → 内置默认值。

### 每个标志的作用

- `showOk`：当模型返回仅 OK 回复时发送 `HEARTBEAT_OK` 确认。
- `showAlerts`：当模型返回非 OK 回复时发送警报内容。
- `useIndicator`：为 UI 状态表面发出指示器事件。

如果**全部三个**都为 false，OpenClaw 完全跳过心跳运行（没有模型调用）。

### 每个通道 vs 每个账户示例

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 所有 Slack 账户
    accounts:
      ops:
        heartbeat:
          showAlerts: false # 仅禁止 ops 账户的警报
  telegram:
    heartbeat:
      showOk: true
```

### 常见模式

| 目标 | 配置 |
| --- | --- |
| 默认行为（静默 OK，警报开启） | （不需要配置） |
| 完全静默（无消息，无指示器） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| 仅指示器（无消息） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| 仅在一个通道中的 OK | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md（可选）

如果工作区中存在 `HEARTBEAT.md` 文件，默认提示告诉代理读取它。把它想象成你的"心跳检查清单"：小、稳定，并且每 30 分钟包含一次是安全的。

如果 `HEARTBEAT.md` 存在但实际上为空（只有空行和像 `# Heading` 这样的 markdown 标题），OpenClaw 跳过心跳运行以节省 API 调用。如果文件缺失，心跳仍然运行，模型决定做什么。

保持它很小（简短的检查清单或提醒）以避免提示膨胀。

示例 `HEARTBEAT.md`：

```md
# 心跳检查清单

- 快速扫描：收件箱中有什么紧急的吗？
- 如果是白天，如果没有其他待办事项，进行轻量级签到。
- 如果任务被阻止，记下_缺少什么_，下次问 Peter。
```

### 代理可以更新 HEARTBEAT.md 吗？

可以 — 如果你要求它。

`HEARTBEAT.md` 只是代理工作区中的一个正常文件，所以你可以（在正常聊天中）告诉代理类似：

- "更新 `HEARTBEAT.md` 添加每日日历检查。"
- "重写 `HEARTBEAT.md` 使它更短，专注于收件箱跟进。"

如果你想让这主动发生，你也可以在你的心跳提示中包含一行："如果检查清单变得过时，更新 HEARTBEAT.md 更好的一个。"

安全注意：不要将机密（API 密钥、电话号码、私人令牌）放入 `HEARTBEAT.md` — 它成为提示上下文的一部分。

## 手动唤醒（按需）

你可以将系统事件加入队列并触发即时心跳：

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

如果多个代理配置了 `heartbeat`，手动唤醒会立即运行每个这些代理的心跳。

使用 `--mode next-heartbeat` 等待下一个计划的节拍。

## 推理传递（可选）

默认情况下，心跳仅传递最终的"答案"负载。

如果你想要透明度，启用：

- `agents.defaults.heartbeat.includeReasoning: true`

启用时，心跳还将传递以 `Reasoning:` 为前缀的单独消息（与 `/reasoning on` 相同形状）。当代理管理多个会话/codex 并且你想看到它决定 ping 你的原因时，这可能有用 — 但它也可能泄露比你想要的更多的内部细节。在群聊中最好保持关闭。

## 成本意识

心跳运行完整的代理回合。较短的间隔消耗更多令牌。保持 `HEARTBEAT.md` 小，并考虑使用更便宜的 `model` 或 `target: "none"`（如果你只想要内部状态更新）。
