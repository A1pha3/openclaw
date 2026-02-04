# 渠道概述

OpenClaw 支持连接多种消息平台，让您的 AI 代理能够通过用户熟悉的通信工具进行交互。

## 支持的渠道

### 内置渠道

这些渠道随 OpenClaw 核心安装，无需额外插件。

| 渠道 | 类型 | 状态 | 说明 |
|------|------|------|------|
| [WhatsApp](/zh-CN/channels/whatsapp) | Web 协议 | 稳定 | 通过 Baileys 库实现 WhatsApp Web |
| [Telegram](/zh-CN/channels/telegram) | Bot API | 稳定 | 支持私聊和群组 |
| [Discord](/zh-CN/channels/discord) | Gateway + REST | 稳定 | 支持服务器和私信 |
| [Slack](/zh-CN/channels/slack) | Socket Mode | 稳定 | 支持频道和私信 |
| [Signal](/zh-CN/channels/signal) | signal-cli | 稳定 | 通过 signal-cli 实现安全通信 |
| [iMessage](/zh-CN/channels/imessage) | 本地 CLI | 稳定 | 仅 macOS |
| [Google Chat](/zh-CN/channels/googlechat) | Webhook | 稳定 | 需要服务账号 |

### 插件渠道

这些渠道需要安装额外插件。

```bash
# 安装插件示例
openclaw plugins install @openclaw/mattermost
```

| 渠道 | 插件包 | 状态 | 说明 |
|------|--------|------|------|
| [Matrix](/zh-CN/channels/matrix) | `@openclaw/matrix` | 稳定 | 去中心化协议 |
| [Microsoft Teams](/zh-CN/channels/msteams) | `@openclaw/msteams` | 稳定 | 企业协作 |
| [Mattermost](/zh-CN/channels/mattermost) | `@openclaw/mattermost` | 稳定 | 开源团队协作 |
| [Twitch](/zh-CN/channels/twitch) | `@openclaw/twitch` | Beta | 直播聊天 |
| [Nostr](/zh-CN/channels/nostr) | `@openclaw/nostr` | Beta | 去中心化社交 |
| [Zalo](/zh-CN/channels/zalo) | `@openclaw/zalo` | Beta | 越南通信应用 |

## 渠道功能对比

### 消息类型支持

| 功能 | WhatsApp | Telegram | Discord | Slack | iMessage |
|------|----------|----------|---------|-------|----------|
| 文本消息 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 图片 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 音频 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 视频 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 文件 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 位置 | ✅ | ✅ | ❌ | ❌ | ❌ |
| 贴纸 | ✅ | ✅ | ✅ | ✅ | ❌ |
| 表情回复 | ✅ | ✅ | ✅ | ✅ | ❌ |
| 投票 | ❌ | ✅ | ✅ | ❌ | ❌ |

### 群组功能

| 功能 | WhatsApp | Telegram | Discord | Slack | iMessage |
|------|----------|----------|---------|-------|----------|
| 群组消息 | ✅ | ✅ | ✅ | ✅ | ✅ |
| @提及 | ✅ | ✅ | ✅ | ✅ | ❌ |
| 话题/线程 | ❌ | ✅ | ✅ | ✅ | ❌ |
| 权限管理 | ❌ | ✅ | ✅ | ✅ | ❌ |
| 历史记录 | 有限 | ✅ | ✅ | ✅ | ✅ |

## 通用配置

### 渠道启用/禁用

每个渠道都可以独立启用或禁用：

```json5
{
  "channels": {
    "whatsapp": { "enabled": true },
    "telegram": { "enabled": true },
    "discord": { "enabled": false }
  }
}
```

### DM 策略

控制如何处理私信：

```json5
{
  "channels": {
    "telegram": {
      "dmPolicy": "pairing"  // pairing | allowlist | open | disabled
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `pairing` | 未知用户需要配对审批（推荐） |
| `allowlist` | 仅允许白名单用户 |
| `open` | 允许所有用户（需配合 `allowFrom: ["*"]`） |
| `disabled` | 禁用所有私信 |

### 群组策略

控制如何处理群组消息：

```json5
{
  "channels": {
    "whatsapp": {
      "groupPolicy": "allowlist",
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

### 提及触发

配置群组中触发机器人响应的条件：

```json5
{
  "agents": {
    "list": [
      {
        "id": "main",
        "groupChat": {
          "mentionPatterns": ["@clawd", "小助手", "openclaw"]
        }
      }
    ]
  }
}
```

## 多账号支持

大多数渠道支持配置多个账号：

```json5
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "name": "主机器人",
          "botToken": "123456:ABC..."
        },
        "alerts": {
          "name": "告警机器人",
          "botToken": "987654:XYZ..."
        }
      }
    }
  }
}
```

## 快速配置指南

### WhatsApp（最常用）

```bash
# 扫码登录
openclaw channels login

# 查看状态
openclaw channels status whatsapp
```

### Telegram

```bash
# 设置 Bot Token
openclaw config set channels.telegram.botToken "YOUR_TOKEN"

# 查看状态
openclaw channels status telegram
```

### Discord

```bash
# 设置 Bot Token
openclaw config set channels.discord.token "YOUR_TOKEN"

# 查看状态
openclaw channels status discord
```

## 渠道状态检查

查看所有渠道状态：

```bash
openclaw channels status
```

带探测的深度检查：

```bash
openclaw channels status --probe
```

## 故障排除

### 渠道无法连接

1. 检查网络连接
2. 验证认证信息（Token/凭证）
3. 运行 `openclaw doctor` 诊断
4. 查看日志 `openclaw logs --tail 100`

### 消息发送失败

1. 确认渠道已连接
2. 检查目标格式是否正确
3. 验证权限和配额

### 收不到消息

1. 检查 allowlist 配置
2. 确认 DM/群组策略
3. 处理待定的配对请求

## 下一步

选择您要使用的渠道，查看详细配置指南：

### 内置渠道

- [WhatsApp 配置](/zh-CN/channels/whatsapp)
- [Telegram 配置](/zh-CN/channels/telegram)
- [Discord 配置](/zh-CN/channels/discord)
- [Slack 配置](/zh-CN/channels/slack)
- [Signal 配置](/zh-CN/channels/signal)
- [iMessage 配置](/zh-CN/channels/imessage)

### 插件渠道

- [Matrix 配置](/zh-CN/channels/matrix)
- [Microsoft Teams 配置](/zh-CN/channels/msteams)
- [Mattermost 配置](/zh-CN/channels/mattermost)

---

## 相关文档

- [新手上路](/zh-CN/start/getting-started) - 首次配置指南
- [CLI 参考](/zh-CN/cli) - 渠道管理命令
- [故障排除](/zh-CN/help/troubleshooting) - 问题解决
