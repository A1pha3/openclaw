---
summary: "AI 模型提供商配置 - Anthropic、OpenAI、Ollama 等模型配置"
read_when:
  - 配置 AI 模型访问
  - 设置 API Key 或 OAuth
  - 选择合适的模型
title: "AI 提供商"
---

# 🤖 AI 提供商

本文档介绍如何配置各种 AI 模型提供商，让 OpenClaw 能够调用不同的 AI 大脑。

## 🎯 支持的提供商

### 内置提供商

| 提供商 | 状态 | 说明 |
|--------|------|------|
| [Anthropic](/zh-CN/providers/anthropic) | ✅ 稳定 | Claude 系列模型（推荐） |
| [OpenAI](/zh-CN/providers/openai) | ✅ 稳定 | GPT 系列模型 |
| [OpenRouter](/zh-CN/providers/openrouter) | ✅ 稳定 | 多模型聚合服务 |
| [Ollama](/zh-CN/providers/ollama) | ✅ 稳定 | 本地模型 |
| [Moonshot](/zh-CN/providers/moonshot) | ✅ 稳定 | Kimi 系列 |
| [GLM](/zh-CN/providers/glm) | ✅ 稳定 | 智谱 AI |
| [MiniMax](/zh-CN/providers/minimax) | ✅ 稳定 | MiniMax 系列 |
| [Qwen](/zh-CN/providers/qwen) | ✅ 稳定 | 通义千问 |

### 插件提供商

| 提供商 | 说明 |
|--------|------|
| GitHub Copilot | 代码辅助 |
| Deepgram | 语音识别 |
| Vercel AI Gateway | Vercel 托管 |

---

## 🔧 基本配置

### 配置结构

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"
      },
      openai: {
        apiKey: "${OPENAI_API_KEY}"
      }
    }
  }
}
```

### 设置 API Key

```bash
# Anthropic
openclaw config set models.providers.anthropic.apiKey "sk-ant-api03-..."

# OpenAI
openclaw config set models.providers.openai.apiKey "sk-..."
```

### 使用环境变量

推荐使用环境变量而非硬编码：

```bash
# 设置环境变量
export ANTHROPIC_API_KEY="sk-ant-api03-..."
export OPENAI_API_KEY="sk-..."

# OpenClaw 会自动读取
```

---

## 🔐 认证方式

### API Key 方式

最简单的方式，适合快速上手：

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"  // 推荐：使用环境变量
        // 或直接写（不推荐）
        // apiKey: "sk-ant-api03-xxx"
      }
    }
  }
}
```

### OAuth 方式（推荐）

更安全，无需存储 API Key：

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": {
        provider: "anthropic",
        mode: "oauth",
        email: "me@example.com"
      }
    }
  }
}
```

**设置 OAuth**：

```bash
# 运行引导配置 OAuth
openclaw onboard
```

### 多账户支持

```json5
{
  auth: {
    profiles: {
      "anthropic:personal@example.com": {
        provider: "anthropic",
        mode: "oauth",
        email: "personal@example.com"
      },
      "anthropic:work@example.com": {
        provider: "anthropic",
        mode: "oauth",
        email: "work@example.com"
      }
    },
    order: {
      anthropic: [
        "anthropic:personal@example.com",
        "anthropic:work@example.com"
      ]
    }
  }
}
```

---

## 📊 模型选择

### 默认模型配置

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### 模型格式

```
<provider>/<model-name>
```

**示例**：

| 提供商 | 模型 | 配置值 |
|--------|------|--------|
| Anthropic | Claude Sonnet 4 | `anthropic/claude-sonnet-4-20250514` |
| Anthropic | Claude Opus 4 | `anthropic/claude-opus-4-20250514` |
| OpenAI | GPT-4o | `openai/gpt-4o` |
| OpenAI | GPT-4o Mini | `openai/gpt-4o-mini` |
| Ollama | Llama 3 | `ollama/llama3` |

### 按代理设置模型

```json5
{
  agents: {
    list: [
      {
        id: "main",
        model: "anthropic/claude-opus-4-20250514"  // 主代理用最强模型
      },
      {
        id: "fast",
        model: "anthropic/claude-haiku-3-5-20241022"  // 快速代理用轻量模型
      }
    ]
  }
}
```

---

## 🔄 模型回退

### 配置回退模型

当主模型不可用时，自动切换到回退模型：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-20250514",
        fallbacks: [
          "anthropic/claude-sonnet-4-20250514",
          "anthropic/claude-haiku-3-5-20241022",
          "openai/gpt-4o"
        ]
      }
    }
  }
}
```

### 回退条件

模型回退会在以下情况触发：

1. API 限流
2. API 错误
3. 网络超时
4. 认证失败

---

## 💰 成本管理

### 查看使用情况

```bash
# 查看 Token 使用
openclaw usage

# 查看成本统计
openclaw usage --cost
```

### 设置使用限制

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "...",
        maxTokens: 100000,  // 每月最大 Token
        maxCost: 100        // 每月最大成本（美元）
      }
    }
  }
}
```

### 成本优化建议

| 策略 | 说明 |
|------|------|
| 使用轻量模型 | 对于简单任务，使用 Haiku 而非 Opus |
| 减少上下文 | 只发送必要的对话历史 |
| 使用流式响应 | 减少等待时间 |
| 设置回退 | 使用低成本模型作为回退 |

---

## 🌍 地区与延迟

### 选择最近的端点

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "...",
        baseUrl: "https://api.anthropic.com"  // 默认
      },
      openai: {
        apiKey: "...",
        baseUrl: "https://api.openai.com/v1"  // 默认
      }
    }
  }
}
```

### 本地模型（Ollama）

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://localhost:11434"
      }
    }
  }
}
```

---

## 🛠️ 故障排除

### 认证失败

```bash
# 检查 API Key
echo $ANTHROPIC_API_KEY

# 测试 API Key
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
  https://api.anthropic.com/v1/messages \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

### 模型不可用

```bash
# 列出可用模型
openclaw models list

# 检查模型名称是否正确
openclaw config get agents.defaults.model
```

### 速率限制

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "...",
        rateLimit: {
          requestsPerMinute: 50,
          tokensPerMinute: 100000
        }
      }
    }
  }
}
```

---

## 📋 提供商配置示例

### Anthropic（推荐）

```json5
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-20250514"
    }
  }
}
```

### OpenAI

```json5
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": "openai/gpt-4o"
    }
  }
}
```

### Ollama（本地）

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://localhost:11434"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": "ollama/llama3"
    }
  }
}
```

### 多提供商

```json5
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "openai": {
        "apiKey": "${OPENAI_API_KEY}"
      },
      "ollama": {
        "baseUrl": "http://localhost:11434"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "openai/gpt-4o",
          "ollama/llama3"
        ]
      }
    }
  }
}
```

---

## 📈 模型性能对比

| 模型 | 速度 | 智能 | 成本 | 适合场景 |
|------|------|------|------|----------|
| Claude Opus 4 | 中 | 最高 | 高 | 复杂推理 |
| Claude Sonnet 4 | 快 | 高 | 中 | 日常对话 |
| Claude Haiku 3 | 最快 | 中 | 低 | 简单任务 |
| GPT-4o | 中 | 高 | 中 | 通用场景 |
| GPT-4o Mini | 快 | 中 | 低 | 成本敏感 |

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw models list` | 列出可用模型 |
| `openclaw config get agents.defaults.model` | 查看当前模型 |
| `openclaw usage` | 查看使用情况 |
| `openclaw health` | 检查 AI 认证状态 |

---

## 📚 相关文档

- [Anthropic 配置](/zh-CN/providers/anthropic) - Claude 配置
- [OpenAI 配置](/zh-CN/providers/openai) - GPT 配置
- [Ollama 配置](/zh-CN/providers/ollama) - 本地模型
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

**选择合适的 AI 模型，让您的助手既智能又高效！** 🦞
