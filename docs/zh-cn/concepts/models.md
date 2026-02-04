---
summary: "模型配置详解 - AI 模型选择、配置和优化"
read_when:
  - 选择合适的 AI 模型
  - 配置模型参数
  - 优化模型使用
title: "模型配置"
---

# 🤖 模型配置

本文档详细介绍 OpenClaw 的 AI 模型配置，包括模型选择、参数调优和性能优化。

## 🎯 模型概览

### 支持的模型提供商

| 提供商 | 模型系列 | 特点 |
|--------|----------|------|
| **Anthropic** | Claude 4/3.5/3 | 擅长推理、长上下文 |
| **OpenAI** | GPT-4o/4o-mini | 通用能力强 |
| **Ollama** | Llama/Mistral 等 | 本地部署 |
| **国产模型** | GLM/Kimi/Qwen | 中文优化 |

### 推荐模型

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| **通用对话** | Claude Sonnet 4 | 平衡智能与速度 |
| **复杂推理** | Claude Opus 4 | 最强推理能力 |
| **代码任务** | Claude Opus 4 | 优秀代码理解 |
| **成本敏感** | Claude Haiku 3 | 低成本快速响应 |
| **本地部署** | Llama 3 70B | 隐私保护 |

---

## ⚙️ 基础配置

### 全局默认模型

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### 按代理配置

```json5
{
  agents: {
    list: [
      {
        id: "main",
        model: "anthropic/claude-opus-4-20250514"
      },
      {
        id: "fast",
        model: "anthropic/claude-haiku-3-5-20241022"
      },
      {
        id: "coding",
        model: "anthropic/claude-sonnet-4-20250514"
      }
    ]
  }
}
```

### 模型格式

```
<provider>/<model-name>
```

**示例**：
```json5
{
  "anthropic/claude-opus-4-20250514",
  "anthropic/claude-sonnet-4-20250514",
  "anthropic/claude-haiku-3-5-20241022",
  "openai/gpt-4o",
  "openai/gpt-4o-mini",
  "ollama/llama3",
  "ollama/mistral"
}
```

---

## 🔄 模型回退

### 配置回退链

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

| 条件 | 说明 |
|------|------|
| API 限流 | 请求被限速 |
| API 错误 | 服务器返回错误 |
| 网络超时 | 请求超时 |
| 认证失败 | API Key 无效 |

---

## 📊 模型参数

### 生成参数

```json5
{
  agents: {
    defaults: {
      modelParameters: {
        temperature: 0.7,        // 0-1， creativity
        topP: 0.9,              // 0-1， vocabulary selection
        maxTokens: 4000,         // 最大输出 Token
        presencePenalty: 0,      // -2 to 2， novelty
        frequencyPenalty: 0      // -2 to 2， repetition
      }
    }
  }
}
```

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `temperature` | 随机性，0 更确定，1 更随机 | 0.1-0.9 |
| `topP` | 核采样，限制词汇选择范围 | 0.9 |
| `maxTokens` | 最大输出长度 | 2000-4000 |
| `presencePenalty` | 惩罚新词汇 | 0 |
| `frequencyPenalty` | 惩罚重复 | 0 |

### 场景化配置

**代码生成**：
```json5
{
  modelParameters: {
    temperature: 0.1,    // 低随机性
    maxTokens: 4000      // 长代码片段
  }
}
```

**创意写作**：
```json5
{
  modelParameters: {
    temperature: 0.8,    // 高随机性
    topP: 0.95
  }
}
```

**精确问答**：
```json5
{
  modelParameters: {
    temperature: 0.2,    // 低随机性
    topP: 0.8
  }
}
```

---

## 💰 成本管理

### Token 限制

```json5
{
  models: {
    limits: {
      daily: 1000000,    // 每日限制
      monthly: 20000000  // 每月限制
    }
  }
}
```

### 成本监控

```bash
# 查看使用情况
openclaw usage

# 设置预算提醒
openclaw usage --alert 80  # 80% 时提醒
```

### 成本优化

| 策略 | 说明 |
|------|------|
| 使用轻量模型 | Haiku 成本约为 Opus 的 1/10 |
| 减少上下文 | 只发送必要的历史 |
| 限制响应长度 | 设置合理的 maxTokens |
| 使用回退 | 故障时自动切换低成本模型 |

---

## 🌍 上下文窗口

### 各模型上下文限制

| 模型 | 上下文 Token |
|------|--------------|
| Claude Opus 4 | 200K |
| Claude Sonnet 4 | 200K |
| Claude Haiku 3 | 200K |
| GPT-4o | 128K |
| GPT-4o-mini | 128K |
| Llama 3 | 8K-128K (取决于版本) |

### 优化上下文使用

```json5
{
  messages: {
    context: {
      maxTokens: 100000,  // 根据模型限制设置
      reserveTokens: 5000  // 为响应预留
    }
  }
}
```

---

## 🔧 故障排除

### 模型不可用

```bash
# 检查模型配置
openclaw config get agents.defaults.model

# 查看可用模型列表
openclaw models list

# 测试 API 连接
curl -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  https://api.anthropic.com/v1/messages \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

### 响应质量差

```bash
# 调整参数
openclaw config set agents.defaults.modelParameters.temperature 0.7

# 检查上下文大小
openclaw sessions context <session-key>

# 清除历史重新开始
openclaw sessions clear <session-key>
```

### 成本过高

```bash
# 查看成本明细
openclaw usage --cost

# 使用低成本模型
openclaw config set agents.defaults.model "anthropic/claude-haiku-3-5-20241022"

# 减少上下文限制
openclaw config set messages.context.maxTokens 30000
```

---

## 📈 性能对比

### 响应速度

| 模型 | 首 Token 时间 | 完整响应（平均） |
|------|---------------|------------------|
| Claude Haiku 3 | ~0.5s | ~2s |
| Claude Sonnet 4 | ~1s | ~5s |
| Claude Opus 4 | ~1.5s | ~8s |
| GPT-4o-mini | ~0.3s | ~1.5s |
| GPT-4o | ~0.8s | ~4s |

### 智能水平

| 模型 | 推理 | 编程 | 创意 | 中文 |
|------|------|------|------|------|
| Claude Opus 4 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Claude Sonnet 4 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Claude Haiku 3 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| GPT-4o | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| GPT-4o-mini | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

---

## 📝 最佳实践

### 开发环境

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514",
      modelParameters: {
        temperature: 0.3,
        maxTokens: 3000
      }
    }
  }
}
```

### 生产环境

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: ["anthropic/claude-haiku-3-5-20241022"]
      },
      modelParameters: {
        temperature: 0.5,
        maxTokens: 2000
      }
    }
  }
}
```

### 成本敏感场景

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-haiku-3-5-20241022",
      modelParameters: {
        temperature: 0.3,
        maxTokens: 1500
      }
    }
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw models list` | 列出可用模型 |
| `openclaw config get agents.defaults.model` | 查看当前模型 |
| `openclaw usage` | 查看使用统计 |
| `openclaw status` | 检查模型状态 |

---

## 📚 相关文档

- [AI 提供商](/zh-CN/providers) - 各提供商配置
- [模型故障转移](/zh-CN/concepts/model-failover) - 故障转移配置
- [上下文管理](/zh-CN/concepts/context) - 上下文优化
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

**选择合适的模型，让 AI 助手既聪明又高效！** 🦞
