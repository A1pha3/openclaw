---
summary: "macOS 应用如何报告网关/Baileys 健康状态"
read_when:
  - 调试 mac 应用健康指示器
title: "健康检查"
---

# macOS 健康检查

如何从菜单栏应用查看链接的频道是否健康。

## 菜单栏

- 状态点现在反映 Baileys 健康状态：
  - 绿色：已链接 + socket 最近已打开。
  - 橙色：连接中/重试中。
  - 红色：已登出或探测失败。
- 第二行显示"linked · auth 12m"或显示失败原因。
- "Run Health Check"菜单项触发按需探测。

## 设置

- General 选项卡增加了健康卡片，显示：链接的认证年龄、会话存储路径/计数、上次检查时间、上次错误/状态码，以及 Run Health Check / Reveal Logs 按钮。
- 使用缓存快照，因此 UI 即时加载并在离线时优雅回退。
- **Channels 选项卡**展示 WhatsApp/Telegram 的频道状态 + 控制（登录二维码、登出、探测、上次断开/错误）。

## 探测工作原理

- 应用每约 60 秒和按需通过 `ShellExecutor` 运行 `openclaw health --json`。探测加载凭据并报告状态，不发送消息。
- 分别缓存上次良好快照和上次错误以避免闪烁；显示每个的时间戳。

## 有疑问时

- 你仍然可以使用 [网关健康](/zh-cn/gateway/health) 中的 CLI 流程（`openclaw status`、`openclaw status --deep`、`openclaw health --json`）并跟踪 `/tmp/openclaw/openclaw-*.log` 中的 `web-heartbeat` / `web-reconnect`。
