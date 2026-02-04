---
summary: "通道连接性的健康检查步骤"
read_when:
  - "诊断 WhatsApp 通道健康状况"
title: "健康检查"
---

# 健康检查（CLI）

验证通道连接性的简短指南，无需猜测。

## 快速检查

- `openclaw status` — 本地摘要：网关可达性/模式、更新提示、链接通道认证年龄、会话 + 最近活动。
- `openclaw status --all` — 完整本地诊断（只读、颜色、安全粘贴用于调试）。
- `openclaw status --deep` — 还探测运行的网关（支持时按通道探测）。
- `openclaw health --json` — 请求运行的网关提供完整健康快照（仅 WS；没有直接 Baileys 套接字）。
- 在 WhatsApp/WebChat 中发送 `/status` 作为独立消息以获取状态回复而不调用代理。
- 日志：尾随 `/tmp/openclaw/openclaw-*.log` 并过滤 `web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound`。

## 深度诊断

- 磁盘上的凭据：`ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json`（mtime 应该很新）。
- 会话存储：`ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json`（路径可以在配置中覆盖）。计数和最近收件人通过 `status` 显示。
- 重新链接流程：当日志中出现状态码 409–515 或 `loggedOut` 时，使用 `openclaw channels logout && openclaw channels login --verbose` 重新链接。（注意：QR 登录流程在状态 515 后自动重启一次配对。）

## 当某些东西失败时

- `logged out` 或状态 409–515 → 使用 `openclaw channels logout` 然后 `openclaw channels login` 重新链接。
- 网关不可达 → 启动它：`openclaw gateway --port 18789`（如果端口繁忙，使用 `--force`）。
- 没有入站消息 → 确认链接的电话在线并且发送者被允许（`channels.whatsapp.allowFrom`）；对于群聊，确保允许列表 + 提及规则匹配（`channels.whatsapp.groups`、`agents.list[].groupChat.mentionPatterns`）。

## 专用"健康"命令

`openclaw health --json` 请求运行的网关提供其健康快照（CLI 没有直接通道套接字）。它在可用时报告链接的凭据/认证年龄、每个通道探测摘要、会话存储摘要和探测持续时间。如果网关不可达或探测失败/超时，它以非零退出。使用 `--timeout <ms>` 覆盖 10 秒默认值。
