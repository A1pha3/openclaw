---
summary: "OpenClaw 模型提供商快速入门指南，包含选择策略、认证流程和配置示例"
read_when:
  - 您想选择模型提供商
  - 您需要 LLM 身份验证和模型选择的快速设置示例
  - 您想了解推荐的提供商配置
title: "模型提供商快速入门"
---

# 模型提供商快速入门

本文档帮助你快速选择和配置适合的 AI 模型提供商。完成本指南后，你将能够根据需求选择最合适的提供商，完成基础配置，并理解高级配置选项。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 了解 OpenClaw 支持的主要模型提供商
- [ ] 能够完成任意一个提供商的认证配置
- [ ] 理解模型引用格式 `<provider>/<model-name>`
- [ ] 知道如何查看可用模型列表

### 进阶目标（建议掌握）⭐⭐

- [ ] 根据业务需求选择最适合的提供商
- [ ] 配置多提供商回退机制
- [ ] 理解不同提供商的定价模型
- [ ] 能够进行成本估算和优化

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计多提供商组合策略
- [ ] 建立成本监控和告警机制
- [ ] 制定团队使用规范

---

## 核心设计理念

### 为什么需要多提供商选择

OpenClaw 的多提供商设计旨在解决以下问题：

1. **避免供应商锁定**：不依赖单一服务商
2. **成本优化**：根据任务选择性价比最高的方案
3. **高可用保障**：主提供商不可用时自动切换
4. **场景适配**：不同任务使用最适合的模型

### 选择提供商的核心考量

| 考量因素 | 问题 | 推荐方案 |
|----------|------|----------|
| **预算** | 预算是否充足？ | 充足选 Anthropic/OpenAI，紧张选 Qwen/Ollama |
| **隐私** | 数据是否敏感？ | 敏感选 Ollama（本地），一般可选云服务 |
| **中文** | 是否主要处理中文？ | 中文优先选 Moonshot/GLM/Qwen |
| **性能** | 任务复杂度如何？ | 复杂选 Claude Opus 4，简单选轻量模型 |
| **延迟** | 实时性要求？ | 高要求选 Ollama（本地）或选择就近区域 |

---

## 推荐配置方案

### 首选：Venice AI（隐私优先）

**推荐理由**：
- 隐私优先的推理服务
- 可选使用 Claude Opus 处理复杂任务
- 提供本地部署选项
- 灵活的定价模型

**默认配置**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "venice/llama-3.3-70b",
        "fallbacks": ["venice/claude-opus-45"]
      }
    }
  }
}
```

**详细配置**：参见 [Venice AI 配置](/zh-CN/providers/venice)

---

## 快速入门（两步完成）

### 第一步：与提供商认证

```bash
# 运行配置向导
openclaw onboard

# 或者直接指定认证方式
openclaw onboard --auth-choice anthropic-api-key
```

### 第二步：设置默认模型

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5"
      }
    }
  }
}
```

---

## 支持的提供商目录

### 云服务提供商

| 提供商 | 认证方式 | 特点 | 文档链接 |
|--------|----------|------|----------|
| **OpenAI** | API Key / Codex | 生态系统最成熟 | [OpenAI 配置](/zh-CN/providers/openai) |
| **Anthropic** | API Key / setup-token | 长上下文最强 | [Anthropic 配置](/zh-CN/providers/anthropic) |
| **OpenRouter** | API Key | 多模型聚合 | [OpenRouter 配置](/zh-CN/providers/openrouter) |
| **Vercel AI Gateway** | API Key | Vercel 托管 | [Vercel AI Gateway 配置](/zh-CN/providers/vercel-ai-gateway) |

### 国产模型提供商

| 提供商 | 认证方式 | 特点 | 文档链接 |
|--------|----------|------|----------|
| **Moonshot AI** | API Key | Kimi 系列，中文优化 | [Moonshot 配置](/zh-CN/providers/moonshot) |
| **GLM** | API Key | 智谱 AI，国产首选 | [GLM 配置](/zh-CN/providers/glm) |
| **Qwen** | OAuth | 免费额度，通义千问 | [Qwen 配置](/zh-CN/providers/qwen) |
| **MiniMax** | API Key | MiniMax 系列 | [MiniMax 配置](/zh-CN/providers/minimax) |
| **Z.AI** | API Key | 新兴国产模型 | [Z.AI 配置](/zh-CN/providers/zai) |

### 本地部署

| 提供商 | 认证方式 | 特点 | 文档链接 |
|--------|----------|------|----------|
| **Ollama** | 无需密钥 | 本地运行，完全隐私 | [Ollama 配置](/zh-CN/providers/ollama) |

### 企业级服务

| 提供商 | 认证方式 | 特点 | 文档链接 |
|--------|----------|------|----------|
| **Amazon Bedrock** | IAM 角色 | 企业级托管 | [Bedrock 配置](/zh-CN/bedrock) |

---

## 模型引用格式

### 标准格式

```
<provider>/<model-name>
```

### 示例

| 提供商 | 模型 | 引用格式 |
|--------|------|----------|
| Anthropic | Claude Sonnet 4 | `anthropic/claude-sonnet-4-20250514` |
| OpenAI | GPT-4o | `openai/gpt-4o` |
| OpenRouter | Claude via OpenRouter | `openrouter/anthropic/claude-sonnet-4-5` |
| Ollama | Llama 3 | `ollama/llama3` |

---

## 快速对比

### 性能对比

| 模型 | 智能水平 | 速度 | 成本 | 适用场景 |
|------|----------|------|------|----------|
| Claude Opus 4 | ⭐⭐⭐⭐⭐ | 中 | 高 | 复杂推理、长文档 |
| Claude Sonnet 4 | ⭐⭐⭐⭐ | 快 | 中 | 日常对话、代码 |
| GPT-4o | ⭐⭐⭐⭐ | 中 | 中 | 通用场景 |
| Llama 3 (Ollama) | ⭐⭐⭐ | 快 | 免费 | 本地部署、隐私 |
| Kimi (Moonshot) | ⭐⭐⭐⭐ | 快 | 中 | 中文场景 |

### 成本对比（每百万 tokens）

| 提供商 | 输入成本 | 输出成本 |
|--------|----------|----------|
| Claude Opus 4 | $15.00 | $75.00 |
| Claude Sonnet 4 | $3.00 | $15.00 |
| GPT-4o | $2.50 | $10.00 |
| GPT-4o Mini | $0.15 | $0.60 |
| Ollama | 免费 | 免费 |

---

## 高级配置

### 多模型回退

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": [
          "anthropic/claude-sonnet-4-20250514",
          "openai/gpt-4o",
          "ollama/llama3"
        ]
      }
    }
  }
}
```

### 按任务分配模型

```json5
{
  "agents": {
    "list": [
      {
        "id": "coder",
        "model": "anthropic/claude-opus-4-5"
      },
      {
        "id": "chat",
        "model": "anthropic/claude-sonnet-4-20250514"
      },
      {
        "id": "fast",
        "model": "anthropic/claude-haiku-3-5-20241022"
      }
    ]
  }
}
```

---

## 相关文档

- [AI 提供商完全指南](/zh-CN/providers/index) - 详细配置和故障排查
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [OAuth 概念](/zh-CN/concepts/oauth) - 认证机制
- [配置参考](/zh-CN/config/reference) - 完整配置选项
- [故障排查](/zh-CN/help/troubleshooting) - 常见问题解决

---

## 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw onboard` | 运行配置向导 |
| `openclaw models list` | 列出可用模型 |
| `openclaw models status` | 查看认证状态 |
| `openclaw usage` | 查看使用统计 |
| `openclaw health` | 检查系统健康 |

---

## 自检清单

### 概念理解
- [ ] 知道如何选择合适的提供商
- [ ] 理解模型引用格式
- [ ] 了解不同提供商的特点

### 动手能力
- [ ] 能够完成任意提供商的认证
- [ ] 能够配置默认模型
- [ ] 能够查看可用模型列表

### 进阶能力
- [ ] 能够配置多模型回退
- [ ] 能够进行成本估算
- [ ] 能够根据场景选择最佳方案
