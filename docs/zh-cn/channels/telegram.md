# Telegram 配置

本文档介绍如何配置 OpenClaw 连接 Telegram Bot。

## 创建 Telegram Bot

### 1. 联系 BotFather

在 Telegram 中搜索 [@BotFather](https://t.me/BotFather) 并发送消息。

### 2. 创建新 Bot

发送命令：

```
/newbot
```

按提示操作：
1. 输入 Bot 名称（显示名）
2. 输入 Bot 用户名（必须以 `bot` 结尾）

### 3. 获取 Token

BotFather 会返回类似这样的 Token：

```
123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

**保存好这个 Token！**

## 配置 OpenClaw

### 快速配置

```bash
openclaw config set channels.telegram.botToken "YOUR_BOT_TOKEN"
```

### 完整配置

```json5
{
  channels: {
    telegram: {
      // 基础设置
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}",
      
      // DM 策略
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789", "@username"],
      
      // 群组设置
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "保持回复简短。",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"]
            }
          }
        }
      },
      
      // 自定义命令
      customCommands: [
        { command: "backup", description: "Git 备份" },
        { command: "generate", description: "生成图片" }
      ],
      
      // 消息设置
      historyLimit: 50,
      replyToMode: "first",
      linkPreview: true,
      
      // 流式输出
      streamMode: "partial",
      
      // 动作开关
      actions: {
        reactions: true,
        sendMessage: true
      },
      
      // 反应通知
      reactionNotifications: "own",
      
      // 媒体限制
      mediaMaxMb: 5,
      
      // 重试策略
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000
      }
    }
  }
}
```

## DM 策略

### 配置选项

| 策略 | 说明 |
|------|------|
| `pairing` | 未知用户需要配对审批（推荐） |
| `allowlist` | 仅允许白名单用户 |
| `open` | 允许所有用户（需 `allowFrom: ["*"]`） |
| `disabled` | 禁用所有私信 |

### allowFrom 格式

```json5
{
  allowFrom: [
    "tg:123456789",    // 用户 ID
    "@username",       // 用户名
    "*"                // 所有用户（仅与 open 策略配合）
  ]
}
```

### 获取用户 ID

发送任意消息给 [@userinfobot](https://t.me/userinfobot)。

## 群组配置

### 群组策略

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",  // allowlist | open | disabled
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

### 特定群组配置

```json5
{
  groups: {
    "-1001234567890": {
      allowFrom: ["@admin", "@moderator"],
      requireMention: false,
      systemPrompt: "这是技术讨论群，请使用专业语言。",
      historyLimit: 100
    }
  }
}
```

### 话题支持

Telegram 超级群组支持话题：

```json5
{
  groups: {
    "-1001234567890": {
      topics: {
        "99": {
          requireMention: false,
          systemPrompt: "这是问答话题。",
          skills: ["search", "docs"]
        }
      }
    }
  }
}
```

## 自定义命令

### 注册命令

```json5
{
  customCommands: [
    { command: "help", description: "显示帮助" },
    { command: "status", description: "查看状态" },
    { command: "search", description: "搜索内容" }
  ]
}
```

### 命令菜单

注册的命令会显示在 Bot 命令菜单中。

## 消息设置

### 回复模式

```json5
{
  replyToMode: "first"  // off | first | all
}
```

| 模式 | 说明 |
|------|------|
| `off` | 不回复原消息 |
| `first` | 仅回复第一条消息 |
| `all` | 回复所有消息 |

### 链接预览

```json5
{
  linkPreview: true  // 是否显示链接预览
}
```

### 历史限制

```json5
{
  historyLimit: 50  // 群组消息上下文数量
}
```

## 流式输出

### 草稿流式

使用 Telegram 草稿功能实时显示输出：

```json5
{
  streamMode: "partial"  // off | partial | block
}
```

### 块流式配置

```json5
{
  streamMode: "block",
  draftChunk: {
    minChars: 200,
    maxChars: 800,
    breakPreference: "paragraph"  // paragraph | newline | sentence
  }
}
```

## 反应通知

```json5
{
  reactionNotifications: "own"  // off | own | all
}
```

| 模式 | 说明 |
|------|------|
| `off` | 不接收反应事件 |
| `own` | 仅接收对自己消息的反应 |
| `all` | 接收所有反应事件 |

## 多账号配置

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "主机器人",
          botToken: "${TELEGRAM_BOT_TOKEN}"
        },
        alerts: {
          name: "告警机器人",
          botToken: "${TELEGRAM_ALERTS_TOKEN}"
        }
      }
    }
  }
}
```

## Webhook 模式

对于高流量场景，使用 Webhook 而非轮询：

```json5
{
  webhookUrl: "https://your-domain.com/telegram-webhook",
  webhookSecret: "your-secret",
  webhookPath: "/telegram-webhook"
}
```

## 网络配置

### 代理设置

```json5
{
  proxy: "socks5://localhost:9050"
}
```

### 网络超时

```json5
{
  network: {
    autoSelectFamily: false
  }
}
```

## 常用命令

### 查看状态

```bash
openclaw channels status telegram
openclaw channels status telegram --probe
```

### 发送消息

```bash
# 发送私信
openclaw message send --channel telegram --target "tg:123456789" --message "你好"

# 发送到群组
openclaw message send --channel telegram --target "group:-1001234567890" --message "群消息"
```

## 故障排除

### Bot 无响应

1. 验证 Token：
```bash
curl https://api.telegram.org/bot<TOKEN>/getMe
```

2. 检查配置：
```bash
openclaw config get channels.telegram
```

3. 检查日志：
```bash
openclaw logs --tail 50
```

### 群组无法使用

1. 确保 Bot 已加入群组
2. 检查 Bot 权限（需要读取消息权限）
3. 检查群组策略和 allowFrom 配置

### 配对问题

```bash
# 查看待审批配对
openclaw pairing list telegram

# 审批
openclaw pairing approve telegram <code>
```

## 最佳实践

1. **使用环境变量存储 Token**: 不要在配置文件中硬编码
2. **启用配对模式**: 保护 Bot 不被滥用
3. **配置群组 requireMention**: 避免过度响应
4. **设置合理的历史限制**: 平衡上下文和 token 消耗

## 下一步

- [Discord 配置](/zh-cn/channels/discord)
- [渠道概述](/zh-cn/channels/index)
- [配置参考](/zh-cn/config/reference)
