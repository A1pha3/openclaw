---
summary: "网关调度器的 Cron 作业 + 唤醒"
read_when:
  - "调度后台作业或唤醒"
  - "连接应该与心跳一起或旁边运行的自动化"
  - "在心跳和 cron 之间为计划任务做出选择"
title: "Cron 作业"
---

# Cron 作业（网关调度器）

> **Cron vs Heartbeat？** 有关何时使用每种的指导，请参阅 [Cron vs Heartbeat](/automation/cron-vs-heartbeat)。

Cron 是网关的内置调度器。它持久化作业，在正确的时间唤醒代理，并且可以选择性地将输出传递回聊天。

如果你想要_"每天早上运行这个"_ 或 _"20 分钟后戳代理"`，cron 就是这个机制。

## TL;DR

- Cron 在**网关内部**运行（不在模型中）。
- 作业持久化在 `~/.openclaw/cron/` 下，因此重启不会丢失计划。
- 两种执行风格：
  - **主会话**：将系统事件加入队列，然后在下次心跳时运行。
  - **隔离**：在 `cron:<jobId>` 中运行专用的代理回合，可选择传递输出。
- 唤醒是一等的：作业可以请求"立即唤醒"与"下次心跳"。

## 快速开始（可操作）

创建一次性提醒，验证它存在，并立即运行：

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id> --force
openclaw cron runs --id <job-id>
```

安排具有传递的重复隔离作业：

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --deliver \
  --channel slack \
  --to "channel:C1234567890"
```

## 工具调用等效项（网关 cron 工具）

有关规范的 JSON 形状和示例，请参阅[工具调用的 JSON 模式](/automation/cron-jobs#json-schema-for-tool-calls)。

## Cron 作业存储在哪里

Cron 作业默认持久化在网关主机的 `~/.openclaw/cron/jobs.json` 中。网关将文件加载到内存并在更改时写回，因此手动编辑仅在网关停止时是安全的。更改时首选 `openclaw cron add/edit` 或 cron 工具调用 API。

## 初学者友好概述

将 cron 作业视为：**何时**运行 + **什么**做。

1. **选择计划**
   - 一次性提醒 → `schedule.kind = "at"`（CLI：`--at`）
   - 重复作业 → `schedule.kind = "every"` 或 `schedule.kind = "cron"`
   - 如果你的 ISO 时间戳省略了时区，它被视为 **UTC**。

2. **选择它在哪里运行**
   - `sessionTarget: "main"` → 在下次心跳时使用主上下文运行。
   - `sessionTarget: "isolated"` → 在 `cron:<jobId>` 中运行专用的代理回合。

3. **选择有效负载**
   - 主会话 → `payload.kind = "systemEvent"`
   - 隔离会话 → `payload.kind = "agentTurn"`

可选：`deleteAfterRun: true` 从存储中删除成功的一次性作业。

## 概念

### 作业

Cron 作业是具有以下内容的存储记录：

- **计划**（它应该何时运行），
- **有效负载**（它应该做什么），
- 可选的**传递**（输出应该发送到哪里）。
- 可选的**代理绑定**（`agentId`）：在特定代理下运行作业；如果缺失或未知，网关回退到默认代理。

作业由稳定的 `jobId` 标识（由 CLI/网关 API 使用）。在代理工具调用中，`jobId` 是规范的；为兼容起见接受传统的 `id`。作业可以通过 `deleteAfterRun: true` 在成功的一次性运行后自动删除。

### 计划

Cron 支持三种计划类型：

- `at`：一次性时间戳（自纪元以来的毫秒）。网关接受 ISO 8601 并转换为 UTC。
- `every`：固定间隔（毫秒）。
- `cron`：具有可选 IANA 时区的 5 字段 cron 表达式。

Cron 表达式使用 `croner`。如果省略时区，则使用网关主机的本地时区。

### 主执行 vs 隔离执行

#### 主会话作业（系统事件）

主作业将系统事件加入队列，可选择性地唤醒心跳运行器。它们必须使用 `payload.kind = "systemEvent"`。

- `wakeMode: "next-heartbeat"`（默认）：事件等待下次计划的心跳。
- `wakeMode: "now"`：事件触发立即心跳运行。

这是当你想要正常的心跳提示 + 主会话上下文时的最佳选择。请参阅 [Heartbeat](/gateway/heartbeat)。

#### 隔离作业（专用 cron 会话）

隔离作业在会话 `cron:<jobId>` 中运行专用的代理回合。

关键行为：

- 提示以 `[cron:<jobId> <job name>]` 为前缀以实现可追溯性。
- 每次运行都从**新的会话 id** 开始（没有先前的对话延续）。
- 摘要发布到主会话（前缀 `Cron`，可配置）。
- 如果 `payload.deliver: true`，输出被传递到频道；否则它保持内部。

将隔离作业用于嘈杂、频繁或"后台杂务"，这些杂务不应该弄乱你的主要聊天历史。

### 有效负载形状（什么运行）

支持两种有效负载类型：

- `systemEvent`：仅主会话，通过心跳提示路由。
- `agentTurn`：仅隔离会话，运行专用的代理回合。

常见的 `agentTurn` 字段：

- `message`：必需的文本提示。
- `model` / `thinking`：可选覆盖（见下文）。
- `timeoutSeconds`：可选超时覆盖。
- `deliver`: `true` 将输出发送到频道目标。
- `channel`: `last` 或特定频道。
- `to`: 频道特定目标（电话/聊天/频道 id）。
- `bestEffortDeliver`：避免在传递失败时使作业失败。

隔离选项（仅用于 `session=isolated`）：

- `postToMainPrefix`（CLI：`--post-prefix`）：主系统事件的前缀。
- `postToMainMode`: `summary`（默认）或 `full`。
- `postToMainMaxChars`: 当 `postToMainMode=full` 时的最大字符数（默认 8000）。

### 模型和推理覆盖

隔离作业（`agentTurn`）可以覆盖模型和推理级别：

- `model`：提供商/模型字符串（例如 `anthropic/claude-sonnet-4-20250514`）或别名（例如 `opus`）
- `thinking`：推理级别（`off`、`minimal`、`low`、`medium`、`high`、`xhigh`；仅 GPT-5.2 + Codex 模型）

注意：你也可以在主会话作业上设置 `model`，但它会改变共享的主会话模型。我们建议仅对隔离作业使用模型覆盖，以避免意外的上下文偏移。

解析优先级：

1. 作业有效负载覆盖（最高）
2. Hook 特定默认值（例如 `hooks.gmail.model`）
3. 代理配置默认值

### 传递（频道 + 目标）

隔离作业可以将输出传递到频道。作业有效负载可以指定：

- `channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost`（插件）/`signal` / `imessage` / `last`
- `to`: 频道特定收件人目标

如果 `channel` 或 `to` 被省略，cron 可以回退到主会话的"最后路由"（代理最后回复的地方）。

传递说明：

- 如果设置了 `to`，cron 即使省略了 `deliver` 也会自动传递代理的最终输出。
- 当你想要最后路由传递而没有明确的 `to` 时使用 `deliver: true`。
- 当 `to` 存在时使用 `deliver: false` 以保持输出内部。

目标格式提醒：

- Slack/Discord/Mattermost（插件）目标应该使用明确的前缀（例如 `channel:<id>`、`user:<id>`）以避免歧义。
- Telegram 主题应该使用 `:topic:` 形式（见下文）。

#### Telegram 传递目标（主题 / 论坛线程）

Telegram 通过 `message_thread_id` 支持论坛主题。对于 cron 传递，你可以将主题/线程编码到 `to` 字段中：

- `-1001234567890`（仅聊天 id）
- `-1001234567890:topic:123`（首选：明确的主题标记）
- `-1001234567890:123`（简写：数字后缀）

也接受带前缀的目标如 `telegram:...` / `telegram:group:...`：

- `telegram:group:-1001234567890:topic:123`

## 工具调用的 JSON 模式

当直接调用网关 `cron.*` 工具时使用这些形状（代理工具调用或 RPC）。CLI 标志接受人类可读的持续时间如 `20m`，但工具调用对 `atMs` 和 `everyMs` 使用纪元毫秒（`at` 时间接受 ISO 时间戳）。

### cron.add 参数

一次性，主会话作业（系统事件）：

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "atMs": 1738262400000 },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

重复，隔离作业与传递：

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "deliver": true,
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffortDeliver": true
  },
  "isolation": { "postToMainPrefix": "Cron", "postToMainMode": "summary" }
}
```

注意：

- `schedule.kind`: `at` (`atMs`)、`every` (`everyMs`) 或 `cron` (`expr`，可选的 `tz`)。
- `atMs` 和 `everyMs` 是纪元毫秒。
- `sessionTarget` 必须是 `"main"` 或 `"isolated"`，并且必须匹配 `payload.kind`。
- 可选字段：`agentId`、`description`、`enabled`、`deleteAfterRun`、`isolation`。
- `wakeMode` 省略时默认为 `"next-heartbeat"`。

### cron.update 参数

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

注意：

- `jobId` 是规范的；为兼容起见接受 `id`。
- 在补丁中使用 `agentId: null` 清除代理绑定。

### cron.run 和 cron.remove 参数

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## 存储和历史

- 作业存储：`~/.openclaw/cron/jobs.json`（网关管理的 JSON）。
- 运行历史：`~/.openclaw/cron/runs/<jobId>.jsonl`（JSONL，自动修剪）。
- 覆盖存储路径：配置中的 `cron.store`。

## 配置

```json5
{
  cron: {
    enabled: true, // 默认 true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // 默认 1
  },
}
```

完全禁用 cron：

- `cron.enabled: false`（配置）
- `OPENCLAW_SKIP_CRON=1`（环境）

## CLI 快速开始

一次性提醒（UTC ISO，成功后自动删除）：

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

一次性提醒（主会话，立即唤醒）：

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

重复隔离作业（传递到 WhatsApp）：

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

重复隔离作业（传递到 Telegram 主题）：

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

具有模型和推理覆盖的隔离作业：

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

代理选择（多代理设置）：

```bash
# 将作业固定到代理"ops"（如果该代理缺失则回退到默认）
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# 切换或清除现有作业的代理
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

手动运行（调试）：

```bash
openclaw cron run <jobId> --force
```

编辑现有作业（补丁字段）：

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

运行历史：

```bash
openclaw cron runs --id <jobId> --limit 50
```

无需创建作业的即时系统事件：

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## 网关 API 表面

- `cron.list`、`cron.status`、`cron.add`、`cron.update`、`cron.remove`
- `cron.run`（强制或到期）、`cron.runs`

对于无需作业的即时系统事件，请使用 [`openclaw system event`](/cli/system)。

## 故障排除

### "什么都不运行"

- 检查 cron 是否启用：`cron.enabled` 和 `OPENCLAW_SKIP_CRON`。
- 检查网关是否持续运行（cron 在网关进程内运行）。
- 对于 `cron` 计划：确认时区（`--tz`）与主机时区。

### Telegram 传递到错误的地方

- 对于论坛主题，使用 `-100…:topic:<id>`，使其明确且无歧义。
- 如果你在日志或存储的"最后路由"目标中看到 `telegram:...` 前缀，那是正常的；cron 传递接受它们并且仍然正确解析主题 ID。
