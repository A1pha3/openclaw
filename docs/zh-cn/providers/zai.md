---
summary: "在 OpenClaw 中使用 Z.AI（GLM 模型）"
read_when:
  - 您想在 OpenClaw 中使用 Z.AI / GLM 模型
  - 您需要简单的 ZAI_API_KEY 配置
title: "Z.AI"
---

# Z.AI

Z.AI 是 **GLM** 模型的 API 平台。它为 GLM 提供 REST API，并使用 API 密钥进行身份验证。请在 Z.AI 控制台中创建您的 API 密钥。OpenClaw 使用 `zai` 提供程序配合 Z.AI API 密钥。

## CLI 设置

```bash
openclaw onboard --auth-choice zai-api-key
# 或非交互式
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## 配置示例

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## 注意事项

- GLM 模型以 `zai/<model>` 的形式提供（例如：`zai/glm-4.7`）。
- 请参阅 [/zh-CN/providers/glm](/zh-CN/providers/glm) 了解模型系列概览。
- Z.AI 使用 Bearer 身份验证，使用您的 API 密钥。
