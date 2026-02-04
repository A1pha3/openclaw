---
summary: "Mac 应用如何嵌入网关 WebChat 及其调试方法"
read_when:
  - 调试 Mac WebChat 视图或回环端口
title: "WebChat"
---

# WebChat (macOS 应用)

macOS 菜单栏应用将 WebChat UI 嵌入为原生 SwiftUI 视图。它连接到网关，默认使用所选代理的**主会话**（带有用于其他会话的会话切换器）。

- **本地模式**：直接连接到本地网关 WebSocket。
- **远程模式**：通过 SSH 转发网关控制端口，并使用该隧道作为数据平面。

## 启动与调试

- 手动：龙虾菜单 → "Open Chat"。
- 用于测试的自动打开：
  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```
- 日志：`./scripts/clawlog.sh`（子系统 `bot.molt`，类别 `WebChatSwiftUI`）。

## 工作原理

- 数据平面：网关 WS 方法 `chat.history`、`chat.send`、`chat.abort`、`chat.inject` 和事件 `chat`、`agent`、`presence`、`tick`、`health`。
- 会话：默认使用主会话（`main`，或当范围为全局时为 `global`）。UI 可以在会话之间切换。
- 新手引导使用专用会话，以将首次运行设置与日常使用分开。

## 安全面

- 远程模式仅通过 SSH 转发网关 WebSocket 控制端口。

## 已知限制

- UI 针对聊天会话优化（不是完整的浏览器沙箱）。

## 架构概览

```
┌─────────────────────────────────────────┐
│           macOS 菜单栏应用               │
├─────────────────────────────────────────┤
│  SwiftUI WebChatView                    │
│     │                                   │
│     ├─ chat.history  ──┐               │
│     ├─ chat.send     ──┤               │
│     ├─ chat.abort    ──┼── WS 方法     │
│     └─ chat.inject   ──┘               │
│                                         │
│     ┌─ chat          ──┐               │
│     ├─ agent         ──┤               │
│     ├─ presence      ──┼── WS 事件     │
│     ├─ tick          ──┤               │
│     └─ health        ──┘               │
└────────────────┬────────────────────────┘
                 │
                 │ WebSocket
                 │
┌────────────────▼────────────────────────┐
│              网关                        │
│  本地: ws://127.0.0.1:18789             │
│  远程: SSH 隧道 → ws://...              │
└─────────────────────────────────────────┘
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| WebChat 无法连接 | 检查网关是否正在运行：`openclaw gateway status` |
| 远程模式连接失败 | 验证 SSH 隧道是否正确建立 |
| 消息不显示 | 查看日志中的 `WebChatSwiftUI` 类别 |
| 会话切换无响应 | 确认网关返回了有效的会话列表 |
