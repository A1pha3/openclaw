---
title: Lobster
summary: "带有可恢复审批门控的类型化工作流运行时"
description: OpenClaw 的类型化工作流运行时 — 带审批门控的可组合管道
read_when:
  - 想要带有显式审批的确定性多步骤工作流
  - 需要恢复工作流而不重新运行之前的步骤
---

# Lobster

Lobster 是一个工作流 shell，让 OpenClaw 可以将多步骤工具序列作为单个确定性操作运行，并带有显式审批检查点。

## 亮点

你的助手可以构建管理自己的工具。请求一个工作流，30 分钟后你就有了一个 CLI 加上作为单个调用运行的管道。Lobster 是缺失的一环：确定性管道、显式审批和可恢复状态。

## 为什么需要 Lobster

今天，复杂的工作流需要许多来回的工具调用。每次调用消耗 token，LLM 必须编排每个步骤。Lobster 将编排移入类型化运行时：

| 特性 | 说明 |
|------|------|
| **一次调用代替多次** | OpenClaw 运行一次 Lobster 工具调用，获得结构化结果 |
| **内置审批** | 有副作用的操作（发送邮件、发表评论）会暂停工作流直到明确批准 |
| **可恢复** | 暂停的工作流返回一个 token；批准后恢复，无需重新运行所有步骤 |

## 为什么是 DSL 而不是普通程序

Lobster 故意设计得很小。目标不是「一门新语言」，而是一个可预测的、AI 友好的管道规范，带有一流的审批和恢复 token。

| 优势 | 说明 |
|------|------|
| **审批/恢复内置** | 普通程序可以提示人类，但没有自定义运行时无法带持久 token _暂停和恢复_ |
| **确定性 + 可审计** | 管道是数据，所以易于记录、对比、重放和审查 |
| **AI 受限表面** | 小语法 + JSON 管道减少「创造性」代码路径，使验证可行 |
| **内置安全策略** | 超时、输出上限、沙箱检查和白名单由运行时强制执行，而不是每个脚本 |
| **仍可编程** | 每个步骤可以调用任何 CLI 或脚本。如果想用 JS/TS，从代码生成 `.lobster` 文件 |

## 工作原理

OpenClaw 以**工具模式**启动本地 `lobster` CLI，并从 stdout 解析 JSON 信封。
如果管道暂停等待审批，工具返回一个 `resumeToken`，你可以稍后用它继续。

## 模式：小 CLI + JSON 管道 + 审批

构建输出 JSON 的小命令，然后链接成单个 Lobster 调用：

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

如果管道请求审批，用 token 恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI 触发工作流；Lobster 执行步骤。审批门控保持副作用显式和可审计。

**示例：将输入项映射为工具调用**

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## JSON 专用 LLM 步骤（llm-task）

对于需要**结构化 LLM 步骤**的工作流，启用可选的 `llm-task` 插件工具并从 Lobster 调用。这保持工作流确定性，同时仍允许用模型进行分类/摘要/草稿。

**启用工具：**

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

**在管道中使用：**

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "给定输入邮件，返回意图和草稿。",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

详情参见 [LLM Task](/tools/llm-task)。

## 工作流文件（.lobster）

Lobster 可以运行带有 `name`、`args`、`steps`、`env`、`condition` 和 `approval` 字段的 YAML/JSON 工作流文件。在 OpenClaw 工具调用中，将 `pipeline` 设置为文件路径。

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

**注意：**

- `stdin: $step.stdout` 和 `stdin: $step.json` 传递前一步的输出
- `condition`（或 `when`）可以根据 `$step.approved` 门控步骤

## 安装 Lobster

在运行 OpenClaw Gateway 的**同一主机**上安装 Lobster CLI（参见 [Lobster 仓库](https://github.com/openclaw/lobster)），并确保 `lobster` 在 `PATH` 中。

如果想使用自定义二进制位置，在工具调用中传递**绝对**路径的 `lobsterPath`。

## 启用工具

Lobster 是**可选**的插件工具（默认不启用）。

**推荐（增量，安全）：**

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

**或按智能体配置：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

避免使用 `tools.allow: ["lobster"]`，除非你打算在限制性白名单模式下运行。

**注意**：白名单对可选插件是可选的。如果你的白名单只列出插件工具（如 `lobster`），OpenClaw 会保持核心工具启用。要限制核心工具，在白名单中也包含你想要的核心工具或组。

## 示例：邮件分类

**没有 Lobster：**

```
用户："检查我的邮件并起草回复"
→ openclaw 调用 gmail.list
→ LLM 总结
→ 用户："为 #2 和 #5 起草回复"
→ LLM 起草
→ 用户："发送 #2"
→ openclaw 调用 gmail.send
（每天重复，不记得分类过什么）
```

**有 Lobster：**

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

**返回 JSON 信封（截断）：**

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 封需要回复，2 封需要操作" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "发送 2 封草稿回复？",
    "items": [],
    "resumeToken": "..."
  }
}
```

**用户批准 → 恢复：**

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

一个工作流。确定性。安全。

## 工具参数

### `run` — 运行管道

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

**运行带参数的工作流文件：**

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### `resume` — 恢复暂停的工作流

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### 可选输入

| 参数 | 说明 |
|------|------|
| `lobsterPath` | Lobster 二进制的绝对路径（省略则使用 `PATH`） |
| `cwd` | 管道的工作目录（默认：当前进程工作目录） |
| `timeoutMs` | 超过此时长则终止子进程（默认：20000） |
| `maxStdoutBytes` | stdout 超过此大小则终止子进程（默认：512000） |
| `argsJson` | 传递给 `lobster run --args-json` 的 JSON 字符串（仅工作流文件） |

## 输出信封

Lobster 返回三种状态之一的 JSON 信封：

| 状态 | 说明 |
|------|------|
| `ok` | 成功完成 |
| `needs_approval` | 已暂停；需要 `requiresApproval.resumeToken` 来恢复 |
| `cancelled` | 明确拒绝或取消 |

工具在 `content`（格式化 JSON）和 `details`（原始对象）中都显示信封。

## 审批

如果存在 `requiresApproval`，检查提示并决定：

| 决定 | 说明 |
|------|------|
| `approve: true` | 恢复并继续有副作用的操作 |
| `approve: false` | 取消并完成工作流 |

使用 `approve --preview-from-stdin --limit N` 将 JSON 预览附加到审批请求，无需自定义 jq/heredoc 胶水代码。

恢复 token 现在是紧凑的：Lobster 在其状态目录下存储工作流恢复状态，只返回一个小的 token 键。

## OpenProse 集成

OpenProse 与 Lobster 配合良好：使用 `/prose` 编排多智能体准备，然后运行 Lobster 管道进行确定性审批。如果 Prose 程序需要 Lobster，通过 `tools.subagents.tools` 为子智能体允许 `lobster` 工具。参见 [OpenProse](/prose)。

## 安全

| 特性 | 说明 |
|------|------|
| **仅本地子进程** | 插件本身不进行网络调用 |
| **无密钥管理** | Lobster 不管理 OAuth；它调用做这些的 OpenClaw 工具 |
| **沙箱感知** | 工具上下文处于沙箱时禁用 |
| **加固** | 如果指定 `lobsterPath` 必须是绝对路径；强制执行超时和输出上限 |

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| `lobster subprocess timed out` | 增加 `timeoutMs`，或拆分长管道 |
| `lobster output exceeded maxStdoutBytes` | 提高 `maxStdoutBytes` 或减少输出大小 |
| `lobster returned invalid JSON` | 确保管道在工具模式下运行并只打印 JSON |
| `lobster failed (code …)` | 在终端运行同样的管道检查 stderr |

## 了解更多

- [插件系统](/plugin)
- [插件工具开发](/plugins/agent-tools)

## 案例研究：社区工作流

一个公开示例：一个「第二大脑」CLI + Lobster 管道，管理三个 Markdown 库（个人、伴侣、共享）。CLI 为统计、收件箱列表和陈旧扫描输出 JSON；Lobster 将这些命令链接成工作流如 `weekly-review`、`inbox-triage`、`memory-consolidation` 和 `shared-task-sync`，每个都有审批门控。AI 在可用时处理判断（分类），不可用时回退到确定性规则。

- 帖子：https://x.com/plattenschieber/status/2014508656335770033
- 仓库：https://github.com/bloomedai/brain-cli

## 相关文档

- [LLM Task](/tools/llm-task) - JSON 专用 LLM 步骤
- [执行审批](/tools/exec-approvals) - 命令执行审批
