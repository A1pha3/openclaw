---
summary: "使用 SSH 隧道和 tailnet 实现远程访问（网关 WebSocket）"
read_when:
  - 运行或排查远程网关设置
title: "远程访问"
---

# 远程访问（SSH、隧道和 tailnet）

本仓库支持"通过 SSH 远程访问"，即在专用主机（桌面/服务器）上运行单个网关（主节点），客户端连接到该网关。

- **对于操作员（你/macOS 应用）：** SSH 隧道是通用回退方案。
- **对于节点（iOS/Android 和未来设备）：** 连接到网关 **WebSocket**（根据需要使用 LAN/tailnet 或 SSH 隧道）。

## 核心概念

- 网关 WebSocket 绑定到 **loopback** 的配置端口（默认为 18789）。
- 对于远程使用，通过 SSH 转发该 loopback 端口（或使用 tailnet/VPN 并减少隧道使用）。

## 常见的 VPN/tailnet 设置（代理所在位置）

将 **网关主机** 视为"代理所在的位置"。它拥有会话、认证配置、频道和状态。你的笔记本电脑/桌面（以及节点）连接到该主机。

### 1）在你的 tailnet 中始终运行网关（VPS 或家庭服务器）

在持久性主机上运行网关，通过 **Tailscale** 或 SSH 访问。

- **最佳体验：** 保持 `gateway.bind: "loopback"` 并使用 **Tailscale Serve** 来控制 UI。
- **回退方案：** 保持 loopback + 从任何需要访问的机器通过 SSH 隧道连接。
- **示例：** [exe.dev](/platforms/exe-dev)（简易 VM）或 [Hetzner](/platforms/hetzner)（生产 VPS）。

当你的笔记本电脑经常休眠但你希望代理始终运行时，这是理想选择。

### 2）家庭桌面运行网关，笔记本电脑远程控制

笔记本电脑**不**运行代理。它远程连接：

- 使用 macOS 应用的 **SSH 远程模式**（设置 → 常规 → "OpenClaw 运行位置"）。
- 应用打开并管理隧道，因此 WebChat + 健康检查"正常工作"。

操作手册：[macOS 远程访问](/platforms/mac/remote)。

### 3）笔记本电脑运行网关，从其他机器远程访问

保持网关本地但安全地暴露它：

- 从其他机器通过 SSH 隧道连接到笔记本电脑，或
- 使用 Tailscale Serve 来控制 UI，并保持网关仅限 loopback。

指南：[Tailscale](/gateway/tailscale) 和 [Web 概览](/web)。

## 命令流（哪里运行什么）

一个网关服务拥有状态 + 频道。节点是外设。

流程示例（Telegram → 节点）：

- Telegram 消息到达 **网关**。
- 网关运行 **代理**并决定是否调用节点工具。
- 网关通过网关 WebSocket（`node.*` RPC）调用 **节点**。
- 节点返回结果；网关回复到 Telegram。

注意：

- **节点不运行网关服务。** 每个主机只应运行一个网关，除非你有意识地运行隔离的配置（参见[多个网关](/gateway/multiple-gateways)）。
- macOS 应用"节点模式"只是通过网关 WebSocket 的节点客户端。

## SSH 隧道（CLI + 工具）

创建到远程网关 WS 的本地隧道：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

隧道建立后：

- `openclaw health` 和 `openclaw status --deep` 现在可以通过 `ws://127.0.0.1:18789` 访问远程网关。
- `openclaw gateway {status,health,send,agent,call}` 也可以通过 `--url` 定位转发的 URL。

注意：将 `18789` 替换为配置的 `gateway.port`（或 `--port`/`OPENCLAW_GATEWAY_PORT`）。

## CLI 远程默认值

你可以持久化远程目标，使 CLI 命令默认使用它：

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

当网关仅限 loopback 时，将 URL 保持为 `ws://127.0.0.1:18789`，并先打开 SSH 隧道。

## 通过 SSH 的聊天 UI

WebChat 不再使用单独的 HTTP 端口。SwiftUI 聊天 UI 直接连接到网关 WebSocket。

- 通过 SSH 转发 `18789`（见上文），然后将客户端连接到 `ws://127.0.0.1:18789`。
- 在 macOS 上，首选应用的"SSH 远程模式"，它会自动管理隧道。

## macOS 应用"SSH 远程模式"

macOS 菜单栏应用可以端到端地驱动相同的设置（远程状态检查、WebChat 和语音唤醒转发）。

操作手册：[macOS 远程访问](/platforms/mac/remote)。

## 安全规则（远程/VPN）

简而言之：**保持网关仅限 loopback**，除非你确定需要绑定。

- **Loopback + SSH/Tailscale Serve** 是最安全的默认设置（不公开暴露）。
- **非 loopback 绑定**（`lan`/`tailnet`/`custom`，或 `auto` 当 loopback 不可用时）必须使用认证令牌/密码。
- `gateway.remote.token` **仅**用于远程 CLI 调用——它**不**启用本地认证。
- `gateway.remote.tlsFingerprint` 在使用 `wss://` 时固定远程 TLS 证书。
- **Tailscale Serve** 可以在 `gateway.auth.allowTailscale: true` 时通过身份标头进行认证。
  如果你希望使用令牌/密码，请将其设置为 `false`。
- 像对待操作员访问一样对待浏览器控制：仅限 tailnet + 故意节点配对。

深入了解：[安全](/gateway/security)。
