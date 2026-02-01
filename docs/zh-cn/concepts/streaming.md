---
summary: "流式传输与分块机制详解：块回复、草稿流式、限制和配置"
read_when:
  - 理解流式传输如何工作
  - 配置块流式或渠道分块行为
  - 调试重复/提前块回复或草稿流式问题
title: "流式传输与分块"
---

# 📡 流式传输与分块

OpenClaw 有两层独立的"流式"机制：

1. **块流式（Block Streaming）** - 渠道层面：将完成的**块**作为助手写入的内容发送（非 token delta）
2. **类 Token 流式（仅 Telegram）** - 更新**草稿气泡**（Typing indicator）

**目前没有真正的 token 流式**发送到外部渠道消息。Telegram 草稿流式是唯一部分流式界面。

---

## 🎯 快速理解

| 机制 | 适用渠道 | 用户体验 | 技术实现 |
|------|---------|----------|----------|
| **块流式** | 所有渠道 | 多条消息逐条出现 | 分块发送 |
| **草稿流式** | 仅 Telegram | "正在输入"气泡实时更新 | Bot API Draft |
| **无流式** | 默认 | 等待完整回复后发送 | 单条消息 |

---

## 📦 块流式（Block Streaming）

### 什么是块流式？

将助手输出分成**粗粒度块**，在内容可用时逐块发送。

**类比**：
- 非流式：写完一整封信再寄出
- 块流式：写完一段就寄一段，用户能更快看到内容

### 工作原理

```
AI 模型输出
    │
    ├─ text_delta/events（模型流式事件）
    │
    ▼
┌─────────────────────┐
│   EmbeddedBlockChunker  │ 分块器
│  ├─ 缓冲文本           │
│  ├─ 检测分块边界        │
│  └─ 产生块            │
└─────────────────────┘
    │
    ▼
块产生 → 立即发送（text_end 模式）
    或
缓冲 → 消息结束时批量发送（message_end 模式）
    │
    ▼
渠道发送（块回复）
```

### 控制参数

```json5
{
  agents: {
    defaults: {
      // 1. 默认开关
      blockStreamingDefault: "on",  // "on" | "off"
      
      // 2. 渠道覆盖（强制开启/关闭）
      // channels.whatsapp.blockStreaming: true
      
      // 3. 分块边界
      blockStreamingBreak: "text_end",  // "text_end" | "message_end"
      
      // 4. 分块大小
      blockStreamingChunk: {
        minChars: 800,      // 最小字符数
        maxChars: 1200,     // 最大字符数
        breakPreference: "paragraph"  // 断点偏好
      },
      
      // 5. 合并设置
      blockStreamingCoalesce: {
        minChars: 100,
        maxChars: 1500,
        idleMs: 500         // 空闲等待时间
      }
    }
  },
  channels: {
    whatsapp: {
      // 6. 渠道硬性限制
      textChunkLimit: 4096,   // 单条消息上限
      chunkMode: "length"     // "length" | "newline"
    },
    discord: {
      // 7. Discord 软限制
      maxLinesPerMessage: 17  // 避免 UI 截断
    }
  }
}
```

### 边界语义详解

#### `text_end` 模式

- 块产生时**立即流式传输**
- 每个 `text_end` 事件触发发送
- 用户体验：消息逐条出现，最实时

```
AI: "这是一个很长的回复..."
        │
        ├─ [块1: 800 chars] → 立即发送
        ├─ [块2: 1000 chars] → 立即发送
        └─ [块3: 剩余] → 立即发送
```

#### `message_end` 模式

- 等待助手消息**完成**
- 然后刷新缓冲输出
- 如缓冲文本超过 `maxChars`，仍会产生多块

```
AI: "这是一个很长的回复..."
        │
        ▼
    [等待完成]
        │
        ▼
    缓冲 3000 chars
        │
        ├─ [块1: 1200 chars] → 发送
        ├─ [块2: 1200 chars] → 发送
        └─ [块3: 600 chars] → 发送
```

### 分块算法（低/高边界）

`EmbeddedBlockChunker` 实现：

1. **低边界**：缓冲 >= `minChars` 才发送（除非强制）
2. **高边界**：优先在 `maxChars` 前分割；如强制，在 `maxChars` 处分割
3. **断点偏好**（按优先级）：
   - `paragraph` → 段落边界（`\n\n`）
   - `newline` → 行边界（`\n`）
   - `sentence` → 句子边界（句号）
   - `whitespace` → 空白字符
   - hard break → 强制截断
4. **代码块保护**：不在围栏内分割，强制时在围栏处关闭+重新打开

`maxChars` 会被渠道 `textChunkLimit` 限制，不能超过渠道上限。

### 合并（Coalescing）

当块流式启用时，OpenClaw 可以**合并连续块**后再发送：

```
原始块：[A] [B] [C] [D]
        │
        ▼
    合并（idleMs 等待）
        │
        ▼
发送块：[A+B] [C+D]
```

**目的**：减少"单行垃圾"，同时保持渐进输出

**参数**：
- `idleMs`：空闲等待时间
- `maxChars`：缓冲上限（超限强制发送）
- `minChars`：最小发送大小（最终发送例外）

**默认调整**：
- Signal/Slack/Discord 默认 `minChars: 1500`（除非覆盖）

### 人性化延迟

块流式启用时，可添加**随机暂停**（第一块后）：

```json5
{
  agents: {
    defaults: {
      humanDelay: {
        mode: "natural",  // "off" | "natural" | "custom"
        // natural: 800-2500ms
        // custom: { minMs: 500, maxMs: 2000 }
      }
    }
  }
}
```

**效果**：多条消息间有自然停顿，更像人类打字

**仅适用于**：块回复（不适用于最终回复或工具摘要）

### 模式对比表

| 配置 | 行为 | 用户体验 |
|------|------|----------|
| `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` | 边生成边发送 | 消息逐条快速出现 |
| `blockStreamingBreak: "message_end"` | 完成后批量发送 | 等待后多条一起出现 |
| `blockStreamingDefault: "off"` | 仅最终回复 | 等待后单条完整消息 |

**渠道注意**：非 Telegram 渠道，块流式**默认关闭**，需显式 `*.blockStreaming: true` 开启。

---

## 💬 Telegram 草稿流式（类 Token）

### 什么是草稿流式？

Telegram 独有的功能：在消息输入框显示**实时更新的草稿**，而非发送真实消息。

**视觉效果**：
```
用户看到：
┌─────────────────────────┐
│ 🤖 Robby 正在输入...    │  ← 草稿气泡
│ "这是回复的第一部分..." │  ← 实时更新
└─────────────────────────┘
        ↓（完成时）
┌─────────────────────────┐
│ 这是回复的完整内容...    │  ← 正式消息
└─────────────────────────┘
```

### 配置

```json5
{
  channels: {
    telegram: {
      streamMode: "partial",  // "partial" | "block" | "off"
      
      // 仅 block 模式有效
      draftChunk: {
        minChars: 200,
        maxChars: 800
      }
    }
  }
}
```

### 模式详解

| 模式 | 行为 | 使用场景 |
|------|------|----------|
| **partial** | 草稿更新为最新流式文本 | 最实时体验 |
| **block** | 草稿按块更新（同 chunker 规则） | 较稳定体验 |
| **off** | 无草稿流式 | 传统体验 |

### 与块流式的关系

- **草稿流式**和**块流式**是独立的
- 草稿流式激活时，块流式被禁用（避免双重流式）
- 最终回复仍是正常消息发送

### 特殊功能：`/reasoning stream`

将推理过程写入草稿气泡（仅 Telegram）：

```
用户：解这个方程 x^2 + 5x + 6 = 0

草稿显示：
"让我解这个二次方程...
使用求根公式：x = (-b ± √(b²-4ac)) / 2a
这里 a=1, b=5, c=6
判别式 = 25 - 24 = 1
所以 x = (-5 ± 1) / 2
x₁ = -2, x₂ = -3"

正式消息：
"解为 x = -2 或 x = -3"
```

---

## 🔧 渠道特定配置

### WhatsApp

```json5
{
  channels: {
    whatsapp: {
      blockStreaming: true,       // 显式开启
      textChunkLimit: 4096,       // 消息上限
      chunkMode: "newline"        // 段落优先分割
    }
  }
}
```

### Discord

```json5
{
  channels: {
    discord: {
      blockStreaming: true,
      maxLinesPerMessage: 17,     // 避免 UI 截断
      textChunkLimit: 2000        // Discord 限制
    }
  }
}
```

### Slack

```json5
{
  channels: {
    slack: {
      blockStreaming: true,
      textChunkLimit: 4000
    }
  }
}
```

### 按账号配置

```json5
{
  channels: {
    whatsapp: {
      accounts: [
        {
          id: "personal",
          blockStreaming: true
        },
        {
          id: "work",
          blockStreaming: false  // 工作账号不流式
        }
      ]
    }
  }
}
```

---

## 🎓 最佳实践

### 选择合适的模式

| 场景 | 推荐配置 | 原因 |
|------|----------|------|
| **快速响应** | `text_end` + 短块 | 最小延迟 |
| **完整思考** | `message_end` | 逻辑完整呈现 |
| **长文生成** | 合并 + 人性化延迟 | 避免刷屏 |
| **代码分享** | `newline` 分块 | 保持代码块完整 |
| **移动优先** | 关闭流式 | 节省流量 |

### 性能考虑

**开启流式的成本**：
- ✅ 更快感知响应
- ❌ 更多 API 调用（每条消息）
- ❌ 复杂会话历史（更多消息记录）

**关闭流式的收益**：
- ✅ 简洁的会话历史
- ✅ 更少的网络请求
- ✅ 更好的移动端体验

### 调试流式问题

**问题：消息重复**
- 检查幂等性 key
- 查看 `blockStreamingCoalesce` 配置

**问题：消息顺序错乱**
- 网络延迟导致
- 考虑增加 `humanDelay`

**问题：块太小/太大**
- 调整 `minChars` 和 `maxChars`
- 修改 `breakPreference`

---

## 📊 配置位置提醒

所有 `blockStreaming*` 配置都在 `agents.defaults` 下，**不在**根配置：

```json5
{
  // ✅ 正确
  agents: {
    defaults: {
      blockStreamingDefault: "on"
    }
  }
  
  // ❌ 错误
  blockStreamingDefault: "on"
}
```

---

## 📚 相关文档

- [系统提示](/zh-CN/concepts/system-prompt) - 提示工程
- [会话管理](/zh-CN/concepts/session) - 会话生命周期
- [队列系统](/zh-CN/concepts/queue) - 消息队列
- [工具系统](/zh-CN/tools) - 工具调用

---

**流式传输是用户体验的关键 - 根据场景选择最适合的模式！** ⚡
