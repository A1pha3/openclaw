---
summary: "节点发现和传输（Bonjour、Tailscale、SSH）用于查找网关"
read_when:
  - "实现或更改 Bonjour 发现/广告"
  - "调整远程连接模式（直接 vs SSH）"
  - "设计远程节点的节点发现和配对"
title: "发现和传输"
---

# 发现和传输

OpenClaw 有两个表面上看起来相似但实际不同的问题：

1. **操作员远程控制**：macOS 菜单栏应用控制在其他地方运行的网关。
2. **节点配对**：iOS/Android（和未来的节点）找到网关并安全配对。

设计目标是将所有网络发现/广告保持在**节点网关**（`openclaw gateway`）中，并让客户端（mac 应用、iOS）作为消费者。

## 术语

- **网关**：拥有状态（会话、配对、节点注册表）并运行通道的单个长期运行的网关进程。大多数设置每台主机使用一个；可能的隔离多网关设置。
- **网关 WS（控制平面）**：默认情况下在 `127.0.0.1:18789` 上的 WebSocket 端点；可以通过 `gateway.bind` 绑定到 LAN/tailnet。
- **直接 WS 传输**：面向 LAN/tailnet 的网关 WS 端点（无 SSH）。
- **SSH 传输（回退）**：通过 SSH 转发 `127.0.0.1:18789` 进行远程控制。
- **旧版 TCP 桥接（已弃用/已移除）**：较旧的节点传输（请参阅[桥接协议](/gateway/bridge-protocol)）；不再为发现做广告。

协议详情：

- [网关协议](/gateway/protocol)
- [桥接协议（旧版）](/gateway/bridge-protocol)

## 为什么我们同时保留"直接"和 SSH

- **直接 WS** 在同一网络和 tailnet 内提供最佳用户体验：
  - 通过 Bonjour 在 LAN 上自动发现
  - 配对令牌 + ACL 由网关拥有
  - 不需要 shell 访问；协议表面可以保持紧凑和可审计
- **SSH** 仍然是通用回退：
  - 适用于任何你有 SSH 访问的地方（甚至跨无关网络）
  - 存活于多播/mDNS 问题
  - 除了 SSH 之外不需要新的入站端口

## 发现输入（客户端如何了解网关在哪里）

### 1) Bonjour / mDNS（仅 LAN）

Bonjour 是尽力而为的，不能跨越网络。它仅用于"同一 LAN"的便利性。

目标方向：

- **网关**通过 Bonjour 广告其 WS 端点。
- 客户端浏览并显示"选择网关"列表，然后存储选定的端点。

故障排除和信标详情：[Bonjour](/gateway/bonjour)。

#### 服务信标详情

- 服务类型：
  - `_openclaw-gw._tcp`（网关传输信标）
- TXT 密钥（非机密）：
  - `role=gateway`
  - `lanHost=<hostname>.local`
  - `sshPort=22`（或任何广告的端口）
  - `gatewayPort=18789`（网关 WS + HTTP）
  - `gatewayTls=1`（仅在启用 TLS 时）
  - `gatewayTlsSha256=<sha256>`（仅在启用 TLS 且指纹可用时）
  - `canvasPort=18793`（默认画布主机端口；服务 `/__openclaw__/canvas/`）
  - `cliPath=<path>`（可选；可运行 `openclaw` 入口点或二进制的绝对路径）
  - `tailnetDns=<magicdns>`（可选提示；当 Tailscale 可用时自动检测）

禁用/覆盖：

- `OPENCLAW_DISABLE_BONJOUR=1` 禁用广告。
- `gateway.bind` 在 `~/.openclaw/openclaw.json` 中控制网关绑定模式。
- `OPENCLAW_SSH_PORT` 覆盖 TXT 中广告的 SSH 端口（默认为 22）。
- `OPENCLAW_TAILNET_DNS` 发布 `tailnetDns` 提示（MagicDNS）。
- `OPENCLAW_CLI_PATH` 覆盖广告的 CLI 路径。

### 2) Tailnet（跨网络）

对于伦敦/维也纳风格的设置，Bonjour 无法提供帮助。推荐的"直接"目标是：

- Tailscale MagicDNS 名称（首选）或稳定的 tailnet IP。

如果网关可以检测到它在 Tailscale 下运行，它会发布 `tailnetDns` 作为客户端的可选提示（包括广域信标）。

### 3) 手动 / SSH 目标

当没有直接路由（或直接被禁用）时，客户端始终可以通过 SSH 连接，方法是转发环回网关端口。

请参阅[远程访问](/gateway/remote)。

## 传输选择（客户端策略）

推荐的客户端行为：

1. 如果配置了配对的直接端点且可到达，使用它。
2. 否则，如果 Bonjour 在 LAN 上找到网关，提供一键"使用此网关"选择并将其保存为直接端点。
3. 否则，如果配置了 tailnet DNS/IP，尝试直接连接。
4. 否则，回退到 SSH。

## 配对 + 认证（直接传输）

网关是节点/客户端准入的事实来源。

- 配对请求在网关中创建/批准/拒绝（请参阅[网关配对](/gateway/pairing)）。
- 网关强制执行：
  - 认证（令牌 / 密钥对）
  - 范围/ACL（网关不是每个方法的原始代理）
  - 速率限制

## 组件职责

- **网关**：广告发现信标，拥有配对决策，并托管 WS 端点。
- **macOS 应用**：帮助你选择网关，显示配对提示，并且仅将 SSH 作为回退使用。
- **iOS/Android 节点**：将 Bonjour 浏览作为便利方式使用，并连接到配对的网关 WS。
