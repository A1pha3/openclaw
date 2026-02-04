---
summary: "OpenClaw 应用、网关节点传输和 PeekabooBridge 的 macOS IPC 架构"
read_when:
  - 编辑 IPC 契约或菜单栏应用 IPC
title: "macOS IPC"
---

# OpenClaw macOS IPC 架构

**当前模型**：本地 Unix 套接字将**节点主机服务**连接到 **macOS 应用**，用于执行审批 + `system.run`。存在一个 `openclaw-mac` 调试 CLI 用于发现/连接检查；代理操作仍通过网关 WebSocket 和 `node.invoke` 流转。UI 自动化使用 PeekabooBridge。

## 设计目标

- 单一 GUI 应用实例拥有所有面向 TCC 的工作（通知、屏幕录制、麦克风、语音、AppleScript）。
- 小型自动化表面：网关 + 节点命令，加上 PeekabooBridge 用于 UI 自动化。
- 可预测的权限：始终相同的签名包 ID，由 launchd 启动，因此 TCC 授权保持有效。

## 工作原理

### 网关 + 节点传输

- 应用运行网关（本地模式）并作为节点连接到它。
- 代理操作通过 `node.invoke` 执行（例如 `system.run`、`system.notify`、`canvas.*`）。

### 节点服务 + 应用 IPC

- 无头节点主机服务连接到网关 WebSocket。
- `system.run` 请求通过本地 Unix 套接字转发到 macOS 应用。
- 应用在 UI 上下文中执行命令，必要时提示，并返回输出。

架构图 (SCI)：

```
代理 -> 网关 -> 节点服务 (WS)
                  |  IPC (UDS + 令牌 + HMAC + TTL)
                  v
              Mac 应用 (UI + TCC + system.run)
```

### PeekabooBridge (UI 自动化)

- UI 自动化使用名为 `bridge.sock` 的单独 UNIX 套接字和 PeekabooBridge JSON 协议。
- 主机优先顺序（客户端侧）：Peekaboo.app → Claude.app → OpenClaw.app → 本地执行。
- 安全性：桥接主机要求允许的 TeamID；仅 DEBUG 时的同 UID 逃生舱由 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` 守护（Peekaboo 约定）。
- 参见：[PeekabooBridge 用法](/zh-cn/platforms/mac/peekaboo) 了解详情。

## 操作流程

- 重启/重建：`SIGN_IDENTITY="Apple Development: <开发者名称> (<TEAMID>)" scripts/restart-mac.sh`
  - 终止现有实例
  - Swift 构建 + 打包
  - 写入/引导/启动 LaunchAgent
- 单实例：如果另一个具有相同包 ID 的实例正在运行，应用会提前退出。

## 加固说明

- 对所有特权表面优先要求 TeamID 匹配。
- PeekabooBridge：`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（仅 DEBUG）可能允许同 UID 调用者用于本地开发。
- 所有通信保持仅本地；不暴露网络套接字。
- TCC 提示仅来自 GUI 应用包；在重建之间保持签名包 ID 稳定。
- IPC 加固：套接字模式 `0600`、令牌、对等 UID 检查、HMAC 挑战/响应、短 TTL。

## 安全层次

| 层 | 机制 | 目的 |
|----|------|------|
| 传输 | Unix 套接字 | 仅本地，无网络暴露 |
| 身份 | TeamID 验证 | 确保只有受信任的应用可以通信 |
| 认证 | HMAC 挑战/响应 | 防止重放攻击 |
| 授权 | 令牌 + TTL | 限制操作时效性 |
| 进程 | 对等 UID 检查 | 确保相同用户上下文 |

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| IPC 连接失败 | 检查套接字文件权限是否为 `0600` |
| TCC 权限丢失 | 确保包 ID 在重建后保持一致 |
| PeekabooBridge 不工作 | 验证是否有受支持的主机应用运行 |
| 调试模式无法使用无签名客户端 | 设置 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` |
