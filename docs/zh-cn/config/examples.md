# 配置示例

本文档提供常见使用场景的配置示例，帮助您快速上手 OpenClaw。

## 基础配置

### 最小配置

只需要一个模型即可开始：

```json5
{
  "agent": {
    "model": "anthropic/claude-opus-4-5"
  }
}
```

### 个人助手配置

适合个人使用的完整配置：

```json5
{
  "agents": {
    "defaults": {
      "workspace": "~/clawd",
      "model": "anthropic/claude-sonnet-4-20250514",
      "userTimezone": "Asia/Shanghai"
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "个人助手",
        "identity": {
          "name": "Clawd"
        }
      }
    ]
  },
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  }
}
```

## 渠道配置示例

### WhatsApp 配置

```json5
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"],
      "groupPolicy": "allowlist",
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

### Telegram 配置

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456789:ABCdefGHIjklMNOpqrSTUvwxYZ",
      "dmPolicy": "pairing",
      "allowFrom": ["123456789"],
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

### Discord 配置

```json5
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "your-bot-token",
      "dm": {
        "policy": "pairing"
      },
      "guilds": {
        "123456789012345678": {
          "allow": true,
          "channels": {
            "987654321098765432": { "requireMention": true }
          }
        }
      }
    }
  }
}
```

### Slack 配置

```json5
{
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "xoxb-your-bot-token",
      "appToken": "xapp-your-app-token",
      "dm": {
        "policy": "pairing"
      }
    }
  }
}
```

### Signal 配置

```json5
{
  "channels": {
    "signal": {
      "enabled": true,
      "account": "+15551234567",
      "cliPath": "signal-cli",
      "dmPolicy": "pairing",
      "allowFrom": ["+15557654321"]
    }
  }
}
```

### iMessage 配置

```json5
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "/usr/local/bin/imsg",
      "dbPath": "/Users/你的用户名/Library/Messages/chat.db",
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"]
    }
  }
}
```

### Matrix 配置

```json5
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.example.org",
      "accessToken": "syt_***",
      "encryption": true,
      "dm": { "policy": "pairing" }
    }
  }
}
```

## 多渠道配置

同时启用多个渠道：

```json5
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "enabled": true,
      "botToken": "123456789:ABCdefGHI...",
      "dmPolicy": "pairing"
    },
    "discord": {
      "enabled": true,
      "token": "your-bot-token",
      "dm": { "policy": "pairing" }
    }
  }
}
```

## 多代理配置

配置多个专用代理：

```json5
{
  "agents": {
    "defaults": {
      "workspace": "~/clawd",
      "model": "anthropic/claude-sonnet-4-20250514"
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "通用助手",
        "model": "anthropic/claude-opus-4-5"
      },
      {
        "id": "coder",
        "name": "编程助手",
        "workspace": "~/code",
        "model": "anthropic/claude-sonnet-4-20250514"
      },
      {
        "id": "writer",
        "name": "写作助手",
        "workspace": "~/writing",
        "model": "openai/gpt-4.5-turbo"
      }
    ]
  }
}
```

## 路由绑定

将特定渠道/用户路由到特定代理：

```json5
{
  "bindings": [
    {
      "match": { "channel": "telegram", "from": "123456789" },
      "agent": "coder"
    },
    {
      "match": { "channel": "whatsapp", "group": "*" },
      "agent": "main"
    },
    {
      "match": { "channel": "discord", "guild": "123456789012345678" },
      "agent": "coder"
    }
  ]
}
```

## 安全配置

### 沙箱模式

隔离非主会话的执行环境：

```json5
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "session",
        "workspaceAccess": "rw",
        "docker": {
          "image": "openclaw-sandbox:latest",
          "cpuLimit": "2",
          "memoryLimit": "2g"
        }
      }
    }
  }
}
```

### 严格访问控制

仅允许特定用户：

```json5
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567", "+15559876543"],
      "groupPolicy": "disabled"
    },
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["123456789"],
      "groupPolicy": "disabled"
    }
  }
}
```

## 远程访问

### Tailscale Serve

通过 Tailscale 暴露网关：

```json5
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "serve"
    }
  }
}
```

### Tailscale Funnel（公网访问）

```json5
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "funnel"
    },
    "auth": {
      "mode": "password",
      "password": "your-secure-password"
    }
  }
}
```

## 工具配置

### 浏览器控制

```json5
{
  "browser": {
    "enabled": true,
    "color": "#FF4500",
    "profile": "~/.clawdbot/browser-profile"
  }
}
```

### 定时任务

```json5
{
  "cron": {
    "jobs": [
      {
        "id": "morning-briefing",
        "schedule": "0 8 * * *",
        "agent": "main",
        "message": "早上好！请给我今天的日程安排和天气预报。",
        "deliverTo": "whatsapp:+15551234567"
      },
      {
        "id": "daily-backup",
        "schedule": "0 2 * * *",
        "agent": "main",
        "message": "执行每日备份任务",
        "deliverTo": "telegram:123456789"
      }
    ]
  }
}
```

## 日志配置

### 详细日志

```json5
{
  "logging": {
    "level": "debug",
    "format": "pretty",
    "file": "~/.clawdbot/logs/openclaw.log",
    "maxSize": "10m",
    "maxFiles": 5
  }
}
```

### 生产环境日志

```json5
{
  "logging": {
    "level": "info",
    "format": "json",
    "file": "/var/log/openclaw/openclaw.log",
    "maxSize": "100m",
    "maxFiles": 10
  }
}
```

## 模型配置

### 多模型配置

```json5
{
  "models": {
    "default": "anthropic/claude-sonnet-4-20250514",
    "fallback": ["openai/gpt-4.5-turbo", "anthropic/claude-3-5-haiku"],
    "routing": {
      "code": "anthropic/claude-opus-4-5",
      "chat": "anthropic/claude-sonnet-4-20250514",
      "quick": "anthropic/claude-3-5-haiku"
    }
  }
}
```

### 自定义端点

```json5
{
  "models": {
    "endpoints": {
      "local-llm": {
        "baseUrl": "http://localhost:11434/v1",
        "model": "llama3:70b"
      }
    }
  }
}
```

## 消息配置

### 群组消息

```json5
{
  "messages": {
    "groupChat": {
      "mentionPatterns": ["@clawd", "小助手", "openclaw"],
      "historyLimit": 50
    },
    "responsePrefix": "",
    "chunkLimit": 4000
  }
}
```

### 自定义响应

```json5
{
  "messages": {
    "welcome": "你好！我是你的 AI 助手，有什么可以帮助你的？",
    "error": "抱歉，处理您的请求时出现了问题。",
    "pairing": "请输入配对码以开始使用：{code}"
  }
}
```

## 完整示例

### 家庭使用配置

```json5
{
  "agents": {
    "defaults": {
      "workspace": "~/clawd",
      "model": "anthropic/claude-sonnet-4-20250514",
      "userTimezone": "Asia/Shanghai"
    },
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "家庭助手",
        "identity": {
          "name": "小助手"
        }
      }
    ]
  },
  "channels": {
    "whatsapp": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567", "+15559876543"],
      "groupPolicy": "allowlist",
      "groups": {
        "family-group-id": { "requireMention": true }
      }
    }
  },
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  },
  "logging": {
    "level": "info"
  }
}
```

### 开发团队配置

```json5
{
  "agents": {
    "defaults": {
      "workspace": "~/projects",
      "model": "anthropic/claude-opus-4-5"
    },
    "list": [
      {
        "id": "dev",
        "default": true,
        "name": "开发助手"
      }
    ]
  },
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "xoxb-your-token",
      "appToken": "xapp-your-token",
      "dm": { "policy": "allowlist", "allowFrom": ["U123456789"] },
      "channels": {
        "C123456789": { "allow": true, "requireMention": true }
      }
    },
    "discord": {
      "enabled": true,
      "token": "your-bot-token",
      "guilds": {
        "123456789012345678": {
          "allow": true,
          "channels": {
            "987654321098765432": { "requireMention": true }
          }
        }
      }
    }
  },
  "browser": {
    "enabled": true
  },
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "serve"
    }
  }
}
```

## 相关文档

- [配置概述](/zh-CN/config)
- [配置参考](/zh-CN/config/reference)
- [渠道概述](/zh-CN/channels)
- [网关配置](/zh-CN/concepts/gateway)
