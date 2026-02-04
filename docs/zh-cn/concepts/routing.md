---
summary: 介绍 OpenClaw 的消息路由系统，包括路由概述、路由流程、绑定配置、匹配优先级、匹配条件、默认代理、多代理路由示例和路由调试
read_when:
  - 需要配置消息路由时
  - 设置多代理路由时
  - 调试路由问题时
title: 消息路由
---

# 消息路由

本文档介绍 OpenClaw 的消息路由系统，包括如何将入站消息路由到正确的 AI 代理。

## 路由概述

当收到消息时，OpenClaw 需要决定：

1. 哪个代理处理这条消息？
2. 使用哪个会话上下文？
3. 应用什么配置？

## 路由流程

```
入站消息
    │
    ▼
┌─────────────┐
│  渠道识别   │ ─→ whatsapp / telegram / discord / ...
└─────────────┘
    │
    ▼
┌─────────────┐
│  账号匹配   │ ─→ default / personal / work / ...
└─────────────┘
    │
    ▼
┌─────────────┐
│  绑定匹配   │ ─→ bindings[]
└─────────────┘
    │
    ▼
┌─────────────┐
│  代理选择   │ ─→ agents.list[]
└─────────────┘
    │
    ▼
┌─────────────┐
│  会话加载   │ ─→ session key
└─────────────┘
```

## 绑定配置

### 基本绑定

```json5
{
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "whatsapp"
      }
    }
  ]
}
```

### 完整绑定示例

```json5
{
  bindings: [
    // 按聊天路由
    {
      agentId: "support",
      match: {
        channel: "telegram",
        peer: { kind: "group", id: "-1001234567890" }
      }
    },
    // 按账号路由
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "business"
      }
    },
    // 按 Discord 服务器路由
    {
      agentId: "gaming",
      match: {
        channel: "discord",
        guildId: "123456789012345678"
      }
    },
    // 通配符账号
    {
      agentId: "default",
      match: {
        channel: "telegram",
        accountId: "*"
      }
    }
  ]
}
```

## 匹配优先级

绑定按以下优先级匹配（从高到低）：

| 优先级 | 匹配条件 | 说明 |
|--------|----------|------|
| 1 | `match.peer` | 特定聊天（DM/群组） |
| 2 | `match.guildId` | Discord 服务器 |
| 3 | `match.teamId` | Teams 团队 |
| 4 | `match.accountId` (精确) | 特定账号 |
| 5 | `match.accountId: "*"` | 任意账号 |
| 6 | 默认代理 | 无匹配时使用 |

### 确定性匹配

同一优先级内，使用 `bindings` 数组中的第一个匹配项。

## 匹配条件

### channel（必需）

指定渠道：

```json5
{
  match: {
    channel: "whatsapp"  // whatsapp | telegram | discord | slack | ...
  }
}
```

### accountId（可选）

指定账号：

```json5
{
  match: {
    channel: "whatsapp",
    accountId: "personal"  // 账号 ID 或 "*"
  }
}
```

### peer（可选）

指定聊天：

```json5
{
  match: {
    channel: "telegram",
    peer: {
      kind: "dm",      // dm | group | channel
      id: "123456789"  // 用户/群组 ID
    }
  }
}
```

### guildId（Discord）

```json5
{
  match: {
    channel: "discord",
    guildId: "123456789012345678"
  }
}
```

### teamId（Microsoft Teams）

```json5
{
  match: {
    channel: "msteams",
    teamId: "team-uuid"
  }
}
```

## 默认代理

当没有绑定匹配时，使用默认代理：

1. `agents.list[]` 中 `default: true` 的代理
2. 如果没有，使用 `agents.list` 的第一个
3. 如果列表为空，使用 ID 为 `"main"` 的隐式代理

```json5
{
  agents: {
    list: [
      { id: "primary", default: true },  // ← 这是默认代理
      { id: "secondary" }
    ]
  }
}
```

## 多代理路由示例

### 场景1：按渠道分离

```json5
{
  agents: {
    list: [
      { id: "whatsapp-agent", workspace: "~/clawd-wa" },
      { id: "telegram-agent", workspace: "~/clawd-tg" }
    ]
  },
  bindings: [
    { agentId: "whatsapp-agent", match: { channel: "whatsapp" } },
    { agentId: "telegram-agent", match: { channel: "telegram" } }
  ]
}
```

### 场景2：按用途分离

```json5
{
  agents: {
    list: [
      { id: "personal", default: true },
      { id: "work" },
      { id: "family" }
    ]
  },
  bindings: [
    // 工作群组
    { agentId: "work", match: { channel: "whatsapp", peer: { kind: "group", id: "work-group@g.us" } } },
    // 家人群组
    { agentId: "family", match: { channel: "whatsapp", peer: { kind: "group", id: "family@g.us" } } }
    // 其他默认到 personal
  ]
}
```

### 场景3：多 WhatsApp 账号

```json5
{
  agents: {
    list: [
      { id: "home", workspace: "~/clawd-home" },
      { id: "office", workspace: "~/clawd-office" }
    ]
  },
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        business: {}
      }
    }
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "office", match: { channel: "whatsapp", accountId: "business" } }
  ]
}
```

## 路由调试

### 查看路由结果

使用详细日志：

```bash
openclaw gateway --verbose
```

### 检查代理绑定

```bash
openclaw config get bindings
openclaw config get agents.list
```

### 模拟路由

（未来功能）

```bash
openclaw route test --channel whatsapp --peer +15555550123
```

## 路由相关配置

### 代理配置

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd"
    },
    list: [
      {
        id: "main",
        default: true,
        workspace: "~/clawd-main",
        model: "anthropic/claude-sonnet-4-20250514"
      }
    ]
  }
}
```

### 会话键生成

路由结果决定会话键：

```
agent:<agentId>:<channel>:<type>:<id>
```

## 最佳实践

### 简单场景

单代理，无需绑定：

```json5
{
  agents: {
    defaults: { workspace: "~/clawd" }
  }
}
```

### 按渠道隔离

```json5
{
  agents: {
    list: [
      { id: "wa", workspace: "~/clawd-wa" },
      { id: "tg", workspace: "~/clawd-tg" }
    ]
  },
  bindings: [
    { agentId: "wa", match: { channel: "whatsapp" } },
    { agentId: "tg", match: { channel: "telegram" } }
  ]
}
```

### 按安全级别隔离

```json5
{
  agents: {
    list: [
      { id: "trusted", sandbox: { mode: "off" } },
      { id: "public", sandbox: { mode: "all", workspaceAccess: "none" } }
    ]
  },
  bindings: [
    { agentId: "trusted", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15555550123" } } }
    // 其他路由到 public
  ]
}
```

## 下一步

- [AI 代理](/zh-CN/concepts/agents) - 代理配置详解
- [会话管理](/zh-CN/concepts/sessions) - 会话系统
- [配置参考](/zh-CN/config/reference) - 完整配置
