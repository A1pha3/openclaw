---
summary: "多代理系统详解 - 配置多个 AI 代理及其协作"
read_when:
  - 配置多个代理
  - 实现代理间协作
  - 管理代理路由
title: "多代理系统"
---

# 🤖 多代理系统

本文档详细介绍 OpenClaw 的多代理系统，包括配置多个 AI 代理、代理路由和协作机制。

## 🎯 为什么需要多代理？

**单一代理的局限**：

```
一个代理处理所有任务：
- ✅ 简单场景足够
- ❌ 任务类型混杂
- ❌ 难以针对优化
- ❌ 无法隔离上下文
```

**多代理的优势**：

```
多个专业代理：
- ✅ 任务专业化
- ✅ 上下文隔离
- ✅ 独立配置
- ✅ 灵活路由
```

---

## 📊 代理架构

### 代理类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **主代理** | 处理主要对话 | `main` |
| **工作代理** | 处理特定任务 | `work`、`coding` |
| **辅助代理** | 提供专业支持 | `helper`、`researcher` |

### 代理关系

```
用户消息
    │
    ▼
┌──────────────┐
│  路由引擎    │ ─→ 选择目标代理
└──────────────┘
    │
    ├──→ main      （主对话）
    ├──→ coding    （代码任务）
    ├──→ work      （工作相关）
    └──→ helper    （辅助任务）
```

---

## ⚙️ 多代理配置

### 基本配置

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,              // 默认代理
        name: "主助手",
        workspace: "~/openclaw/workspace-main",
        model: "anthropic/claude-opus-4-20250514"
      },
      {
        id: "coding",
        name: "编程助手",
        workspace: "~/openclaw/workspace-coding",
        model: "anthropic/claude-sonnet-4-20250514",
        identity: {
          name: "CodeBot",
          emoji: "👨‍💻"
        }
      },
      {
        id: "work",
        name: "工作助手",
        workspace: "~/openclaw/workspace-work",
        model: "anthropic/claude-sonnet-4-20250514"
      }
    ]
  }
}
```

### 默认配置

```json5
{
  agents: {
    defaults: {
      workspace: "~/openclaw/workspace",
      model: "anthropic/claude-sonnet-4-20250514",
      sandbox: {
        mode: "non-main"
      }
    }
  }
}
```

---

## 🔀 代理路由

### 基本路由

```json5
{
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "whatsapp",
        accountId: "personal"
      }
    },
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "work"
      }
    }
  ]
}
```

### 路由匹配规则

| 优先级 | 匹配条件 | 说明 |
|--------|----------|------|
| 1 | `match.peer` | 特定聊天（DM/群组） |
| 2 | `match.guildId` | Discord 服务器 |
| 3 | `match.teamId` | Teams 团队 |
| 4 | `match.accountId` | 特定账号 |
| 5 | `match.channel` | 特定渠道 |
| 6 | 默认代理 | 无匹配时 |

### 按消息内容路由

```json5
{
  bindings: [
    {
      agentId: "coding",
      match: {
        channel: "*",
        content: {
          pattern: "/code|编程|代码|debug"  // 包含关键词
        }
      }
    }
  ]
}
```

---

## 🔄 代理间通信

### 启用代理通信

```json5
{
  tools: {
    agentToAgent: {
      enabled: true,
      allow: ["main", "coding", "work"]
    }
  }
}
```

### 调用子代理

```json5
{
  agents: {
    list: [
      {
        id: "main",
        subagents: {
          allowAgents: ["coding", "helper"],
          autoInvoke: {
            enabled: true,
            patterns: [
              { pattern: "写代码", agent: "coding" },
              { pattern: "搜索", agent: "helper" }
            ]
          }
        }
      }
    ]
  }
}
```

### 代理消息传递

```bash
# 发送消息给另一个代理
openclaw message send --target agent:coding --message "请帮我写一个函数..."
```

---

## 📋 工作区管理

### 独立工作区

```json5
{
  agents: {
    list: [
      {
        id: "main",
        workspace: "~/openclaw/workspace-main",
        // 读取 workspace-main/ 下的文件
      },
      {
        id: "coding",
        workspace: "~/openclaw/workspace-coding",
        // 读取 workspace-coding/ 下的文件
      }
    ]
  }
}
```

### 工作区模板

```
~/openclaw/
├── workspace-main/          # 主代理工作区
│   ├── AGENTS.md
│   ├── SOUL.md
│   ├── USER.md
│   ├── MEMORY.md
│   └── skills/
│
├── workspace-coding/        # 编程代理工作区
│   ├── AGENTS.md           # 专注代码
│   ├── SOUL.md
│   ├── USER.md
│   └── skills/
│
└── workspace-work/          # 工作代理工作区
    ├── AGENTS.md
    └── USER.md
```

---

## 🔧 代理配置示例

### 个人+工作分离

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "non-main" }
    },
    list: [
      {
        id: "personal",
        default: true,
        workspace: "~/openclaw/personal",
        identity: {
          name: "小助手",
          emoji: "🦞"
        }
      },
      {
        id: "work",
        workspace: "~/openclaw/work",
        identity: {
          name: "工作助手",
          emoji: "💼"
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "personal",
      match: { channel: "whatsapp", accountId: "personal" }
    },
    {
      agentId: "work",
      match: { channel: "whatsapp", accountId: "work" }
    }
  ]
}
```

### 专业分工

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        workspace: "~/openclaw/main",
        model: "anthropic/claude-opus-4-20250514"
      },
      {
        id: "coder",
        workspace: "~/openclaw/code",
        model: "anthropic/claude-sonnet-4-20250514",
        tools: {
          allow: ["read", "write", "edit", "bash", "lsp"],
          deny: ["browser"]
        }
      },
      {
        id: "researcher",
        workspace: "~/openclaw/research",
        model: "anthropic/claude-haiku-3-5-20241022",
        tools: {
          allow: ["web", "search"],
          deny: ["bash", "write"]
        }
      }
    ]
  }
}
```

---

## 📊 代理监控

### 查看代理状态

```bash
# 列出所有代理
openclaw agents list

# 查看代理状态
openclaw agents status main
openclaw agents status coding
```

**输出示例**：
```
代理状态：
┌────────┬────────────┬──────────┬─────────┐
│ 名称   │ 模型                       │ 会话数 │ 状态  │
├────────┼────────────┬──────────┼─────────┤
│ main   │ claude-opus-4             │ 12     │ 运行中│
│ coding │ claude-sonnet-4           │ 5      │ 运行中│
│ work   │ claude-sonnet-4           │ 3      │ 运行中│
└────────┴────────────┴──────────┴─────────┘
```

### 代理使用统计

```bash
# 查看消息统计
openclaw status | grep agents

# 查看会话分布
openclaw sessions stats
```

---

## 🐛 故障排除

### 消息路由错误

```bash
# 查看当前绑定
openclaw config get bindings

# 测试路由
openclaw message send --target test --message "test"

# 查看路由日志
openclaw logs --lines 50 | grep routing
```

### 代理不可用

```bash
# 检查代理状态
openclaw agents status <agent-id>

# 查看代理配置
openclaw config get agents.list

# 重启代理
openclaw gateway restart
```

---

## 📈 最佳实践

### 简单场景

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        sandbox: { mode: "off" }  // 无沙箱
      }
    ]
  }
}
```

### 多用户场景

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    list: [
      { id: "user1", workspace: "~/openclaw/user1" },
      { id: "user2", workspace: "~/openclaw/user2" }
    ]
  },
  bindings: [
    { agentId: "user1", match: { channel: "whatsapp", accountId: "user1" } },
    { agentId: "user2", match: { channel: "whatsapp", accountId: "user2" } }
  ]
}
```

### 生产环境

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: ["anthropic/claude-haiku-3-5-20241022"]
      }
    },
    list: [
      { id: "public", sandbox: { workspaceAccess: "none" } },
      { id: "internal" }
    ]
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw agents list` | 列出代理 |
| `openclaw agents status` | 查看状态 |
| `openclaw agents add` | 添加代理 |
| `openclaw agents remove` | 删除代理 |
| `openclaw binding` | 管理路由 |

---

## 📚 相关文档

- [代理系统](/zh-CN/concepts/agents) - 代理详解
- [会话管理](/zh-CN/concepts/sessions) - 会话系统
- [消息路由](/zh-CN/concepts/routing) - 路由配置
- [配置参考](/zh-CN/config/reference) - 完整配置

---

**多代理系统让 AI 助手更专业、更高效！** 🦞
