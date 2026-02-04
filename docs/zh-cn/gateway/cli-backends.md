---
summary: "CLI 后端：通过本地 AI CLI 的纯文本回退"
read_when:
  - "你希望 API 提供商失败时有可靠的回退"
  - "你正在运行 Claude Code CLI 或其他本地 AI CLI 并想重用它们"
  - "你需要支持会话和图像的纯文本、无工具路径"
title: "CLI 后端"
---

# CLI 后端（回退运行时）

OpenClaw 可以在 API 提供商宕机、速率受限或暂时行为异常时将**本地 AI CLI**作为**纯文本回退**运行。这是故意保守的设计：

- **工具被禁用**（没有工具调用）。
- **文本输入 → 文本输出**（可靠）。
- **支持会话**（因此后续回合保持连贯）。
- **如果 CLI 接受图像路径，则可以传递图像**。

这是设计为**安全网**而不是主要路径。当你想要"始终可用"的文本响应而不依赖外部 API 时使用它。

## 适合初学者的快速入门

你可以**无需任何配置**使用 Claude Code CLI（OpenClaw 附带了内置默认配置）：

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI 也可以直接使用：

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

如果你的网关在 launchd/systemd 下运行且 PATH 最小，只需添加命令路径：

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

就这样。除了 CLI 本身之外，不需要密钥，也不需要额外的认证配置。

## 作为回退使用

将 CLI 后端添加到你的回退列表，以便仅在主要模型失败时运行：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

注意：

- 如果你使用 `agents.defaults.models`（允许列表），必须包含 `claude-cli/...`。
- 如果主要提供商失败（认证、速率限制、超时），OpenClaw 将尝试下一个 CLI 后端。

## 配置概览

所有 CLI 后端位于：

```
agents.defaults.cliBackends
```

每个条目由**提供商 ID**（例如 `claude-cli`、`my-cli`）键入。提供商 ID 成为模型引用的左侧：

```
<provider>/<model>
```

### 示例配置

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## 工作原理

1. **根据提供商前缀**（`claude-cli/...`）选择后端。
2. **使用相同的 OpenClaw 提示 + 工作区上下文构建系统提示**。
3. **执行 CLI** 并带有会话 ID（如果支持），以便历史记录保持一致。
4. **解析输出**（JSON 或纯文本）并返回最终文本。
5. **持久化每个后端的会话 ID**，以便后续使用相同的 CLI 会话。

## 会话

- 如果 CLI 支持会话，设置 `sessionArg`（例如 `--session-id`）或 `sessionArgs`（占位符 `{sessionId}`），当 ID 需要插入多个标志时。
- 如果 CLI 使用**恢复子命令**并带有不同标志，设置 `resumeArgs`（恢复时替换 `args`）和可选的 `resumeOutput`（用于非 JSON 恢复）。
- `sessionMode`：
  - `always`：始终发送会话 ID（如果没有存储，则使用新的 UUID）。
  - `existing`：仅在之前存储了会话 ID 时才发送。
  - `none`：从不发送会话 ID。

## 图像（传递）

如果你的 CLI 接受图像路径，设置 `imageArg`：

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw 会将 base64 图像写入临时文件。如果设置了 `imageArg`，这些路径将作为 CLI 参数传递。如果缺少 `imageArg`，OpenClaw 将文件路径附加到提示（路径注入），这对于从纯路径自动加载本地文件的 CLI 来说足够了（Claude Code CLI 行为）。

## 输入 / 输出

- `output: "json"`（默认）尝试解析 JSON 并提取文本 + 会话 ID。
- `output: "jsonl"` 解析 JSONL 流（Codex CLI `--json`）并提取最后的代理消息以及存在的 `thread_id`。
- `output: "text"` 将 stdout 视为最终响应。

输入模式：

- `input: "arg"`（默认）将提示作为最后一个 CLI 参数传递。
- `input: "stdin"` 通过 stdin 发送提示。
- 如果提示非常长且设置了 `maxPromptArgChars`，则使用 stdin。

## 默认值（内置）

OpenClaw 为 `claude-cli` 附带了默认值：

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
: "--session-id- `sessionArg"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw 还为 `codex-cli` 附带了默认值：

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

仅在需要时覆盖（常见情况：绝对 `command` 路径）。

## 限制

- **没有 OpenClaw 工具**（CLI 后端永远不会收到工具调用）。某些 CLI 可能仍然运行自己的代理工具。
- **没有流式传输**（收集 CLI 输出然后返回）。
- **结构化输出**取决于 CLI 的 JSON 格式。
- **Codex CLI 会话**通过文本输出恢复（没有 JSONL），这比初始的 `--json` 运行更结构化。OpenClaw 会话仍然正常工作。

## 故障排除

- **找不到 CLI**：将 `command` 设置为完整路径。
- **错误的模型名称**：使用 `modelAliases` 将 `provider/model` 映射到 CLI 模型。
- **没有会话连续性**：确保设置了 `sessionArg` 并且 `sessionMode` 不是 `none`（Codex CLI 当前无法使用 JSON 输出恢复）。
- **图像被忽略**：设置 `imageArg`（并验证 CLI 支持文件路径）。
