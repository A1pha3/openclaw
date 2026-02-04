---
summary: "OpenClaw 支持的模型提供商（LLM）"
read_when:
  - 您想选择模型提供商
  - 您想要 LLM 身份验证 + 模型选择的快速设置示例
title: "模型提供商快速入门"
---

# 模型提供商

OpenClaw 可以使用许多 LLM 提供商。选择一个，进行身份验证，然后将默认模型设置为 `provider/model`。

## 重点：Venice (Venice AI)

Venice 是我们推荐的 Venice AI 设置，用于以隐私优先的推理，并可选择使用 Opus 处理最困难的任务。

- 默认：`venice/llama-3.3-70b`
- 综合最佳：`venice/claude-opus-45` (Opus 仍然是最强大的)

参见 [Venice AI](/zh-CN/providers/venice)。

## 快速入门（两步）

1. 与提供商进行身份验证（通常通过 `openclaw onboard`）。
2. 设置默认模型：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } },
}
```

## 支持的提供商（入门套件）

- [OpenAI (API + Codex)](/zh-CN/providers/openai)
- [Anthropic (API + Claude Code CLI)](/zh-CN/providers/anthropic)
- [OpenRouter](/zh-CN/providers/openrouter)
- [Vercel AI Gateway](/zh-CN/providers/vercel-ai-gateway)
- [Moonshot AI (Kimi + Kimi Coding)](/zh-CN/providers/moonshot)
- [Synthetic](/zh-CN/providers/synthetic)
- [OpenCode Zen](/zh-CN/providers/opencode)
- [Z.AI](/zh-CN/providers/zai)
- [GLM models](/zh-CN/providers/glm)
- [MiniMax](/zh-CN/providers/minimax)
- [Venice (Venice AI)](/zh-CN/providers/venice)
- [Amazon Bedrock](/zh-CN/bedrock)

有关完整的提供商目录（xAI、Groq、Mistral 等）和高级配置，请参见[模型提供商](/zh-CN/concepts/model-providers)。
