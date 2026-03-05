---
read_when:
  - 你需要按提供商分类的模型设置参考
  - 你需要模型提供商的示例配置或 CLI 新手引导命令
  - 你想配置自定义模型提供商
summary: "模型提供商完整指南：内置提供商、自定义配置、认证方式、CLI 命令和最佳实践"
title: "模型提供商"
---

# 模型提供商

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 内置提供商的类型和特点
- [ ] 掌握主要提供商的配置方法
- [ ] 区分内置提供商和自定义提供商
- [ ] 使用 CLI 命令配置模型认证

### 进阶目标（建议掌握）

- [ ] 配置自定义模型提供商
- [ ] 理解 OpenAI 兼容端点的配置
- [ ] 处理多提供商和回退策略
- [ ] 调试提供商认证问题

---

## 💡 提供商类型

### 内置 vs 自定义

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    提供商类型                                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  内置提供商（pi-ai 目录）                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 无需 models.providers 配置                                    │   │
│  │  • 开箱即用                                                    │   │
│  │  • 自动处理认证和模型                                        │   │
│  │                                                                 │   │
│  │  示例：                                                          │   │
│  │  • anthropic（Anthropic 官方）                                   │   │
│  │  • openai（OpenAI 官方）                                         │   │
│  │  • openai-codex（OpenAI Code/Codex）                             │   │
│  │  • google（Gemini）                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  自定义提供商（models.providers 配置）                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 需要 models.providers 或 models.json                           │   │
│  │  • 支持 OpenAI 兼容端点                                        │   │
│  │  • 可自定义 baseUrl、apiKey、models                              │   │
│  │                                                                 │   │
│  │  示例：                                                          │   │
│  │  • moonshot（月之暗面 Kimi）                                       │   │
│  │  • minimax（MiniMax AI）                                          │   │
│  │  • ollama（本地运行时）                                          │   │
│  │  • LM Studio（本地代理）                                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 模型引用格式

```
格式：provider/model

示例：
• anthropic/claude-sonnet-4-20250514
• openai/gpt-4o
• openrouter/moonshotai/kimi-k2
• ollama/llama3.3

注意：提供商标/别名规范化为小写
z.ai/* → zai/*
z-ai/* → zai/*
```

---

## 🚀 快速开始

### CLI 向导（推荐）

```bash
# 启动向导
openclaw onboard

# 选择认证方式
openclaw onboard --auth-choice token           # API 密钥
openclaw onboard --auth-choice openai-api-key    # OpenAI API 密钥
openclaw onboard --auth-choice openai-codex      # OpenAI OAuth
openclaw onboard --auth-choice opencode-zen      # OpenCode Zen
```

### 快速命令

```bash
# 列出可用模型
openclaw models list

# 设置默认模型
openclaw models set anthropic/claude-sonnet-4-20250514

# 查看状态
openclaw models status
```

---

## 🏢 内置提供商

### Anthropic

```
┌─────────────────────────────────────────────────────────────────────────┐
│  提供商：anthropic                                                   │
├─────────────────────────────────────────────────────────────────────────┤
│  认证方式：                                                              │
│  • ANTHROPIC_API_KEY（API 密钥，推荐）                              │
│  • claude setup-token（订阅令牌）                                   │
│                                                                         │
│  示例模型：                                                              │
│  • anthropic/claude-sonnet-4-20250514                               │
│  • anthropic/claude-opus-4-20250514                                │
│  • anthropic/claude-haiku-3-5-20241022                              │
│                                                                         │
│  CLI 命令：                                                              │
│  openclaw onboard --auth-choice token                                   │
│  openclaw models auth paste-token --provider anthropic                │
└─────────────────────────────────────────────────────────────────────────┘
```

### OpenAI

```
┌─────────────────────────────────────────────────────────────────────────┐
│  提供商：openai                                                       │
├─────────────────────────────────────────────────────────────────────────┤
│  认证方式：OPENAI_API_KEY                                            │
│                                                                         │
│  示例模型：                                                              │
│  • openai/gpt-4o                                                     │
│  • openai/gpt-4o-mini                                                │
│  • openai/o1-preview                                               │
│                                                                         │
│  CLI 命令：                                                              │
│  openclaw onboard --auth-choice openai-api-key                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### OpenAI Code (Codex)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  提供商：openai-codex                                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  认证方式：OAuth (ChatGPT)                                            │
│                                                                         │
│  示例模型：                                                              │
│  • openai-codex/gpt-5.2                                              │
│                                                                         │
│  CLI 命令：                                                              │
│  openclaw onboard --auth-choice openai-codex                             │
│  openclaw models auth login --provider openai-codex                      │
└─────────────────────────────────────────────────────────────────────────┘
```

### Google Gemini

```
┌─────────────────────────────────────────────────────────────────────────┐
│  提供商：google                                                        │
├─────────────────────────────────────────────────────────────────────────┤
│  认证方式：GEMINI_API_KEY                                             │
│                                                                         │
│  示例模型：                                                              │
│  • google/gemini-2.5-pro-preview                                   │
│  • google/gemini-2.0-flash-exp                                     │
│                                                                         │
│  CLI 命令：                                                              │
│  openclaw onboard --auth-choice gemini-api-key                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### Z.AI (GLM)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  提供商：zai                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│  认证方式：ZAI_API_KEY                                               │
│                                                                         │
│  示例模型：                                                              │
│  • zai/glm-4-plus                                                  │
│  • zai/glm-4-plus-0115                                             │
│                                                                         │
│  CLI 命令：                                                              │
│  openclaw onboard --auth-choice zai-api-key                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### 其他内置提供商

| 提供商 | 认证 | 示例模型 |
|--------|------|----------|
| **openrouter** | `OPENROUTER_API_KEY` | `openrouter/anthropic/claude-sonnet-4-5` |
| **xai** | `XAI_API_KEY` | `xai/grok-beta` |
| **groq** | `GROQ_API_KEY` | `groq/llama-3.3-70b` |
| **cerebras** | `CEREBRAS_API_KEY` | `cerebras/zai-glm-4.7` |
| **mistral** | `MISTRAL_API_KEY` | `mistral/mistral-large` |
| **github-copilot** | `COPILOT_GITHUB_TOKEN` | `github-copilot/gpt-4` |
| **opencode** | `OPENCODE_API_KEY` | `opencode/claude-opus-4-5` |

---

## 🔧 自定义提供商

### Moonshot AI (Kimi)

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }]
      }
    }
  }
}
```

### Kimi Coding

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } }
  }
}
```

### MiniMax

```json5
{
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.chat/v1",
        apiKey: "${MINIMAX_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "MiniMax-MX-7B-beta", name: "MiniMax MX 7B" }
        ]
      }
    }
  }
}
```

### Ollama（本地）

```bash
# 安装 Ollama
# 访问 https://ollama.ai 下载

# 拉取模型
ollama pull llama3.3

# 配置
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

### 本地代理（LM Studio、vLLM）

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" }
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

---

## ⚙️ 配置参数说明

### 模型属性

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `reasoning` | `false` | 是否支持推理 |
| `input` | `["text"]` | 支持的输入类型 |
| `cost` | `0` | Token 成本 |
| `contextWindow` | `200000` | 上下文窗口大小 |
| `maxTokens` | `8192` | 最大输出 Token 数 |

### API 类型

| API | 说明 | 使用场景 |
|-----|------|----------|
| `openai-completions` | OpenAI 兼容补全 | 大多数模型 |
| `anthropic-messages` | Anthropic 消息 API | Claude 模型 |
| `anthropic-completions` | Anthropic 补全 API | 旧版 Claude |

---

## 🛠️ CLI 命令参考

### 认证命令

```bash
# 向导式认证
openclaw onboard

# API 密钥认证
openclaw onboard --auth-choice <provider>

# OAuth 登录
openclaw models auth login --provider <provider>

# setup-token 导入
openclaw models auth setup-token --provider anthropic
openclaw models auth paste-token --provider anthropic
```

### 模型管理

```bash
# 列出模型
openclaw models list [--all] [--provider <name>]

# 设置默认模型
openclaw models set <provider/model>

# 设置图像模型
openclaw models set-image <provider/model>

# 查看状态
openclaw models status [--plain] [--json]
```

### 别名管理

```bash
# 添加别名
openclaw models aliases add <alias> <provider/model>

# 列出别名
openclaw models aliases list

# 删除别名
openclaw models aliases remove <alias>
```

---

## 🎯 提供商对比

### 编程能力

| 提供商 | 编程 | 工具使用 | 推荐场景 |
|--------|------|----------|----------|
| **GLM (z.ai)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 代码开发 |
| **Claude Sonnet** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 平衡性能 |
| **GPT-4o** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 通用任务 |
| **MiniMax** | ⭐⭐⭐ | ⭐⭐⭐ | 创意写作 |

### 成本效益

| 提供商 | 成本 | 性价比 | 适用场景 |
|--------|------|--------|----------|
| **Haiku** | 低 | ⭐⭐⭐⭐ | 快速任务 |
| **Sonnet** | 中 | ⭐⭐⭐⭐ | 日常使用 |
| **Opus** | 高 | ⭐⭐⭐ | 复杂任务 |
| **OpenRouter 免费模型** | 免费 | ⭐⭐⭐ | 测试/备用 |

---

## 🐛 故障排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| "Model not allowed" | 不在白名单 | 添加到 `agents.defaults.models` |
| 认证失败 | API 密钥无效 | 检查密钥配置 |
| 速率限制 | 超过配额 | 等待或使用回退 |
| 网络错误 | 无法访问端点 | 检查 `baseUrl` |

### 调试技巧

```bash
# 查看模型状态
openclaw models status --verbose

# 检查认证状态
openclaw models status --check

# 查看可用模型
openclaw models list --all

# 测试模型连接
# 通过实际发送消息测试
```

---

## 🎯 最佳实践

### 选择建议

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    提供商选择指南                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  代码开发（推荐）：                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  主要：zai/glm-4-plus                                          │   │
│  │  回退：anthropic/claude-sonnet-4                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  通用任务（推荐）：                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  主要：anthropic/claude-sonnet-4                               │   │
│  │  回退：openai/gpt-4o                                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  创意写作（推荐）：                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  主要：minimax/minimax-mini                                      │   │
│  │  回退：zai/glm-4-plus                                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  本地开发：                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • ollama（本地运行时）                                          │   │
│  │  • LM Studio/vLLM（本地代理）                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 认证安全

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    认证安全最佳实践                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  环境变量（推荐）：                                                        │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  export OPENAI_API_KEY="sk-..."                                 │   │
│  │  export ANTHROPIC_API_KEY="sk-ant..."                            │   │
│  │                                                                 │   │
│  │  优势：                                                          │   │
│  │  ✅ 不写入配置文件                                               │   │
│  │  ✅ 防止意外提交                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  配置文件：                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  apiKey: "${ENV_VAR}"                                           │   │
│  │                                                                 │   │
│  │  注意事项：                                                        │   │
│  │  • 将 openclaw.json 添加到 .gitignore                             │   │
│  │  • 永远不要提交 API 密钥                                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [模型 CLI](/concepts/models) | 模型管理 |
| [模型故障转移](/concepts/model-failover) | 故障转移 |
| [配置参考](/config/reference) | 完整配置 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置内置提供商 | ⭐⭐⭐⭐⭐ |
| 配置自定义提供商 | ⭐⭐⭐⭐ |
| 选择合适提供商 | ⭐⭐⭐⭐ |
| 调试认证问题 | ⭐⭐⭐ |

---

> **💡 专家提示**：使用 `openclaw models status --check` 可以定期检查认证状态，在密钥过期前获得警告！
