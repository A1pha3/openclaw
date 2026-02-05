---
summary: "AI 模型提供商配置完全指南 - Anthropic、OpenAI、Ollama 等模型配置与选择策略"
read_when:
  - 配置 AI 模型访问
  - 设置 API Key 或 OAuth
  - 选择合适的模型提供商
  - 理解提供商特性和适用场景
title: "AI 提供商完全指南"
---

# 🤖 AI 提供商完全指南

本文档作为 OpenClaw AI 模型提供商配置的权威参考，帮助你理解不同提供商的特点、选择策略和最佳实践。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 AI 提供商的核心概念和 OpenClaw 的集成机制
- [ ] 掌握 API 密钥和 OAuth 两种认证方式的原理和适用场景
- [ ] 能够独立完成 Anthropic 或 OpenAI 的基础配置
- [ ] 理解模型引用格式 `<provider>/<model-name>` 的含义

### 进阶目标（建议掌握）⭐⭐

- [ ] 根据业务需求选择最适合的模型提供商
- [ ] 配置多提供商回退机制实现高可用
- [ ] 理解成本管理策略并优化使用成本
- [ ] 能够诊断和解决常见的认证问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计多提供商架构实现成本和性能的平衡
- [ ] 制定团队的模型使用规范和成本控制策略
- [ ] 能够针对特定场景定制提供商配置

---

## 核心设计思想

### 为什么需要多提供商支持

在理解具体配置之前，我们需要先理解**设计者为什么提供多提供商支持**。这不仅帮助你更好地使用 OpenClaw，还能让你在遇到问题时做出更好的设计决策。

**问题背景**：单一 AI 提供商存在以下风险：
- 服务商故障导致业务中断
- 特定模型在某些任务上表现不佳
- 成本控制缺乏灵活性
- 地理位置导致的延迟问题

**OpenClaw 的解决方案**：
1. **多提供商抽象层**：统一调用接口，屏蔽底层差异
2. **模型回退机制**：主模型不可用时自动切换
3. **认证配置文件**：支持多账户管理和 OAuth

**设计原则**：
- 灵活性优先：让用户能够自由选择
- 渐进式复杂度：简单配置即可上手，高级配置满足定制需求
- 安全性保障：敏感信息不硬编码，支持环境变量

---

## 支持的提供商概览

### 内置提供商

| 提供商 | 状态 | 核心优势 | 适用场景 |
|--------|------|----------|----------|
| [Anthropic](/zh-CN/providers/anthropic) | ✅ 稳定 | Claude 系列，长上下文强，指令遵循优秀 | 复杂推理、长文档分析、高级对话 |
| [OpenAI](/zh-CN/providers/openai) | ✅ 稳定 | GPT 系列，生态系统成熟 | 通用场景、代码生成、多模态 |
| [OpenRouter](/zh-CN/providers/openrouter) | ✅ 稳定 | 多模型聚合，统一 API | 模型对比、成本优化、灵活切换 |
| [Ollama](/zh-CN/providers/ollama) | ✅ 稳定 | 本地部署，零延迟，完全隐私 | 隐私敏感场景、离线使用、成本控制 |
| [Moonshot](/zh-CN/providers/moonshot) | ✅ 稳定 | Kimi 系列，中文优化 | 中文场景、长文本处理 |
| [GLM](/zh-CN/providers/glm) | ✅ 稳定 | 智谱 AI，国产模型 | 国产化需求、中文场景 |
| [MiniMax](/zh-CN/providers/minimax) | ✅ 稳定 | MiniMax 系列 | 特定优化场景 |
| [Qwen](/zh-CN/providers/qwen) | ✅ 稳定 | 通义千问，免费额度 | 成本敏感、中文场景 |

### 插件提供商

| 提供商 | 说明 | 安装方式 |
|--------|------|----------|
| GitHub Copilot | 代码辅助 | 插件市场 |
| Deepgram | 语音识别 | 插件市场 |
| Vercel AI Gateway | Vercel 托管 | 插件市场 |

---

## 专家思维模型：提供商选择框架

### 决策树

```
Q: 你的首要考虑因素是什么？
├── 隐私和数据安全
│   └── → Ollama（本地部署）
│
├── 成本控制
│   ├── 预算充足
│   │   └── → Anthropic Claude（最强性能）
│   ├── 预算中等
│   │   └── → OpenRouter（灵活切换）
│   └── 预算有限
│       └── → Qwen（免费额度）或 Ollama（本地）
│
├── 中文场景优化
│   └── → Moonshot / GLM / Qwen（国产模型）
│
├── 通用场景
│   └── → OpenAI（生态系统成熟）
│
└── 特定任务性能
    └── → 参考模型性能对比表
```

### 性能与成本权衡矩阵

| 优先级 | 推荐方案 | 说明 |
|--------|----------|------|
| 性能优先 | Claude Opus 4 | 最高智能，适合复杂任务 |
| 平衡方案 | Claude Sonnet 4 | 性能与速度的良好平衡 |
| 成本优先 | GPT-4o Mini / Haiku 3 | 低成本，适合简单任务 |
| 极速响应 | 本地 Ollama | 无网络延迟 |

---

## 认证方式详解

### 认证方式对比

| 认证方式 | 安全性 | 配置复杂度 | 适用场景 |
|----------|--------|------------|----------|
| API 密钥 | 中 | ⭐ 简单 | 快速上手、个人使用 |
| OAuth | 高 | ⭐⭐ 中等 | 企业环境、多用户 |
| Setup-Token | 高 | ⭐⭐ 中等 | Anthropic Claude 订阅 |

### API Key 方式

**原理**：使用提供商颁发的 API 密钥进行身份验证。

**配置**：

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"  // 推荐：使用环境变量
      },
      openai: {
        apiKey: "${OPENAI_API_KEY}"
      }
    }
  }
}
```

**命令行设置**：

```bash
# Anthropic
openclaw config set models.providers.anthropic.apiKey "sk-ant-api03-..."

# OpenAI
openclaw config set models.providers.openai.apiKey "sk-..."
```

**环境变量配置**：

```bash
# 设置环境变量
export ANTHROPIC_API_KEY="sk-ant-api03-..."
export OPENAI_API_KEY="sk-..."

# OpenClaw 会自动读取这些环境变量
```

### OAuth 方式（推荐）

**原理**：通过 OAuth 协议进行身份验证，无需存储 API 密钥，支持令牌自动刷新。

**配置**：

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

**设置步骤**：

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

## 模型选择与配置

### 模型引用格式

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

## 模型回退机制

### 为什么需要回退

在生产环境中，单一模型可能面临以下问题：
- API 限流导致请求失败
- 网络问题导致超时
- 模型服务临时不可用
- 认证令牌过期

回退机制确保在主模型不可用时，系统能够自动切换到备用模型，保证服务连续性。

### 配置回退模型

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

### 回退触发条件

模型回退会在以下情况触发：

1. **API 限流**：返回 429 状态码
2. **API 错误**：返回 5xx 状态码
3. **网络超时**：请求超过配置的超时时间
4. **认证失败**：401/403 状态码（不包括令牌过期）

---

## 成本管理

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

### 成本优化策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| 使用轻量模型 | 对于简单任务，使用 Haiku 而非 Opus | 简单问答、状态回复 |
| 减少上下文 | 只发送必要的对话历史 | 长对话场景 |
| 使用流式响应 | 减少等待时间，提升用户体验 | 实时交互场景 |
| 设置回退 | 使用低成本模型作为回退 | 生产环境高可用 |

---

## 地区与延迟优化

### 选择最近的端点

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "...",
        baseUrl: "https://api.anthropic.com"  // 默认端点
      },
      openai: {
        apiKey: "...",
        baseUrl: "https://api.openai.com/v1"  // 默认端点
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

## 故障排查指南

### 故障排查思维模型

```
1. 观察现象
   └── 记录错误信息、复现步骤、环境信息
   
2. 形成假设
   └── 基于经验和模式，推测可能的原因
   
3. 设计实验
   └── 确定如何验证假设（最小化实验）
   
4. 执行实验
   └── 控制变量，逐个验证假设
   
5. 得出结论
   └── 找到根本原因
   
6. 验证修复
   └── 确保修复有效且不引入新问题
```

### 常见问题与解决方案

#### 问题一：认证失败

**症状**：返回 401 错误，提示认证失败

**排查步骤**：

```bash
# 1. 检查 API Key 是否设置正确
echo $ANTHROPIC_API_KEY

# 2. 验证 API Key 是否有效
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
  https://api.anthropic.com/v1/messages \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'

# 3. 检查配置文件中的 API Key 引用
openclaw config get models.providers.anthropic.apiKey
```

**可能原因**：
- API Key 已过期或被撤销
- API Key 格式错误
- 环境变量未正确加载
- 配置文件语法错误

#### 问题二：模型不可用

**症状**：提示模型不存在或不支持

**排查步骤**：

```bash
# 1. 列出可用模型
openclaw models list

# 2. 检查模型名称是否正确
openclaw config get agents.defaults.model

# 3. 检查模型是否在提供商支持列表中
```

#### 问题三：速率限制

**症状**：返回 429 错误，提示请求过于频繁

**解决方案**：

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

**缓解策略**：
- 实现请求队列和限流
- 使用回退模型分散请求
- 升级提供商配额

---

## 提供商配置示例

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

### 多提供商高可用

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

## 模型性能对比

| 模型 | 速度 | 智能 | 成本 | 适用场景 |
|------|------|------|------|----------|
| Claude Opus 4 | 中 | 最高 | 高 | 复杂推理、长文档分析 |
| Claude Sonnet 4 | 快 | 高 | 中 | 日常对话、代码编写 |
| Claude Haiku 3 | 最快 | 中 | 低 | 简单任务、快速响应 |
| GPT-4o | 中 | 高 | 中 | 通用场景、多模态 |
| GPT-4o Mini | 快 | 中 | 低 | 成本敏感场景 |
| Llama 3 (Ollama) | 快 | 中 | 免费 | 本地部署、隐私敏感 |

---

## 相关命令速查

| 命令 | 说明 |
|------|------|
| `openclaw models list` | 列出可用模型 |
| `openclaw config get agents.defaults.model` | 查看当前模型 |
| `openclaw usage` | 查看使用情况 |
| `openclaw health` | 检查 AI 认证状态 |
| `openclaw onboard` | 运行配置向导 |

---

## 相关文档

- [Anthropic 配置](/zh-CN/providers/anthropic) - Claude 配置详解
- [OpenAI 配置](/zh-CN/providers/openai) - GPT 配置详解
- [Ollama 配置](/zh-CN/providers/ollama) - 本地模型配置
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [配置参考](/zh-CN/config/reference) - 完整配置选项
- [故障排查](/zh-CN/help/troubleshooting) - 常见问题解决

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：配置 Anthropic 作为默认提供商

**步骤**：
1. 获取 Anthropic API Key
2. 设置环境变量
3. 运行 `openclaw onboard` 完成配置
4. 验证配置：`openclaw health`

**成功标准**：能够成功调用 Claude 模型

### 练习 2：配置多提供商回退 ⭐⭐

**任务**：配置主备模型实现高可用

**步骤**：
1. 配置 Anthropic 和 OpenAI 两个提供商
2. 设置回退模型链
3. 测试回退机制（临时禁用主模型）

**成功标准**：主模型不可用时自动切换到备用模型

### 练习 3：成本优化实践 ⭐⭐⭐

**任务**：优化模型使用成本

**步骤**：
1. 分析当前使用情况：`openclaw usage --cost`
2. 识别高成本场景
3. 为不同任务配置不同模型
4. 设置使用限制

**成功标准**：成本降低 30% 以上，同时保持服务质量

---

## 自检清单

完成本章节学习后，请自检以下能力：

### 概念理解
- [ ] 能够解释 API Key 和 OAuth 的区别
- [ ] 理解模型回退机制的设计原理
- [ ] 知道如何根据场景选择合适的提供商

### 动手能力
- [ ] 能够独立完成任意提供商的配置
- [ ] 能够配置多提供商回退
- [ ] 能够诊断认证相关问题

### 问题解决
- [ ] 能够解决认证失败问题
- [ ] 能够处理速率限制
- [ ] 能够优化延迟和成本

---

**选择合适的 AI 模型，让您的助手既智能又高效！** 🦞
