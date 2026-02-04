---
summary: "OpenClaw 何时显示打字指示符以及如何调整它们"
read_when:
  - 更改打字指示符行为或默认值
title: "打字指示符"
---

# 打字指示符

打字指示符在运行活动时发送到聊天频道。使用 `agents.defaults.typingMode` 控制**何时**开始打字，使用 `typingIntervalSeconds` 控制**多久**刷新一次。

## 默认值

当 `agents.defaults.typingMode` **未设置**时，OpenClaw 保持旧行为：

- **直接聊天**：模型循环一开始打字就开始。
- **带有提及的群组聊天**：模型循环一开始打字就开始。
- **没有提及的群组聊天**：仅当消息文本开始流式传输时才开始打字。
- **心跳运行**：禁用打字。

## 模式

设置 `agents.defaults.typingMode` 为以下之一：

- `never` — 永不显示打字指示符。
- `instant` — **一旦模型循环开始**就开始打字，即使运行后来只返回静默回复令牌。
- `thinking` — 在**第一个推理增量**时开始打字（需要运行的 `reasoningLevel: "stream"`）。
- `message` — 在**第一个非静默文本增量**时开始打字（忽略 `NO_REPLY` 静默令牌）。

"它多早触发"的顺序：
`never` → `message` → `thinking` → `instant`

## 配置

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

你可以覆盖每个会话的模式或节奏：

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```

## 注意

- `message` 模式不会显示纯静默回复的打字指示符（例如用于抑制输出的 `NO_REPLY` 令牌）。
- `thinking` 仅在运行流式传输推理时触发（`reasoningLevel: "stream"`）。
  如果模型不发出推理增量，打字不会开始。
- 心跳从不显示打字，无论模式如何。
- `typingIntervalSeconds` 控制**刷新节奏**，而不是开始时间。默认是 6 秒。
