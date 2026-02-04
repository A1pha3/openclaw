---
summary: "OpenClaw Pi AI 集成详解：嵌入式 Pi 代理、系统提示构建、扩展机制和工具适配"
read_when:
  - 你需要了解 OpenClaw 如何集成 Pi AI
  - 你想了解嵌入式 Pi 会话的工作原理
  - 你需要配置 Pi 扩展或工具适配
title: "Pi AI 集成"
---

# Pi AI 集成

OpenClaw 集成了 **Pi AI**（一个专为编码设计的 AI 助手），通过嵌入式架构将 Pi 的能力融入 OpenClaw 的消息处理和代理系统。

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw Gateway                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Embedded Pi Session                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │  │
│  │  │ Pi Agent    │  │ Extensions  │  │ OpenClaw     │   │  │
│  │  │ Core        │  │ (Compaction │  │ Tool Adapter │   │  │
│  │  │             │  │  etc.)      │  │              │   │  │
│  │  └─────────────┘  └─────────────┘  └──────────────┘   │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              OpenClaw Tool Suite                       │  │
│  │  read, edit, glob, bash, exec, browser, etc.          │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 核心组件

### Pi 嵌入式运行器

`src/agents/pi-embedded-runner.ts` 是核心运行器，负责：

- 创建和管理嵌入式 Pi 会话
- 处理工具调用和结果
- 管理会话状态和历史
- 处理错误和故障转移

### Pi 嵌入式块处理器

`src/agents/pi-embedded-block-chunker.ts` 处理流式输出：

- 解析 `<thinking>` 和 `<final>` 标签
- 分离推理内容和最终答案
- 处理代码块和格式化文本

### Pi 扩展

`src/agents/pi-extensions/` 包含多个扩展模块：

| 扩展 | 功能 |
|------|------|
| `compaction-safeguard.ts` | 上下文压缩保护 |
| `context-pruning.ts` | 上下文剪枝 |
| `tool-normalizer.ts` | 工具调用标准化 |
| `result-sanitizer.ts` | 结果清理 |

## 系统提示构建

Pi 的系统提示由多个组件构建：

### 引导文件

| 文件 | 内容 | 最大字符数 |
|------|------|------------|
| `AGENTS.md` | 代理配置 | 50000 |
| `SOUL.md` | 核心价值观 | 20000 |
| `TOOLS.md` | 工具定义 | 20000 |
| `IDENTITY.md` | 身份定义 | 20000 |
| `USER.md` | 用户信息 | 20000 |
| `HEARTBEAT.md` | 心跳配置 | 20000 |
| `BOOTSTRAP.md` | 引导指令 | 20000 |

### 引导上下文构建

`buildBootstrapContext()` 函数：

```typescript
interface BootstrapContext {
  soul: string;          // SOUL.md 内容
  tools: string;         // TOOLS.md 内容（可能截断）
  agents: string;        // AGENTS.md 内容
  identity: string;      // IDENTITY.md 内容
  user: string;          // USER.md 内容（可能截断）
  heartbeat: string;     // HEARTBEAT.md 内容
  bootstrap: string;     // BOOTSTRAP.md 内容（如果存在）
  maxChars: number;      // 最大字符数限制
}

function buildBootstrapContext(
  params: BootstrapParams
): BootstrapContext {
  const files = await loadBootstrapFiles(workspace);
  
  // 合并 SOUL + AGENTS
  let soulAndAgents = files.soul + "\n\n" + files.agents;
  
  // 如果超过限制，截断 TOOLS
  if (soulAndAgents.length > params.maxChars - 5000) {
    const tools = truncateToolsSection(files.tools, params.maxChars - soulAndAgents.length - 5000);
    return { soul, tools, ... };
  }
  
  return { soul, tools: files.tools, agents: files.agents, ... };
}
```

### 工具定义截断

当工具定义过长时，系统会按优先级截断：

1. 保留 `description`（工具用途）
2. 保留 `inputSchema`（输入参数）
3. 截断 `commentary`（详细说明）
4. 删除冗余示例

### 块标签处理

Pi 使用块标签分隔不同类型的内容：

```markdown
<thinking>
这里是推理过程，模型思考如何解决问题。
</thinking>

<final>
这里是最终答案，直接返回给用户。
</final>
```

#### 块标签处理逻辑

```typescript
interface BlockProcessingResult {
  text: string;
  thinking: string | null;
  hasThinking: boolean;
  hasFinal: boolean;
  thinkingLevel: ThinkingLevel;
}

function processBlocks(
  chunk: string,
  state: BlockProcessorState
): BlockProcessingResult {
  // 提取 <thinking> 内容
  if (chunk.includes("<thinking>")) {
    state.thinking += extractThinking(chunk);
    state.hasThinking = true;
  }
  
  // 提取 <final> 内容
  if (chunk.includes("<final>")) {
    state.final += extractFinal(chunk);
    state.hasFinal = true;
  }
  
  // 清理标签
  const cleanedText = stripBlockTags(chunk, state);
  
  return {
    text: cleanedText,
    thinking: state.thinking,
    hasThinking: state.hasThinking,
    hasFinal: state.hasFinal,
    thinkingLevel: detectThinkingLevel(chunk),
  };
}
```

## 工具适配层

OpenClaw 提供了自己的工具集，适配 Pi 的工具调用接口：

### 工具创建

```typescript
function createOpenClawCodingTools(
  params: ToolParams
): Record<string, Tool> {
  return {
    // 核心工具
    read: tool("read", {
      description: "Read a file from the filesystem",
      parameters: {
        type: "object",
        properties: {
          filePath: { type: "string" },
          limit: { type: "number" },
          offset: { type: "number" },
        },
        required: ["filePath"],
      },
    }),
    
    edit: tool("edit", {
      description: "Edit a file by replacing oldString with newString",
      parameters: {
        type: "object",
        properties: {
          filePath: { type: "string" },
          oldString: { type: "string" },
          newString: { type: "string" },
          replaceAll: { type: "boolean" },
        },
        required: ["filePath", "oldString", "newString"],
      },
    }),
    
    // 其他工具...
  };
}
```

### Claude 风格别名

为兼容 Claude 的工具调用习惯，添加了别名：

```typescript
const CLAUDE_TOOL_ALIASES: Record<string, string[]> = {
  "bash": ["shell", "cmd", "command"],
  "glob": ["ls", "find", "list"],
  "grep": ["search", "find", "rg"],
  "read": ["cat", "view", "open"],
  "edit": ["update", "modify", "change"],
  "write": ["create", "new", "save"],
};
```

## 消息格式处理

### 消息角色规范化

Pi 使用特定的消息角色格式：

```typescript
const PI_MESSAGE_ROLES = ["user", "assistant", "tool_use", "tool_result"] as const;

function sanitizeSessionMessages(
  messages: Message[]
): Message[] {
  return messages.map((msg) => {
    // 确保角色有效
    if (!PI_MESSAGE_ROLES.includes(msg.role as any)) {
      return { ...msg, role: "assistant" };
    }
    
    // 清理内容
    return {
      ...msg,
      content: sanitizeContent(msg.content),
    };
  });
}
```

### 回复指令处理

Pi 支持特殊的回复指令：

```typescript
interface ReplyDirectives {
  mediaUrls: string[];      // [[media:url]]
  audioAsVoice: boolean;    // [[voice]]
  replyToId: string | null; // [[reply:id]]
}

function consumeReplyDirectives(
  chunk: string
): { text: string; directives: ReplyDirectives } {
  const mediaUrls = extractDirectives(chunk, /\\[\\[media:([^\\]]+)\\]\\]/g);
  const audioAsVoice = chunk.includes("[[voice]]");
  const replyToId = extractDirective(chunk, /\\[\\[reply:([^\\]]+)\\]\\]/, 1);
  
  return {
    text: chunk
      .replace(/\\[\\[media:[^\\]]+\\]\\]/g, "")
      .replace("[[voice]]", "")
      .replace(/\\[\\[reply:[^\\]]+\\]\\]/g, ""),
    directives: { mediaUrls, audioAsVoice, replyToId },
  };
}
```

## 错误处理

### 错误分类

```typescript
function classifyError(errorText: string): ErrorType {
  if (isContextOverflowError(errorText)) {
    return "context_overflow";
  }
  if (isCompactionFailureError(errorText)) {
    return "compaction_failure";
  }
  if (isAuthError(errorText)) {
    return "auth";
  }
  if (isRateLimitError(errorText)) {
    return "rate_limit";
  }
  return "unknown";
}

function isContextOverflowError(errorText: string): boolean {
  return (
    errorText.includes("context_length_exceeded") ||
    errorText.includes("too many tokens") ||
    errorText.includes("maximum context length")
  );
}
```

### Thinking 级别降级

当请求的 thinking 级别不受支持时，自动降级：

```typescript
function pickFallbackThinkingLevel(
  params: FallbackParams
): ThinkingLevel | null {
  const { message, attempted } = params;
  
  if (attempted === "high" && message.includes("not supported")) {
    return "medium";
  }
  if (attempted === "medium" && message.includes("not supported")) {
    return "low";
  }
  if (attempted === "low" && message.includes("not supported")) {
    return "none";
  }
  
  return null;
}
```

## 故障转移机制

当主提供商失败时，自动切换到备用提供商：

```typescript
async function runWithFailover(
  params: RunParams
): Promise<Result> {
  for (const provider of params.failoverProviders) {
    try {
      return await runWithProvider(provider);
    } catch (error) {
      const reason = classifyFailoverReason(error.message);
      
      if (shouldContinueFailover(reason)) {
        continue;
      }
      throw error;
    }
  }
}

function classifyFailoverReason(errorText: string): FailoverReason {
  if (errorText.includes("auth")) return "auth";
  if (errorText.includes("rate limit")) return "rate_limit";
  if (errorText.includes("quota")) return "quota";
  if (errorText.includes("timeout")) return "timeout";
  return "unknown";
}
```

## 沙箱集成

在沙箱模式下，工具和路径会受到限制：

```typescript
interface SandboxInfo {
  enabled: boolean;
  root: string | null;
  workspaceDir: string;
}

async function resolveSandboxContext(
  params: SandboxParams
): Promise<SandboxInfo> {
  if (!params.config.sandbox?.enabled) {
    return { enabled: false, root: null, workspaceDir: params.workspaceDir };
  }
  
  // 解析沙箱路径
  const sandboxRoot = resolveSandboxRoot(params.sessionKey);
  
  return {
    enabled: true,
    root: sandboxRoot,
    workspaceDir: path.join(sandboxRoot, "workspace"),
  };
}
```

## 与 Pi CLI 的区别

| 方面 | Pi CLI | OpenClaw 嵌入式 |
|------|--------|-----------------|
| 调用方式 | `pi` 命令 / RPC | SDK via `createAgentSession()` |
| 工具 | 默认编码工具 | 自定义 OpenClaw 工具套件 |
| 系统提示 | AGENTS.md + prompts | 按通道/上下文动态生成 |
| 会话存储 | `~/.pi/agent/sessions/` | `~/.openclaw/agents/<agentId>/sessions/` |
| 认证 | 单一凭证 | 多配置轮询 |
| 扩展 | 从磁盘加载 | 程序化 + 磁盘路径 |
| 事件处理 | TUI 渲染 | 基于回调（onBlockReply 等） |

## 测试

Pi 集成的测试覆盖：

- `src/agents/pi-embedded-block-chunker.test.ts` - 块处理
- `src/agents/pi-embedded-helpers.*.test.ts` - 辅助函数
- `src/agents/pi-embedded-runner.*.test.ts` - 运行器
- `src/agents/pi-embedded-subscribe.*.test.ts` - 订阅处理
- `src/agents/pi-extensions/*.test.ts` - 扩展模块

运行测试：

```bash
pnpm test -- --grep "pi-embedded"
```

## 相关文档

- [代理](/concepts/agents) - OpenClaw 代理系统
- [会话管理](/concepts/session) - 会话状态管理
- [工具](/tools) - OpenClaw 工具集
- [沙箱模式](/concepts/sandbox) - 安全沙箱
