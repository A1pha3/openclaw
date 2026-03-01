---
summary: "钩子系统完整指南——OpenClaw 事件驱动的自动化框架，通过 HOOK.md 配置和 handler.ts 实现自定义自动化行为"
read_when:
  - 理解钩子系统的工作原理和事件驱动架构
  - 创建自定义钩子处理特定事件
  - 开发复杂的自动化工作流
  - 将钩子打包为插件或独立分发
title: "钩子（Hooks）"
---

# 🔧 钩子（Hooks）

钩子是 OpenClaw 的**事件驱动自动化框架**，它们允许你在特定事件发生时执行自定义操作。想象一下这样的场景：当收到包含"紧急"关键词的消息时自动发送通知；当代理开始或结束运行时记录日志；当消息发送失败时自动重试或告警——这些都是钩子系统能够实现的功能。通过钩子，你可以将 OpenClaw 从一个被动的 AI 助手转变为一个能够主动响应和自动化处理任务的智能系统。

> **钩子与自动化的区别**
>
> 钩子系统和自动化系统（Automation）在定位上有所不同：自动化系统更适合处理定时任务和复杂的工作流程，强调"在什么时间做什么事"；而钩子系统更适合处理事件驱动的即时响应，强调"当某事发生时立即响应"。两者可以配合使用——自动化中的步骤可以触发钩子，钩子可以调用自动化脚本。理解这个区别有助于你在合适的场景选择合适的工具。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解钩子系统的核心概念和事件模型
- 掌握 HOOK.md 配置文件的基本语法
- 创建简单的消息处理钩子
- 使用模板变量访问上下文数据

### 进阶目标（建议掌握）

- 开发复杂的多条件钩子规则
- 实现与外部系统的 HTTP 集成
- 处理错误和实现重试逻辑
- 优化钩子性能和资源使用

### 专家目标（挑战）

- 开发自定义钩子类型
- 将钩子打包为可分发的插件
- 设计企业级的钩子审计系统
- 实现跨实例的钩子同步

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速上手** | 首次使用钩子 | 30 分钟 | 创建第一个钩子 |
| **日常使用** | 需要自动化响应 | 1 小时 | 常用钩子类型和配置 |
| **高级开发** | 需要复杂逻辑 | 2 小时 | handler.ts 和条件处理 |
| **专家扩展** | 开发钩子系统 | 4 小时 | 自定义钩子类型和插件 |

---

## 🧠 原理解释：钩子系统架构

### 事件驱动架构

钩子系统基于**发布-订阅（Publish-Subscribe）**模式实现。当系统中发生某个事件时（比如收到消息），事件会被发布到事件总线，然后所有订阅了该事件的钩子都会被触发执行。

**事件流转流程**：

```
┌─────────────────────────────────────────────────────────────────┐
│                       事件发生                                  │
│              例如：用户发送了一条消息                             │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                       事件总线                                  │
│         接收事件 → 验证事件类型 → 分发给订阅者                   │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                     钩子匹配引擎                                 │
│         遍历所有钩子 → 匹配触发条件 → 确定执行顺序                 │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                       执行引擎                                  │
│         加载钩子配置 → 执行动作 → 处理结果 → 记录日志             │
└─────────────────────────────────────────────────────────────────┘
```

### 钩子类型分类

钩子系统支持多种类型的事件触发：

| 类别 | 事件类型 | 触发时机 | 典型用途 |
|------|----------|----------|----------|
| **消息事件** | messageReceived | 收到新消息时 | 自动回复、内容过滤 |
| | messageSent | 消息发送后 | 发送确认、通知 |
| | messageFailed | 消息发送失败时 | 自动重试、告警 |
| **代理事件** | agentRunStart | 代理开始处理时 | 记录日志、初始化 |
| | agentRunEnd | 代理处理完成时 | 统计耗时、清理资源 |
| | agentError | 代理出错时 | 错误通知、恢复 |
| **会话事件** | sessionCreated | 创建新会话时 | 初始化上下文 |
| | sessionEnded | 会话结束时 | 保存记录、清理 |
| **系统事件** | scheduled | 定时触发时 | 定时任务 |
| | error | 发生错误时 | 错误处理、告警 |
| | startup | Gateway 启动时 | 初始化配置 |
| | shutdown | Gateway 关闭时 | 保存状态 |

### 钩子执行上下文

每个钩子执行时都会获得一个**上下文对象**，包含与事件相关的所有信息：

**消息事件上下文**：

| 变量 | 类型 | 说明 |
|------|------|------|
| `message.id` | string | 消息唯一 ID |
| `message.text` | string | 消息文本内容 |
| `message.senderId` | string | 发送者标识 |
| `message.channel` | string | 来源渠道类型 |
| `message.chatId` | string | 聊天会话 ID |
| `message.mentions` | string[] | 提及的用户列表 |
| `message.timestamp` | Date | 消息时间戳 |

**代理事件上下文**：

| 变量 | 类型 | 说明 |
|------|------|------|
| `agent.id` | string | 代理 ID |
| `agent.name` | string | 代理名称 |
| `session.id` | string | 会话 ID |
| `session.key` | string | 会话键 |
| `trigger` | string | 触发方式 |

---

## 🚀 快速开始

### 创建第一个钩子

让我们创建一个简单的自动回复钩子，当收到包含"hello"的消息时自动回复：

**步骤一：创建钩子目录结构**

```bash
# 创建钩子目录
mkdir -p ~/.openclaw/hooks/hello-world

# 创建配置文件
touch ~/.openclaw/hooks/hello-world/HOOK.md
```

**步骤二：编写 HOOK.md 配置**

```markdown
# ~/.openclaw/hooks/hello-world/HOOK.md

name: hello-world
description: 当收到 hello 消息时自动回复

# 触发条件
on:
  messageReceived: true

# 执行动作
then:
  - type: if
    condition: "{{message.text}}" contains "hello"
    then:
      - type: sendText
        text: "👋 你好！我是 OpenClaw AI 助手。有什么可以帮助你的吗？"
```

**步骤三：测试钩子**

向你的 OpenClaw 发送一条包含"hello"的消息，你应该会收到自动回复。

### 验证钩子是否生效

```bash
# 列出所有已安装的钩子
openclaw hooks list

# 查看钩子状态
openclaw hooks status hello-world

# 查看钩子执行日志
openclaw hooks logs --follow

# 启用/禁用钩子
openclaw hooks enable hello-world
openclaw hooks disable hello-world
```

---

## 📁 钩子目录结构

### 存放位置

钩子可以存放在以下位置，OpenClaw 会按优先级扫描：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | `~/.openclaw/hooks/` | 用户全局钩子目录 |
| 2 | `<workspace>/.openclaw/hooks/` | 工作区钩子目录 |
| 3 | `~/.openclaw/extensions/*/hooks/` | 插件内置钩子 |

### 标准目录结构

```
my-hook/
├── HOOK.md           # 钩子配置（必需）
├── handler.ts        # 处理函数（可选，复杂逻辑使用）
├── handler.js        # JavaScript 处理函数（可选）
├── package.json      # 依赖声明（可选）
├── README.md         # 钩子文档
└── assets/           # 静态资源（可选）
```

---

## ⚙️ HOOK.md 配置详解

### 基本结构

```markdown
# 钩子元数据
name: my-hook
description: 我的自定义钩子
version: 1.0.0
author: Your Name

# 触发条件
on:
  eventType: conditions

# 执行逻辑
then:
  - type: action1
    ...
```

### 触发条件（on）

**简单触发**（无条件触发）：

```markdown
on:
  messageReceived: true
```

**带条件的触发**：

```markdown
on:
  messageReceived:
    # 必须匹配的条件
    text:
      - pattern: "hello"
        type: contains
    senderId:
      - value: "allowed-user"
```

**条件操作符**：

| 操作符 | 说明 | 示例 |
|--------|------|------|
| `contains` | 包含文本 | `text contains "hello"` |
| `matches` | 正则匹配 | `text matches "^error:"` |
| `equals` | 完全匹配 | `channel equals "telegram"` |
| `startsWith` | 前缀匹配 | `senderId startsWith "admin:"` |
| `in` | 在列表中 | `channel in ["telegram", "slack"]` |

### 执行动作（then）

#### 条件执行

```markdown
then:
  - type: if
    condition: "{{message.text}}" contains "error"
    then:
      - type: sendText
        text: "检测到错误报告"
    else:
      - type: sendText
        text: "收到你的消息"
```

#### 发送消息

```markdown
then:
  - type: sendText
    text: "自动回复消息"
    
  - type: sendText
    channel: "telegram"
    chatId: "123456"
    text: "指定渠道消息"
```

#### 调用工具

```markdown
then:
  - type: callTool
    name: "bash"
    args:
      command: echo "Hello from hook"
      
  - type: callTool
    name: "web_search"
    args:
      query: "{{message.text}}"
```

#### HTTP 请求

```markdown
then:
  - type: httpRequest
    method: POST
    url: "https://api.example.com/webhook"
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{config.apiKey}}"
    body:
      event: "message"
      text: "{{message.text}}"
```

#### 延迟执行

```markdown
then:
  - type: delay
    duration: 5s  # 支持 s(秒)、m(分)、h(时)
    
  - type: sendText
    text: "5秒后发送的消息"
```

#### 循环执行

```markdown
then:
  - type: forEach
    items: "{{message.mentions}}"
    itemVar: "mention"
    do:
      - type: sendText
        text: "Mentioned: {{mention}}"
```

---

## 📊 模板变量参考

### 消息相关变量

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `{{message.id}}` | 消息 ID | "msg_12345" |
| `{{message.text}}` | 消息文本 | "Hello World" |
| `{{message.senderId}}` | 发送者 ID | "user_67890" |
| `{{message.channel}}` | 渠道类型 | "telegram" |
| `{{message.chatId}}` | 聊天 ID | "chat_11111" |
| `{{message.mentions}}` | 提及列表 | ["@bot"] |
| `{{message.attachments}}` | 附件列表 | [] |
| `{{message.timestamp}}` | 时间戳 | "2026-02-05T10:00:00Z" |

### 会话相关变量

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `{{session.id}}` | 会话 ID | "sess_abc" |
| `{{session.key}}` | 会话键 | "main" |
| `{{session.agentId}}` | 代理 ID | "agent_default" |

### 代理相关变量

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `{{agent.id}}` | 代理 ID | "default" |
| `{{agent.name}}` | 代理名称 | "Main Agent" |

### 系统变量

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `{{timestamp}}` | 当前时间戳 | ISO 8601 格式 |
| `{{randomId}}` | 随机 ID | "abc123" |
| `{{config.*}}` | 配置值 | `{{config.apiKey}}` |

### 工具输出变量

```markdown
then:
  - type: callTool
    name: "bash"
    id: "get-time"
    args:
      command: date +"%Y-%m-%d"
      
  - type: sendText
    text: "当前日期: {{tool.get-time.output}}"
```

---

## 💻 使用 handler.ts 实现复杂逻辑

当 HOOK.md 的配置能力无法满足需求时，可以使用 TypeScript/JavaScript 实现复杂的钩子逻辑。

### 基本结构

```typescript
// handler.ts
import type { HookContext, HookHandler } from "openclaw/hooks";

export const handler: HookHandler = async (ctx: HookContext) => {
  // 访问消息
  const message = ctx.message;
  if (!message) {
    ctx.logger.warn("No message in context");
    return;
  }
  
  // 发送回复
  await ctx.sendText(`收到你的消息: ${message.text}`);
  
  // 调用工具
  const result = await ctx.callTool("bash", {
    command: `echo "${message.text}"`
  });
  
  // 记录日志
  ctx.logger.info("Hook executed", { 
    messageId: message.id,
    hasResult: result.success 
  });
};

export default handler;
```

### 上下文接口

```typescript
interface HookContext {
  // 消息信息
  message?: {
    id: string;
    text: string;
    senderId: string;
    channel: string;
    chatId: string;
    mentions?: string[];
    attachments?: Attachment[];
  };
  
  // 会话信息
  session?: {
    id: string;
    key: string;
    agentId: string;
  };
  
  // 代理信息
  agent?: {
    id: string;
    name: string;
  };
  
  // 状态管理
  state: Record<string, unknown>;
  
  // 发送消息
  sendText(text: string): Promise<void>;
  sendTextTo(text: string, chatId: string): Promise<void>;
  
  // 调用工具
  callTool(name: string, args: Record<string, unknown>): Promise<ToolResult>;
  
  // HTTP 请求
  httpRequest(options: HttpRequestOptions): Promise<HttpResponse>;
  
  // 延迟
  delay(ms: number): Promise<void>;
  
  // 日志
  logger: {
    info(msg: string, data?: Record<string, unknown>): void;
    warn(msg: string, data?: Record<string, unknown>): void;
    error(msg: string, data?: Record<string, unknown>): void;
  };
  
  // 配置访问
  config: {
    get<T>(key: string): T | undefined;
  };
}
```

### 高级示例：智能路由钩子

```typescript
// handler.ts
import type { HookContext, HookHandler } from "openclaw/hooks";

export const handler: HookHandler = async (ctx: HookContext) => {
  const message = ctx.message;
  if (!message) return;
  
  // 根据消息内容决定路由
  const text = message.text.toLowerCase();
  
  // 紧急消息处理
  if (text.includes("紧急") || text.includes("urgent")) {
    await ctx.sendText("🚨 已标记为紧急，正在通知相关人员...");
    
    // 调用通知工具
    await ctx.callTool("notification_send", {
      channel: "ops",
      priority: "critical",
      message: `紧急消息来自 ${message.senderId}: ${message.text}`
    });
    
    return;
  }
  
  // 技术问题处理
  if (text.includes("bug") || text.includes("错误")) {
    // 收集诊断信息
    const diagnostics = await ctx.callTool("system_diagnostics", {});
    
    // 创建问题工单
    await ctx.callTool("ticket_create", {
      title: `问题报告: ${message.text.substring(0, 50)}...`,
      description: message.text,
      attachments: diagnostics.data,
      priority: "medium"
    });
    
    await ctx.sendText("📝 已为你创建问题工单，我们会尽快处理。");
    return;
  }
  
  // 默认处理
  await ctx.sendText("收到你的消息！如果是技术问题，请提供更多细节。");
};

export default handler;
```

---

## 📦 钩子打包为插件

### 插件集成

将钩子打包为 OpenClaw 插件，便于分发和安装：

```typescript
// extensions/my-hooks/index.ts
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api: PluginAPI) {
  // 从 hooks 目录注册所有钩子
  registerPluginHooksFromDir(api, "./hooks");
}
```

**插件目录结构**：

```
extensions/my-hooks/
├── openclaw.plugin.json
├── package.json
├── index.ts
└── hooks/
    ├── hook-one/
    │   ├── HOOK.md
    │   └── handler.ts
    └── hook-two/
        ├── HOOK.md
        └── handler.ts
```

### 钩子包（Hook Packs）

钩子包是一组相关钩子的集合，支持独立分发：

```
my-hook-pack/
├── hooks/
│   ├── hook-1/
│   │   ├── HOOK.md
│   │   └── handler.ts
│   └── hook-2/
│       ├── HOOK.md
│       └── handler.ts
├── HOOKS.md        # 包的元数据
├── README.md
└── package.json
```

**HOOKS.md 元数据**：

```markdown
---
name: my-hook-pack
version: 1.0.0
description: 我的钩子包
author: Your Name
dependencies:
  - "@openclaw/core": ">=1.0.0"
---
```

---

## 🔍 调试钩子

### 启用调试日志

```bash
# 查看钩子调试日志
openclaw hooks debug --follow

# 查看特定钩子的日志
openclaw hooks logs --hook hello-world --follow
```

### 测试模式

在配置中启用测试模式，不会实际执行操作：

```markdown
then:
  - type: sendText
    text: "测试消息"
    _test: true  # 标记为测试
```

### 性能分析

```markdown
# 在 HOOK.md 中启用性能追踪
name: my-hook
description: 性能测试钩子

performance:
  enabled: true
  logThreshold: 100  # 超过 100ms 记录日志

on:
  messageReceived: true

then:
  # ... 钩子逻辑
```

---

## 🚨 故障排除

### 钩子不触发

**检查清单**：

```bash
# 1. 检查钩子是否已加载
openclaw hooks list

# 2. 检查钩子是否启用
openclaw hooks status <hook-name>

# 3. 检查触发条件是否匹配
# 查看日志确认条件判断
openclaw logs | grep hook

# 4. 验证配置语法
openclaw hooks validate <hook-name>
```

### 权限错误

```bash
# 确保钩子文件有正确的权限
chmod -R 700 ~/.openclaw/hooks/

# 检查 SELinux（如果启用）
ls -Z ~/.openclaw/hooks/
```

### 循环执行

如果钩子导致无限循环：

```markdown
# 1. 使用条件限制执行频率
on:
  messageReceived:
    # 只处理首次消息，不处理机器人自己的消息
    senderId:
      - valuePattern: "bot_*"
        negate: true

# 2. 利用状态管理跟踪已处理的消息
then:
  - type: if
    condition: "{{state.processed}}" not equals "{{message.id}}"
    then:
      - type: setState
        key: "processed"
        value: "{{message.id}}"
      # ... 执行逻辑
```

### 性能问题

**诊断步骤**：

```bash
# 1. 查看钩子执行时间
openclaw hooks logs --format detailed

# 2. 检查慢钩子
# 启用了性能追踪后，超过阈值的钩子会被标记

# 3. 优化建议
# - 减少工具调用次数
# - 使用缓存避免重复操作
# - 简化条件判断逻辑
```

---

## 📋 完整示例

### 示例一：自动问候新用户

```markdown
# ~/.openclaw/hooks/welcome-new-users/HOOK.md

name: welcome-new-users
description: 欢迎新用户加入聊天

on:
  messageReceived: true

then:
  - type: if
    condition: |
      {{message.text}} matches /^(hi|hello|你好|嗨)/i
      && {{message.chatId}} not in {{state.welcomedUsers}}
    then:
      - type: sendText
        text: |
          👋 欢迎 {{message.senderId}}！
          
          我是 OpenClaw，一个 AI 助手。
          输入 /help 查看可用命令。
      - type: setState
        key: "welcomedUsers.{{message.chatId}}"
        value: |
          {{concat(
            (state.welcomedUsers.[{{message.chatId}}] or []),
            [{{message.senderId}}]
          )}}
```

### 示例二：定时发送提醒

```markdown
# ~/.openclaw/hooks/daily-reminder/HOOK.md

name: daily-reminder
description: 每天早上 9 点发送提醒

on:
  scheduled: true

then:
  - type: if
    condition: "{{cron.minute}}" == "0" && "{{cron.hour}}" == "9"
    then:
      - type: sendText
        text: |
          ☀️ 早上好！
          
          今日待办事项：
          1. 查看邮件
          2. 更新任务列表
          3. 回顾昨天的工作
          
          祝你有美好的一天！ 🌟
```

### 示例三：错误通知到 Slack

```markdown
# ~/.openclaw/hooks/error-notifier/HOOK.md

name: error-notifier
description: 发送错误通知到 Slack

on:
  error: true

then:
  - type: if
    condition: "{{error.type}}" == "critical"
    then:
      - type: httpRequest
        method: POST
        url: "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
        headers:
          Content-Type: "application/json"
        body:
          text: |
            🚨 OpenClaw 发生严重错误！

            *错误类型*: {{error.type}}
            *消息*: {{error.message}}
            *时间*: {{timestamp}}
```

---

## 📖 相关文档

- [自动化](/zh-CN/automation)——Cron 和 Webhook 自动化
- [消息系统](/zh-CN/concepts/messages)——消息处理详解
- [工具系统](/zh-CN/tools)——可用工具列表
- [插件开发](/zh-CN/developer/plugin-development)——开发 OpenClaw 插件
- [配置参考](/zh-CN/config/reference)——配置选项详解
