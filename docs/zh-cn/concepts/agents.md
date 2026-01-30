# AI 代理系统

本文档详细介绍 Moltbot 的 AI 代理（Agent）系统，包括配置、工作流程和高级特性。

## 代理概述

代理是 Moltbot 的核心组件，负责：

- 接收和处理用户消息
- 调用 AI 模型生成回复
- 执行工具调用（文件操作、浏览器、命令等）
- 管理会话上下文和历史

## 代理架构

```
入站消息
    │
    ▼
┌─────────────┐
│  路由引擎   │ ─→ 选择代理
└─────────────┘
    │
    ▼
┌─────────────┐
│  会话管理   │ ─→ 加载上下文
└─────────────┘
    │
    ▼
┌─────────────┐
│  AI 模型    │ ─→ 生成响应
└─────────────┘
    │
    ├─→ 工具调用 ─→ 执行 ─→ 继续
    │
    ▼
┌─────────────┐
│  消息发送   │ ─→ 回复用户
└─────────────┘
```

## 基础配置

### 默认代理

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### 代理列表

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        workspace: "~/clawd",
        identity: {
          name: "Clawd",
          emoji: "🦞",
          theme: "helpful assistant"
        }
      }
    ]
  }
}
```

## 代理身份

### 配置身份

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",       // 名称
          emoji: "🦥",            // 表情
          theme: "helpful sloth", // 主题
          avatar: "avatars/samantha.png"  // 头像
        }
      }
    ]
  }
}
```

### 身份用途

身份信息用于：

1. **提及触发**: `identity.name` 自动添加到群组触发词
2. **确认反应**: `identity.emoji` 用作消息确认表情
3. **响应前缀**: 当设置为 `"auto"` 时，使用 `identity.name`

## 工作区配置

### 基本工作区

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd"
    }
  }
}
```

### 多代理工作区

```json5
{
  agents: {
    list: [
      { id: "personal", workspace: "~/clawd-personal" },
      { id: "work", workspace: "~/clawd-work" }
    ]
  }
}
```

### 工作区文件

代理会读取工作区中的以下文件：

| 文件 | 用途 |
|------|------|
| `AGENTS.md` | 代理行为指南 |
| `SOUL.md` | 代理个性定义 |
| `USER.md` | 用户信息 |
| `MEMORY.md` | 长期记忆 |
| `TOOLS.md` | 工具使用说明 |

## 模型配置

### 设置默认模型

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### 模型格式

```
<provider>/<model-name>
```

示例：
- `anthropic/claude-sonnet-4-20250514`
- `openai/gpt-4o`
- `openrouter/anthropic/claude-3.5-sonnet`

### 模型回退

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: [
          "openai/gpt-4o",
          "anthropic/claude-haiku-3-5-20241022"
        ]
      }
    }
  }
}
```

### 按代理设置模型

```json5
{
  agents: {
    list: [
      {
        id: "premium",
        model: "anthropic/claude-opus-4-20250514"
      },
      {
        id: "fast",
        model: "anthropic/claude-haiku-3-5-20241022"
      }
    ]
  }
}
```

## 沙箱模式

### 沙箱概述

沙箱将代理的操作隔离在 Docker 容器中，提高安全性。

### 沙箱模式

| 模式 | 说明 |
|------|------|
| `off` | 无沙箱，完全访问主机 |
| `non-main` | 非主会话使用沙箱 |
| `all` | 所有会话使用沙箱 |

### 配置沙箱

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",        // session | agent | shared
        workspaceAccess: "rw",   // none | ro | rw
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

### 工作区访问

| 级别 | 说明 |
|------|------|
| `none` | 无文件系统访问 |
| `ro` | 只读访问 |
| `rw` | 读写访问 |

## 工具配置

### 工具限制

```json5
{
  agents: {
    list: [
      {
        id: "restricted",
        tools: {
          allow: ["read", "sessions_list", "sessions_history"],
          deny: ["write", "exec", "browser"]
        }
      }
    ]
  }
}
```

### 高级工具

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"]
      }
    }
  }
}
```

## 多代理路由

### 基本路由

```json5
{
  agents: {
    list: [
      { id: "personal", default: true },
      { id: "work" }
    ]
  },
  bindings: [
    { agentId: "personal", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "work" } }
  ]
}
```

### 路由匹配

匹配条件优先级：

1. `match.peer` - 特定聊天
2. `match.guildId` - Discord 服务器
3. `match.teamId` - Teams 团队
4. `match.accountId` - 特定账号
5. `match.accountId: "*"` - 任意账号
6. 默认代理

### 按聊天路由

```json5
{
  bindings: [
    {
      agentId: "support",
      match: {
        channel: "telegram",
        peer: { kind: "group", id: "-1001234567890" }
      }
    }
  ]
}
```

## 子代理

### 允许子代理

```json5
{
  agents: {
    list: [
      {
        id: "main",
        subagents: {
          allowAgents: ["helper", "researcher"]  // 或 ["*"] 允许所有
        }
      }
    ]
  }
}
```

### 代理间通信

```json5
{
  tools: {
    agentToAgent: {
      enabled: true,
      allow: ["main", "helper"]
    }
  }
}
```

## 群组聊天配置

### 提及触发

```json5
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@clawd", "小助手", "机器人"]
        }
      }
    ]
  }
}
```

### 群组系统提示

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          systemPrompt: "保持回复简短，使用中文。"
        }
      }
    }
  }
}
```

## 会话配置

### 会话键

会话键格式：`agent:<agentId>:<channel>:<type>:<id>`

示例：
- `agent:main:whatsapp:dm:+15555550123`
- `agent:main:telegram:group:-1001234567890`

### 主会话

默认情况下，同一用户的多个 DM 会话合并到 `main`：

```json5
{
  session: {
    mainKey: "main"
  }
}
```

### 历史限制

```json5
{
  channels: {
    telegram: {
      historyLimit: 50,        // 群组历史
      dmHistoryLimit: 30       // DM 历史
    }
  }
}
```

## 代理生命周期

### 初始化

1. 加载配置
2. 初始化工作区
3. 注册工具
4. 加载 OAuth/API 密钥

### 消息处理

1. 路由到代理
2. 加载会话
3. 准备上下文
4. 调用 AI 模型
5. 执行工具（如有）
6. 发送响应

### 清理

- 会话自动保存
- 沙箱容器定期清理

## 监控与调试

### 查看代理状态

```bash
moltbot agents list
moltbot agents status <agentId>
```

### 查看会话

```bash
moltbot sessions list
moltbot sessions history <sessionKey>
```

### 调试模式

```bash
moltbot gateway --verbose
```

## 最佳实践

### 个人使用

单代理，无沙箱：

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      sandbox: { mode: "off" }
    }
  }
}
```

### 团队使用

多代理，沙箱隔离：

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    list: [
      { id: "team", workspace: "~/clawd-team" },
      { id: "admin", sandbox: { mode: "off" } }
    ]
  }
}
```

### 公开服务

严格限制：

```json5
{
  agents: {
    list: [
      {
        id: "public",
        sandbox: {
          mode: "all",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history"],
          deny: ["write", "exec", "browser"]
        }
      }
    ]
  }
}
```

## 下一步

- [消息路由](/zh-cn/concepts/routing) - 路由配置详解
- [会话管理](/zh-cn/concepts/sessions) - 会话系统详解
- [配置参考](/zh-cn/config/reference) - 完整配置选项
