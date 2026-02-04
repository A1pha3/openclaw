---
summary: "在 OpenClaw 中使用小米 MiMo (mimo-v2-flash)"
read_when:
  - 你想在 OpenClaw 中使用小米 MiMo 模型
  - 你需要配置 XIAOMI_API_KEY
title: "小米 MiMo"
---

# 小米 MiMo

小米 MiMo 是 **MiMo** 模型的 API 平台。它提供与 OpenAI 和 Anthropic 格式兼容的 REST API，并使用 API 密钥进行身份验证。在 [小米 MiMo 控制台](https://platform.xiaomimimo.com/#/console/api-keys)中创建你的 API 密钥。OpenClaw 使用 `xiaomi` 提供商和小米 MiMo API 密钥。

## 模型概述

- **mimo-v2-flash**: 262144 token 上下文窗口，兼容 Anthropic Messages API。
- Base URL: `https://api.xiaomimimo.com/anthropic`
- Authorization: `Bearer $XIAOMI_API_KEY`

## CLI 设置

```bash
openclaw onboard --auth-choice xiaomi-api-key
# 或非交互式
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## 配置片段

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 注意事项

- 模型引用: `xiaomi/mimo-v2-flash`。
- 当设置了 `XIAOMI_API_KEY` 时（或存在身份验证配置文件），提供商会自动注入。
- 参阅 [/zh-CN/concepts/model-providers](/zh-CN/concepts/model-providers) 了解提供商规则。
