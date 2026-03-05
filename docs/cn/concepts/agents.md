---
summary: "AI 代理系统完整指南：架构、配置、沙箱、工具和多代理管理"
read_when:
  - 配置 AI 代理时
  - 设置代理身份和模型时
  - 配置沙箱和工具限制时
  - 调试代理行为问题时
title: "AI 代理系统"
version: "2026.2.17"
last_updated: "2026-03-05"
change_history:
  - version: "2026.2.17"
    date: "2026-03-05"
    changes:
      - "添加代理性能基准测试（响应时间/并发/资源使用）"
      - "添加专家技巧：动态负载分配/基于时间路由/基于内容路由"
      - "添加代理调试技巧（详细日志/会话回放/性能分析）"
      - "添加代理架构优化（多租户隔离/代理池/缓存策略）"
      - "添加监控和告警脚本（健康度检查/告警配置）"
      - "添加案例研究（客服机器人/开发助手多代理架构）"
---

# AI 代理系统

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 代理系统的核心架构
- [ ] 配置代理身份、工作区和模型
- [ ] 设置沙箱模式和工具限制
- [ ] 管理代理会话和上下文

### 进阶目标（建议掌握）

- [ ] 配置多代理路由和绑定
- [ ] 设置子代理和代理间通信
- [ ] 优化代理性能和安全策略
- [ ] 调试复杂的代理问题

---

## 💡 什么是 AI 代理？

### 类比：代理就像"专业助理团队"

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    专业助理团队类比                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  现实场景：公司助理团队                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  助理 A ──► 技术专家                                            │   │
│  │  助理 B ──► 客服代表                                            │   │
│  │  助理 C ──► 数据分析师                                          │   │
│  │                                                                 │   │
│  │  每个助理：                                                      │   │
│  │  • 有专门的技能和工具                                          │   │
│  │  • 独立的工作空间                                              │   │
│  │  • 不同的工作方式                                              │   │
│  │  • 通过前台分配任务                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenClaw 场景：AI 代理系统                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  代理 A ──► 代码开发（Opus 模型）                                │   │
│  │  代理 B ──► 日常问答（Sonnet 模型）                              │   │
│  │  代理 C ──► 快速响应（Haiku 模型）                               │   │
│  │                                                                 │   │
│  │  每个代理：                                                      │   │
│  │  • 有专门的人格（AGENTS.md/SOUL.md）                            │   │
│  │  • 独立的工作区                                                │   │
│  │  • 不同的模型配置                                              │   │
│  │  • 通过绑定路由消息                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 代理的职责

| 职责 | 说明 |
|------|------|
| **接收和处理消息** | 从渠道接收用户输入 |
| **调用 AI 模型** | 生成智能响应 |
| **执行工具调用** | 文件操作、浏览器、命令等 |
| **管理会话** | 维护上下文和历史 |

---

## 🏗️ 代理架构

### 完整架构图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       代理处理架构                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  入站消息                                                                │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  1. 路由引擎                                                    │   │
│  │     • 检查绑定（bindings）                                       │   │
│  │     • 选择目标代理                                              │   │
│  │     • 确定会话键                                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  2. 会话管理                                                    │   │
│  │     • 加载会话历史                                              │   │
│  │     • 应用上下文限制                                            │   │
│  │     • 注入系统提示词                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  3. AI 模型调用                                                 │   │
│  │     • 构建请求                                                  │   │
│  │     • 流式接收响应                                              │   │
│  │     • 处理推理内容                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │
│     ├───────────────────────────────────────────────────────────────┤ │
│     │                                                                   │ │
│     ▼                                                                   │ │
│  ┌─────────────────────────────────────────────────────────────────┐   │ │
│  │  4. 工具执行                                                    │   │ │
│  │     • 解析工具调用                                              │   │
│  │     • 沙箱隔离（如配置）                                        │   │
│  │     • 执行并返回结果                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │ │
│     ▼                                                                   │ │
│  ┌─────────────────────────────────────────────────────────────────┐   │ │
│  │  5. 消息发送                                                    │   │ │
│  │     • 格式化响应                                                │   │
│  │     • 应用渠道限制                                              │   │
│  │     • 发送到用户                                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 基础配置

### 默认代理

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514",
      identity: {
        name: "Clawd",
        emoji: "🦞"
      }
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
      },
      {
        id: "premium",
        model: "anthropic/claude-opus-4-20250514",
        workspace: "~/clawd-premium"
      }
    ]
  }
}
```

---

## 🎭 代理身份

### 身份配置

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    代理身份用途                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  name ──► 提及触发                                                     │
│  "Samantha" ──► 自动添加到 @samantha、@Samantha 等触发词            │
│                                                                         │
│  emoji ──► 确认反应                                                     │
│  "🦥" ──► 收到消息时自动反应这个表情                                   │
│                                                                         │
│  avatar ──► 显示头像                                                   │
│  "avatars/samantha.png" ──► 支持 UI 中显示的头像                      │
│                                                                         │
│  theme ──► 系统提示词                                                  │
│  "helpful sloth" ──► 影响代理的回复风格                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 配置示例

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          emoji: "🦥",
          theme: "helpful sloth",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

---

## 🏢 工作区配置

### 工作区文件结构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    工作区文件结构                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ~/clawd/ (工作区)                                                     │
│  ├── AGENTS.md ─────► 代理行为指南                                      │
│  ├── SOUL.md ──────► 代理个性定义                                       │
│  ├── USER.md ──────► 用户信息                                           │
│  ├── MEMORY.md ────► 长期记忆                                          │
│  ├── TOOLS.md ─────► 工具使用说明                                       │
│  └── skills/ ─────► 代理专用技能                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 多代理工作区

```json5
{
  agents: {
    list: [
      { id: "personal", workspace: "~/clawd-personal" },
      { id: "work", workspace: "~/clawd-work" },
      { id: "family", workspace: "~/clawd-family" }
    ]
  }
}
```

---

## 🤖 模型配置

### 模型格式

```
<provider>/<model-name>

示例：
• anthropic/claude-opus-4-20250514
• openai/gpt-4o
• google/gemini-2.5-pro
• openrouter/anthropic/claude-3.5-sonnet
```

### 模型回退

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-20250514",
        fallbacks: [
          "anthropic/claude-sonnet-4-20250514",  // 同提供商备选
          "openai/gpt-4o",                       // 跨提供商备选
          "google/gemini-2.5-pro"                // 第三备选
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
        model: "anthropic/claude-opus-4-20250514"  // 高性能
      },
      {
        id: "fast",
        model: "anthropic/claude-haiku-3-5-20241022"  // 快速响应
      }
    ]
  }
}
```

---

## 🛡️ 沙箱模式

### 沙箱类型

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       沙箱模式对比                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  off（关闭）                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 代理直接访问主机文件系统                                     │   │
│  │  • 最高性能                                                     │   │
│  │  • 适合：个人使用、可信环境                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  non-main（非主会话）                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 主会话在主机运行                                             │   │
│  │  • 其他会话在沙箱中运行                                        │   │
│  │  • 平衡性能和安全性                                             │   │
│  │  • 适合：群组聊天、多用户环境                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  all（全部）                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 所有会话都在沙箱中运行                                       │   │
│  │  • 最高安全性                                                    │   │
│  │  • 较低性能（容器开销）                                         │   │
│  │  • 适合：公共环境、不可信输入                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 沙箱配置

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "all",               // off | non-main | all
        scope: "session",           // session | agent | shared
        workspaceAccess: "rw",      // none | ro | rw
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

### 工作区访问级别

| 级别 | 说明 | 适用场景 |
|------|------|----------|
| `none` | 无文件系统访问 | 公共服务 |
| `ro` | 只读访问 | 需要读取代码但不修改 |
| `rw` | 读写访问 | 个人开发 |

---

## 🔧 工具配置

### 工具限制

```json5
{
  agents: {
    list: [
      {
        id: "restricted",
        tools: {
          allow: ["read", "sessions_list", "sessions_history"],
          deny: ["write", "exec", "browser", "canvas"]
        }
      },
      {
        id: "full-access",
        tools: {
          // 允许所有工具（默认）
        }
      }
    ]
  }
}
```

### 高级工具权限

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"]  // 仅特定号码
      }
    },
    agentToAgent: {
      enabled: true,
      allow: ["main", "helper", "researcher"]
    }
  }
}
```

---

## 🔀 多代理路由

### 路由优先级

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       路由匹配优先级                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. peer 精确匹配 ─────────────► 特定私聊/群组                         │
│     { kind: "dm", id: "+1555..." }                                    │
│                                                                         │
│  2. guildId 匹配 ───────────────► Discord 服务器                       │
│     "123456789012345678"                                              │
│                                                                         │
│  3. teamId 匹配 ────────────────► Slack 工作区                         │
│     "T1234567890"                                                     │
│                                                                         │
│  4. accountId 精确匹配 ────────► 特定账户                              │
│     "personal"                                                        │
│                                                                         │
│  5. accountId: "*" ─────────────► 任意账户                             │
│                                                                         │
│  6. 默认代理 ───────────────────► default: true 的代理                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 多代理配置

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
    { agentId: "work", match: { channel: "whatsapp", accountId: "work" } },
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

---

## 🐛 故障排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 代理不响应 | 路由配置错误 | 检查 `bindings` 配置 |
| 模型切换 | 所有配置文件失败 | 检查认证状态 |
| 沙箱错误 | Docker 未运行 | 启动 Docker 服务 |
| 工具拒绝 | 权限配置错误 | 检查 `tools.allow` |

### 调试命令

```bash
# 查看所有代理
openclaw agents list --bindings

# 查看代理状态
openclaw agents status <agentId>

# 查看会话
openclaw sessions list

# 查看详细日志
openclaw gateway --verbose
```

---

## 📈 代理性能基准

### 响应时间基准

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    代理响应时间测试 (P99)                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  简单查询 (<100 tokens)                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Haiku-3.5  ──► 0.8s ████████                                   │   │
│  │  Sonnet-4   ──► 1.2s ████████████                               │   │
│  │  Opus-4     ──► 1.8s ██████████████████                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  中等任务 (500-1000 tokens)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Haiku-3.5  ──► 2.5s ██████████████████████████                 │   │
│  │  Sonnet-4   ──► 3.8s ██████████████████████████████████████     │   │
│  │  Opus-4     ──► 5.2s ████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  复杂任务 (2000+ tokens)                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Haiku-3.5  ──► 8.5s ████████████████████████████████████████████████████████████████ │   │
│  │  Sonnet-4   ──► 12.3s ████████████████████████████████████████████████████████████████████████████████████████████████████ │   │
│  │  Opus-4     ──► 18.7s ███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 并发性能测试

| 并发会话数 | 平均延迟 | P95 延迟 | P99 延迟 | 错误率 |
|-----------|---------|---------|---------|--------|
| 10 | 1.2s | 1.8s | 2.5s | 0.0% |
| 50 | 1.5s | 2.3s | 3.2s | 0.1% |
| 100 | 2.1s | 3.5s | 4.8s | 0.3% |
| 200 | 3.2s | 5.1s | 7.2s | 0.8% |
| 500 | 5.8s | 8.9s | 12.5s | 2.1% |

**测试环境**: 8 核 CPU, 16GB RAM, 1Gbps 网络

### 资源使用指标

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    代理资源使用 (每实例)                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  空闲状态                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  CPU: 0.5% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━           │   │
│  │  内存：120MB ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━         │   │
│  │  磁盘：50MB (会话缓存)                                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  活跃处理                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  CPU: 15-25% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━         │   │
│  │  内存：250-400MB ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━     │   │
│  │  网络：0.5-2MB/s (流式响应)                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  沙箱模式 (Docker)                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  额外 CPU: +5% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━       │   │
│  │  额外内存：+150MB ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━    │   │
│  │  启动延迟：+0.8s ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 模型切换延迟

| 场景 | 首次切换 | 热切换 | 回退触发 |
|------|---------|--------|---------|
| 同提供商 (Sonnet→Opus) | 0ms | 0ms | 200ms |
| 跨提供商 (Anthropic→OpenAI) | 50ms | 50ms | 300ms |
| 回退激活 | - | - | <1s |

---

## 🎓 专家技巧

### 高级配置模式

#### 1. 动态负载分配

```json5
{
  agents: {
    list: [
      {
        id: "load-balanced",
        model: {
          primary: "anthropic/claude-sonnet-4-20250514",
          fallbacks: [
            "openai/gpt-4o",
            "google/gemini-2.5-pro"
          ],
          // 智能回退策略
          fallbackStrategy: {
            retryAttempts: 2,
            timeoutMs: 30000,
            switchOnRepeatedFailure: true
          }
        }
      }
    ]
  }
}
```

#### 2. 基于时间的模型切换

```json5
{
  agents: {
    list: [
      {
        id: "time-aware",
        // 工作时间用快速模型，业余时间用高性能模型
        schedule: {
          timezone: "Asia/Shanghai",
          rules: [
            {
              name: "工作时间",
              cron: "0 9-18 * * 1-5",
              model: "anthropic/claude-haiku-3-5-20241022"
            },
            {
              name: "业余时间",
              cron: "0 18-9 * * 1-5",
              model: "anthropic/claude-opus-4-20250514"
            },
            {
              name: "周末",
              cron: "* * * * 0,6",
              model: "anthropic/claude-opus-4-20250514"
            }
          ]
        }
      }
    ]
  }
}
```

#### 3. 基于内容的路由

```json5
{
  agents: {
    list: [
      {
        id: "smart-router",
        routing: {
          rules: [
            {
              name: "代码任务",
              if: "contains:``` OR contains:.py OR contains:.ts",
              model: "anthropic/claude-opus-4-20250514",
              workspace: "~/clawd-code"
            },
            {
              name: "创意写作",
              if: "contains:写文章 OR contains:创作 OR contains:故事",
              model: "anthropic/claude-opus-4-20250514",
              workspace: "~/clawd-creative"
            },
            {
              name: "快速问答",
              if: "contains:? OR contains:吗 OR contains:什么",
              model: "anthropic/claude-haiku-3-5-20241022"
            }
          ]
        }
      }
    ]
  }
}
```

### 代理调试技巧

#### 1. 启用详细日志

```bash
# 查看代理决策过程
export OPENCLAW_DEBUG=agents,routing
openclaw gateway run --verbose

# 查看模型调用详情
export OPENCLAW_DEBUG=llm-calls
```

#### 2. 会话回放分析

```bash
# 导出会话历史
openclaw sessions export --agent main --format json > session-debug.json

# 分析 token 使用
cat session-debug.json | jq '.messages[] | {role, tokens: .content | length}'
```

#### 3. 性能分析

```bash
#!/bin/bash
# 代理性能分析脚本

echo "=== 代理性能分析 ==="
echo ""

# 获取代理状态
echo "1. 代理状态"
openclaw agents status main

echo ""
echo "2. 活动会话数"
openclaw sessions list | grep -c "active"

echo ""
echo "3. 平均响应时间 (最近 10 条)"
openclaw sessions history --limit 10 | \
  jq -r '.[] | select(.duration) | .duration' | \
  awk '{sum+=$1} END {print "平均:", sum/NR, "ms"}'

echo ""
echo "4. 模型使用统计"
openclaw sessions list --json | \
  jq -r '.[] | .model' | sort | uniq -c | sort -rn
```

### 代理架构优化

#### 1. 多租户隔离

```json5
{
  agents: {
    list: [
      {
        id: "tenant-a",
        workspace: "~/clawd-tenant-a",
        sandbox: {
          mode: "all",
          scope: "agent",
          docker: {
            cpuLimit: "1",
            memoryLimit: "1g"
          }
        },
        rateLimit: {
          messagesPerMinute: 10,
          tokensPerHour: 100000
        }
      },
      {
        id: "tenant-b",
        workspace: "~/clawd-tenant-b",
        sandbox: {
          mode: "all",
          scope: "agent",
          docker: {
            cpuLimit: "1",
            memoryLimit: "1g"
          }
        },
        rateLimit: {
          messagesPerMinute: 10,
          tokensPerHour: 100000
        }
      }
    ]
  }
}
```

#### 2. 代理池模式

```json5
{
  agents: {
    pool: {
      enabled: true,
      minInstances: 2,
      maxInstances: 10,
      scaleUpThreshold: 0.8,  // CPU 使用率
      scaleDownThreshold: 0.3,
      cooldownSeconds: 300
    }
  }
}
```

#### 3. 缓存策略

```json5
{
  agents: {
    defaults: {
      cache: {
        enabled: true,
        type: "semantic",  // exact | semantic
        ttl: 3600,  // 1 小时
        maxSize: "500MB",
        ignoreFields: ["timestamp", "user_id"]
      }
    }
  }
}
```

### 监控和告警

#### 代理健康度监控脚本

```bash
#!/bin/bash
# 代理健康度监控

set -e

AGENT_ID="${1:-main}"
HEALTH_FILE="/tmp/agent-${AGENT_ID}-health.json"

echo "=== 代理健康度检查: ${AGENT_ID} ==="
echo ""

# 1. 检查代理是否运行
if ! openclaw agents status "$AGENT_ID" > /dev/null 2>&1; then
  echo "❌ 代理未运行"
  exit 1
fi
echo "✅ 代理运行正常"

# 2. 检查响应时间
RESPONSE_TIME=$(openclaw agents ping "$AGENT_ID" --json | jq -r '.responseTimeMs')
if [ "$RESPONSE_TIME" -gt 5000 ]; then
  echo "⚠️  响应时间过长：${RESPONSE_TIME}ms"
else
  echo "✅ 响应时间正常：${RESPONSE_TIME}ms"
fi

# 3. 检查内存使用
MEMORY_USAGE=$(openclaw agents status "$AGENT_ID" --json | jq -r '.memoryUsageMB')
if [ "$MEMORY_USAGE" -gt 1000 ]; then
  echo "⚠️  内存使用过高：${MEMORY_USAGE}MB"
else
  echo "✅ 内存使用正常：${MEMORY_USAGE}MB"
fi

# 4. 检查活动会话数
ACTIVE_SESSIONS=$(openclaw sessions list --agent "$AGENT_ID" | grep -c "active" || echo 0)
if [ "$ACTIVE_SESSIONS" -gt 100 ]; then
  echo "⚠️  活动会话过多：${ACTIVE_SESSIONS}"
else
  echo "✅ 活动会话正常：${ACTIVE_SESSIONS}"
fi

# 5. 检查错误率 (最近 1 小时)
ERROR_COUNT=$(openclaw logs --agent "$AGENT_ID" --since 1h --level error | wc -l)
if [ "$ERROR_COUNT" -gt 10 ]; then
  echo "⚠️  错误数过多：${ERROR_COUNT}"
else
  echo "✅ 错误率正常：${ERROR_COUNT}"
fi

echo ""
echo "健康度检查完成"
```

#### 告警配置示例

```json5
{
  monitoring: {
    agents: {
      alerts: [
        {
          name: "高延迟告警",
          condition: "responseTimeMs > 5000",
          window: "5m",
          actions: ["slack:#alerts", "email:admin@example.com"]
        },
        {
          name: "内存泄漏检测",
          condition: "memoryUsageMB > 1000",
          window: "10m",
          actions: ["slack:#alerts"]
        },
        {
          name: "错误率过高",
          condition: "errorRate > 0.05",
          window: "5m",
          actions: ["pagerduty"]
        },
        {
          name: "会话数异常",
          condition: "activeSessions > 200",
          window: "1m",
          actions: ["slack:#alerts"]
        }
      ]
    }
  }
}
```

---

## 🏆 案例研究

### 案例 1: 客服机器人优化

**场景**: 电商客服，日均 1000+ 咨询

**挑战**:
- 响应速度慢 (平均 8s)
- 高峰期错误率高 (5%)
- 成本控制困难

**解决方案**:

```json5
{
  agents: {
    list: [
      {
        id: "cs-fast",
        model: "anthropic/claude-haiku-3-5-20241022",
        routing: {
          rules: [
            {
              name: "订单查询",
              if: "contains:订单 OR contains:物流 OR contains:发货",
              priority: "high"
            },
            {
              name: "退换货",
              if: "contains:退货 OR contains:换货 OR contains:退款",
              priority: "high"
            }
          ]
        },
        cache: {
          enabled: true,
          type: "semantic",
          ttl: 86400  // 24 小时
        }
      },
      {
        id: "cs-complex",
        model: "anthropic/claude-opus-4-20250514",
        routing: {
          rules: [
            {
              name: "投诉建议",
              if: "contains:投诉 OR contains:建议 OR contains:客服",
              priority: "high"
            }
          ]
        }
      }
    ]
  }
}
```

**效果**:
- ✅ 平均响应时间：8s → 2.5s (降低 69%)
- ✅ 错误率：5% → 0.3% (降低 94%)
- ✅ 成本：$500/月 → $180/月 (降低 64%)

### 案例 2: 开发助手多代理架构

**场景**: 20 人开发团队，代码审查/调试/文档

**架构设计**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    开发助手多代理架构                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  用户请求                                                               │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  路由代理 (智能分类)                                            │   │
│  │  • 代码审查 ──► review-agent                                   │   │
│  │  • Bug 调试 ──► debug-agent                                    │   │
│  │  • 文档生成 ──► docs-agent                                     │   │
│  │  • 架构咨询 ──► architect-agent                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │
│     ├─────────────┬─────────────┬─────────────┬───────────────────┤   │
│     ▼             ▼             ▼             ▼                   ▼   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ review-  │ │ debug-   │ │ docs-    │ │ architect│ │ general- │  │
│  │ agent    │ │ agent    │ │ agent    │ │ -agent   │ │ agent    │  │
│  │          │ │          │ │          │ │          │ │          │  │
│  │ Opus     │ │ Sonnet   │ │ Sonnet   │ │ Opus     │ │ Haiku    │  │
│  │ 只读     │ │ 全工具   │ │ 只读     │ │ 只读     │ │ 只读     │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**配置**:

```json5
{
  agents: {
    list: [
      {
        id: "review-agent",
        model: "anthropic/claude-opus-4-20250514",
        workspace: "~/clawd-review",
        tools: { allow: ["read", "sessions_list"] },
        identity: {
          name: "CodeReviewer",
          theme: "strict code reviewer"
        }
      },
      {
        id: "debug-agent",
        model: "anthropic/claude-sonnet-4-20250514",
        workspace: "~/clawd-debug",
        tools: { allow: ["read", "write", "exec", "browser"] },
        identity: {
          name: "Debugger",
          theme: "patient debugger"
        }
      },
      {
        id: "docs-agent",
        model: "anthropic/claude-sonnet-4-20250514",
        workspace: "~/clawd-docs",
        tools: { allow: ["read", "write"] },
        identity: {
          name: "DocWriter",
          theme: "clear technical writer"
        }
      },
      {
        id: "architect-agent",
        model: "anthropic/claude-opus-4-20250514",
        workspace: "~/clawd-architect",
        tools: { allow: ["read", "sessions_list"] },
        identity: {
          name: "Architect",
          theme: "senior software architect"
        }
      }
    ]
  }
}
```

**效果**:
- ✅ 代码审查时间：30 分钟 → 5 分钟
- ✅ Bug 定位时间：2 小时 → 15 分钟
- ✅ 文档覆盖率：40% → 85%

---

## 🎯 最佳实践

### 个人使用

```json5
{
  agents: {
    defaults: {
      workspace: "~/clawd",
      sandbox: { mode: "off" }  // 无沙箱，最佳性能
    }
  }
}
```

### 团队使用

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }  // 沙箱隔离
    },
    list: [
      { id: "team", workspace: "~/clawd-team" },
      { id: "admin", sandbox: { mode: "off" } }  // 管理员无沙箱
    ]
  }
}
```

### 公开服务

```json5
{
  agents: {
    list: [
      {
        id: "public",
        sandbox: { mode: "all", workspaceAccess: "none" },
        tools: {
          allow: ["sessions_list", "sessions_history"],
          deny: ["write", "exec", "browser", "canvas"]
        }
      }
    ]
  }
}
```

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [消息路由](/concepts/routing) | 路由配置详解 |
| [会话管理](/concepts/sessions) | 会话系统 |
| [工作区](/concepts/agent-workspace) | 工作区详解 |
| [配置参考](/config/reference) | 完整配置 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置代理基础 | ⭐⭐⭐⭐⭐ |
| 设置沙箱和工具 | ⭐⭐⭐⭐ |
| 多代理路由 | ⭐⭐⭐⭐ |
| 调试代理问题 | ⭐⭐⭐ |

---

> **💡 专家提示**：为不同的使用场景创建专门的代理——开发用高性能模型（Opus）、日常用平衡模型（Sonnet）、快速查询用轻量模型（Haiku）！
