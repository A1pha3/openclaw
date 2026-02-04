---
summary: "Bonjour/mDNS 发现 + 调试（网关信标、客户端和常见故障模式）"
read_when:
  - "在 macOS/iOS 上调试 Bonjour 发现问题"
  - "更改 mDNS 服务类型、TXT 记录或发现用户体验"
title: "Bonjour 发现"
---

# Bonjour / mDNS 发现

OpenClaw 使用 Bonjour（mDNS / DNS-SD）作为**仅 LAN 的便利方式**来发现活动的网关（WebSocket 端点）。它是尽力而为的，**不能**取代 SSH 或基于 Tailnet 的连接。

## 通过 Tailscale 实现的广域 Bonjour（单播 DNS-SD）

如果节点和网关在不同网络上，多播 mDNS 不会跨越边界。你可以通过切换到通过 Tailscale 的**单播 DNS-SD**（"广域 Bonjour"）来保持相同的发现用户体验。

高级步骤：

1. 在网关机上运行 DNS 服务器（可通过 Tailnet 访问）。
2. 为专用区域发布 DNS-SD 记录 `_openclaw-gw._tcp`（示例：`openclaw.internal.`）。
3. 配置 Tailscale **分割 DNS**，使你选择的域通过该 DNS 服务器为客户端解析（包括 iOS）。

OpenClaw 支持任何发现域；`openclaw.internal.` 只是一个示例。iOS/Android 节点同时浏览 `local.` 和你配置的广域域。

### 网关配置（推荐）

```json5
{
  gateway: { bind: "tailnet" }, // 仅 tailnet（推荐）
  discovery: { wideArea: { enabled: true } }, // 启用广域 DNS-SD 发布
}
```

### 一次性 DNS 服务器设置（网关机）

```bash
openclaw dns setup --apply
```

这会安装 CoreDNS 并将其配置为：

- 仅在网关的 Tailscale 接口上监听端口 53
- 从 `~/.openclaw/dns/<domain>.db` 为你选择的域（示例：`openclaw.internal.`）提供服务

从 tailnet 连接的机器验证：

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS 设置

在 Tailscale 管理控制台中：

- 添加指向网关 tailnet IP 的名称服务器（UDP/TCP 53）。
- 添加分割 DNS，使你的发现域使用该名称服务器。

一旦客户端接受 tailnet DNS，iOS 节点就可以在你的发现域中浏览 `_openclaw-gw._tcp`，而无需多播。

### 网关监听器安全性（推荐）

网关 WS 端口（默认 `18789`）默认绑定到环回。对于 LAN/tailnet 访问，显式绑定并保持认证启用。

对于仅 tailnet 的设置：

- 在 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
- 重启网关（或重启 macOS 菜单栏应用）。

## 广告内容

只有网关广告 `_openclaw-gw._tcp`。

## 服务类型

- `_openclaw-gw._tcp` — 网关传输信标（由 macOS/iOS/Android 节点使用）。

## TXT 密钥（非机密提示）

网关广告小的非机密提示以使 UI 流程方便：

- `role=gateway`
- `displayName=<友好名称>`
- `lanHost=<hostname>.local`
- `gatewayPort=<port>`（网关 WS + HTTP）
- `gatewayTls=1`（仅在启用 TLS 时）
- `gatewayTlsSha256=<sha256>`（仅在启用 TLS 且指纹可用时）
- `canvasPort=<port>`（仅在画布主机启用时；默认 `18793`）
- `sshPort=<port>`（未覆盖时默认为 22）
- `transport=gateway`
- `cliPath=<path>`（可选；可运行 `openclaw` 入口点的绝对路径）
- `tailnetDns=<magicdns>`（可选提示，当 Tailnet 可用时）

## 在 macOS 上调试

有用的内置工具：

- 浏览实例：
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
- 解析一个实例（替换 `<instance>`）：
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

如果浏览有效但解析失败，你通常会遇到 LAN 策略或 mDNS 解析器问题。

## 在网关日志中调试

网关写入滚动日志文件（在启动时打印为 `gateway log file: ...`）。查找 `bonjour:` 行，特别是：

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`

## 在 iOS 节点上调试

iOS 节点使用 `NWBrowser` 发现 `_openclaw-gw._tcp`。

要捕获日志：

- 设置 → 网关 → 高级 → **发现调试日志**
- 设置 → 网关 → 高级 → **发现日志** → 复现 → **复制**

日志包括浏览器状态转换和结果集更改。

## 常见故障模式

- **Bonjour 不跨网络**：使用 Tailnet 或 SSH。
- **多播被阻止**：某些 Wi-Fi 网络禁用 mDNS。
- **睡眠/接口变化**：macOS 可能暂时删除 mDNS 结果；重试。
- **浏览有效但解析失败**：保持机器名称简单（避免表情符号或标点符号），然后重启网关。服务实例名称派生自主机名，因此过于复杂的名称可能会混淆某些解析器。

## 转义的实例名称（`\032`）

Bonjour/DNS-SD 通常将服务实例名称中的字节转义为十进制 `\DDD` 序列（例如，空格变成 `\032`）。

- 这在协议级别是正常的。
- UI 应该解码以进行显示（iOS 使用 `BonjourEscapes.decode`）。

## 禁用 / 配置

- `OPENCLAW_DISABLE_BONJOUR=1` 禁用广告（旧版：`OPENCLAW_DISABLE_BONJOUR`）。
- `gateway.bind` 在 `~/.openclaw/openclaw.json` 中控制网关绑定模式。
- `OPENCLAW_SSH_PORT` 覆盖 TXT 中广告的 SSH 端口（旧版：`OPENCLAW_SSH_PORT`）。
- `OPENCLAW_TAILNET_DNS` 在 TXT 中发布 MagicDNS 提示（旧版：`OPENCLAW_TAILNET_DNS`）。
- `OPENCLAW_CLI_PATH` 覆盖广告的 CLI 路径（旧版：`OPENCLAW_CLI_PATH`）。

## 相关文档

- 发现策略和传输选择：[发现](/gateway/discovery)
- 节点配对 + 审批：[网关配对](/gateway/pairing)
