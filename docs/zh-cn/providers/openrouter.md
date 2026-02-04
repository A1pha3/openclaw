---
summary: "使用 OpenRouter 的统一 API 在 OpenClaw 中访问许多模型"
read_when:
  - 你想要一个用于许多 LLM 的单一 API 密钥
  - 你想通过 OpenRouter 在 OpenClaw 中运行模型
title: "OpenRouter"
---

# OpenRouter

OpenRouter 提供了一个**统一 API**，将请求路由到单个端点和 API 密钥背后的许多模型。它是 OpenAI 兼容的，因此大多数 OpenAI SDK 可以通过切换基础 URL 来工作。

## CLI 设置

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 配置片段

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## 说明

- 模型引用格式为 `openrouter/<provider>/<model>`。
- 有关更多模型/提供商选项，请参阅 [/concepts/model-providers](/concepts/model-providers)。
- OpenRouter 在底层使用你的 API 密钥的 Bearer 令牌。
