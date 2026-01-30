# 配置参考

本文档提供 Moltbot 配置的完整参考。

## 配置结构

```json5
{
  // 代理配置
  agents: { /* ... */ },
  
  // 路由绑定
  bindings: [ /* ... */ ],
  
  // 渠道配置
  channels: { /* ... */ },
  
  // 网关配置
  gateway: { /* ... */ },
  
  // 消息配置
  messages: { /* ... */ },
  
  // 日志配置
  logging: { /* ... */ },
  
  // 工具配置
  tools: { /* ... */ },
  
  // 模型配置
  models: { /* ... */ },
  
  // 认证配置
  auth: { /* ... */ },
  
  // 环境变量
  env: { /* ... */ },
  
  // 命令配置
  commands: { /* ... */ },
  
  // 会话配置
  session: { /* ... */ }
}
```

## agents - 代理配置

### agents.defaults

全局代理默认值：

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
          image: "moltbot-sandbox:latest",
          cpuLimit: "2",
          memoryLimit: "2g"
        }
      }
    }
  }
}
```

### agents.list

代理列表：

```json5
{
  agents: {
    list: [
      {
        id: "main",               // 唯一标识（必需）
        default: true,            // 是否为默认代理
        name: "Main Agent",       // 显示名称
        workspace: "~/clawd",     // 工作区（覆盖 defaults）
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

## bindings - 路由绑定

```json5
{
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "whatsapp",       // 必需
        accountId: "default",      // 可选
        peer: {                    // 可选
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

## logging - 日志配置

```json5
{
  logging: {
    level: "info",                    // debug | info | warn | error
    file: "/tmp/moltbot/moltbot.log",
    consoleLevel: "info",
    consoleStyle: "pretty",           // pretty | compact | json
    redactSensitive: "tools",         // off | tools
    redactPatterns: [
      "\\bsk-[A-Za-z0-9_-]{8,}\\b"
    ]
  }
}
```

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

## commands - 命令配置

```json5
{
  commands: {
    native: "auto",       // auto | true | false
    text: true,           // 解析斜杠命令
    bash: false,          // 允许 !command
    bashForegroundMs: 2000,
    config: false,        // 允许 /config
    debug: false,         // 允许 /debug
    restart: false,       // 允许 /restart
    useAccessGroups: true
  }
}
```

## session - 会话配置

```json5
{
  session: {
    mainKey: "main"       // DM 会话合并键
  }
}
```

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

规则：
- 仅匹配大写变量名
- 缺失变量会报错
- 使用 `$${VAR}` 转义

## 下一步

- [配置示例](/zh-cn/config/examples)
- [配置概述](/zh-cn/config/index)
- [快速入门](/zh-cn/start/quick-start)
