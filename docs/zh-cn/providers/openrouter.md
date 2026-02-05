---
summary: "OpenRouter 统一 API 聚合服务完全指南，包含配置、最佳实践和多模型管理"
read_when:
  - 你想要一个用于许多 LLM 的单一 API 密钥
  - 你想通过 OpenRouter 在 OpenClaw 中运行模型
  - 你需要灵活切换不同提供商的模型
title: "OpenRouter 完全配置指南"
---

# OpenRouter 完全配置指南

本文档详细介绍如何在 OpenClaw 中配置和使用 OpenRouter 统一 API 聚合服务。完成本指南后，你将理解 OpenRouter 的价值主张，掌握多模型统一管理的技能，并能够灵活切换不同提供商的模型。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 OpenRouter 的核心概念和价值
- [ ] 能够完成 OpenRouter 的基础配置
- [ ] 掌握模型引用格式 `openrouter/<provider>/<model>`
- [ ] 知道如何查看可用模型列表

### 进阶目标（建议掌握）⭐⭐

- [ ] 根据需求选择最优模型
- [ ] 配置多模型回退和切换
- [ ] 理解成本追踪和优化策略
- [ ] 能够诊断连接问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计多提供商组合策略
- [ ] 实现成本最优化方案
- [ ] 建立模型性能基准测试

---

## 核心概念解析

### 为什么选择 OpenRouter

**问题背景**：管理多个 AI 提供商需要维护多个 API 密钥、不同的 SDK、各自的计费系统。

**OpenRouter 的解决方案**：

| 特性 | 说明 | 优势 |
|------|------|------|
| **统一 API** | 单一端点访问多模型 | 简化集成 |
| **单一密钥** | 一个 API 密钥访问所有模型 | 降低管理复杂度 |
| **透明定价** | 统一的计费系统 | 清晰成本 |
| **灵活切换** | 无需改代码即可切换模型 | 快速实验 |
| **模型聚合** | 200+ 模型可选 | 最大灵活性 |

**适用场景**：

- 需要对比不同模型性能
- 希望灵活切换提供商
- 简化多提供商管理
- 需要统一的计费系统

### OpenRouter vs 直接使用提供商

| 特性 | OpenRouter | 直接使用提供商 |
|------|------------|----------------|
| 模型数量 | 200+ | 1 个提供商 |
| API 密钥 | 1 个 | 多个 |
| SDK | OpenAI 兼容 | 各家不同 |
| 计费 | 统一账单 | 各自账单 |
| 延迟 | 额外一层 | 更低 |
| 成本 | 可能略高 | 可能略低 |
| 灵活性 | 最高 | 受限 |

### 模型引用格式

OpenRouter 使用层级化的模型引用格式：

```
openrouter/<provider>/<model-name>
```

**示例**：

| 提供商 | 原始模型 | OpenRouter 引用 |
|--------|----------|-----------------|
| Anthropic | claude-sonnet-4-20250514 | `openrouter/anthropic/claude-sonnet-4-20250514` |
| OpenAI | gpt-4o | `openrouter/openai/gpt-4o` |
| Google | gemini-pro | `openrouter/google/gemini-pro` |
| Meta | llama-3.1-405b | `openrouter/meta/llama-3.1-405b` |

---

## 快速开始

### 获取 API 密钥

1. 访问 [OpenRouter](https://openrouter.ai)
2. 注册账户并登录
3. 导航至「Keys」页面
4. 点击「Create Key」
5. 设置密钥名称和限额
6. **复制并保存密钥**

### 基础配置

**命令行配置**：

```bash
openclaw onboard --auth-choice apiKey \
  --token-provider openrouter \
  --token "$OPENROUTER_API_KEY"
```

**配置文件**：

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: {
        primary: "openrouter/anthropic/claude-sonnet-4-20250514"
      }
    }
  }
}
```

### 验证配置

```bash
# 查看可用模型
openclaw models list | grep openrouter

# 测试连接
openclaw health
```

---

## 模型选择与管理

### 查看可用模型

```bash
# OpenRouter 查看
curl -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  https://openrouter.ai/api/v1/models

# OpenClaw 查看
openclaw models list | grep openrouter
```

### 推荐模型组合

**平衡方案**（推荐）：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/anthropic/claude-sonnet-4-20250514",
        "fallbacks": [
          "openrouter/openai/gpt-4o",
          "openrouter/google/gemini-pro"
        ]
      }
    }
  }
}
```

**成本优化方案**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/openai/gpt-4o-mini",
        "fallbacks": [
          "openrouter/anthropic/claude-haiku-3-5-20241022"
        ]
      }
    }
  }
}
```

**最高性能方案**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "openrouter/meta/llama-3.1-405b",
          "openrouter/openai/gpt-4o"
        ]
      }
    }
  }
}
```

### 模型性能对比

| 模型 | 来源 | 智能 | 速度 | 成本 | 适用场景 |
|------|------|------|------|------|----------|
| Claude Opus 4 | Anthropic | ⭐⭐⭐⭐⭐ | 中 | 高 | 复杂推理 |
| Claude Sonnet 4 | Anthropic | ⭐⭐⭐⭐ | 快 | 中 | 日常任务 |
| GPT-4o | OpenAI | ⭐⭐⭐⭐ | 中 | 中 | 通用 |
| GPT-4o Mini | OpenAI | ⭐⭐⭐ | 快 | 低 | 简单任务 |
| Llama 3.1 405B | Meta | ⭐⭐⭐⭐ | 中 | 低 | 开源偏好 |
| Gemini Pro | Google | ⭐⭐⭐⭐ | 快 | 中 | 多模态 |

---

## 成本管理

### 理解 OpenRouter 定价

OpenRouter 作为聚合层，会加上服务费：

```
实际成本 = 提供商原始价格 × (1 + OpenRouter 服务费)
```

**服务费**：约 0.5% - 1%

### 成本追踪

```bash
# 查看 OpenRouter 使用情况
# 访问：https://openrouter.ai/activity

# 通过 OpenClaw 查看
openclaw usage --cost
```

### 成本优化策略

**策略一：模型分级**

```json5
{
  "agents": {
    "list": [
      {
        "id": "complex",
        "model": "openrouter/anthropic/claude-opus-4-20250514"
      },
      {
        "id": "simple",
        "model": "openrouter/openai/gpt-4o-mini"
      }
    ]
  }
}
```

**策略二：设置使用限额**

在 OpenRouter 仪表板设置每日/每月限额。

---

## 高级配置

### 自定义请求头

```json5
{
  "models": {
    "providers": {
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}",
        "headers": {
          "HTTP-Referer": "https://your-site.com",
          "X-Title": "Your App"
        }
      }
    }
  }
}
```

### 代理配置

```json5
{
  "models": {
    "providers": {
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}",
        "proxy": "http://your-proxy:7890"
      }
    }
  }
}
```

---

## 故障排查指南

### 常见问题与解决方案

#### 问题一：认证失败（401）

**症状**：
```
Error: Invalid API Key
Status: 401
```

**排查步骤**：

```bash
# 1. 验证 API Key
curl -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  https://openrouter.ai/api/v1/models

# 2. 检查 API Key 格式
echo $OPENROUTER_API_KEY

# 3. 检查是否包含 'sk-or-' 前缀
```

**解决方案**：
- 在 OpenRouter 仪表板重新创建 API Key
- 更新环境变量
- 验证 Key 是否被禁用

#### 问题二：模型不可用

**症状**：
```
Error: Model not found
Status: 404
```

**排查步骤**：

```bash
# 1. 查看可用模型列表
openclaw models list | grep openrouter

# 2. 检查模型名称格式
# 正确：openrouter/anthropic/claude-sonnet-4-20250514
# 错误：openrouter/claude-sonnet-4-20250514
```

#### 问题三：速率限制

**症状**：
```
Error: Rate limit exceeded
Status: 429
```

**排查步骤**：

```bash
# 1. 查看当前使用量
# 访问：https://openrouter.ai/activity

# 2. 检查账户限额
# 访问：https://openrouter.ai/settings/limits
```

**解决方案**：
- 升级账户配额
- 实现请求队列
- 使用回退模型

#### 问题四：连接超时

**症状**：
```
Error: Connection timeout
Status: 000
```

**排查步骤**：

```bash
# 1. 测试网络连接
curl -v https://openrouter.ai/api/v1/models

# 2. 检查代理设置
env | grep -i proxy

# 3. 测试 OpenRouter 可达性
ping openrouter.ai
```

**解决方案**：

```json5
{
  "models": {
    "providers": {
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}",
        "timeout": 60000,
        "proxy": "http://your-proxy:7890"
      }
    }
  }
}
```

---

## 适用场景分析

### 推荐使用 OpenRouter 的场景

| 场景 | 优势 | 示例 |
|------|------|------|
| 模型对比测试 | 快速切换，无需改代码 | A/B 测试不同模型 |
| 开发测试环境 | 统一接口，简化开发 | 开发环境使用 |
| 多提供商需求 | 单一密钥，多模型 | 灵活切换 |
| 统一计费 | 透明成本 | 团队协作 |

### 不适用场景

- **生产环境高要求**：额外延迟可能不可接受
- **成本极度敏感**：OpenRouter 服务费
- **固定单一模型**：直接使用提供商更简单

### 混合策略

```json5
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "openrouter/openai/gpt-4o",
          "openrouter/anthropic/claude-sonnet-4-20250514"
        ]
      }
    }
  }
}
```

---

## 相关命令速查

| 命令 | 说明 |
|------|------|
| `openclaw onboard --auth-choice apiKey --token-provider openrouter` | 配置 OpenRouter |
| `openclaw models list` | 列出可用模型 |
| `openclaw health` | 检查认证状态 |
| `openclaw usage --cost` | 查看成本统计 |

---

## 相关文档

- [AI 提供商概览](/zh-CN/providers/index) - 所有提供商对比
- [OpenAI 配置](/zh-CN/providers/openai) - 直接使用 OpenAI
- [Anthropic 配置](/zh-CN/providers/anthropic) - 直接使用 Anthropic
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [故障排查](/zh-CN/help/troubleshooting) - 常见问题解决

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：配置 OpenRouter 并测试

**步骤**：
1. 获取 OpenRouter API Key
2. 配置 OpenClaw
3. 测试模型调用
4. 查看可用模型列表

**成功标准**：能够使用 OpenRouter 调用任意模型

### 练习 2：模型对比测试 ⭐⭐

**任务**：对比不同模型的表现

**步骤**：
1. 选择 3 个不同提供商的模型
2. 配置回退链
3. 发送相同请求到不同模型
4. 记录响应质量和延迟

**成功标准**：有完整的对比报告

### 练习 3：成本优化 ⭐⭐⭐

**任务**：优化 OpenRouter 使用成本

**步骤**：
1. 分析当前使用模式
2. 配置模型分级
3. 设置使用限额
4. 监控成本变化

**成功标准**：成本降低 30% 以上

---

## 自检清单

### 概念理解
- [ ] 理解 OpenRouter 的价值主张
- [ ] 知道模型引用格式
- [ ] 理解成本计算方式

### 动手能力
- [ ] 能够完成 OpenRouter 配置
- [ ] 能够配置模型回退
- [ ] 能够查看成本统计

### 问题解决
- [ ] 能够诊断认证问题
- [ ] 能够处理速率限制
- [ ] 能够优化使用成本
