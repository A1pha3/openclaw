---
summary: "跨渠道表情回应的统一语义"
read_when:
  - 在任何渠道处理表情回应功能
title: "表情回应"
---

# 表情回应工具

OpenClaw 在各消息渠道间统一了表情回应（Reaction）的语义：

## 核心规则

| 参数 | 说明 |
|------|------|
| `emoji` | 添加回应时**必填** |
| `emoji=""` | 空字符串 = 移除机器人的回应（如渠道支持） |
| `remove: true` | 移除指定表情（需同时提供 `emoji`） |

## 各渠道行为差异

| 渠道 | 空 `emoji` 行为 | `remove: true` 行为 |
|------|----------------|---------------------|
| **Discord / Slack** | 移除机器人在该消息上的**所有**回应 | 仅移除指定表情 |
| **Google Chat** | 移除应用的回应 | 仅移除指定表情 |
| **Telegram** | 移除机器人的回应 | 同样移除回应，但工具验证仍需非空 `emoji` |
| **WhatsApp** | 移除机器人回应 | 映射为空 emoji（仍需提供 `emoji` 参数） |
| **Signal** | 当 `channels.signal.reactionNotifications` 启用时，入站回应通知会触发系统事件 |

## 使用示例

```typescript
// 添加回应
await reaction({ messageId: "...", emoji: "👍" })

// 移除指定回应
await reaction({ messageId: "...", emoji: "👍", remove: true })

// 移除所有回应（Discord/Slack/Google Chat）
await reaction({ messageId: "...", emoji: "" })
```

## 注意事项

- 不是所有渠道都支持完整的回应操作
- Signal 的回应通知需要在配置中显式启用
- WhatsApp 的回应 API 有特殊限制，详见 [WhatsApp 渠道](/zh-cn/channels/whatsapp)
