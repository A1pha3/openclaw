---
summary: "用于外部 CLI (signal-cli, imsg) 和网关模式的 RPC 适配器"
read_when:
  - 添加或修改外部 CLI 集成
  - 调试 RPC 适配器 (signal-cli, imsg)
title: "RPC 适配器"
---

# RPC 适配器

OpenClaw 通过 JSON-RPC 集成外部 CLI。目前使用两种模式。

## 模式 A: HTTP 守护进程 (signal-cli)

- `signal-cli` 作为守护进程运行，使用 HTTP 上的 JSON-RPC。
- 事件流为 SSE (`/api/v1/events`)。
- 健康探测：`/api/v1/check`。
- 当 `channels.signal.autoStart=true` 时，OpenClaw 拥有生命周期管理权。

参见 [Signal](/zh-CN/channels/signal) 了解设置和端点。

## 模式 B: stdio 子进程 (imsg)

- OpenClaw 作为子进程生成 `imsg rpc`。
- JSON-RPC 通过 stdin/stdout 逐行分隔（每行一个 JSON 对象）。
- 无需 TCP 端口，无需守护进程。

使用的核心方法：

- `watch.subscribe` → 通知（`method: "message"`）
- `watch.unsubscribe`
- `send`
- `chats.list`（探测/诊断）

参见 [iMessage](/zh-CN/channels/imessage) 了解设置和寻址（推荐使用 `chat_id`）。

## 适配器指南

- 网关拥有进程（启动/停止与提供程序生命周期绑定）。
- 保持 RPC 客户端具有弹性：设置超时，退出时重启。
- 优先使用稳定 ID（例如 `chat_id`）而非显示字符串。
