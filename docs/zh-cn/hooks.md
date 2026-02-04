---
summary: "OpenClaw 钩子：事件驱动的自动化框架，用于在特定事件发生时执行自定义操作"
read_when:
  - 你希望在特定事件发生时自动执行操作
  - 你需要了解如何编写自定义钩子来处理消息、代理事件等
  - 你想将钩子打包为插件或独立的钩子包
title: "钩子（Hooks）"
---

# 钩子（Hooks）

钩子是 OpenClaw 的**事件驱动自动化框架**。它们允许你在特定事件发生时执行自定义操作，例如：

- 当收到新消息时
- 当代理开始或结束运行时
- 当消息发送失败时
- 定时触发任务（使用 cron 表达式）

钩子通过 `HOOK.md` 配置文件定义每个钩子的触发条件，通过 `handler.ts` 或 `handler.js` 实现具体的处理逻辑。

## 快速入门

如果你从未使用过钩子，以下是最简单的入门方式：

1. **查看已安装的钩子**：

   ```bash
   openclaw hooks list
   ```

2. **创建你的第一个钩子**：

   在你的工作区或全局钩子目录中创建：

   ```
   ~/.openclaw/hooks/my-first-hook/
   ├── HOOK.md
   └── handler.ts
   ```

3. **配置钩子**：

   在 `HOOK.md` 中定义触发条件：

   ```markdown
   on:
     messageReceived: true
   then:
     run:
       type: if # 条件执行
         condition: "{{message.text}}" contains "hello"
         then:
           - type: sendText
             text: "Hi there! 👋"
   ```

4. **测试钩子**：

   向你的 OpenClaw 发送一条包含 "hello" 的消息，你应该会收到自动回复。

## 钩子目录结构

钩子可以存放在多个位置，OpenClaw 会按以下顺序扫描：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | `~/.openclaw/hooks/` | 用户全局钩子目录 |
| 2 | `<workspace>/.openclaw/hooks/` | 工作区钩子目录 |
| 3 | `~/.openclaw/extensions/*/hooks/` | 插件内置钩子 |

### 标准目录结构

每个钩子都是一个包含配置和实现文件的目录：

```
my-hook/
├── HOOK.md           # 钩子配置（触发条件、执行逻辑）
├── handler.ts        # 处理函数（可选，复杂逻辑使用）
├── handler.js        # JavaScript 处理函数（可选）
├── package.json      # 依赖声明（可选）
└── tsconfig.json     # TypeScript 配置（可选）
```

## HOOK.md 配置详解

`HOOK.md` 是钩子的核心配置文件，使用 YAML 格式定义触发条件和执行逻辑。

### 基本结构

```markdown
# 钩子元数据
name: my-hook
description: 我的第一个钩子

# 触发条件
on:
  eventType: conditions

# 执行逻辑
then:
  - type: action1
    ...
```

### 触发条件（on）

触发条件定义了何时执行钩子。以下是所有支持的事件类型：

| 事件类型 | 说明 | 条件字段 |
|----------|------|----------|
| `messageReceived` | 收到消息时 | `text`, `senderId`, `channel` |
| `messageSent` | 消息发送后 | `text`, `recipientId`, `channel` |
| `agentRunStart` | 代理开始运行时 | `sessionKey`, `trigger` |
| `agentRunEnd` | 代理结束运行时 | `sessionKey`, `outcome` |
| `error` | 发生错误时 | `errorType`, `message` |
| `scheduled` | 定时触发 | `cron` |

### 执行逻辑（then）

`then` 部分定义了触发后执行的操作。支持多种操作类型：

#### 条件执行

```markdown
then:
  - type: if
    condition: "{{message.text}}" contains "hello"
    then:
      - type: sendText
        text: "Hello!"
    else:
      - type: sendText
        text: "Not a greeting"
```

#### 发送消息

```markdown
then:
  - type: sendText
    text: "自动回复消息"
```

#### 调用工具

```markdown
then:
  - type: callTool
    name: "bash"
    args:
      command: echo "Hello from hook"
```

#### 发送 HTTP 请求

```markdown
then:
  - type: httpRequest
    method: POST
    url: "https://api.example.com/webhook"
    body:
      event: "message"
      text: "{{message.text}}"
```

#### 延迟执行

```markdown
then:
  - type: delay
    duration: 5s
  - type: sendText
    text: "5秒后发送"
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

## 模板变量

钩子配置中可以使用模板变量来访问上下文数据：

### 消息相关变量

| 变量 | 说明 |
|------|------|
| `{{message.text}}` | 消息文本内容 |
| `{{message.senderId}}` | 发送者 ID |
| `{{message.channel}}` | 渠道类型 |
| `{{message.chatId}}` | 聊天会话 ID |
| `{{message.mentions}}` | 提及的用户列表 |
| `{{message.attachments}}` | 附件列表 |

### 会话相关变量

| 变量 | 说明 |
|------|------|
| `{{session.id}}` | 会话 ID |
| `{{session.key}}` | 会话键 |
| `{{session.agentId}}` | 代理 ID |

### 代理相关变量

| 变量 | 说明 |
|------|------|
| `{{agent.name}}` | 代理名称 |
| `{{agent.id}}` | 代理 ID |

### 工具输出变量

当使用 `callTool` 后，可以通过变量引用工具输出：

```markdown
then:
  - type: callTool
    name: "bash"
    id: "get-time"
    args:
      command: date +"%Y-%m-%d %H:%M:%S"
  - type: sendText
    text: "当前时间: {{tool.get-time.output}}"
```

## 完整示例

### 示例 1：自动问候新用户

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
          输入 `/help` 查看可用命令。
      - type: setState
        key: "welcomedUsers.{{message.chatId}}"
        value: |
          {{concat(
            (state.welcomedUsers.[{{message.chatId}}] or []),
            [{{message.senderId}}]
          )}}
```

### 示例 2：定时发送提醒

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

### 示例 3：错误通知

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

## 使用 handler.ts 实现复杂逻辑

对于复杂的钩子逻辑，建议使用 TypeScript/JavaScript 实现：

### 基本结构

```typescript
// handler.ts
import type { HookContext, HookHandler } from "openclaw/hooks";

export const handler: HookHandler = async (ctx: HookContext) => {
  // 访问消息
  const message = ctx.message;
  
  // 发送回复
  await ctx.sendText(`收到你的消息: ${message.text}`);
  
  // 调用工具
  const result = await ctx.callTool("bash", {
    command: `echo "${message.text}"`,
  });
  
  // 记录日志
  ctx.logger.info("Hook executed", { messageId: message.id });
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
  
  // 发送消息（指定聊天）
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
}
```

## 钩子打包为插件

你可以将钩子打包为 OpenClaw 插件：

```typescript
// extensions/my-hooks/index.ts
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api: PluginAPI) {
  // 从 hooks 目录注册所有钩子
  registerPluginHooksFromDir(api, "./hooks");
}
```

插件的目录结构：

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

## 钩子包（Hook Packs）

钩子包是一组相关钩子的集合，可以独立分发和安装：

```
my-hook-pack/
├── hooks/
│   ├── hook-1/
│   │   ├── HOOK.md
│   │   └── handler.ts
│   └── hook-2/
│       ├── HOOK.md
│       └── handler.ts
├── HOOKS.md      # 包的元数据
└── README.md
```

### HOOKS.md 包元数据

```markdown
---
name: my-hook-pack
version: 1.0.0
description: 我的钩子包
author: Your Name
---
```

## 调试钩子

### 启用调试日志

```bash
openclaw hooks debug --follow
```

### 测试模式

在配置中启用测试模式，不会实际执行操作：

```markdown
then:
  - type: sendText
    text: "测试消息"
    _test: true  # 标记为测试
```

## 故障排除

### 钩子不触发

1. 检查触发条件是否正确
2. 确认钩子已启用：`openclaw hooks list`
3. 查看日志：`openclaw logs --follow | grep hook`

### 权限错误

确保钩子文件有正确的权限：

```bash
chmod -R 700 ~/.openclaw/hooks/
```

### 循环执行

如果钩子导致无限循环：

1. 使用条件限制执行频率
2. 利用状态管理跟踪已处理的消息
3. 设置 `_maxRuns` 限制最大执行次数

## 最佳实践

1. **保持简洁**：每个钩子只做一件事
2. **错误处理**：始终处理可能的错误情况
3. **日志记录**：添加适当的日志便于调试
4. **性能考虑**：避免在钩子中执行长时间操作
5. **安全性**：不要在钩子中硬编码敏感信息

## 相关文档

- [自动化](/automation) - 了解 cron 和其他自动化方式
- [消息系统](/concepts/messages) - 消息处理详解
- [工具](/tools) - 可用工具列表
- [插件开发](/developer/plugins) - 开发 OpenClaw 插件
