---
summary: "群组管理详解 - 群组策略、权限和配置"
read_when:
  - 配置群组访问策略
  - 设置群组权限
  - 管理群组行为
title: "群组管理"
---

# 👥 群组管理

本文档详细介绍 OpenClaw 的群组管理功能，包括策略配置、权限控制和最佳实践。

## 🎯 群组策略类型

### 三种策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| **allowlist** | 仅允许白名单群组 | 受控环境 |
| **open** | 允许所有群组 | 开放环境 |
| **disabled** | 禁用群组功能 | 仅私聊 |

### 配置策略

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist"
    },
    discord: {
      groupPolicy: "allowlist"
    },
    whatsapp: {
      groupPolicy: "allowlist"
    }
  }
}
```

---

## 📋 白名单管理

### 全局白名单

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {                    // 使用 * 作为全局白名单
          requireMention: true    // 需要 @提及
        }
      }
    }
  }
}
```

### 特定群组配置

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {                           // 默认配置
          requireMention: true,
          allowFrom: ["*"]              // 允许所有人
        },
        "-1001234567890": {              // 技术讨论群
          requireMention: false,         // 不需要 @提及
          allowFrom: ["@team-member"],   // 仅团队成员
          systemPrompt: "技术讨论，请提供代码示例。"
        },
        "-1009876543210": {              # 支持群
          requireMention: true,
          allowFrom: ["*"],
          systemPrompt: "技术支持，快速响应问题。"
        }
      }
    }
  }
}
```

### 添加/移除群组

```bash
# 添加群组到白名单
openclaw config set channels.telegram.groups."-1001234567890" '{"requireMention":true}'

# 移除群组
openclaw config unset channels.telegram.groups."-1001234567890"
```

---

## 🔐 权限控制

### 用户权限

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          allowFrom: [
            "@admin",           # 管理员
            "@moderator",       # moderator
            "@team-member"      # 团队成员
          ],
          denyFrom: [
            "@banned-user"      # 禁止用户
          ]
        }
      }
    }
  }
}
```

### 权限级别

| 级别 | 说明 |
|------|------|
| `allowFrom: ["*"]` | 允许所有人 |
| `allowFrom: ["@user"]` | 仅允许特定用户 |
| `denyFrom: ["@user"]` | 禁止特定用户 |

---

## ⚙️ 群组行为配置

### 基本配置

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          // 是否需要 @提及
          requireMention: true,
          
          // 历史消息数
          historyLimit: 50,
          
          // 系统提示词
          systemPrompt: "你是技术助手，提供准确的信息。",
          
          # 是否允许回复
          allowReplies: true
        }
      }
    }
  }
}
```

### 高级配置

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          requireMention: true,
          historyLimit: 100,
          
          // 话题配置
          topics: {
            "99": {
              requireMention: false,
              systemPrompt: "问答话题。"
            }
          },
          
          // 命令配置
          commands: {
            help: true,
            status: true,
            restart: false  # 禁止重启命令
          },
          
          # 响应限制
          rateLimit: {
            perUser: 10,    # 每用户每分钟
            perGroup: 50    # 群组每分钟
          }
        }
      }
    }
  }
}
```

---

## 📊 群组监控

### 查看群组状态

```bash
# 列出所有群组
openclaw channels status | grep group

# 查看特定群组
openclaw sessions list --channel telegram | grep "-1001234567890"

# 群组消息统计
openclaw sessions stats --by-channel
```

### 日志记录

```json5
{
  logging: {
    groups: {
      enabled: true,
      includeMessages: false,  # 不记录消息内容
      includeMetadata: true    # 记录元数据
    }
  }
}
```

---

## 🔄 群组生命周期

### 创建群组

1. 将群组 ID 添加到白名单
2. 配置基本行为
3. 设置系统提示词（如需要）

### 更新配置

```bash
# 更新群组配置
openclaw config set channels.telegram.groups."-1001234567890" '{
  "requireMention": true,
  "systemPrompt": "新的提示词。"
}'
```

### 移除群组

```bash
# 从白名单移除
openclaw config unset channels.telegram.groups."-1001234567890"

# 清除群组会话
openclaw sessions clear agent:main:telegram:group:-1001234567890
```

---

## 🛡️ 安全考虑

### 防止滥用

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {
          requireMention: true,
          rateLimit: {
            perUser: 5,     # 严格限制
            perGroup: 30
          },
          denyFrom: ["@spam-user", "@bot"]
        }
      }
    }
  }
}
```

### 敏感操作保护

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          commands: {
            restart: false,
            config: false,
            security: false
          }
        }
      }
    }
  }
}
```

---

## 📈 最佳实践

### 公开群组

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {
          requireMention: true,
          allowFrom: ["*"],
          systemPrompt: "欢迎使用 OpenClaw！请 @提及 我来获取帮助。",
          historyLimit: 50,
          rateLimit: {
            perUser: 10,
            perGroup: 100
          }
        }
      }
    }
  }
}
```

### 团队内部群组

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "-1001234567890": {
          requireMention: false,      # 不需要 @提及
          allowFrom: [
            "@engineer-1",
            "@engineer-2",
            "@engineer-3"
          ],
          systemPrompt: "团队内部助手。可以直接对话，不需要 @提及。",
          historyLimit: 200,
          commands: {
            restart: true,
            config: true
          }
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
| `openclaw config` | 配置群组 |
| `openclaw sessions` | 管理会话 |
| `openclaw channels status` | 查看状态 |
| `openclaw logs` | 查看日志 |

---

## 📚 相关文档

- [群组消息](/zh-CN/concepts/group-messages) - 消息处理
- [渠道配置](/zh-CN/channels) - 各渠道配置
- [会话管理](/zh-CN/concepts/sessions) - 会话系统
- [配对与安全](/zh-CN/start/pairing) - 安全策略

---

**合理的群组管理，让 AI 助手在群聊中发挥最大价值！** 🦞
