---
summary: "群组消息处理详解 - @提及、话题和群组配置"
read_when:
  - 配置群组聊天行为
  - 设置 @提及规则
  - 管理话题线程
title: "群组消息"
---

# 👥 群组消息

本文档详细介绍 OpenClaw 在群组聊天中的消息处理机制，包括 @提及、话题和配置选项。

## 🎯 群组消息概述

群组消息与私聊有显著区别：

| 特性 | 私聊（DM） | 群组 |
|------|------------|------|
| 参与者 | 两人 | 多人 |
| 激活方式 | 自动 | @提及或配置 |
| 上下文 | 共享主会话 | 独立会话 |
| 隐私 | 较高 | 较低 |

---

## @提及触发

### 配置提及模式

```json5
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: [
            "@openclaw",     // @机器人名称
            "@小助手",       // 中文昵称
            "@机器人",       // 通用称呼
            "OpenClaw"       # 直接名称
          ]
        }
      }
    ]
  }
}
```

### 渠道特定配置

**Telegram**：
```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true },  // 所有群组需要 @提及
        "-1001234567890": {              // 特定群组
          requireMention: false,         # 该群组不需要 @提及
          allowFrom: ["@admin"]         # 仅允许特定用户
        }
      }
    }
  }
}
```

**Discord**：
```json5
{
  channels: {
    discord: {
      guilds: {
        "123456789012345678": {
          requireMention: true,
          channels: {
            "general": { allow: true },
            "help": { allow: true, requireMention: true }
          }
        }
      }
    }
  }
}
```

**WhatsApp**：
```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

---

## 🧵 话题线程

### Telegram 话题支持

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "99": {                    // 话题 ID
              requireMention: false,
              systemPrompt: "这是问答话题，请提供详细解答。",
              skills: ["search", "docs"]
            },
            "100": {
              requireMention: true,
              systemPrompt: "技术讨论，保持专业。"
            }
          }
        }
      }
    }
  }
}
```

### 话题创建

当用户在 Telegram 群组中创建新话题时：
1. OpenClaw 自动识别话题
2. 为该话题创建独立会话上下文
3. 应用话题特定的系统提示词

---

## 📊 群组会话管理

### 独立会话模式

```json5
{
  session: {
    groupIsolation: true  // 每个群组独立会话
  }
}
```

**会话键格式**：
```
agent:main:telegram:group:-1001234567890
                                   └──┬───────────┘
                                      └── 群组 ID
```

### 群组白名单

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": { requireMention: true },  // 所有群组
        "-1001234567890": {              // 特定群组
          allowFrom: ["@team-member"]   // 白名单用户
        }
      }
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `allowlist` | 仅允许白名单中的群组 |
| `open` | 允许所有群组 |
| `disabled` | 禁用群组消息 |

---

## ⚙️ 群组配置

### 群组特定系统提示词

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          systemPrompt: "这是技术讨论群，请使用专业语言，提供准确的代码示例。"
        },
        "-1009876543210": {
          systemPrompt: "这是休闲聊天群，可以轻松幽默，保持友好氛围。"
        }
      }
    }
  }
}
```

### 消息历史限制

```json5
{
  channels: {
    telegram: {
      historyLimit: 100,       // 群组消息数
      dmHistoryLimit: 50       # DM 消息数
    },
    discord: {
      historyLimit: 50,
      dmHistoryLimit: 30
    }
  }
}
```

---

## 🔔 群组通知

### 消息确认

```json5
{
  messages: {
    ackReaction: "👀",  // 已读反应
    groupChat: {
      mentionNotify: true,   // @提及时通知
      replyNotify: true      # 回复时通知
    }
  }
}
```

### 活动模式

```markdown
在群组中发送：
/activation mention   # 仅 @提及时响应（默认）
/activation always    # 始终响应
```

---

## 📈 群组使用统计

### 查看群组活动

```bash
# 列出所有群组会话
openclaw sessions list --type group

# 查看群组消息统计
openclaw sessions stats --by-channel

# 查看特定群组历史
openclaw sessions history agent:main:telegram:group:-1001234567890
```

---

## 🐛 故障排除

### 机器人不响应

```bash
# 检查群组配置
openclaw config get channels.telegram.groups

# 检查提及模式
openclaw config get agents.defaults.groupChat.mentionPatterns

# 查看是否被禁用
openclaw config get channels.telegram.groupPolicy
```

### 消息丢失

```bash
# 检查历史限制
openclaw config get channels.telegram.historyLimit

# 查看日志
openclaw logs --lines 100 | grep "group"
```

### 错误的群组上下文

```bash
# 清除群组会话
openclaw sessions clear agent:main:telegram:group:-1001234567890

# 重新配置
openclaw config set channels.telegram.groups."-1001234567890".systemPrompt "..."
```

---

## 📝 最佳实践

### 技术讨论群

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          requireMention: true,
          systemPrompt: "技术讨论群。规则：1. 提供准确的代码示例；2. 解释原理；3. 引用官方文档。",
          historyLimit: 100
        }
      }
    }
  }
}
```

### 多群组管理

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {
          requireMention: true,
          systemPrompt: "默认群组行为：礼貌、专业、简洁。"
        },
        "-1001234567890": {
          requireMention: false,
          systemPrompt: "技术支持群：详细解答，步骤清晰。"
        },
        "-1009876543210": {
          requireMention: true,
          systemPrompt: "反馈群：快速响应，收集意见。"
        }
      }
    }
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw sessions list --type group` | 列出群组会话 |
| `openclaw message send --target group:*` | 发送到群组 |
| `openclaw pairing approve` | 审批群组请求 |
| `openclaw config` | 配置群组设置 |

---

## 📚 相关文档

- [会话管理](/zh-CN/concepts/sessions) - 会话系统
- [消息系统](/zh-CN/concepts/messages) - 消息处理
- [渠道配置](/zh-CN/channels) - 各渠道详细配置
- [配对与安全](/zh-CN/start/pairing) - 安全策略

---

**合理的群组配置，让 AI 助手在群聊中既活跃又不打扰！** 🦞
