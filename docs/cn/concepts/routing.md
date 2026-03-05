---
read_when:
  - 需要配置消息路由时
  - 设置多代理路由时
  - 调试路由问题时
summary: "消息路由完整指南：路由概述、流程、绑定配置、匹配优先级、多代理场景和最佳实践"
title: "消息路由"
---

# 消息路由

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 消息路由的核心概念
- [ ] 掌握绑定配置的语法和用法
- [ ] 了解匹配优先级的判断逻辑
- [ ] 配置简单的多代理路由

### 进阶目标（建议掌握）

- [ ] 设计复杂的多代理路由架构
- [ ] 使用 peer 和 guildId 精确路由
- [ ] 配置按安全级别隔离的代理
- [ ] 调试和解决路由问题

---

## 💡 为什么需要消息路由？

### 类比：路由就像"电话总机"

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    路由系统类比                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  传统电话总机：                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  来电者 ──► 总机接线员 ──► 转接到正确的分机                      │   │
│  │                                                                 │   │
│  │  接线员判断：                                                    │   │
│  │  • 这是销售电话吗？ → 转销售部                                  │   │
│  │  • 这是客户投诉吗？ → 转客服部                                  │   │
│  │  • 这是老板的电话吗？ → 转老板办公室                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenClaw 消息路由：                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  入站消息 ──► 路由系统 ──► 分发给正确的 AI 代理                   │   │
│  │                                                                 │   │
│  │  路由判断：                                                      │   │
│  │  • 这是 WhatsApp 消息吗？ → 转个人代理                          │   │
│  │  • 这是工作群组吗？ → 转工作代理                                │   │
│  │  • 这是陌生人消息吗？ → 转受限代理（沙箱模式）                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  核心价值：                                                              │
│  ✅ 上下文隔离 — 工作和个人消息分开                                    │
│  ✅ 个性化配置 — 不同场景不同行为                                      │
│  ✅ 安全控制 — 未知来源使用受限模式                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 路由决策

当收到消息时，OpenClaw 需要决定：

1. **哪个代理**处理这条消息？
2. **哪个会话**上下文？
3. **什么配置**策略？

---

## 🔄 路由流程

### 完整决策流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    路由决策流程                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  入站消息                                                                │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────┐                                                    │
│  │  1. 渠道识别    │  whatsapp / telegram / discord / slack / ...      │
│  └────────┬────────┘                                                    │
│           │                                                             │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │  2. 账号匹配    │  personal / business / default                   │
│  └────────┬────────┘                                                    │
│           │                                                             │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │  3. 绑定匹配    │  按优先级检查 bindings[]                         │
│  │  │              │  • peer（聊天）                                  │
│  │  │              │  • guildId（Discord 服务器）                     │
│  │  │              │  • teamId（Teams 团队）                          │
│  │  │              │  • accountId（账号）                             │
│  └────────┬────────┘                                                    │
│           │                                                             │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │  4. 代理选择    │  agents.list[] 或默认代理                         │
│  └────────┬────────┘                                                    │
│           │                                                             │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │  5. 会话加载    │  agent:<id>:<channel>:<type>:<id>                │
│  └────────┬────────┘                                                    │
│           │                                                             │
│           ▼                                                             │
│      智能体处理                                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 会话键生成

路由结果决定会话键格式：

```
agent:<agentId>:<channel>:<type>:<id>
```

**示例**：
- `agent:main:whatsapp:dm:+15555550123` — WhatsApp 私聊
- `agent:work:telegram:group:-1001234567890` — Telegram 群组
- `agent:gaming:discord:guild:123456789012345678` — Discord 服务器

---

## ⚙️ 绑定配置

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
    // 1. 按聊天路由（最高优先级）
    {
      agentId: "support",
      match: {
        channel: "telegram",
        peer: { kind: "group", id: "-1001234567890" }
      }
    },

    // 2. 按账号路由
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "business"
      }
    },

    // 3. 按 Discord 服务器路由
    {
      agentId: "gaming",
      match: {
        channel: "discord",
        guildId: "123456789012345678"
      }
    },

    // 4. 通配符账号（最低优先级）
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

---

## 🎯 匹配优先级

### 优先级表

绑定按以下优先级匹配（从高到低）：

| 优先级 | 匹配条件 | 说明 | 示例 |
|--------|----------|------|------|
| **1** | `match.peer` | 特定聊天（DM/群组） | 某个群组用专用代理 |
| **2** | `match.guildId` | Discord 服务器 | 游戏服务器用游戏代理 |
| **3** | `match.teamId` | Teams 团队 | 工作团队用工作代理 |
| **4** | `match.accountId`（精确） | 特定账号 | 个人号 vs 商业号 |
| **5** | `match.accountId: "*"` | 任意账号 | 该渠道的通配代理 |
| **6** | 默认代理 | 无匹配时使用 | `default: true` 的代理 |

### 确定性匹配

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    匹配决策示例                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  配置：                                                                  │
│  bindings: [                                                            │
│    { agentId: "A", match: { channel: "telegram", peer: {...} } },      │
│    { agentId: "B", match: { channel: "telegram" } }                     │
│  ]                                                                      │
│                                                                         │
│  来自匹配 peer 的消息 → 代理 A（优先级更高）                            │
│  其他 Telegram 消息 → 代理 B                                             │
│                                                                         │
│  重要规则：                                                              │
│  • 同一优先级内，使用 bindings 数组中的第一个匹配项                     │
│  • 更具体的规则应该放在数组前面                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 匹配条件详解

### channel（必需）

指定渠道：

```json5
{
  match: {
    channel: "whatsapp"  // whatsapp | telegram | discord | slack | signal | imessage
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
      kind: "dm",      // dm（私聊） | group（群组） | channel（频道）
      id: "123456789"  // 用户/群组 ID
    }
  }
}
```

### guildId（Discord 专用）

指定 Discord 服务器：

```json5
{
  match: {
    channel: "discord",
    guildId: "123456789012345678"
  }
}
```

### teamId（Microsoft Teams 专用）

指定 Teams 团队：

```json5
{
  match: {
    channel: "msteams",
    teamId: "team-uuid"
  }
}
```

---

## 🤖 默认代理

### 默认代理选择逻辑

当没有绑定匹配时，按以下顺序选择默认代理：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    默认代理选择                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 检查 agents.list[] 中 default: true 的代理                          │
│     │                                                                   │
│     ▼ （找到）                                                          │
│  使用该代理 ✓                                                           │
│                                                                         │
│     ▼ （未找到）                                                        │
│  2. 使用 agents.list 的第一个代理                                       │
│     │                                                                   │
│     ▼ （列表为空）                                                      │
│  3. 使用 ID 为 "main" 的隐式代理                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 配置示例

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

---

## 🌟 多代理路由场景

### 场景 1：按渠道分离

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    按渠道分离                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  WhatsApp ──► whatsapp-agent ──► ~/clawd-wa 工作区                      │
│  Telegram ──► telegram-agent ──► ~/clawd-tg 工作区                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

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

### 场景 2：按用途分离

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    按用途分离                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  工作群组 ──► work 代理（正式、专业）                                    │
│  家人群组 ──► family 代理（友好、随意）                                  │
│  其他消息 ──► personal 代理（默认）                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

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
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "work-group@g.us" }
      }
    },
    // 家人群组
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "family@g.us" }
      }
    }
    // 其他默认到 personal
  ]
}
```

### 场景 3：多 WhatsApp 账号

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    多账号分离                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  个人账号（personal）──► home 代理 ──► ~/clawd-home 工作区              │
│  工作账号（business）──► office 代理 ──► ~/clawd-office 工作区          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

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

### 场景 4：按安全级别隔离

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    按安全级别隔离                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  受信任用户（白名单）──► trusted 代理 ──► 沙箱关闭，完全访问             │
│  未知来源 ────────────► public 代理 ──► 沙箱全开，无工作区访问           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

```json5
{
  agents: {
    list: [
      {
        id: "trusted",
        sandbox: { mode: "off" }
      },
      {
        id: "public",
        sandbox: { mode: "all", workspaceAccess: "none" }
      }
    ]
  },
  bindings: [
    // 仅受信任的私聊使用 trusted 代理
    {
      agentId: "trusted",
      match: {
        channel: "whatsapp",
        peer: { kind: "dm", id: "+15555550123" }
      }
    }
    // 其他所有消息路由到 public（因为 public 可设为 default）
  ]
}
```

---

## 🔧 路由相关配置

### 代理配置

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
        workspace: "~/clawd-main"
      },
      {
        id: "secondary",
        workspace: "~/clawd-secondary",
        model: "anthropic/claude-haiku-3-5-20241022"
      }
    ]
  }
}
```

### 代理级别配置覆盖

| 配置项 | 说明 |
|--------|------|
| `workspace` | 工作区路径 |
| `model` | 使用的模型 |
| `sandbox.mode` | 沙箱模式 |
| `systemPrompt` | 自定义系统提示 |

---

## 🐛 路由调试

### 查看路由结果

使用详细日志查看路由决策：

```bash
openclaw gateway --verbose
```

日志输出示例：
```
[INFO] Routing message from channel=whatsapp peer=+15555550123
[INFO] Matched binding: agentId=personal (priority=4)
[INFO] Session key: agent:personal:whatsapp:dm:+15555550123
```

### 检查代理绑定

```bash
# 查看所有绑定
openclaw config get bindings

# 查看代理列表
openclaw config get agents.list

# 验证配置
openclaw config validate
```

### 常见路由问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 消息无响应 | 代理 ID 不存在 | 检查 `agentId` 是否在 `agents.list` 中 |
| 路由到错误代理 | 优先级顺序错误 | 将更具体的规则放在数组前面 |
| 所有消息同一代理 | 绑定未匹配 | 检查 `match.channel` 等条件是否正确 |
| 会话上下文混乱 | 会话键冲突 | 确保每个路由有唯一的会话键 |

### 调试技巧

```bash
# 1. 测试单个渠道
openclaw channels status --probe

# 2. 查看活跃会话
openclaw sessions list --active 60

# 3. 重置会话（测试用）
openclaw sessions reset <sessionId>
```

---

## 🎯 最佳实践

### 简单场景

单代理，无需绑定：

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd"
    }
  }
}
```

### 按渠道隔离

```
适用场景：不同渠道需要不同配置
```

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

### 按用途隔离

```
适用场景：工作/个人/家庭需要不同行为
```

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
    { agentId: "work", match: { channel: "whatsapp", peer: { kind: "group", id: "work-group@g.us" } } },
    { agentId: "family", match: { channel: "whatsapp", peer: { kind: "group", id: "family@g.us" } } }
  ]
}
```

### 配置建议

| 实践 | 说明 |
|------|------|
| **具体规则优先** | 将更具体的 match 规则放在 bindings 数组前面 |
| **明确默认代理** | 设置 `default: true` 避免歧义 |
| **命名清晰** | 使用有意义的 agentId（work、personal、family） |
| **测试路由** | 使用 `--verbose` 验证路由是否符合预期 |

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [智能体运行时](/concepts/agent) | 代理配置详解 |
| [会话管理](/concepts/session) | 会话系统 |
| [智能体工作区](/concepts/agent-workspace) | 工作区隔离 |
| [配置参考](/config/reference) | 完整配置选项 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置基本路由绑定 | ⭐⭐⭐⭐⭐ |
| 理解匹配优先级 | ⭐⭐⭐⭐ |
| 设计多代理架构 | ⭐⭐⭐ |
| 调试路由问题 | ⭐⭐⭐ |

---

> **💡 专家提示**：路由规则按数组顺序匹配，更具体的规则一定要放在通配规则之前！
