---
summary: "用于列出会话、获取历史记录和发送跨会话消息的代理会话工具"
read_when:
  - 添加或修改会话工具
title: "会话工具"
---

# 会话工具

目标：小型、难以误用的工具集，以便代理可以列出会话、获取历史记录并发送到另一个会话。

## 工具名称

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

## 关键模型

- 主要直接聊天桶始终是字面密钥 `"main"`（解析为当前代理的主密钥）。
- 组聊天使用 `agent:<agentId>:<channel>:group:<id>` 或 `agent:<agentId>:<channel>:channel:<id>`（传递完整密钥）。
- Cron 作业使用 `cron:<job.id>`。
- Hooks 使用 `hook:<uuid>`，除非显式设置。
- 节点会话使用 `node-<nodeId>`，除非显式设置。

`global` 和 `unknown` 是保留值，永不被列出。如果 `session.scope = "global"`，我们为所有工具别名为 `main`，以便调用者永远不会看到 `global`。

## sessions_list

将会话列为行数组。

参数：

- `kinds?: string[]` 过滤器：任何 `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` 最大行数（默认值：服务器默认值，钳制例如 200）
- `activeMinutes?: number` 仅最近 N 分钟内更新的会话
- `messageLimit?: number` 0 = 无消息（默认值 0）；>0 = 包含最后 N 条消息

行为：

- `messageLimit > 0` 获取每个会话的 `chat.history` 并包含最后 N 条消息。
- 工具结果在列表输出中被过滤；使用 `sessions_history` 获取工具消息。
- 当在**沙箱化**的代理会话中运行时，会话工具默认为**仅生成可见性**（见下文）。

行形状（JSON）：

- `key`：会话密钥（字符串）
- `kind`：`main | group | cron | hook | node | other`
- `channel`：`whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName`（组显示标签，如果可用）
- `updatedAt`（毫秒）
- `sessionId`
- `model`、`contextTokens`、`totalTokens`
- `thinkingLevel`、`verboseLevel`、`systemSent`、`abortedLastRun`
- `sendPolicy`（如果设置，会话覆盖）
- `lastChannel`、`lastTo`
- `deliveryContext`（规范化 `{ channel, to, accountId }`，如果可用）
- `transcriptPath`（从存储目录 + sessionId 派生的尽力路径）
- `messages?`（仅当 `messageLimit > 0`）

## sessions_history

获取一个会话的记录。

参数：

- `sessionKey`（必需；接受会话密钥或来自 `sessions_list` 的 `sessionId`）
- `limit?: number` 最大消息数（服务器钳制）
- `includeTools?: boolean`（默认值 false）

行为：

- `includeTools=false` 过滤 `role: "toolResult"` 消息。
- 以原始记录格式返回消息数组。
- 当给出 `sessionId` 时，OpenClaw 将其解析为对应的会话密钥（缺失的 ID 报错）。

## sessions_send

发送消息到另一个会话。

参数：

- `sessionKey`（必需；接受会话密钥或来自 `sessions_list` 的 `sessionId`）
- `message`（必需）
- `timeoutSeconds?: number`（默认值 >0；0 = 即发即忘）

行为：

- `timeoutSeconds = 0`：排队并返回 `{ runId, status: "accepted" }`。
- `timeoutSeconds > 0`：等待最多 N 秒完成，然后返回 `{ runId, status: "ok", reply }`。
- 如果等待超时：`{ runId, status: "timeout", error }`。运行继续；稍后调用 `sessions_history`。
- 如果运行失败：`{ runId, status: "error", error }`。
- 宣布传递在主要运行完成后运行并且是尽力而为的；`status: "ok"` 不保证宣布已传递。
- 通过网关 `agent.wait`（服务器端）等待，因此重新连接不会丢弃等待。
- 代理到代理消息上下文被注入到主要运行中。
- 主要运行完成后，OpenClaw 运行**回复回环**：
  - 第 2+ 轮在请求者和目标代理之间交替。
  - 完全回复 `REPLY_SKIP` 以停止乒乓。
  - 最大轮次是 `session.agentToAgent.maxPingPongTurns`（0-5，默认 5）。
- 一旦循环结束，OpenClaw 运行**代理到代理宣布步骤**（仅目标代理）：
  - 完全回复 `ANNOUNCE_SKIP` 以保持沉默。
  - 任何其他回复都发送到目标频道。
  - 宣布步骤包括原始请求 + 第 1 轮回复 + 最新的乒乓回复。

## 频道字段

- 对于组，`channel` 是会话条目上记录的频道。
- 对于直接聊天，`channel` 从 `lastChannel` 映射。
- 对于 cron/hook/node，`channel` 是 `internal`。
- 如果缺失，`channel` 是 `unknown`。

## 安全 / 发送策略

基于频道/聊天类型的策略阻止（不是每个会话 ID）。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

运行时覆盖（每个会话条目）：

- `sendPolicy: "allow" | "deny"`（未设置 = 继承配置）
- 可通过 `sessions.patch` 或仅所有者 `/send on|off|inherit`（独立消息）设置。

强制执行点：

- `chat.send` / `agent`（网关）
- 自动回复传递逻辑

## sessions_spawn

在隔离的会话中生成子代理运行，并在请求者聊天频道中宣布结果。

参数：

- `task`（必需）
- `label?`（可选；用于日志/UI）
- `agentId?`（可选；如果允许，在另一个代理 ID 下生成）
- `model?`（可选；覆盖子代理模型；无效值报错）
- `runTimeoutSeconds?`（默认值 0；设置时，在 N 秒后中止子代理运行）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：

- `agents.list[].subagents.allowAgents`：允许通过 `agentId` 的代理 ID 列表（`["*"]` 允许任何）。默认值：仅请求者代理。

发现：

- 使用 `agents_list` 发现允许用于 `sessions_spawn` 的代理 ID。

行为：

- 使用 `deliver: false` 启动新的 `agent:<agentId>:subagent:<uuid>` 会话。
- 子代理默认为完整工具集**减去会话工具**（可通过 `tools.subagents.tools` 配置）。
- 子代理不允许调用 `sessions_spawn`（没有子代理 → 子代理生成）。
- 始终非阻塞：立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 完成后，OpenClaw 运行子代理**宣布步骤**并将结果发布到请求者聊天频道。
- 在宣布步骤期间完全回复 `ANNOUNCE_SKIP` 以保持沉默。
- 宣布回复被规范化为 `Status`/`Result`/`Notes`；`Status` 来自运行时结果（不是模型文本）。
- 子代理会话在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动归档。
- 宣布回复包括统计行（运行时、代币、sessionKey/sessionId、记录路径和可选成本）。

## 沙箱会话可见性

沙箱化会话可以使用会话工具，但默认情况下它们只看到通过 `sessions_spawn` 生成的会话。

配置：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // 默认："spawned"
        sessionToolsVisibility: "spawned", // 或 "all"
      },
    },
  },
}
```
