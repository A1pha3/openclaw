---
summary: "AI 代理运行时详解：pi-mono 基础、工作区契约和会话引导"
read_when:
  - 想要了解 Agent 如何工作
  - 配置代理运行时、工作区引导或会话行为
  - 理解引导文件（AGENTS.md, SOUL.md 等）的作用
title: "AI 代理运行时"
---

# 🤖 AI 代理运行时

OpenClaw 运行一个**嵌入式代理运行时**，基于 **pi-mono** 代码库衍生而来。

---

## 🎯 一句话理解

Agent 是 OpenClaw 的**大脑**，它接收消息、思考、调用 AI 模型、执行工具，然后回复。就像你通过 WhatsApp 聊天的"虚拟助手"。

---

## 🏗️ 架构位置

```
┌─────────────────────────────────────────┐
│           用户消息（WhatsApp/Telegram）   │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│              Gateway 网关               │
│  ├─ 接收消息                            │
│  ├─ 路由到 Agent                        │
│  ├─ 管理 WebSocket 连接                  │
│  └─ 发送回复                            │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│           🤖 Agent 运行时               │
│  ├─ 加载工作区文件（AGENTS.md等）        │
│  ├─ 构建上下文（系统提示+历史）          │
│  ├─ 调用 AI 模型                        │
│  ├─ 执行工具（read/exec/edit等）        │
│  ├─ 流式传输回复                        │
│  └─ 保存会话历史                        │
└─────────────────────────────────────────┘
```

---

## 📁 工作区（Workspace）- 必须的

### 什么是工作区？

工作区是 Agent 的**唯一工作目录**（`cwd`），所有工具操作都在这个目录下进行。

**默认位置**：`~/.openclaw/workspace`

**为什么重要？**
- **安全性**：Agent 不能随意访问整个文件系统
- **组织性**：所有项目文件集中管理
- **持久性**：重启后配置和记忆还在

### 工作区结构

```
~/.openclaw/workspace/
├── AGENTS.md           # 操作指南 + 记忆
├── SOUL.md            # 个性、边界、语气
├── TOOLS.md           # 工具使用说明
├── IDENTITY.md        # Agent 名称/氛围/表情
├── USER.md            # 用户信息 + 称呼偏好
├── MEMORY.md          # 长期记忆（仅主会话加载）
├── HEARTBEAT.md       # 心跳任务清单
├── memory/            # 每日记忆日志
│   ├── 2025-01-20.md
│   ├── 2025-01-21.md
│   └── ...
└── skills/            # 本地技能
    └── ...
```

**创建工作区：**
```bash
openclaw setup
```

这会创建 `~/.openclaw/openclaw.json`（如缺失）并初始化工作区文件。

### 沙箱模式下的工作区

如果启用了 `agents.defaults.sandbox`，非主会话可以使用隔离的工作区：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        workspaceRoot: "~/.openclaw/sandbox-workspaces"
      }
    }
  }
}
```

每个沙箱会话获得独立子目录。

---

## 📄 引导文件（Bootstrap Files）- 注入式配置

### 什么是引导文件？

这些是**用户可编辑的 Markdown 文件**，在会话的第一回合被**注入到 Agent 上下文中**，告诉 Agent 它是谁、如何行事、记住什么。

**为什么用文件而非配置？**
- **可编辑**：随时修改，即时生效
- **版本控制**：可用 git 管理
- **透明**：你知道 Agent 知道什么
- **灵活**：可包含任意指令

### 引导文件清单

| 文件 | 用途 | 加载时机 | 安全级别 |
|------|------|----------|----------|
| **AGENTS.md** | 操作指南 + 记忆 | 所有会话 | 公开 |
| **SOUL.md** | 个性、边界、语气 | 所有会话 | 公开 |
| **TOOLS.md** | 工具使用说明 | 所有会话 | 公开 |
| **IDENTITY.md** | Agent 名称/表情 | 所有会话 | 公开 |
| **USER.md** | 用户档案 | 所有会话 | 半公开 |
| **HEARTBEAT.md** | 心跳任务 | 所有会话 | 公开 |
| **BOOTSTRAP.md** | 首次引导仪式 | 仅首次 | 一次性 |
| **MEMORY.md** | 长期记忆 | **仅主会话** | 私密 |

### MEMORY.md 安全警告

**⚠️ 重要**：`MEMORY.md` **只在主会话（直接与你聊天的会话）中加载**，绝不会在共享上下文（Discord 群组、与其他人的会话）中加载。

**原因**：安全 - 包含个人上下文，不应泄露给陌生人。

### 文件处理逻辑

**空文件**：跳过（不注入任何内容）

**大文件**：自动截断，保持提示精简
- 默认限制：`agents.defaults.bootstrapMaxChars = 20000`
- 截断标记：添加 "[truncated]" 标记

**缺失文件**：注入单行 "missing file" 标记
- 可用 `openclaw setup` 创建默认模板

**BOOTSTRAP.md 特殊逻辑**：
- 仅当工作区全新（无其他引导文件）时创建
- 完成仪式后可删除
- 不会重新创建

### 禁用引导文件创建

对于预配置的工作区：

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

---

## 🛠️ 内置工具

### 核心工具

这些工具**始终可用**（受工具策略限制）：

| 工具 | 用途 | 示例 |
|------|------|------|
| **read** | 读取文件 | `read file.txt` |
| **write** | 写入文件 | `write file.txt "内容"` |
| **edit** | 编辑文件 | `edit file.txt "旧" "新"` |
| **exec** | 执行命令 | `exec ls -la` |
| **process** | 进程管理 | `process list` |
| **browser** | 浏览器控制 | `browser navigate https://...` |
| **message** | 发送消息 | `message send --target ...` |

### 可选工具

- **apply_patch**：应用代码补丁
  - 由 `tools.exec.applyPatch` 控制
  - TOOLS.md **不**控制哪些工具存在，只提供使用建议

---

## 🧩 技能系统

### 技能加载路径（优先级从高到低）

1. **工作区技能**：`<workspace>/skills/`
2. **托管/本地技能**：`~/.openclaw/skills/`
3. **捆绑技能**：随安装提供

**名称冲突**：工作区 wins（覆盖）

### 技能配置

技能可通过配置/环境变量控制：

```json5
{
  skills: {
    web: {
      enabled: true,
      config: {
        apiKey: "..."
      }
    }
  }
}
```

详见 [技能配置](/zh-CN/tools/skills-config)。

---

## 🔌 pi-mono 集成

### 关系说明

OpenClaw **重用** pi-mono 的部分代码：
- ✅ 模型调用
- ✅ 工具定义

但**自己实现**：
- ❌ 会话管理
- ❌ 服务发现
- ❌ 工具连接

**不存在**：
- ❌ pi-coding 代理运行时
- ❌ `~/.pi/agent` 设置
- ❌ `<workspace>/.pi` 配置

### 为什么分叉？

pi-mono 是一个通用的 AI 编码代理，OpenClaw 需要：
- 多渠道支持（WhatsApp/Telegram/Discord）
- 中心化网关架构
- 不同的会话管理模型
- 更灵活的工具系统

---

## 💾 会话存储

### 存储位置

会话记录（JSONL 格式）：
```
~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl
```

**会话 ID**：由 OpenClaw 生成，保持稳定

**不读取**：
- ❌ 旧版 Pi/Tau 会话文件夹
- ❌ 其他 Agent 的会话

### 会话格式

每行一个 JSON 对象：

```json
{"role": "user", "content": "你好", "timestamp": "2025-01-20T10:00:00Z"}
{"role": "assistant", "content": "你好！有什么我可以帮助你的吗？", "timestamp": "2025-01-20T10:00:05Z"}
{"role": "tool", "name": "read", "result": "...", "timestamp": "2025-01-20T10:00:10Z"}
```

---

## 🎮 流式传输控制

### 队列模式（Queue Mode）

当用户在 AI 正在回复时发送新消息：

**steer 模式**（默认）：
1. 新消息注入当前运行
2. 队列在每个**工具调用后**检查
3. 如有队列消息，跳过当前助手的剩余工具调用
4. 错误工具结果："Skipped due to queued user message."
5. 队列用户消息注入到下一次助手响应前

**followup/collect 模式**：
- 入站消息等待当前回合结束
- 然后新 Agent 回合开始，携带队列载荷

详见 [队列](/zh-CN/concepts/queue)。

### 块流式传输（Block Streaming）

将助手输出分块发送（非 Telegram 渠道默认关闭）：

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "on",     // 默认开启
      blockStreamingBreak: "text_end", // text_end 或 message_end
      blockStreamingChunk: {
        minChars: 800,
        maxChars: 1200,
        breakPreference: "paragraph"   // paragraph | newline | sentence
      },
      blockStreamingCoalesce: {
        minChars: 100,
        maxChars: 1500,
        idleMs: 500
      }
    }
  }
}
```

**边界语义**：
- `text_end`：块产生时立即流式传输
- `message_end`：等待助手消息完成，然后刷新缓冲输出

**通道覆盖**：
非 Telegram 渠道需要显式 `*.blockStreaming: true` 开启。

详见 [流式传输](/zh-CN/concepts/streaming)。

---

## 🔧 模型引用格式

### 格式规范

模型引用在第一个 `/` 处分割：

```
<provider>/<model-name>
```

**示例**：
- `anthropic/claude-sonnet-4-20250514`
- `openai/gpt-4o`
- `openrouter/moonshotai/kimi-k2`（OpenRouter 风格）

**省略 provider**：
- 视为别名或**默认提供商**的模型
- 仅当模型 ID 不含 `/` 时有效

### 配置示例

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

---

## ⚙️ 最小配置

至少设置：

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace"
    }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"]  // 强烈推荐
    }
  }
}
```

---

## 🧠 理解 Agent 生命周期

```
1. 初始化
   ├─ 加载配置
   ├─ 初始化工作区
   ├─ 注册工具
   └─ 加载 OAuth/API 密钥

2. 消息处理（每消息）
   ├─ 路由到代理
   ├─ 加载/创建会话
   ├─ 准备上下文
   │   ├─ 注入引导文件
   │   ├─ 加载会话历史
   │   └─ 构建系统提示
   ├─ 调用 AI 模型
   ├─ 执行工具（如有）
   │   ├─ 解析工具调用
   │   ├─ 执行
   │   └─ 返回结果
   ├─ 生成最终回复
   └─ 保存会话历史

3. 清理（定期）
   ├─ 会话自动保存
   ├─ 沙箱容器清理
   └─ 旧会话归档
```

---

## 📚 相关文档

- [会话管理](/zh-CN/concepts/session) - 会话系统详解
- [记忆系统](/zh-CN/concepts/memory) - 记忆如何工作
- [工作区详解](/zh-CN/concepts/agent-workspace) - 完整工作区布局
- [流式传输](/zh-CN/concepts/streaming) - 流式回复机制
- [工具系统](/zh-CN/tools) - 所有可用工具
- [技能系统](/zh-CN/tools/skills) - 技能开发和使用

---

**Agent 是 OpenClaw 的大脑，理解它如何工作，你就能更好地驾驭这个系统！** 🦞
