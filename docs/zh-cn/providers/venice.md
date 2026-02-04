---
summary: "在 OpenClaw 中使用以隐私为重点的 Venice AI 模型"
read_when:
  - 你想在 OpenClaw 中进行以隐私为重点的推理
  - 你需要 Venice AI 设置指导
title: "Venice AI"
---

# Venice AI（Venice 亮点）

**Venice** 是我们对隐私优先推理的亮点设置，可选择性地访问专有模型。

Venice AI 提供以隐私为重点的 AI 推理，支持无审查模型并通过其匿名代理访问主要专有模型。所有推理默认是私人的——不对你的数据进行训练，不记录日志。

## 为什么在 OpenClaw 中使用 Venice

- **开源模型的私人推理**（无日志）。
- **无审查模型**，当你需要时。
- **匿名访问**专有模型（Opus/GPT/Gemini），当质量很重要时。
- OpenAI 兼容的 `/v1` 端点。

## 隐私模式

Venice 提供两个隐私级别——理解这一点对于选择模型至关重要：

| 模式 | 描述 | 模型 |
|------|------|------|
| **私人** | 完全私人。提示/响应**永远不会存储或记录**。临时的。 | Llama、Qwen、DeepSeek、Venice Uncensored 等 |
| **匿名** | 通过 Venice 代理，剥离元数据。底层提供商（OpenAI、Anthropic）看到匿名请求。 | Claude、GPT、Gemini、Grok、Kimi、MiniMax |

## 功能

- **以隐私为重点**：在"私人"（完全私人）和"匿名"（代理）模式之间选择
- **无审查模型**：访问没有内容限制的模型
- **主要模型访问**：通过 Venice 的匿名代理使用 Claude、GPT-5.2、Gemini
- **OpenAI 兼容 API**：标准 `/v1` 端点，易于集成
- **流式传输**：✅ 所有模型支持
- **函数调用**：✅ 选择性模型支持（检查模型能力）
- **视觉**：✅ 具有视觉能力的模型支持
- **无硬速率限制**：极端使用可能适用公平使用节流

## 设置

### 1. 获取 API 密钥

1. 在 [venice.ai](https://venice.ai) 注册
2. 转到 **设置 → API 密钥 → 创建新密钥**
3. 复制你的 API 密钥（格式：`vapi_xxxxxxxxxxxx`）

### 2. 配置 OpenClaw

**选项 A：环境变量**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**选项 B：交互式设置（推荐）**

```bash
openclaw onboard --auth-choice venice-api-key
```

这将：

1. 提示输入你的 API 密钥（或使用现有的 `VENICE_API_KEY`）
2. 显示所有可用的 Venice 模型
3. 让你选择默认模型
4. 自动配置提供商

**选项 C：非交互式**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3. 验证设置

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```

## 模型选择

设置后，OpenClaw 显示所有可用的 Venice 模型。根据你的需求选择：

- **默认（我们的选择）**：`venice/llama-3.3-70b` 用于私人的、平衡的性能。
- **最佳整体质量**：`venice/claude-opus-45` 用于困难任务（Opus 仍然是最强的）。
- **隐私**：选择"私人"模型以获得完全私人的推理。
- **能力**：选择"匿名"模型通过 Venice 的代理访问 Claude、GPT、Gemini。

随时更改你的默认模型：

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

列出所有可用的模型：

```bash
openclaw models list | grep venice
```

## 通过 `openclaw configure` 配置

1. 运行 `openclaw configure`
2. 选择 **模型/认证**
3. 选择 **Venice AI**

## 我应该使用哪个模型？

| 用例 | 推荐模型 | 原因 |
|------|----------|------|
| **一般聊天** | `llama-3.3-70b` | 全面的好，完全私人 |
| **最佳整体质量** | `claude-opus-45` | Opus 对于困难任务仍然最强 |
| **隐私 + Claude 质量** | `claude-opus-45` | 通过匿名代理获得最佳推理 |
| **编码** | `qwen3-coder-480b-a35b-instruct` | 代码优化，262k 上下文 |
| **视觉任务** | `qwen3-vl-235b-a22b` | 最佳私人视觉模型 |
| **无审查** | `venice-uncensored` | 无内容限制 |
| **快速 + 便宜** | `qwen3-4b` | 轻量级，仍有能力 |
| **复杂推理** | `deepseek-v3.2` | 强推理，私人 |

## 可用模型（共 25 个）

### 私人模型（15个）——完全私人，无日志

| 模型 ID | 名称 | 上下文（token）| 功能 |
|---------|------|------------------|------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | 一般 |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | 快速，轻量级 |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | 复杂任务 |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | 推理 |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | 一般 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | 代码 |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | 一般 |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | 视觉 |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | 快速，推理 |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | 推理 |
| `venice-uncensored` | Venice Uncensored | 32k | 无审查 |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | 视觉 |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | 视觉 |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | 一般 |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | 推理，多语言 |

### 匿名模型（10个）——通过 Venice 代理

| 模型 ID | 原始模型 | 上下文（token）| 功能 |
|---------|----------|------------------|------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | 推理，视觉 |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | 推理，视觉 |
| `openai-gpt-52` | GPT-5.2 | 262k | 推理 |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | 推理，视觉 |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | 推理，视觉 |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | 推理，视觉 |
| `grok-41-fast` | Grok 4.1 Fast | 262k | 推理，视觉 |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | 推理，代码 |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | 推理 |
| `minimax-m21` | MiniMax M2.1 | 202k | 推理 |

## 模型发现

当设置了 `VENICE_API_KEY` 时，OpenClaw 会自动从 Venice API 发现模型。如果 API 不可访问，它会回退到静态目录。

`/models` 端点是公共的（列出不需要认证），但推理需要有效的 API 密钥。

## 流式传输和工具支持

| 功能 | 支持 |
|------|------|
| **流式传输** | ✅ 所有模型 |
| **函数调用** | ✅ 大多数模型（在 API 中检查 `supportsFunctionCalling`） |
| **视觉/图像** | ✅ 标记为"视觉"功能的模型 |
| **JSON 模式** | ✅ 通过 `response_format` 支持 |

## 定价

Venice 使用基于积分的系统。查看 [venice.ai/pricing](https://venice.ai/pricing) 了解当前费率：

- **私人模型**： generally lower cost
- **匿名模型**：与直接 API 定价相似 + 小额 Venice 费用

## 比较：Venice vs 直接 API

| 方面 | Venice（匿名）| 直接 API |
|------|--------------|----------|
| **隐私** | 元数据剥离，匿名 | 你的账户链接 |
| **延迟** | +10-50ms（代理）| 直接 |
| **功能** | 大多数功能支持 | 完整功能 |
| **计费** | Venice 积分 | 提供商计费 |

## 使用示例

```bash
# 使用默认私人模型
openclaw chat --model venice/llama-3.3-70b

# 通过 Venice 使用 Claude（匿名）
openclaw chat --model venice/claude-opus-45

# 使用无审查模型
openclaw chat --model venice/venice-uncensored

# 使用视觉模型和图像
openclaw chat --model venice/qwen3-vl-235b-a22b

# 使用编码模型
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```

## 故障排除

### API 密钥不被识别

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

确保密钥以 `vapi_` 开头。

### 模型不可用

Venice 模型目录动态更新。运行 `openclaw models list` 查看当前可用的模型。有些模型可能暂时离线。

### 连接问题

Venice API 位于 `https://api.venice.ai/api/v1`。确保你的网络允许 HTTPS 连接。

## 配置文件示例

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 链接

- [Venice AI](https://venice.ai)
- [API 文档](https://docs.venice.ai)
- [定价](https://venice.ai/pricing)
- [状态](https://status.venice.ai)
