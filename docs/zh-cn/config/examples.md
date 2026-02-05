---
summary: "OpenClaw 常见使用场景的配置示例，包括基础配置、渠道配置、多渠道、多代理、路由绑定、安全配置、工具、日志、模型和消息配置"
read_when:
  - 需要参考配置示例时
  - 设置特定功能需要模板时
  - 配置新渠道或功能时
title: "配置示例"
---

# 配置示例

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解不同场景的配置需求
- [ ] 参考示例完成标准配置
- [ ] 理解配置项的作用和关系

### 进阶目标（建议掌握）

- [ ] 根据示例进行配置定制
- [ ] 组合多种配置方案
- [ ] 优化现有配置

### 专家目标（挑战）

- [ ] 设计复杂的配置架构
- [ ] 解决特殊场景需求
- [ ] 创建自定义配置模板

---

## 为什么需要配置示例？

配置示例是学习和实践的最佳起点。通过真实的配置案例，你可以：

- 快速理解配置项的实际应用
- 避免常见的配置错误
- 学习最佳实践

### 示例使用指南

```
查找需求 → 选择场景 → 参考示例 → 定制修改 → 测试验证
```

---

## 适用场景分析

### 场景分类

| 场景类型 | 复杂度 | 典型需求 |
|---------|-------|---------|
| 基础配置 | ⭐ | 最小可用配置 |
| 单一渠道 | ⭐⭐ | 单一消息渠道 |
| 多渠道 | ⭐⭐⭐ | 多个消息渠道 |
| 多代理 | ⭐⭐⭐ | 功能分离协作 |
| 企业部署 | ⭐⭐⭐⭐ | 大规模生产环境 |

---

## 基础配置示例

### 最小配置

只需要一个模型即可开始使用：

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  }
}
```

### 个人助手配置

适合个人使用的完整配置：

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514",
      userTimezone: "Asia/Shanghai"
    },
    list: [
      {
        id: "main",
        default: true,
        name: "个人助手",
        identity: {
          name: "Clawd"
        }
      }
    ]
  },
  gateway: {
    port: 18789,
    bind: "loopback"
  }
}
```

---

## 渠道配置示例

### WhatsApp 配置

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

### Telegram 配置

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456789:ABCdefGHIjklMNOpqrSTUvwxYZ",
      dmPolicy: "pairing",
      allowFrom: ["123456789"],
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

### Discord 配置

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      dm: {
        policy: "pairing"
      },
      guilds: {
        "123456789012345678": {
          allow: true,
          channels: {
            "987654321098765432": { requireMention: true }
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
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-your-bot-token",
      appToken: "xapp-your-app-token",
      dm: {
        policy: "pairing"
      }
    }
  }
}
```

### Signal 配置

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

### iMessage 配置

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/你的用户名/Library/Messages/chat.db",
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"]
    }
  }
}
```

### Matrix 配置

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

---

## 多渠道配置

同时启用多个渠道的完整配置：

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"]
    },
    telegram: {
      enabled: true,
      botToken: "123456789:ABCdefGHI...",
      dmPolicy: "pairing"
    },
    discord: {
      enabled: true,
      token: "your-bot-token",
      dm: { policy: "pairing" }
    }
  }
}
```

---

## 多代理配置

配置多个专用代理，实现功能分离：

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514"
    },
    list: [
      {
        id: "main",
        default: true,
        name: "通用助手",
        model: "anthropic/claude-opus-4-5"
      },
      {
        id: "coder",
        name: "编程助手",
        workspace: "~/code",
        model: "anthropic/claude-sonnet-4-20250514"
      },
      {
        id: "writer",
        name: "写作助手",
        workspace: "~/writing",
        model: "openai/gpt-4.5-turbo"
      }
    ]
  }
}
```

---

## 路由绑定配置

将特定渠道或用户路由到特定代理：

```json5
{
  bindings: [
    {
      match: { channel: "telegram", from: "123456789" },
      agent: "coder"
    },
    {
      match: { channel: "whatsapp", group: "*" },
      agent: "main"
    },
    {
      match: { channel: "discord", guild: "123456789012345678" },
      agent: "coder"
    }
  ]
}
```

---

## 安全配置示例

### 沙箱模式配置

隔离非主会话的执行环境：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "rw",
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

### 严格访问控制

仅允许特定用户访问：

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567", "+15559876543"],
      groupPolicy: "disabled"
    },
    telegram: {
      enabled: true,
      dmPolicy: "allowlist",
      allowFrom: ["123456789"],
      groupPolicy: "disabled"
    }
  }
}
```

---

## 远程访问配置

### Tailscale Serve

通过 Tailscale 暴露网关：

```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    tailscale: {
      mode: "serve"
    }
  }
}
```

### Tailscale Funnel

公网访问配置（需要密码认证）：

```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    tailscale: {
      mode: "funnel"
    },
    auth: {
      mode: "password",
      password: "your-secure-password"
    }
  }
}
```

---

## 工具配置示例

### 浏览器控制

```json5
{
  browser: {
    enabled: true,
    color: "#FF4500",
    profile: "~/.clawdbot/browser-profile"
  }
}
```

### 定时任务

```json5
{
  cron: {
    jobs: [
      {
        id: "morning-briefing",
        schedule: "0 8 * * *",
        agent: "main",
        message: "早上好！请给我今天的日程安排和天气预报。",
        deliverTo: "whatsapp:+15551234567"
      },
      {
        id: "daily-backup",
        schedule: "0 2 * * *",
        agent: "main",
        message: "执行每日备份任务",
        deliverTo: "telegram:123456789"
      }
    ]
  }
}
```

---

## 日志配置示例

### 详细日志（开发环境）

```json5
{
  logging: {
    level: "debug",
    format: "pretty",
    file: "~/.clawdbot/logs/openclaw.log",
    maxSize: "10m",
    maxFiles: 5
  }
}
```

### 生产环境日志

```json5
{
  logging: {
    level: "info",
    format: "json",
    file: "/var/log/openclaw/openclaw.log",
    maxSize: "100m",
    maxFiles: 10
  }
}
```

---

## 模型配置示例

### 多模型配置

```json5
{
  models: {
    default: "anthropic/claude-sonnet-4-20250514",
    fallback: ["openai/gpt-4.5-turbo", "anthropic/claude-3-5-haiku"],
    routing: {
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
  models: {
    endpoints: {
      "local-llm": {
        baseUrl: "http://localhost:11434/v1",
        model: "llama3:70b"
      }
    }
  }
}
```

---

## 消息配置示例

### 群组消息配置

```json5
{
  messages: {
    groupChat: {
      mentionPatterns: ["@clawd", "小助手", "openclaw"],
      historyLimit: 50
    },
    responsePrefix: "",
    chunkLimit: 4000
  }
}
```

### 自定义响应消息

```json5
{
  messages: {
    welcome: "你好！我是你的 AI 助手，有什么可以帮助你的？",
    error: "抱歉，处理您的请求时出现了问题。",
    pairi
```

我将继续完成这个被截断的配置示例，并补充完整的内容。这段配置展示了 AI 助手的消息模板和错误处理设置，定义了欢迎消息、错误提示和配对流程的定制化文本。这些配置帮助个性化 AI 助手的交互体验，确保在不同场景下能给出友好、准确的响应。 ng: "请输入配对码以开始使用：{code}"
  }
}
```

---

## 完整配置示例

### 家庭使用配置

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514",
      userTimezone: "Asia/Shanghai"
    },
    list: [
      {
        id: "main",
        default: true,
        name: "家庭助手",
        identity: {
          name: "小助手"
        }
      }
    ]
  },
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567", "+15559876543"],
      groupPolicy: "allowlist",
      groups: {
        "family-group-id": { requireMention: true }
      }
    }
  },
  gateway: {
    port: 18789,
    bind: "loopback"
  },
  logging: {
    level: "info"
  }
}
```

### 开发团队配置

```json5
{
  agents: {
    defaults: {
      workspace: "~/projects",
      model: "anthropic/claude-opus-4-5"
    },
    list: [
      {
        id: "dev",
        default: true,
        name: "开发助手"
      }
    ]
  },
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-your-token",
      appToken: "xapp-your-token",
      dm: { policy: "allowlist", allowFrom: ["U123456789"] },
      channels: {
        "C123456789": { allow: true, requireMention: true }
      }
    },
    discord: {
      enabled: true,
      token: "your-bot-token",
      guilds: {
        "123456789012345678": {
          allow: true,
          channels: {
            "987654321098765432": { requireMention: true }
          }
        }
      }
    }
  },
  browser: {
    enabled: true
  },
  gateway: {
    port: 18789,
    bind: "loopback",
    tailscale: {
      mode: "serve"
    }
  }
}
```

---

## 专家思维模型：配置优化框架

### 配置检查清单

```
配置前检查：
  □ 是否需要该功能
  □ 配置项的有效值范围
  □ 依赖的配置项
  □ 安全影响

配置时检查：
  □ 语法是否正确
  □ 路径是否正确
  □ 敏感信息是否保护

配置后检查：
  □ 功能是否正常
  □ 日志是否有错误
  □ 性能是否受影响
```

### 常见配置错误

| 错误类型 | 示例 | 解决方案 |
|---------|------|---------|
| 语法错误 | 缺少逗号、引号 | 使用验证工具 |
| 路径错误 | 错误的文件路径 | 检查路径是否存在 |
| 权限问题 | 文件不可读 | 检查文件权限 |
| 依赖缺失 | 使用未定义的配置 | 检查依赖配置 |

---

## 故障排查指南

### 配置不生效

**排查步骤**：

1. 检查配置文件路径
2. 验证 JSON 语法
3. 检查配置合并
4. 查看日志输出

```bash
# 验证配置
openclaw doctor

# 查看当前配置
openclaw config get

# 检查日志
openclaw logs --verbose
```

### 功能异常

**排查步骤**：

1. 检查相关配置项
2. 检查依赖配置
3. 检查权限设置
4. 测试功能验证

---

## 最佳实践

### 配置组织

- 按功能模块拆分配置
- 使用包含文件管理复杂配置
- 保留配置版本历史

### 安全保护

- 使用环境变量存储敏感信息
- 限制配置文件的访问权限
- 定期轮换 API Key

### 性能优化

- 减少不必要的配置嵌套
- 优化日志级别
- 合理设置缓存策略

---

## 总结

配置示例是学习和实践的重要参考。通过本教程，你可以：

- 快速完成标准配置
- 学习最佳实践
- 避免常见错误
- 优化现有配置

建议从最小配置开始，逐步添加所需功能，并参考示例进行定制。

---

## 进阶学习路径

| 级别 | 主题 | 资源 |
|-----|------|-----|
| ⭐ | 基础配置 | [配置概述](/zh-CN/config/index) |
| ⭐⭐ | 配置参考 | [配置参考](/zh-CN/config/reference) |
| ⭐⭐⭐ | 高级配置 | [安全配置](/zh-CN/config/security) |
| ⭐⭐⭐⭐ | 企业部署 | [运维指南](/zh-CN/operators) |

---

## 相关文档

- [配置概述](/zh-CN/config/index) - 配置系统介绍
- [配置参考](/zh-CN/config/reference) - 完整配置选项
- [CLI 参考](/zh-CN/cli) - 命令行工具
- [故障排除](/zh-CN/help/troubleshooting) - 问题排查

---

**丰富的配置示例帮助你快速配置 OpenClaw！** 🦞