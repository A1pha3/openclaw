---
summary: "上下文管理详解 - 对话上下文、上下文窗口和上下文优化"
read_when:
  - 了解上下文如何工作
  - 配置上下文大小
  - 优化 Token 使用
title: "上下文管理"
---

# 🧠 上下文管理

本文档详细介绍 OpenClaw 的上下文管理系统，帮助您理解如何高效管理对话上下文。

## 🎯 什么是上下文？

**上下文（Context）** 是 AI 理解对话所需的历史信息，包括：

```
┌─────────────────────────────────────────────────────────┐
│                        上下文                           │
├─────────────────────────────────────────────────────────┤
│  系统提示词    │ AI 的角色、行为约束、工具说明          │
│  对话历史      │ 用户和 AI 之间的消息记录               │
│  工作区文件    │ 读取的代码、文档等内容                 │
│  用户信息      │ 关于用户的背景信息                     │
│  长期记忆      │ 跨会话记住的重要信息                   │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 上下文组成

### 系统提示词

```
┌─────────────────────────────────────────┐
│  系统提示词                              │
├─────────────────────────────────────────┤
│  你是 OpenClaw，一个 AI 助手。           │
│  - 你可以帮助用户完成各种任务            │
│  - 你可以读取和编辑文件                  │
│  - 你可以执行命令                        │
│  - 你必须遵守用户的安全规则              │
└─────────────────────────────────────────┘
```

### 对话历史

```
┌─────────────────────────────────────────┐
│  对话历史                                │
├─────────────────────────────────────────┤
│  [用户] 请帮我创建一个文件               │
│  [AI] 好的，我来创建文件。               │
│  [用户] 文件内容是什么？                 │
│  [AI] 文件包含以下内容：...              │
└─────────────────────────────────────────┘
```

### 工作区上下文

```
┌─────────────────────────────────────────┐
│  工作区上下文                            │
├─────────────────────────────────────────┤
│  已读取文件：                            │
│  - /workspace/src/index.ts              │
│  - /workspace/README.md                 │
│                                          │
│  已执行命令：                            │
│  - npm install                          │
│  - npm run build                        │
└─────────────────────────────────────────┘
```

---

## ⚙️ 上下文配置

### 基础配置

```json5
{
  messages: {
    context: {
      maxMessages: 50,         // 最大消息数
      maxTokens: 100000,       // 最大 Token 数
      includeSystem: true,     // 包含系统提示词
      includeHistory: true     // 包含对话历史
    }
  }
}
```

### 按渠道配置

```json5
{
  channels: {
    telegram: {
      historyLimit: 50,        // 群组历史
      dmHistoryLimit: 30,      // DM 历史
      contextLimit: 20000      // 上下文 Token 限制
    },
    whatsapp: {
      historyLimit: 20,
      contextLimit: 30000
    }
  }
}
```

### 工作区文件限制

```json5
{
  agents: {
    defaults: {
      bootstrapMaxChars: 20000,  // 工作区文件最大字符数
      skipBootstrap: false        // 跳过工作区初始化
    }
  }
}
```

---

## 🔄 上下文生命周期

```
1. 加载上下文
   │
   ├── 加载系统提示词
   ├── 加载对话历史
   ├── 加载工作区文件
   └── 加载用户信息
   │
   ▼
2. 准备上下文
   │
   ├── 裁剪过长的历史
   ├── 优化 Token 使用
   └── 格式化消息
   │
   ▼
3. 发送到 AI
   │
   └── AI 处理并生成响应
   │
   ▼
4. 更新上下文
   │
   ├── 添加新消息到历史
   └── 更新工作区状态
```

---

## ✂️ 上下文裁剪

### 自动裁剪

当上下文超过限制时，自动裁剪：

```json5
{
  messages: {
    context: {
      strategy: "summarize",    // summarize | oldest | relevant
      preserveRecent: 5,        // 保留最近 N 条消息
      summarizeThreshold: 0.8   // 超过 80% 时压缩
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `summarize` | 压缩旧消息为摘要 |
| `oldest` | 删除最旧的消息 |
| `relevant` | 保留相关消息 |

### 手动裁剪

```bash
# 查看上下文大小
openclaw sessions context <session-key>

# 手动压缩上下文
openclaw sessions compact <session-key>
```

---

## 🎯 Token 管理

### Token 计算

| 内容类型 | Approx. Tokens |
|----------|----------------|
| 1 个英文单词 | 0.75 |
| 1 个中文字符 | 1.5 |
| 1 行代码 | 10-15 |
| 1 个系统提示词 | 1000-3000 |

### 优化建议

```json5
{
  messages: {
    context: {
      maxTokens: 50000,  // 根据模型限制设置
      reserveTokens: 5000  // 为 AI 响应预留
    }
  }
}
```

### Token 统计

```bash
# 查看 Token 使用
openclaw status | grep tokens

# 查看会话 Token
openclaw sessions history <session-key> --tokens
```

---

## 💾 长期记忆

### 记忆文件

```
~/.openclaw/workspace/
├── MEMORY.md           # 长期记忆
└── SESSIONSUMMARY.md   # 会话摘要
```

### 记忆内容

```markdown
# 用户信息
- 名称：张三
- 偏好：喜欢简洁的回答
- 工作：软件工程师

# 项目背景
- 项目名称：OpenClaw
- 主要语言：TypeScript
- 代码库规模：大型

# 重要偏好
- 使用中文交流
- 代码需要详细注释
```

### 自动记忆

```json5
{
  memory: {
    enabled: true,
    autoSummary: true,      // 自动生成摘要
    important提取: true,    // 提取重要信息
    maxMemoryChars: 10000   // 记忆最大字符数
  }
}
```

---

## 📈 上下文优化策略

### 高效提示词

```markdown
<!-- 避免 -->
你是一个有用的助手，请帮助用户完成各种任务。

<!-- 推荐 -->
你是 OpenClaw，一个 AI 编程助手。
- 擅长：代码审查、调试、架构设计
- 风格：简洁、准确、注重可维护性
- 约束：不写冗余代码，不跳过安全检查
```

### 历史消息精简

```json5
{
  messages: {
    context: {
      includeActions: true,    // 仅包含操作结果
      skipDuplicates: true,    // 跳过重复内容
      compressCode: true       // 压缩代码片段
    }
  }
}
```

### 会话分离

```json5
{
  session: {
    mainKey: "main",       // DM 合并到主会话
    groupIsolation: true   // 群组会话独立
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw sessions list` | 列出会话 |
| `openclaw sessions history` | 查看会话历史 |
| `openclaw sessions compact` | 压缩会话上下文 |
| `openclaw memory` | 管理记忆 |
| `openclaw status` | 查看上下文使用 |

---

## 📚 相关文档

- [会话管理](/zh-CN/concepts/sessions) - 会话系统
- [消息系统](/zh-CN/concepts/messages) - 消息处理
- [记忆系统](/zh-CN/concepts/memory) - 长期记忆
- [系统提示词](/zh-CN/concepts/system-prompt) - 提示词配置

---

**合理的上下文管理，让 AI 助手既聪明又高效！** 🦞
