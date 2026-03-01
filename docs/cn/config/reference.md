---
summary: "OpenClaw 配置参考 - 完整配置选项包括代理、路由绑定、渠道、网关、消息、日志、工具、模型、认证等"
read_when:
  - 需要查找特定配置项时
  - 配置出现问题需要调试时
  - 了解完整配置结构时
title: "配置参考"
---

# 配置参考

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解配置文件的整体结构
- [ ] 掌握核心配置块的作用
- [ ] 查找和理解特定配置项
- [ ] 完成标准配置

### 进阶目标（建议掌握）

- [ ] 配置多渠道和多代理
- [ ] 优化消息处理和路由
- [ ] 配置工具和权限

### 专家目标（挑战）

- [ ] 设计复杂的配置架构
- [ ] 自定义配置验证
- [ ] 优化大规模部署配置

---

## 为什么需要配置参考？

本参考文档提供 OpenClaw 所有配置项的详细说明。在实际配置中，你会经常需要查阅特定配置项的作用和有效值。本文档帮助你快速定位和理解配置项。

### 配置结构概览

```
OpenClaw 配置
    │
    ├── agents          → AI 代理配置
    ├── bindings        → 消息路由绑定
    ├── channels        → 消息渠道配置
    ├── gateway        → 网关服务配置
    ├── messages       → 消息处理配置
    ├── logging        → 日志配置
    ├── tools          → 工具配置
    ├── models         → 模型提供商配置
    ├── auth           → 认证配置
    ├── env            → 环境变量配置
    ├── commands       → 聊天命令配置
    └── session        → 会话配置
```

---

## 核心概念速查

| 配置块 | 必需性 | 主要作用 | 复杂度 |
|-------|-------|---------|-------|
| agents | ✅ | 定义 AI 代理行为 | ⭐⭐ |
| bindings | ⭐ | 消息路由规则 | ⭐⭐ |
| channels | ✅ | 渠道连接配置 | ⭐⭐⭐ |
| gateway | ⭐ | 网关服务设置 | ⭐ |
| messages | ⭐ | 消息处理规则 | ⭐⭐ |
| logging | ⭐ | 日志输出配置 | ⭐ |
| tools | ⭐ | 工具权限配置 | ⭐⭐ |
| models | ⭐ | 模型 API 配置 | ⭐⭐ |
| auth | ⭐ | 认证策略配置 | ⭐⭐ |
| env | ⭐ | 环境变量注入 | ⭐ |
| commands | ⭐ | 聊天命令配置 | ⭐ |
| session | ⭐ | 会话行为配置 | ⭐ |

---

## 适用场景分析

### 场景一：单一代理单渠道

**需求**：简单的个人 AI 助手

**必需配置**：

```
agents + channels（至少一个渠道）
```

### 场景二：多代理多渠道

**需求**：团队协作或功能分离

**必需配置**：

```
agents（多个代理）+ bindings（路由规则）+ channels（多个渠道）
```

### 场景三：企业级部署

**需求**：复杂环境的高级配置

**建议配置**：

```
所有配置块（按需）
```

---

## 专家思维模型：配置查找框架

当需要查找特定配置时，专家会采用以下思维框架：

### 配置查找流程

```
需要配置 → 确定功能域 → 找到对应配置块 → 查找具体配置项 → 配置生效
```

### 功能域对应关系

| 功能需求 | 对应配置块 |
|---------|-----------|
| AI 行为设置 | agents |
| 消息路由 | bindings |
| 渠道连接 | channels |
| 服务端口 | gateway |
| 响应格式 | messages |
| 日志输出 | logging |
| 工具权限 | tools |
| API 配置 | models |
| 认证方式 | auth |
| 敏感信息 | env |
| 聊天命令 | commands |

---

## agents - 代理配置

### agents.defaults

全局代理默认值，应用于所有未单独配置的代理：

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",           // 工作区目录
      model: "anthropic/claude-sonnet-4-20250514",  // 默认模型
      repoRoot: "~/Projects/repo",    // 仓库根目录（可选）
      skipBootstrap: false,           // 跳过工作区初始化
      bootstrapMaxChars: 20000,       // 工作区文件最大字符
      userTimezone: "Asia/Shanghai",  // 用户时区
      timeFormat: "auto",             // 时间格式 (auto | 12 | 24)
      
      // 沙箱配置
      sandbox: {
        mode: "non-main",        // off | non-main | all
        scope: "session",        // session | agent | shared
        workspaceAccess: "rw",   // none | ro | rw
        workspaceRoot: "~/.clawdbot/sandboxes",
        docker: {
          image: "openclaw-sandbox:latest",
          cpuLimit: "2",
          memoryLimit: "2g"
        }
      }
    }
  }
}
```

### agents.list

代理列表，定义具体的代理实例：

```json5
{
  agents: {
    list: [
      {
        id: "main",               // 唯一标识（必需）
        default: true,            // 是否为默认代理
        name: "Main Agent",       // 显示名称
        workspace: "~/clawd",    // 工作区（覆盖 defaults）
        agentDir: "~/.clawdbot/agents/main/agent",
        model: "anthropic/claude-opus-4-20250514",
        
        // 身份配置
        identity: {
          name: "Clawd",
          emoji: "🦞",
          theme: "helpful assistant",
          avatar: "avatars/clawd.png"
        },
        
        // 群组聊天配置
        groupChat: {
          mentionPatterns: ["@clawd", "小助手"]
        },
        
        // 沙箱覆盖
        sandbox: {
          mode: "off"
        },
        
        // 工具限制
        tools: {
          allow: ["read", "write", "exec"],
          deny: ["browser"]
        },
        
        // 子代理配置
        subagents: {
          allowAgents: ["helper"]
        }
      }
    ]
  }
}
```

### 沙箱模式说明

| 模式 | 描述 | 适用场景 |
|-----|------|---------|
| off | 不使用沙箱 | 本地开发、个人使用 |
| non-main | 仅非主会话使用沙箱 | 群组隔离、团队使用 |
| all | 所有会话使用沙箱 | 高安全要求环境 |

---

## bindings - 路由绑定

配置消息路由规则，将不同来源的消息路由到不同的代理：

```json5
{
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "whatsapp",       // 必需：渠道类型
        accountId: "default",      // 可选：账户标识
        peer: {                    // 可选：消息来源
          kind: "dm",              // dm | group | channel
          id: "+15555550123"
        },
        guildId: "123456",         // Discord 服务器
        teamId: "team-uuid"        // Teams 团队
      }
    }
  ]
}
```

### 匹配规则优先级

1. 精确匹配（最高）
2. 通配符匹配
3. 默认规则（最低）

---

## channels - 渠道配置

### 通用选项

```json5
{
  channels: {
    defaults: {
      groupPolicy: "allowlist"   // allowlist | open | disabled
    }
  }
}
```

### channels.whatsapp

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      },
      accounts: {
        default: {},
        business: {}
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 50,
      sendReadReceipts: true
    }
  }
}
```

### channels.telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true }
      },
      customCommands: [
        { command: "help", description: "帮助" }
      ],
      historyLimit: 50,
      replyToMode: "first",
      linkPreview: true,
      streamMode: "partial",
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own",
      mediaMaxMb: 5
    }
  }
}
```

### channels.discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "${DISCORD_BOT_TOKEN}",
      mediaMaxMb: 8,
      allowBots: false,
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["123456789"],
        groupEnabled: false
      },
      guilds: {
        "123456789012345678": {
          slug: "my-server",
          requireMention: false,
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true }
          }
        }
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      replyToMode: "off",
      actions: {
        reactions: true,
        messages: true
      }
    }
  }
}
```

### channels.slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "${SLACK_BOT_TOKEN}",
      appToken: "${SLACK_APP_TOKEN}",
      dm: {
        enabled: true,
        policy: "pairing"
      },
      channels: {
        "#general": { allow: true, requireMention: true }
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      thread: {
        historyScope: "thread",
        inheritParent: false
      }
    }
  }
}
```

---

## gateway - 网关配置

```json5
{
  gateway: {
    bind: "loopback",      // loopback | tailnet | lan | <ip>
    port: 18789,
    auth: {
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    },
    canvasHost: {
      enabled: true,
      port: 18793
    }
  }
}
```

### bind 选项说明

| 值 | 描述 | 适用场景 |
|---|------|---------|
| loopback | 仅本地访问 | 本地开发、个人使用 |
| tailnet | Tailscale 网络 | 远程访问、安全网络 |
| lan | 本地网络 | 局域网部署 |
| <ip> | 特定 IP | 定制部署 |

---

## messages - 消息配置

```json5
{
  messages: {
    responsePrefix: "🦞",      // 或 "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
    
    groupChat: {
      historyLimit: 50
    },
    
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize"
    },
    
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000
      }
    },
    
    tts: {
      enabled: false,
      provider: "elevenlabs",
      voiceId: "..."
    }
  }
}
```

---

## logging - 日志配置

```json5
{
  logging: {
    level: "info",                    // debug | info | warn | error
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",           // pretty | compact | json
    redactSensitive: "tools",         // off | tools
    redactPatterns: [
      "\\bsk-[A-Za-z0-9_-]{8,}\\b"    // 敏感信息脱敏
    ]
  }
}
```

### 日志级别说明

| 级别 | 描述 | 适用场景 |
|-----|------|---------|
| debug | 详细调试信息 | 开发调试 |
| info | 一般信息 | 日常监控 |
| warn | 警告信息 | 问题排查 |
| error | 错误信息 | 故障处理 |

---

## tools - 工具配置

```json5
{
  tools: {
    elevated: {
      enabled: false,
      allowFrom: {
        whatsapp: ["+15555550123"]
      }
    },
    agentToAgent: {
      enabled: false,
      allow: ["main", "helper"]
    },
    web: {
      search: {
        apiKey: "${BRAVE_SEARCH_API_KEY}"
      }
    }
  }
}
```

---

## models - 模型配置

```json5
{
  models: {
    providers: {
      openai: {
        apiKey: "${OPENAI_API_KEY}"
      },
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"
      },
      openrouter: {
        apiKey: "${OPENROUTER_API_KEY}"
      },
      custom: {
        baseUrl: "https://api.custom.com/v1",
        apiKey: "${CUSTOM_API_KEY}"
      }
    }
  }
}
```

---

## auth - 认证配置

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": {
        provider: "anthropic",
        mode: "oauth",
        email: "me@example.com"
      }
    },
    order: {
      anthropic: ["anthropic:me@example.com"]
    }
  }
}
```

---

## env - 环境变量

```json5
{
  env: {
    vars: {
      CUSTOM_VAR: "value"
    },
    shellEnv: {
      enabled: false,
      timeoutMs: 15000
    }
  }
}
```

---

## commands - 命令配置

```json5
{
  commands: {
    native: "auto",       // auto | true | false
    text: true,           // 解析斜杠命令
    bash: false,          // 允许 !command
    bashForegroundMs: 2000,
    config: false,        // 允许 /config
    debug: false,        // 允许 /debug
    restart: false,      // 允许 /restart
    useAccessGroups: true
  }
}
```

---

## session - 会话配置

```json5
{
  session: {
    mainKey: "main"       // DM 会话合并键
  }
}
```

---

## 配置包含

```json5
{
  agents: { "$include": "./agents.json5" },
  channels: {
    "$include": [
      "./channels/whatsapp.json5",
      "./channels/telegram.json5"
    ]
  }
}
```

---

## 环境变量替换

```json5
{
  gateway: {
    auth: {
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

### 替换规则

| 语法 | 行为 |
|-----|------|
| `${VAR}` | 替换为变量值 |
| `${VAR:-default}` | 变量未设置时使用默认值 |
| `${VAR?error}` | 变量未设置时报错 |
| `$${VAR}` | 转义，保留为 `${VAR}` |

---

## 总结

本参考文档提供了 OpenClaw 所有配置项的详细说明。在实际使用中，建议：

1. 从最小配置开始，逐步添加
2. 使用模块化配置管理复杂设置
3. 善用环境变量保护敏感信息
4. 定期检查配置是否需要优化

---

## 进阶学习路径

| 级别 | 主题 | 资源 |
|-----|------|-----|
| ⭐ | 基础配置 | [配置概述](/zh-CN/config/index) |
| ⭐⭐ | 配置示例 | [配置示例](/zh-CN/config/examples) |
| ⭐⭐⭐ | 高级配置 | [安全配置](/zh-CN/config/security) |
| ⭐⭐⭐⭐ | 企业部署 | [运维指南](/zh-CN/operators) |

---

## 相关文档

- [配置概述](/zh-CN/config/index) - 配置系统介绍
- [配置示例](/zh-CN/config/examples) - 完整配置示例
- [CLI 参考](/zh-CN/cli) - 命令行工具
- [故障排除](/zh-CN/help/troubleshooting) - 问题排查

---

**完整的配置参考帮助你精准控制 OpenClaw 的每一个细节！** 🦞