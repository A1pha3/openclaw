---
summary: "通过 grammY 集成 Telegram Bot API，包含设置说明"
read_when:
  - 使用 Telegram 或 grammY 通道时
title: grammY
---

# grammY 集成（Telegram Bot API）

## 为什么选择 grammY

- TS 优先的 Bot API 客户端，内置长轮询 + webhook 辅助函数、中间件、错误处理、速率限制器。
- 比手动编写 fetch + FormData 更简洁的媒体辅助函数；支持所有 Bot API 方法。
- 可扩展：通过自定义 fetch 支持代理、会话中间件（可选）、类型安全的上下文。

## 我们提供的功能

- **单一客户端路径：** 已移除基于 fetch 的实现；grammY 现在是唯一的 Telegram 客户端（发送 + 网关），默认启用 grammY 限流器。
- **网关：** `monitorTelegramProvider` 构建 grammY `Bot`，连接提及/白名单过滤，通过 `getFile`/`download` 下载媒体，使用 `sendMessage`/`sendPhoto`/`sendVideo`/`sendAudio`/`sendDocument` 发送回复。通过 `webhookCallback` 支持长轮询或 webhook。
- **代理：** 可选的 `channels.telegram.proxy` 使用 `undici.ProxyAgent`，通过 grammY 的 `client.baseFetch`。
- **Webhook 支持：** `webhook-set.ts` 封装 `setWebhook`/`deleteWebhook`；`webhook.ts` 托管回调，包含健康检查 + 优雅关闭。当设置 `channels.telegram.webhookUrl` 时，网关启用 webhook 模式（否则使用长轮询）。
- **会话：** 直接对话合并到智能体主会话（`agent:<agentId>:<mainKey>`）；群组使用 `agent:<agentId>:telegram:group:<chatId>`；回复路由回同一通道。
- **配置选项：** `channels.telegram.botToken`、`channels.telegram.dmPolicy`、`channels.telegram.groups`（白名单 + 提及默认值）、`channels.telegram.allowFrom`、`channels.telegram.groupAllowFrom`、`channels.telegram.groupPolicy`、`channels.telegram.mediaMaxMb`、`channels.telegram.linkPreview`、`channels.telegram.proxy`、`channels.telegram.webhookSecret`、`channels.telegram.webhookUrl`。
- **草稿流式传输：** 可选的 `channels.telegram.streamMode` 在私有主题对话中使用 `sendMessageDraft`（Bot API 9.3+）。这与通道块流式传输是分开的。
- **测试：** grammY 模拟覆盖 DM + 群组提及过滤和出站发送；欢迎提供更多媒体/webhook 测试用例。

## 开放性问题

- 如果遇到 Bot API 429 错误，可选的 grammY 插件（限流器）。
- 添加更多结构化的媒体测试（贴纸、语音笔记）。
- 使 webhook 监听端口可配置（目前固定为 8787，除非通过网关连接）。
