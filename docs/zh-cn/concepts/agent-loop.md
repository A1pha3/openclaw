---
summary: "代理循环生命周期、流式传输和等待语义"
read_when:
  - 你需要代理循环或生命周期事件的详细演练
title: "代理循环"
---

# 代理循环（OpenClaw）

代理循环是代理的完整"真实"运行：摄取 → 上下文组装 → 模型推理 →
工具执行 → 流式回复 → 持久化。它是将消息转换为操作和最终回复的权威路径，
同时保持会话状态一致。

在 OpenClaw 中，循环是每个会话的单个序列化运行，它在模型思考、调用工具和流式输出时发出生命周期和流事件。本文档解释了这个真实循环是如何端到端连接的。

## 入口点

- 网关 RPC：`agent` 和 `agent.wait`。
- CLI：`agent` 命令。

## 工作原理（高级）

1. `agent` RPC 验证参数，解析会话（sessionKey/sessionId），持久化会话元数据，立即返回 `{ runId, acceptedAt }`。
2. `agentCommand` 运行代理：
   - 解析模型 + thinking/verbose 默认值
   - 加载技能快照
   - 调用 `runEmbeddedPiAgent`（pi-agent-core 运行时）
   - 如果嵌入式循环没有发出，则发出**生命周期结束/错误**
3. `runEmbeddedPiAgent`：
   - 通过每个会话 + 全局队列序列化运行
   - 解析模型 + 认证配置文件并构建 pi 会话
   - 订阅 pi 事件并流式传输助手/工具增量
   - 强制超时 → 如果超出则中止运行
   - 返回 payload + 使用元数据
4. `subscribeEmbeddedPiSession` 将 pi-agent-core 事件桥接到 OpenClaw `agent` 流：
   - 工具事件 => `stream: "tool"`
   - 助手增量 => `stream: "assistant"`
   - 生命周期事件 => `stream: "lifecycle"`（`phase: "start" | "end" | "error"`）
5. `agent.wait` 使用 `waitForAgentJob`：
   - 等待 **runId** 的**生命周期结束/错误**
   - 返回 `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## 队列 + 并发

- 运行按会话密钥（会话车道）序列化，可选地通过全局车道。
- 这可以防止工具/会话竞争并保持会话历史一致。
- 消息频道可以选择队列模式（收集/转向/后续），从而馈入此车道系统。
  参见[命令队列](/concepts/queue)。

## 会话 + 工作区准备

- 工作区被解析并创建；沙箱化运行可能会重定向到沙箱工作区根目录。
- 技能被加载（或从快照重用）并注入到环境和提示中。
- 引导/上下文文件被解析并注入到系统提示报告中。
- 获取会话写锁；`SessionManager` 在流式传输之前打开并准备。

## 提示组装 + 系统提示

- 系统提示由 OpenClaw 的基础提示、技能提示、引导上下文和每个运行的覆盖构建。
- 强制执行特定于模型的限制和压缩保留令牌。
- 参见[系统提示](/concepts/system-prompt)了解模型看到的内容。

## 钩子点（你可以拦截的地方）

OpenClaw 有两个钩子系统：

- **内部钩子**（网关钩子）：用于命令和生命周期事件的事件驱动脚本。
- **插件钩子**：代理/工具生命周期和网关管道中的扩展点。

### 内部钩子（网关钩子）

- **`agent:bootstrap`**：在构建引导文件时运行，在系统提示最终确定之前。
  使用它来添加/删除引导上下文文件。
- **命令钩子**：`/new`、`/reset`、`/stop` 和其他命令事件（参见 Hooks 文档）。

参见[Hooks](/hooks)了解设置和示例。

### 插件钩子（代理 + 网关生命周期）

这些在代理循环或网关管道内运行：

- **`before_agent_start`**：在运行开始前注入上下文或覆盖系统提示。
- **`agent_end`**：检查完成后的最终消息列表和运行元数据。
- **`before_compaction` / `after_compaction`**：观察或注释压缩周期。
- **`before_tool_call` / `after_tool_call`**：拦截工具参数/结果。
- **`tool_result_persist`**：同步转换工具结果，然后再写入会话记录。
- **`message_received` / `message_sending` / `message_sent`**：入站 + 出站消息钩子。
- **`session_start` / `session_end`**：会话生命周期边界。
- **`gateway_start` / `gateway_stop`**：网关生命周期事件。

参见[Plugins](/plugin#plugin-hooks)了解钩子 API 和注册详情。

## 流式传输 + 部分回复

- 助手增量从 pi-agent-core 流式传输并作为 `assistant` 事件发出。
- 块流式传输可以在 `text_end` 或 `message_end` 上发出部分回复。
- 推理流式传输可以作为单独的流或作为块回复发出。
- 参见[流式传输](/concepts/streaming)了解分块和块回复行为。

## 工具执行 + 消息工具

- 工具开始/更新/结束事件在 `tool` 流上发出。
- 工具结果在记录/发出之前被清理大小和图像负载。
- 消息工具发送被跟踪以抑制重复的助手确认。

## 回复塑造 + 抑制

- 最终 payload 从以下内容组装：
  - 助手文本（和可选推理）
  - 内联工具摘要（当 verbose + 允许时）
  - 模型错误时的助手错误文本
- `NO_REPLY` 被视为静默令牌并从传出 payload 中过滤。
- 消息工具重复项从最终 payload 列表中移除。
- 如果没有可渲染的 payload 剩余且工具出错，则发出回退工具错误回复
  （除非消息工具已经发送了用户可见的回复）。

## 压缩 + 重试

- 自动压缩发出 `compaction` 流事件并可以触发重试。
- 重试时，重置内存缓冲区和工具摘要以避免重复输出。
- 参见[压缩](/concepts/compaction)了解压缩管道。

## 事件流（当前）

- `lifecycle`：由 `subscribeEmbeddedPiSession` 发出（并在回退时由 `agentCommand` 发出）
- `assistant`：从 pi-agent-core 流式传输的增量
- `tool`：从 pi-agent-core 流式传输的工具事件

## 聊天频道处理

- 助手增量被缓冲到聊天 `delta` 消息中。
- 聊天 `final` 在**生命周期结束/错误**时发出。

## 超时

- `agent.wait` 默认：30s（只是等待）。`timeoutMs` 参数覆盖。
- 代理运行时：`agents.defaults.timeoutSeconds` 默认 600s；在 `runEmbeddedPiAgent` 中止计时器中强制执行。

## 哪些地方可能提前结束

- 代理超时（中止）
- AbortSignal（取消）
- 网关断开连接或 RPC 超时
- `agent.wait` 超时（仅等待，不停止代理）
