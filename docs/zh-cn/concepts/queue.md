---
summary: "序列化入站自动回复运行的设计"
read_when:
  - 更改自动回复执行或并发
title: "命令队列"
---

# 命令队列（2026-01-16）

我们通过一个微小的进程内队列序列化入站自动回复运行（所有频道），以防止多个代理运行碰撞，同时仍允许跨会话的安全并行。

## 为什么

- 自动回复运行可能很昂贵（LLM 调用），并且当多个入站消息接近时可能会碰撞。
- 序列化避免争夺共享资源（会话文件、日志、CLI stdin）并减少上游速率限制的机会。

## 它如何工作

- 车道感知 FIFO 队列以可配置的并发上限排出每个车道（未配置车道默认为 1；主车道默认为 4，子代理为 8）。
- `runEmbeddedPiAgent` 按**会话密钥**（车道 `session:<key>`）入队，以保证每个会话只有一个活动运行。
- 然后每个会话运行被排队到一个**全局车道**（默认为 `main`），因此总体并行度由 `agents.defaults.maxConcurrent` 限制。
- 当启用详细日志时，排队的运行如果等待超过 ~2s 开始则发出简短通知。
- 打字指示符仍立即在入队时触发（当频道支持时），因此用户体验不变，而我们在等待我们的轮次。

## 队列模式（每个频道）

入站消息可以转向当前运行、等待后续轮次，或两者兼而有之：

- `steer`：立即注入当前运行（在下一个工具边界之后取消挂起的工具调用）。如果未流式传输，回退到后续。
- `followup`：在当前运行结束后排队等待下一个代理轮次。
- `collect`：将所有排队的消息合并为**单个**后续轮次（如果消息针对不同的频道/线程，它们单独排出以保留路由）。
- `steer-backlog`（也称为 `steer+backlog`）：现在转向**并且**保留消息用于后续轮次。
- `interrupt`（旧版）：中止该会话的活动运行，然后运行最新的消息。
- `queue`（旧版别名）：与 `steer` 相同。

Steer-backlog 意味着你可以在转向的运行之后获得后续回复，因此流式传输表面可能看起来像重复项。如果你想要每个入站消息一个回复，优先使用 `collect`/`steer`。
发送 `/queue collect` 作为独立命令（每个会话）或设置 `messages.queue.byChannel.discord: "collect"`。

默认值（当在配置中未设置时）：

- 所有表面 → `collect`

通过 `messages.queue` 全局或按频道配置：

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

## 队列选项

选项适用于 `followup`、`collect` 和 `steer-backlog`（以及回退到后续时的 `steer`）：

- `debounceMs`：在开始后续轮次之前等待安静（防止"继续，继续"）。
- `cap`：每个会话的最大排队消息数。
- `drop`：溢出策略（`old`、`new`、`summarize`）。

Summarize 保留被丢弃消息的简短项目符号列表，并将其作为合成后续提示注入。
默认值：`debounceMs: 1000`、`cap: 20`、`drop: summarize`。

## 每个会话覆盖

- 发送 `/queue <mode>` 作为独立命令以存储当前会话的模式。
- 选项可以组合：`/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` 或 `/queue reset` 清除会话覆盖。

## 范围和保证

- 适用于使用网关回复管道的所有入站频道的自动回复代理运行（WhatsApp web、Telegram、Slack、Discord、Signal、iMessage、webchat 等）。
- 默认车道（`main`）对于入站 + 主要心跳是进程范围的；设置 `agents.defaults.maxConcurrent` 以允许会话并行运行。
- 可能存在额外车道（例如 `cron`、`subagent`），因此后台作业可以并行运行而不会阻止入站回复。
- 每个会话车道保证只有一个代理运行一次接触给定的会话。
- 无外部依赖或后台工作线程；纯 TypeScript + promises。

## 故障排除

- 如果命令似乎卡住，启用详细日志并查找"排队 ...ms"行以确认队列正在排出。
- 如果你需要队列深度，启用详细日志并观察队列计时行。
