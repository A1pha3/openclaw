---
summary: "群组管理完整指南：策略类型、白名单管理、权限控制、行为配置和安全最佳实践"
read_when:
  - 配置群组访问策略
  - 设置群组权限
  - 管理群组行为
title: "群组管理"
---

# 👥 群组管理

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的三种群组策略类型
- [ ] 配置群组白名单和权限控制
- [ ] 设置 requireMention 和 allowFrom 参数
- [ ] 管理群组会话和配置

### 进阶目标（建议掌握）

- [ ] 配置群组级别的系统提示词
- [ ] 设置群组速率限制防止滥用
- [ ] 配置话题和命令权限
- [ ] 实施群组安全策略

---

## 💡 为什么需要群组管理？

### 类比：群组管理就像"门卫系统"

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    群组管理类比                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  没有群组管理的混乱场景：                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  任何人都可以在任何群组中召唤 AI                               │   │
│  │                                                                 │   │
│  │  问题：                                                          │   │
│  │  • AI 可能被加入不当群组                                       │   │
│  │  • 无法控制谁能使用 AI                                         │   │
│  │  • 可能被滥用或刷屏                                             │   │
│  │  • 无法针对不同群组定制行为                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  有群组管理的有序场景：                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  门卫检查                                                       │   │
│  │  ├── 是否在白名单中？                                          │   │
│  │  ├── 是否有权限使用？                                          │   │
│  │  └── 是否需要 @提及？                                           │   │
│  │                                                                 │   │
│  │  优势：                                                          │   │
│  │  ✅ 精确控制 AI 在哪些群组活跃                                  │   │
│  │  ✅ 灵活的权限管理                                              │   │
│  │  ✅ 可定制的群组行为                                            │   │
│  │  ✅ 防止滥用和刷屏                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 群组策略类型

### 三种策略

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    策略选择对比                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  allowlist（白名单，推荐）                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  仅白名单群组可以访问 AI                                        │   │
│  │                                                                 │   │
│  │  适用：                                                          │   │
│  │  • 受控环境（公司/团队）                                       │   │
│  │  • 需要精确控制的场景                                           │   │
│  │  • 避免意外暴露                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  open（开放）                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  所有群组都可以访问 AI                                        │   │
│  │                                                                 │   │
│  │  适用：                                                          │   │
│  │  • 公开服务                                                     │   │
│  │  • 社区机器人                                                   │   │
│  │  • 开放测试环境                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  disabled（禁用）                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  完全禁用群组功能                                             │   │
│  │                                                                 │   │
│  │  适用：                                                          │   │
│  │  • 仅私聊使用                                                   │   │
│  │  • 暂时关闭群组功能                                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 配置策略

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist"      // allowlist | open | disabled
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
        "*": {                      // 全局默认配置
          requireMention: true,      // 需要 @提及才响应
          allowFrom: ["*"]          // 允许所有人
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
        "*": {                           // 默认：需要 @提及
          requireMention: true,
          allowFrom: ["*"]
        },
        "-1001234567890": {              // 技术讨论群：不需要 @，仅团队
          requireMention: false,
          allowFrom: ["@team-member"],
          systemPrompt: "技术讨论群。请提供代码示例和详细描述。"
        },
        "-1009876543210": {              // 支持群：需要 @，所有人
          requireMention: true,
          allowFrom: ["*"],
          systemPrompt: "技术支持群。请描述您遇到的问题。"
        }
      }
    }
  }
}
```

### 群组 ID 获取

```bash
# Telegram：通过 /status 命令
/status

# Discord：使用服务器 ID
# 服务器设置 → 开发者模式 → 复制 ID

# WhatsApp：群组邀请链接中的 ID
# chat.whatsapp.com/<groupId>
```

### 添加/移除群组

```bash
# 添加群组
openclaw config set channels.telegram.groups."-1001234567890" '{
  "requireMention": true,
  "allowFrom": ["*"]
}'

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
            "@admin",           // 管理员
            "@moderator",       // 版主
            "@team-member"      // 团队成员
          ],
          denyFrom: [
            "@banned-user"      // 禁止的用户
          ]
        }
      }
    }
  }
}
```

### 权限级别

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    权限控制层级                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  allowFrom: ["*"] ─────────► 允许所有人                             │
│                                                                         │
│  allowFrom: ["@user"] ─────► 仅特定用户                             │
│                                                                         │
│  denyFrom: ["@user"] ───────► 禁止特定用户                           │
│                                                                         │
│  组合使用：                                                            │
│  allowFrom: ["@member", "@guest"]                                     │
│  denyFrom: ["@banned"]                                               │
│  ─────► 仅 member 和 guest 可用，banned 禁止                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ 群组行为配置

### 基本配置

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          requireMention: true,           // 是否需要 @提及
          historyLimit: 50,                // 历史消息数
          systemPrompt: "你是技术助手。",   // 群组专用提示词
          allowReplies: true              // 是否允许回复
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

          // 话题配置（Telegram 群组话题）
          topics: {
            "99": {
              requireMention: false,       // 此话题不需要 @
              systemPrompt: "话题 99：问答专区"
            }
          },

          // 命令权限控制
          commands: {
            help: true,                   // 允许 /help
            status: true,                 // 允许 /status
            restart: false,               // 禁止 /restart
            config: false                 // 禁止配置命令
          },

          // 响应速率限制
          rateLimit: {
            perUser: 10,                  // 每用户每分钟
            perGroup: 50                  // 群组每分钟
          }
        }
      }
    }
  }
}
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
          requireMention: true,            // 强制 @提及
          rateLimit: {
            perUser: 5,                     // 严格限制
            perGroup: 30
          },
          denyFrom: ["@spam-user", "@bot"] // 禁止已知滥用者
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
            restart: false,               // 禁止重启
            config: false,                 // 禁止修改配置
            security: false,               // 禁止安全命令
            admin: false                   // 禁止管理命令
          }
        }
      }
    }
  }
}
```

### 安全最佳实践

| 实践 | 说明 |
|------|------|
| **默认白名单** | 使用 `allowlist` 策略 |
| **强制 @提及** | 启用 `requireMention: true` |
| **速率限制** | 设置合理的 `rateLimit` |
| **命令限制** | 禁止敏感命令 |
| **用户过滤** | 使用 `denyFrom` 禁止问题用户 |

---

## 🎯 场景配置

### 公开群组

```json5
{
  channels: {
    telegram: {
      groupPolicy: "allowlist",
      groups: {
        "*": {
          requireMention: true,            // 需要 @提及
          allowFrom: ["*"],
          systemPrompt: "欢迎使用！请 @提及 我来获取帮助。",
          historyLimit: 50,
          rateLimit: {
            perUser: 10,
            perGroup: 100
          },
          commands: {
            help: true,
            status: true,
            config: false                  // 禁止配置命令
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
          requireMention: false,           // 不需要 @
          allowFrom: [
            "@engineer-1",
            "@engineer-2",
            "@engineer-3"
          ],
          systemPrompt: "团队内部助手。可以直接对话。",
          historyLimit: 200,
          commands: {
            restart: true,                 // 允许重启
            config: true                   // 允许配置
          }
        }
      }
    }
  }
}
```

### 技术讨论群

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      groups: {
        "123456789012345678": {
          requireMention: false,
          allowFrom: ["@developer"],
          systemPrompt: "技术讨论群。请提供代码示例和详细描述。",
          historyLimit: 100,
          rateLimit: {
            perUser: 20,
            perGroup: 200
          }
        }
      }
    }
  }
}
```

---

## 🔧 管理命令

| 命令 | 说明 |
|------|------|
| `openclaw config` | 配置群组 |
| `openclaw sessions list` | 列出群组会话 |
| `openclaw channels status` | 查看渠道状态 |
| `openclaw logs` | 查看日志 |

### 群组监控

```bash
# 列出群组会话
openclaw sessions list --channel telegram

# 查看群组状态
openclaw channels status --probe

# 统计消息
openclaw sessions stats --by-channel
```

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [群组消息](/concepts/group-messages) | 消息处理 |
| [渠道配置](/channels) | 各渠道配置 |
| [会话管理](/concepts/sessions) | 会话系统 |
| [配对与安全](/start/pairing) | 安全策略 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置群组策略 | ⭐⭐⭐⭐⭐ |
| 管理白名单 | ⭐⭐⭐⭐ |
| 设置权限控制 | ⭐⭐⭐⭐ |
| 防止群组滥用 | ⭐⭐⭐ |

---

> **💡 专家提示**：对于公开群组，始终启用 `requireMention: true` 和合理的 `rateLimit`，这是防止滥用最有效的双重保护！
