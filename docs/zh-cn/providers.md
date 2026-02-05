---
summary: "AI 提供商完整指南——配置 Anthropic Claude、OpenAI GPT、Moonshot Kimi、GLM、Ollama 等 AI 模型，包含 API 配置、模型选择、成本优化和故障排除"
read_when:
  - 选择和配置 AI 模型提供商
  - 设置 API 密钥和认证
  - 了解不同模型的特点和适用场景
  - 优化 AI 响应质量和成本
title: "AI 提供商"
---

# 🤖 AI 提供商

OpenClaw 支持多种 AI 模型提供商，让你能够根据需求选择最适合的 AI 大脑。不同的提供商具有各自的特点、价格和适用场景，正确选择和配置 AI 提供商是获得最佳使用体验的关键。本文档将详细介绍各提供商的特点、配置方法和最佳实践。

> **AI 提供商的核心作用**
>
> AI 提供商是 OpenClaw 系统的"大脑"，负责理解和生成自然语言、处理复杂任务、提供智能建议。每个提供商都有其独特的优势：Anthropic Claude 在编程和复杂推理方面表现出色；OpenAI GPT 具有广泛的生态支持；国产模型在中文场景下有独特优势。理解这些差异并选择合适的提供商，能够显著提升 AI 助手的效果。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解各 AI 提供商的特点和差异
- 完成至少一个提供商的 API 配置
- 选择适合自己需求的 AI 模型
- 理解 API 调用和 token 消耗的基本概念

### 进阶目标（建议掌握）

- 配置多个提供商实现故障转移
- 优化提示词以获得更好的模型输出
- 管理 API 成本和调用限制
- 实现模型切换和负载均衡

### 专家目标（挑战）

- 开发自定义提供商适配器
- 实现提示词优化和评估系统
- 设计成本监控和告警机制
- 集成专业领域的微调模型

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速上手** | 首次配置 AI 模型 | 20 分钟 | 选择一个提供商，完成基础配置 |
| **进阶配置** | 需要优化模型效果 | 1 小时 | 多提供商配置、成本管理 |
| **专家部署** | 生产环境运维 | 2 小时 | 高可用、成本优化、监控 |

---

## 🧠 原理解释：AI 模型是如何工作的

在选择和配置 AI 提供商之前，理解 AI 模型的基本工作原理有助于更好地使用它们。

### 大语言模型的核心概念

**Token：语言的基本单位**

AI 模型不像人类那样处理"单词"，而是处理称为"Token"的基本单位。一个 Token 可能是一个完整的单词，也可能只是一个单词的一部分。例如：

- 英文单词 "learning" 通常是 1 个 Token
- 中文 "学习" 通常是 1-2 个 Token
- 标点符号和空格也占用 Token

理解 Token 的概念对于管理成本和控制响应长度至关重要。大多数 API 的计费都是基于 Token 数量，而不是请求次数。

**上下文窗口：模型的"记忆"**

上下文窗口是指模型在生成响应时能够"看到"的 Token 数量。较大的上下文窗口意味着模型可以处理更长的对话历史和更复杂的指令。例如：

- Claude 3.5 Sonnet：200K Token 上下文窗口
- GPT-4o：128K Token 上下文窗口
- Kimi K2：200K Token 上下文窗口

**推理能力：模型的"思考"过程**

不同模型在不同类型的任务上表现不同。推理能力包括：

- **逻辑推理**——数学计算、因果分析
- **代码理解**——程序分析、调试建议
- **创意写作**——文章生成、风格模仿
- **多语言处理**——翻译、跨语言理解

### 提供商架构对比

| 提供商 | API 架构 | 认证方式 | 速率限制 | 特色功能 |
|--------|----------|----------|----------|----------|
| Anthropic | REST + WebSocket | API Key | 动态调整 | 系统提示词优化 |
| OpenAI | REST + SSE | API Key | 固定配额 | 函数调用优化 |
| 国内厂商 | REST | API Key + 备案 | 灵活 | 中文优化 |

---

## 🎛️ 提供商对比与选择

### 推荐提供商概览

| 提供商 | 核心优势 | 适用场景 | 价格定位 |
|--------|----------|----------|----------|
| **Anthropic Claude** | 编程能力强、推理优秀 | 开发、日常对话 | 中高 |
| **OpenAI GPT** | 生态丰富、功能全面 | 通用场景 | 中高 |
| **Moonshot Kimi** | 中文优化、长文本 | 中文创作、长文档 | 中等 |
| **GLM** | 国产模型、中文强 | 中文场景 | 中等 |
| **MiniMax** | 成本低、速度快 | 高频调用 | 低 |
| **Ollama** | 本地运行、隐私优先 | 隐私敏感场景 | 无 |

### 提供商选择决策树

```
我的主要使用场景是什么？
├── 专业编程和复杂推理
│   └── 推荐 Anthropic Claude
│
├── 需要丰富的第三方集成
│   └── 推荐 OpenAI GPT
│
├── 主要使用中文
│   ├── 需要长文本处理 → Moonshot Kimi
│   ├── 需要平衡成本 → GLM
│   └── 需要快速响应 → MiniMax
│
└── 隐私优先或离线使用
    └── 推荐 Ollama（本地部署）
```

---

## 🏢 Anthropic（Claude）

Anthropic 是一家专注于 AI 安全和研究的公司，其 Claude 模型在编程能力、逻辑推理和安全性方面表现出色。OpenClaw 将 Anthropic 作为默认推荐提供商。

### 为什么选择 Claude

Claude 的独特优势在于：**卓越的编程能力**——Claude 在代码生成、理解和调试方面表现优秀，特别适合作为开发助手；**安全性设计**——Claude 从设计之初就注重安全性和有用性，减少有害输出；**长上下文处理**——Claude 3 系列支持超长上下文窗口，适合处理长文档和复杂对话；**系统提示词优化**——Claude 对系统提示词有特殊优化，可以更好地理解和执行复杂指令。

### 支持的模型

| 模型名称 | 上下文窗口 | 特点 | 推荐场景 |
|----------|------------|------|----------|
| Claude Opus 4 | 200K | 最强能力，适合复杂任务 | 专业编程、深度分析 |
| Claude Sonnet 4 | 200K | 平衡性能与成本 | 日常对话、一般任务 |
| Claude Haiku 4 | 200K | 快速响应，适合简单任务 | 快速问答、简单查询 |

### API 配置

**方法一：环境变量配置（推荐）**

```bash
# 设置 API 密钥
export ANTHROPIC_API_KEY="sk-ant-api03-xxxxx"

# 验证配置
echo $ANTHROPIC_API_KEY
```

**方法二：配置文件配置**

在 `~/.openclaw/openclaw.json` 中添加：

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "sk-ant-api03-xxxxx", // 建议使用环境变量
        defaultModel: "anthropic/claude-sonnet-4-20250514"
      }
    }
  }
}
```

**方法三：交互式配置**

```bash
# 启动配置向导
openclaw config edit

# 或者使用 set 命令
openclaw config set models.providers.anthropic.apiKey "sk-ant-api03-xxxxx"
```

### 获取 API 密钥

1. 访问 [Anthropic Console](https://console.anthropic.com/)
2. 登录或创建账户
3. 导航到 "API Keys" 部分
4. 点击 "Create Key" 生成新的 API 密钥
5. 复制并安全保存密钥（密钥只会显示一次）

### 高级配置

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}", // 使用环境变量
        defaultModel: "anthropic/claude-sonnet-4-20250514",
        
        // API 端点（通常不需要修改）
        apiEndpoint: "https://api.anthropic.com/v1",
        
        // 请求选项
        options: {
          maxTokens: 8192,          // 最大输出 Token
          temperature: 0.7,         // 创造性（0-1）
          topP: 0.9,                // 采样策略
        },
        
        // 超时设置
        timeout: 60000,             // 毫秒
        maxRetries: 3               // 最大重试次数
      }
    }
  }
}
```

---

## 🏢 OpenAI（GPT）

OpenAI 是 AI 领域的先驱，其 GPT 系列模型拥有最广泛的生态支持和最成熟的应用场景。

### 为什么选择 GPT

GPT 的独特优势在于：**生态成熟**——大量的第三方工具、库和集成方案；**函数调用**——GPT 的函数调用机制非常成熟，适合构建 AI 应用；**多模态能力**——支持图像、音频等多种输入；**社区资源**——丰富的预训练提示词和最佳实践。

### 支持的模型

| 模型名称 | 上下文窗口 | 特点 | 推荐场景 |
|----------|------------|------|----------|
| GPT-4o | 128K | 最新旗舰模型，多模态 | 复杂任务、图像理解 |
| GPT-4o-mini | 128K | 成本优化版 | 简单任务、高频调用 |
| GPT-4 Turbo | 128K | 平衡性能与成本 | 通用场景 |

### API 配置

**方法一：环境变量配置**

```bash
# 设置 API 密钥
export OPENAI_API_KEY="sk-xxxxx"
```

**方法二：配置文件配置**

```json5
{
  models: {
    providers: {
      openai: {
        apiKey: "${OPENAI_API_KEY}",
        defaultModel: "openai/gpt-4o-2024-08-06",
        organization: "org-xxxxx", // 可选，组织 ID
        
        options: {
          maxTokens: 4096,
          temperature: 0.7,
        }
      }
    }
  }
}
```

### 函数调用配置

GPT 的函数调用是构建 AI 应用的核心能力：

```json5
{
  tools: {
    enabled: true,
    openaiFunctions: {
      enabled: true,
      // 函数定义
      functions: [
        {
          name: "get_weather",
          description: "获取指定位置的天气信息",
          parameters: {
            type: "object",
            properties: {
              location: {
                type: "string",
                description: "城市名称，如 北京"
              }
            },
            required: ["location"]
          }
        }
      ]
    }
  }
}
```

---

## 🏢 Moonshot（月之暗面）

Moonshot AI 是中国的 AI 初创公司，其 Kimi 系列模型在中文理解和长文本处理方面表现优秀。

### 为什么选择 Kimi

Kimi 的独特优势在于：**中文优化**——对中文语境有深入理解，成语、俗语、文化背景都能准确把握；**超长上下文**——支持高达 200 万 Token 的上下文窗口，适合处理长文档；**长文本能力**——特别擅长长文章写作、长对话保持；**本土化服务**——国内节点，延迟低，服务稳定。

### 支持的模型

| 模型名称 | 上下文窗口 | 特点 | 推荐场景 |
|----------|------------|------|----------|
| Kimi K2 | 200K | 最新旗舰 | 长文本、专业写作 |
| Kimi K1.5 | 128K | 平衡性能 | 日常对话、翻译 |

### API 配置

```json5
{
  models: {
    providers: {
      moonshot: {
        apiKey: "${MOONSHOT_API_KEY}",
        defaultModel: "moonshot/kimi-k2",
        apiEndpoint: "https://api.moonshot.cn/v1",
        
        options: {
          maxTokens: 8192,
          temperature: 0.7,
        }
      }
    }
  }
}
```

### 获取 API 密钥

1. 访问 [Moonshot AI开放平台](https://platform.moonshot.cn/)
2. 注册账户并完成实名认证
3. 创建 API Key
4. 充值账户（按 Token 计费）

---

## 🏢 智谱 AI（GLM）

智谱 AI 是清华大学技术团队创立的 AI 公司，其 GLM 系列模型在中文场景下有出色表现。

### 为什么选择 GLM

GLM 的独特优势在于：**学术背景**——基于清华大学的前沿研究；**中文理解**——对中文语义有深入理解；**多版本选择**——提供从轻量到旗舰的多个版本；**企业服务**——提供专业的企业级支持。

### API 配置

```json5
{
  models: {
    providers: {
      glm: {
        apiKey: "${GLM_API_KEY}",
        defaultModel: "glm-4-plus",
        apiEndpoint: "https://open.bigmodel.cn/api/paas/v4",
        
        options: {
          maxTokens: 4096,
          temperature: 0.9,
        }
      }
    }
  }
}
```

---

## 🏢 MiniMax

MiniMax 是一家专注于高效能 AI 模型的公司，其模型以低成本和快速响应著称。

### 为什么选择 MiniMax

MiniMax 的独特优势在于：**成本优势**——相比国际模型，价格更具竞争力；**响应速度快**——推理效率高，延迟低；**稳定服务**——国内节点，服务稳定可靠；**简洁接口**——API 设计简洁，易于集成。

### API 配置

```json5
{
  models: {
    providers: {
      minimax: {
        apiKey: "${MINIMAX_API_KEY}",
        defaultModel: "abab6.5s-chat",
        apiEndpoint: "https://api.minimax.chat/v1",
        
        options: {
          maxTokens: 4096,
          temperature: 0.7,
        }
      }
    }
  }
}
```

---

## 🏢 Ollama（本地部署）

Ollama 允许你在本地机器上运行 AI 模型，完全掌控数据和计算资源。

### 为什么选择 Ollama

Ollama 的独特优势在于：**隐私保护**——数据不需要离开本地；**零 API 成本**——无需支付 API 调用费用；**离线可用**——不依赖网络连接；**完全控制**——可以部署任何兼容的模型。

### 安装和配置

**步骤一：安装 Ollama**

```bash
# macOS/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Windows
# 下载安装包：https://ollama.ai/download
```

**步骤二：拉取模型**

```bash
# 拉取 llama3.2（推荐 3GB+ RAM）
ollama pull llama3.2

# 拉取 qwen2.5（中文优化，2GB+ RAM）
ollama pull qwen2.5

# 拉取 deepseek-r1（编程优化）
ollama pull deepseek-r1
```

**步骤三：OpenClaw 配置**

```json5
{
  models: {
    providers: {
      ollama: {
        apiEndpoint: "http://localhost:11434/v1",
        defaultModel: "llama3.2",
        
        options: {
          // 本地模型通常需要更高的 temperature
          temperature: 0.7,
          numCtx: 4096,  // 上下文长度
        }
      }
    }
  }
}
```

### Ollama 模型推荐

| 模型名称 | 大小 | 内存需求 | 特点 |
|----------|------|----------|------|
| llama3.2 | 2GB | 4GB | 通用能力强 |
| qwen2.5 | 2GB | 4GB | 中文优秀 |
| deepseek-r1 | 4GB | 8GB | 编程专用 |
| mistral | 4GB | 8G | 平衡性能 |

---

## 🔧 模型选择与优化

### 按任务类型选择模型

| 任务类型 | 推荐模型 | 理由 |
|----------|----------|------|
| **编程开发** | Claude Opus 4 | 代码理解能力强 |
| **日常对话** | Claude Haiku / GPT-4o-mini | 快速响应 |
| **长文本写作** | Kimi K2 | 200K 上下文 |
| **中文创作** | Kimi / GLM | 中文优化 |
| **成本敏感** | MiniMax / Ollama | 价格低 |
| **隐私优先** | Ollama | 本地部署 |

### 提示词优化技巧

**技巧一：明确任务类型**

```markdown
<!-- 不推荐 -->
帮我写一些内容

<!-- 推荐 -->
请帮我写一封专业的商务邮件，收件人是合作伙伴，
主题是讨论项目合作细节，语调要正式但友好。
```

**技巧二：提供上下文**

```markdown
<!-- 不推荐 -->
这个代码有什么问题？

<!-- 推荐 -->
以下是我用 Python 编写的函数，用于计算斐波那契数列。
但是当输入大于 50 时，程序运行非常慢。请分析问题并给出优化方案：
[代码内容]
```

**技巧三：指定输出格式**

```markdown
<!-- 不推荐 -->
告诉我关于这个项目的所有信息

<!-- 推荐 -->
请用以下格式总结项目信息：
## 项目概述
[一句话介绍]

## 核心功能
- 功能 1：[描述]
- 功能 2：[描述]

## 技术栈
- 前端：[技术]
- 后端：[技术]
```

---

## 💰 成本管理

### 理解定价模型

| 提供商 | 输入定价 | 输出定价 | 计量单位 |
|--------|----------|----------|----------|
| Claude | $3/1M tokens | $15/1M tokens | 每百万 Token |
| GPT-4o | $2.5/1M tokens | $10/1M tokens | 每百万 Token |
| Kimi | ¥20/1M tokens | ¥20/1M tokens | 每百万 Token |
| Ollama | 免费 | 免费 | 本地计算 |

### 成本优化策略

**策略一：选择合适的模型**

简单任务使用轻量模型，复杂任务再切换到旗舰模型：

```json5
{
  models: {
    providers: {
      anthropic: {
        // 配置默认使用轻量模型
        defaultModel: "anthropic/claude-haiku-4-20250514",
        
        // 为特定任务配置不同模型
        models: {
          "anthropic/claude-opus-4-20250514": {
            description: "复杂编程任务",
            enabled: true
          },
          "anthropic/claude-sonnet-4-20250514": {
            description: "日常对话",
            enabled: true
          }
        }
      }
    }
  }
}
```

**策略二：限制上下文长度**

```json5
{
  models: {
    defaults: {
      // 限制最大上下文
      maxContextTokens: 50000,
      // 限制响应长度
      maxResponseTokens: 2000
    }
  }
}
```

**策略三：监控使用量**

```bash
# 查看使用统计
openclaw usage

# 设置预算告警
openclaw config set models.budget.alertThreshold 0.8
openclaw config set models.budget.monthlyLimit 100
```

---

## 🚨 故障排除

### 问题一：API 密钥无效

**症状**：返回 401 错误，提示认证失败。

**解决方案**：
1. 检查 API 密钥是否正确复制，没有多余空格
2. 确认 API 密钥是否已激活
3. 验证密钥是否有足够的权限
4. 检查提供商账户是否正常

### 问题二：速率限制

**症状**：返回 429 错误，提示请求过于频繁。

**解决方案**：
1. 减少请求频率
2. 实现请求队列和重试机制
3. 考虑升级服务套餐
4. 使用缓存减少重复请求

### 问题三：响应超时

**症状**：请求长时间等待后超时。

**解决方案**：
1. 检查网络连接
2. 减少请求的复杂度
3. 增加超时设置
4. 考虑使用更快的模型

### 问题四：输出质量差

**症状**：AI 响应不准确或不相关。

**解决方案**：
1. 优化提示词，提供更多上下文
2. 尝试调整 temperature 参数
3. 切换到更高级的模型
4. 分解复杂问题为简单问题

---

## 📋 配置最佳实践

### 开发环境配置

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}",
        defaultModel: "anthropic/claude-sonnet-4-20250514",
        options: {
          temperature: 0.7,
          maxTokens: 4096
        }
      }
    }
  }
}
```

### 生产环境配置

```json5
{
  models: {
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}",
        defaultModel: "anthropic/claude-opus-4-20250514",
        
        // 生产环境优化
        options: {
          temperature: 0.5,  // 降低创造性，提高一致性
          maxTokens: 8192
        },
        
        // 故障转移配置
        failover: {
          enabled: true,
          providers: ["openai", "moonshot"]
        },
        
        // 缓存配置
        cache: {
          enabled: true,
          ttl: 3600  // 1小时缓存
        }
      }
    }
  }
}
```

---

## 📖 相关文档

- [配置参考](/zh-CN/config/reference)——完整配置选项
- [模型配置](/zh-CN/concepts/models)——模型概念详解
- [故障排除](/zh-CN/help/troubleshooting)——问题解决指南
- [成本优化](/zh-CN/concepts/models#成本优化)——高级成本管理
