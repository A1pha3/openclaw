---
summary: "用于网关控制面板的集成 Tailscale Serve/Funnel"
read_when:
  - 在 localhost 之外暴露网关控制 UI
  - 自动化 tailnet 或公共控制面板访问
title: "Tailscale"
---

# Tailscale（网关控制面板）

OpenClaw 可以自动配置 Tailscale **Serve**（tailnet）或 **Funnel**（公共）
用于网关控制面板和 WebSocket 端口。这让网关保持绑定到 loopback，而
Tailscale 提供 HTTPS、路由和（对于 Serve）身份标头。

## 模式

- `serve`：通过 `tailscale serve` 的仅限 tailnet 的 Serve。网关保持在 `127.0.0.1`。
- `funnel`：通过 `tailscale funnel` 的公共 HTTPS。OpenClaw 需要共享密码。
- `off`：默认（无 Tailscale 自动化）。

## 认证

设置 `gateway.auth.mode` 来控制握手：

- `token`（当设置了 `OPENCLAW_GATEWAY_TOKEN` 时的默认设置）
- `password`（通过 `OPENCLAW_GATEWAY_PASSWORD` 或配置的共享密钥）

当 `tailscale.mode = "serve"` 且 `gateway.auth.allowTailscale` 为 `true` 时，
有效的 Serve 代理请求可以通过 Tailscale 身份标头（`tailscale-user-login`）进行认证，无需提供令牌/密码。
OpenClaw 通过本地 Tailscale 守护进程（`tailscale whois`）解析 `x-forwarded-for` 地址并与标头匹配来验证身份，然后才接受它。
OpenClaw 只有在请求来自 loopback 并带有 Tailscale 的 `x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host` 标头时，才将请求视为 Serve。
要要求明确凭证，设置 `gateway.auth.allowTailscale: false` 或强制使用 `gateway.auth.mode: "password"`。

## 配置示例

### 仅限 tailnet（Serve）

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

打开：`https://<magicdns>/`（或你配置的 `gateway.controlUi.basePath`）

### 仅限 tailnet（绑定到 Tailnet IP）

当你希望网关直接监听 Tailnet IP 时使用（无 Serve/Funnel）。

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

从另一个 Tailnet 设备连接：

- 控制 UI：`http://<tailscale-ip>:18789/`
- WebSocket：`ws://<tailscale-ip>:18789`

注意：loopback（`http://127.0.0.1:18789`）在此模式下**不会**工作。

### 公共互联网（Funnel + 共享密码）

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

优先使用 `OPENCLAW_GATEWAY_PASSWORD`，而不是将密码提交到磁盘。

## CLI 示例

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 注意

- Tailscale Serve/Funnel 需要安装并登录 `tailscale` CLI。
- `tailscale.mode: "funnel"` 拒绝启动，除非认证模式是 `password` 以避免公共暴露。
- 如果你希望 OpenClaw 在关闭时撤销 `tailscale serve` 或 `tailscale funnel` 配置，设置 `gateway.tailscale.resetOnExit`。
- `gateway.bind: "tailnet"` 是直接 Tailnet 绑定（无 HTTPS，无 Serve/Funnel）。
- `gateway.bind: "auto"` 优先使用 loopback；如果你想要仅限 tailnet，使用 `tailnet`。
- Serve/Funnel 仅暴露**网关控制 UI + WS**。节点通过相同的网关 WS 端点连接，因此 Serve 可以用于节点访问。

## 浏览器控制（远程网关 + 本地浏览器）

如果你在一台机器上运行网关，但想在另一台机器上驱动浏览器，
在浏览器机器上运行**节点主机**，并让两者保持在同一个 tailnet 上。
网关会将浏览器操作代理到节点；不需要单独的控制服务器或 Serve URL。

避免将 Funnel 用于浏览器控制；将节点配对视为操作员访问。

## Tailscale 先决条件 + 限制

- Serve 需要为你的 tailnet 启用 HTTPS；如果缺少，CLI 会提示。
- Serve 注入 Tailscale 身份标头；Funnel 不会。
- Funnel 需要 Tailscale v1.38.3+、MagicDNS、HTTPS 启用，以及 funnel 节点属性。
- Funnel 仅支持端口 `443`、`8443` 和 `10000` 上的 TLS。
- macOS 上的 Funnel 需要开源 Tailscale 应用变体。

## 了解更多

- Tailscale Serve 概览：https://tailscale.com/kb/1312/serve
- `tailscale serve` 命令：https://tailscale.com/kb/1242/tailscale-serve
- Tailscale Funnel 概览：https://tailscale.com/kb/1223/tailscale-funnel
- `tailscale funnel` 命令：https://tailscale.com/kb/1311/tailscale-funnel
