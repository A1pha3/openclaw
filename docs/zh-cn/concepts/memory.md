---
summary: "记忆系统详解：工作区文件 + 自动记忆刷新 + 向量搜索"
read_when:
  - 想要了解 OpenClaw 记忆如何工作
  - 配置文件布局和自动刷新机制
  - 设置语义记忆搜索
title: "记忆系统"
---

# 🧠 记忆系统

OpenClaw 的记忆是**纯 Markdown 文件**，存放在 Agent 工作区。这些文件是真相的来源；模型只"记住"被写入磁盘的内容。

---

## 🎯 核心概念

### 记忆 ≠ 上下文

| 概念 | 存储位置 | 持续时间 | 用途 |
|------|----------|----------|------|
| **上下文（Context）** | 内存 | 当前会话 | AI 模型当前看到的 |
| **短期记忆** | 会话历史 | 直到修剪 | 近期对话 |
| **长期记忆** | `memory/` + `MEMORY.md` | 永久 | 持久化知识 |

**关键区别**：
- **上下文** = AI 当前窗口中的内容（有限，会遗忘）
- **记忆** = 写入文件的内容（持久，可搜索）

### 记忆工作流程

```
重要信息/决策
      │
      ▼
┌─────────────┐
│  写入文件   │ ─→ `memory/YYYY-MM-DD.md` 或 `MEMORY.md`
└─────────────┘
      │
      ▼
┌─────────────┐
│  索引构建   │ ─→ 向量数据库（用于语义搜索）
└─────────────┘
      │
      ▼
┌─────────────┐
│  按需加载   │ ─→ Agent 需要时搜索/读取
└─────────────┘
```

---

## 📁 记忆文件布局

### 两层记忆架构

**1. 每日日志（`memory/YYYY-MM-DD.md`）**

- **用途**：日常记录（追加写入）
- **读取**：会话启动时读取今天 + 昨天
- **内容**：琐碎信息、临时上下文、每日摘要

**2. 长期记忆（`MEMORY.md`）**

- **用途**：精选的长期记忆
- **读取**：**仅主会话**（私密，不共享）
- **内容**：决策、偏好、持久事实、个人背景

**为什么只加载到主会话？**

安全考虑 - `MEMORY.md` 可能包含：
- 个人身份信息
- 敏感决策
- 私人偏好

不应在 Discord 群组或与他人共享的会话中暴露。

### 完整文件结构

```
~/.openclaw/workspace/
├── memory/                      # 每日日志目录
│   ├── 2025-01-15.md           # 旧日志（可归档）
│   ├── 2025-01-16.md
│   ├── 2025-01-17.md
│   ├── 2025-01-18.md           # 昨天（自动读取）
│   └── 2025-01-19.md           # 今天（自动读取）
│
├── MEMORY.md                    # 长期记忆（精选）
│   # 个人偏好
│   # 重要决策
│   # 持久知识
│   # 项目背景
│
└── ... 其他工作区文件
```

---

## ✍️ 何时写入记忆

### 应该写入的内容

| 类型 | 示例 | 存储位置 |
|------|------|----------|
| **决策** | "使用 React 而非 Vue" | `MEMORY.md` |
| **偏好** | "喜欢用中文回复" | `MEMORY.md` |
| **持久事实** | "项目使用 TypeScript" | `MEMORY.md` |
| **个人信息** | "我是软件工程师" | `MEMORY.md` |
| **每日琐事** | "今天解决了 bug X" | `memory/YYYY-MM-DD.md` |
| **运行上下文** | "正在调试登录功能" | `memory/YYYY-MM-DD.md` |

### 写入方式

**显式请求**：
```
用户：记住我喜欢用 2 空格缩进
Agent：已记录到 MEMORY.md
```

**自动提取**：
```
用户：顺便说一下，我叫张三
Agent：[自动写入 USER.md 和 MEMORY.md]
```

**提示词引导**：
可以提示 Agent 主动存储记忆：
```
如果有任何重要信息，请写入记忆文件
```

---

## 🔄 自动记忆刷新（预压缩触发）

### 什么是自动刷新？

当会话**接近自动压缩**时，OpenClaw 触发一个**静默的 Agentic 回合**，提醒模型在上下文被压缩前写入持久记忆。

### 工作原理

```
会话使用 80% 上下文窗口
        │
        ▼
┌───────────────┐
│ 触发记忆刷新 │
│ （静默回合） │
└───────────────┘
        │
        ▼
Agent 检查是否有持久化记忆
        │
        ├─ 有 → 写入 memory/YYYY-MM-DD.md
        │
        └─ 无 → 回复 NO_REPLY（用户不可见）
        │
        ▼
继续正常会话
```

### 配置

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,    // 保留 20k tokens
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,  // 软阈值
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store."
        }
      }
    }
  }
}
```

### 关键细节

| 配置项 | 说明 |
|--------|------|
| **软阈值** | `contextWindow - reserveTokensFloor - softThresholdTokens` |
| **静默** | 默认 `NO_REPLY`，用户看不到这个回合 |
| **双提示** | 用户提示 + 系统提示追加 |
| **每周期一次** | 每次压缩周期只刷新一次 |
| **可写工作区** | 如果 `workspaceAccess: "ro"` 或 `"none"` 则跳过 |

---

## 🔍 向量记忆搜索

### 什么是语义搜索？

传统搜索（关键词匹配）：
- 搜索"苹果"
- 只能找到包含"苹果"这个词的内容

语义搜索（向量相似度）：
- 搜索"苹果"
- 能找到"水果"、"iPhone"、"红色圆形食物"等相关内容
- **理解含义，不只是匹配文字**

### 技术原理

```
文本 → Embedding 模型 → 向量（768/1536 维）
                                        ↓
搜索查询 → Embedding 模型 → 查询向量
                                        ↓
                              相似度计算（余弦相似度）
                                        ↓
                              返回最相关的记忆
```

### 默认配置

- **默认启用**：是
- **监听文件变化**：是（防抖处理）
- **远程 embedding**：默认（OpenAI 或 Gemini）
- **本地 embedding**：可选（node-llama-cpp）
- **加速**：使用 sqlite-vec（可用时）

### 提供商选择优先级

如果没有设置 `memorySearch.provider`，自动选择：

1. **local**：如果配置了 `memorySearch.local.modelPath` 且文件存在
2. **openai**：如果能解析到 OpenAI key
3. **gemini**：如果能解析到 Gemini key
4. **禁用**：直到配置完成

### 远程 Embedding

**需要 API Key**：
- OpenAI：`OPENAI_API_KEY`
- Gemini：`GEMINI_API_KEY` 或 `models.providers.google.apiKey`

**注意**：Codex OAuth **不覆盖** embedding，需要单独的 API key。

### 本地 Embedding

使用 node-llama-cpp，可能需要：
```bash
pnpm approve-builds
```

配置：
```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "local",
        local: {
          modelPath: "~/models/all-MiniLM-L6-v2.gguf"
        }
      }
    }
  }
}
```

### Gemini Embedding

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "gemini",
        model: "gemini-embedding-001",
        remote: {
          apiKey: "YOUR_GEMINI_API_KEY"
        }
      }
    }
  }
}
```

### 自定义 OpenAI 兼容端点

（OpenRouter, vLLM, 代理）

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai",
        model: "text-embedding-3-small",
        remote: {
          baseUrl: "https://api.example.com/v1/",
          apiKey: "YOUR_API_KEY",
          headers: {
            "X-Custom-Header": "value"
          }
        }
      }
    }
  }
}
```

### 备用提供商

如果主提供商失败，可使用备用：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai",
        fallback: "gemini"  // 或 "local", "none"
      }
    }
  }
}
```

### 批量索引

OpenAI 和 Gemini 支持批量 API：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai",
        remote: {
          batch: {
            enabled: true,          // 默认启用
            wait: true,             // 等待完成
            pollIntervalMs: 5000,   // 轮询间隔
            timeoutMinutes: 30,     // 超时
            concurrency: 2          // 并行作业数
          }
        }
      }
    }
  }
}
```

**为什么 OpenAI 批量又快又便宜？**

- 一次提交多个 embedding 请求
- OpenAI 异步处理
- 批量 API 有折扣价格
- [OpenAI Batch API 文档](https://platform.openai.com/docs/api-reference/batch)

---

## 📂 额外记忆路径

索引工作区外的 Markdown 文件：

```json5
{
  agents: {
    defaults: {
      memorySearch: {
        extraPaths: [
          "../team-docs",           // 相对工作区
          "/srv/shared-notes/overview.md"  // 绝对路径
        ]
      }
    }
  }
}
```

**规则**：
- 支持绝对和相对路径
- 目录递归扫描 `.md` 文件
- 仅索引 Markdown
- 忽略符号链接

---

## 🎓 最佳实践

### 个人使用

```
MEMORY.md
├── 我是谁（基本信息）
├── 我的偏好（沟通风格、技术栈）
├── 重要决策（项目选择、架构决策）
└── 持久知识（常用命令、项目背景）

memory/YYYY-MM-DD.md
├── 每日摘要
├── 遇到的问题
├── 解决方案
└── 临时上下文
```

### 团队使用

```
# 个人工作区 MEMORY.md
个人偏好、学习笔记

# 团队共享（通过 extraPaths）
../team-docs/
├── 项目架构.md
├── 开发规范.md
└── API 文档.md
```

### 记忆维护

**定期回顾**（每周/每月）：
1. 回顾 `memory/*.md` 文件
2. 提取重要信息到 `MEMORY.md`
3. 删除过时信息
4. 归档旧日志（移动到 `memory/archive/`）

---

## 🔧 故障排除

### 记忆搜索不工作

**检查清单**：
1. 是否启用？`memorySearch.enabled`
2. 提供商配置正确？API key 有效？
3. 文件是否有内容？空文件不索引
4. 查看日志：`openclaw logs --follow`

### 向量索引很慢

**优化**：
- 使用本地 embedding（无网络延迟）
- 启用 sqlite-vec 加速
- 减少 `extraPaths` 中的大目录
- 使用批量索引（OpenAI/Gemini）

### 记忆文件太大

**管理**：
- 归档旧日志：`mv memory/2024-* memory/archive/`
- 定期压缩 `MEMORY.md`
- 删除不再需要的信息

---

## 📚 相关文档

- [上下文管理](/zh-CN/concepts/context) - 理解上下文窗口
- [会话管理](/zh-CN/concepts/session) - 会话生命周期
- [压缩机制](/zh-CN/concepts/compaction) - 自动压缩
- [系统提示](/zh-CN/concepts/system-prompt) - 提示工程
- [参考：会话管理 + 压缩](/zh-CN/reference/session-management-compaction)

---

**好记性不如烂笔头 - 让 Agent 帮你记住一切！** 📝
