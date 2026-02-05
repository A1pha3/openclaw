---
summary: "OpenAI API 密钥和 Codex 订阅配置完全指南，包含认证方式、最佳实践和故障排查"
read_when:
  - 你想在 OpenClaw 中使用 OpenAI 模型
  - 你想使用 Codex 订阅认证而不是 API 密钥
  - 你需要了解 API 密钥管理和安全最佳实践
title: "OpenAI 完全配置指南"
---

# OpenAI 完全配置指南

本文档详细介绍 OpenClaw 中 OpenAI 的两种认证方式：API 密钥和 Codex 订阅。完成本指南后，你将能够根据实际需求选择最适合的认证方式，并掌握故障排查技能。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 OpenAI API 和 Codex 订阅的区别
- [ ] 能够完成 API 密钥的基础配置
- [ ] 掌握模型引用格式 `openai/<model-name>`
- [ ] 能够进行基本的连接测试

### 进阶目标（建议掌握）⭐⭐

- [ ] 根据需求选择合适的认证方式
- [ ] 配置多模型使用和回退
- [ ] 理解速率限制和成本控制
- [ ] 能够诊断认证相关问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计多账户管理策略
- [ ] 实现成本优化方案
- [ ] 建立团队使用规范

---

## 核心概念解析

### 为什么有两种认证方式

OpenAI 提供两种访问方式，这反映了不同的用户群体和使用场景：

| 认证方式 | 目标用户 | 定价模式 | 适用场景 |
|----------|----------|----------|----------|
| API 密钥 | 开发者 | 按使用量付费 | 开发测试、生产应用 |
| Codex 订阅 | ChatGPT 用户 | 订阅制 | 个人使用、团队协作 |

**设计决策背景**：

**API 密钥方式**：
- **优点**：灵活度高、按需付费、无需订阅
- **缺点**：需要管理计费、可能产生意外费用
- **适合**：生产环境、需要精确成本控制的场景

**Codex 订阅方式**：
- **优点**：使用 ChatGPT 订阅、无需单独计费
- **缺点**：功能受限（非完整 API 访问）
- **适合**：个人使用、预算有限的用户

### API 版本和兼容性

OpenAI API 使用版本化机制确保向后兼容：

```
https://api.openai.com/v1/models
```

- **v1**：当前主版本，包含所有最新功能
- **版本升级**：OpenAI 会提前通知，迁移通常简单

---

## 认证方式详解

### 方式一：OpenAI API 密钥

**原理**：使用 OpenAI 平台颁发的 API 密钥进行身份验证。

**获取步骤**：

1. 访问 [OpenAI 仪表板](https://platform.openai.com/api-keys)
2. 点击「Create new secret key」
3. 设置密钥名称和权限
4. **立即复制并保存**（密钥只显示一次）

**配置方式**：

```bash
# 方式一：使用向导配置
openclaw onboard --auth-choice openai-api-key

# 方式二：非交互式设置
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

**配置文件**：

```json5
{
  env: { OPENAI_API_KEY: "sk-proj-..." },
  agents: {
    defaults: {
      model: { primary: "openai/gpt-4o" }
    }
  }
}
```

**环境变量配置**：

```bash
# 在 ~/.zshrc 或 ~/.bashrc 中添加
export OPENAI_API_KEY="sk-proj-your-key-here"

# 或在当前会话中设置
export OPENAI_API_KEY="sk-proj-your-key-here"
```

### 方式二：OpenAI Codex 订阅

**原理**：使用 ChatGPT/Codex 订阅进行身份验证，无需 API 密钥。

**适用场景**：
- 已有 ChatGPT Plus/Pro 订阅
- 不想单独管理 API 密钥
- 只需要基本的模型访问

**配置步骤**：

```bash
# 方式一：在向导中选择 Codex OAuth
openclaw onboard --auth-choice openai-codex

# 方式二：直接运行 OAuth 流程
openclaw models auth login --provider openai-codex
```

**配置文件**：

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-4o" }
    }
  }
}
```

**注意**：Codex 使用不同的模型前缀 `openai-codex/`，与 API 密钥方式区分。

---

## API 密钥管理最佳实践

### 安全性原则

**核心原则**：绝不将 API 密钥硬编码或提交到版本控制。

### 错误做法（❌）

```json5
{
  "OPENAI_API_KEY": "sk-proj-xxxxxxxxxxxx"
}
```

**正确做法（✅）**：

```json5
{
  "env": {
    "OPENAI_API_KEY": "${OPENAI_API_KEY}"
  }
}
```

### 密钥轮换策略

**定期轮换**：建议每 90 天更换一次 API 密钥。

```bash
# 1. 在 OpenAI 仪表板创建新密钥
# 2. 更新本地环境变量
export OPENAI_API_KEY="sk-proj-new-key..."

# 3. 验证新密钥可用
openclaw health

# 4. 删除旧密钥（在 OpenAI 仪表板）
```

### 访问控制

**最小权限原则**：
- 创建专用密钥用于 OpenClaw
- 限制密钥的使用范围
- 设置使用预算和限制

---

## 模型配置详解

### 支持的模型

| 模型名称 | 引用格式 | 特点 | 适用场景 |
|----------|----------|------|----------|
| GPT-4o | `openai/gpt-4o` | 多模态、最强性能 | 复杂任务、代码生成 |
| GPT-4o Mini | `openai/gpt-4o-mini` | 轻量、快速 | 简单问答、成本敏感 |
| GPT-4 Turbo | `openai/gpt-4-turbo` | 大上下文窗口 | 长文档处理 |
| GPT-3.5 Turbo | `openai/gpt-3.5-turbo` | 性价比高 | 基础对话 |

### 模型选择思维模型

```
Q: 需要处理什么类型的任务？
├── 复杂推理 / 代码生成
│   └── → GPT-4o 或 GPT-4 Turbo
│
├── 简单对话 / 快速响应
│   └── → GPT-4o Mini
│
├── 长文档分析（>128K tokens）
│   └── → GPT-4 Turbo
│
└── 成本敏感场景
    └── → GPT-4o Mini 或 GPT-3.5 Turbo
```

### 配置示例

**基础配置**：

```json5
{
  "agents": {
    "defaults": {
      "model": "openai/gpt-4o"
    }
  }
}
```

**多模型配置**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openai/gpt-4o",
        "fallbacks": [
          "openai/gpt-4o-mini",
          "anthropic/claude-sonnet-4-20250514"
        ]
      }
    }
  }
}
```

**按任务类型分配**：

```json5
{
  "agents": {
    "list": [
      {
        "id": "coder",
        "model": "openai/gpt-4o"
      },
      {
        "id": "chat",
        "model": "openai/gpt-4o-mini"
      }
    ]
  }
}
```

---

## 速率限制与成本控制

### 理解速率限制

OpenAI 对 API 请求施加速率限制，防止滥用并确保服务稳定性。

**限制类型**：
- **RPM**（Requests Per Minute）：每分钟请求数
- **TPM**（Tokens Per Minute）：每分钟 Token 数
- **RPD**（Requests Per Day）：每天请求数

### 查看当前限制

```bash
# 在 OpenAI 仪表板查看：https://platform.openai.com/limits
```

### 配置请求限制

```json5
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "rateLimit": {
          "requestsPerMinute": 50,
          "tokensPerMinute": 100000
        }
      }
    }
  }
}
```

### 成本估算

| 模型 | 输入价格（$ / 1M tokens） | 输出价格（$ / 1M tokens） |
|------|---------------------------|---------------------------|
| GPT-4o | $2.50 | $10.00 |
| GPT-4o Mini | $0.15 | $0.60 |
| GPT-4 Turbo | $10.00 | $30.00 |

**示例成本计算**：
- 1000 次对话请求，每次约 1000 输入 tokens + 500 输出 tokens
- 使用 GPT-4o Mini：1000 × ($0.15 × 0.001 + $0.60 × 0.0005) = $0.45
- 使用 GPT-4o：1000 × ($2.50 × 0.001 + $10.00 × 0.0005) = $7.50

### 成本优化策略

**策略一：任务分级**

```json5
{
  "agents": {
    "list": [
      {
        "id": "complex",
        "model": "openai/gpt-4o"
      },
      {
        "id": "simple",
        "model": "openai/gpt-4o-mini"
      }
    ]
  }
}
```

**策略二：上下文精简**

```json5
{
  "agents": {
    "defaults": {
      "model": "openai/gpt-4o",
      "context": {
        "maxHistory": 10,  // 减少对话历史
        "summarize": true  // 启用摘要压缩
      }
    }
  }
}
```

---

## 故障排查指南

### 故障排查思维模型

```
1. 观察错误信息
   └── 记录完整的错误消息和状态码
   
2. 检查认证状态
   └── 环境变量、API Key 格式、有效期
   
3. 验证网络连接
   └── curl 测试、代理设置
   
4. 检查速率限制
   └── 是否超过配额
   
5. 查看日志
   └── OpenClaw 日志提供详细信息
```

### 常见问题与解决方案

#### 问题一：认证失败（401 错误）

**症状**：
```
Error: Invalid API Key or unauthorized
Status: 401
```

**排查步骤**：

```bash
# 1. 验证环境变量
echo $OPENAI_API_KEY

# 2. 检查 API Key 格式
# 正确格式：sk-proj-xxxxx...
# 错误格式：缺少 sk- 前缀

# 3. 验证 API Key 有效性
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/models
```

**可能原因**：
- API Key 已过期或被撤销
- API Key 格式错误
- 组织订阅已过期
- 账户被暂停

**解决方案**：
1. 在 OpenAI 仪表板创建新密钥
2. 更新环境变量
3. 验证新密钥权限

#### 问题二：速率限制（429 错误）

**症状**：
```
Error: Rate limit exceeded
Status: 429
Retry-After: 5
```

**排查步骤**：

```bash
# 1. 检查当前使用量
openclaw usage

# 2. 查看速率限制状态
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/usage
```

**解决方案**：

```json5
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "rateLimit": {
          "requestsPerMinute": 50,
          "tokensPerMinute": 100000
        },
        "retry": {
          "maxAttempts": 3,
          "backoff": "exponential"
        }
      }
    }
  }
}
```

**缓解策略**：
- 实现请求队列
- 使用回退模型
- 升级账户配额

#### 问题三：网络超时

**症状**：请求超时，无响应

**排查步骤**：

```bash
# 1. 测试网络连接
curl -v https://api.openai.com/v1/models

# 2. 检查代理设置
env | grep -i proxy

# 3. 测试 OpenAI API 可达性
ping api.openai.com
```

**解决方案**：

```json5
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "timeout": 60000,  // 60秒超时
        "proxy": "http://your-proxy:7890"  // 如需要代理
      }
    }
  }
}
```

#### 问题四：模型不存在

**症状**：
```
Error: Model not found
Status: 404
```

**排查步骤**：

```bash
# 1. 列出可用模型
openclaw models list

# 2. 检查模型名称是否正确
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/models | jq '.data[].id'
```

**解决方案**：使用正确的模型名称，如 `gpt-4o` 而非 `gpt-4`。

---

## 适用场景分析

### 适用场景

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| 代码生成和调试 | GPT-4o | 强大的代码理解和生成能力 |
| 长文档分析 | GPT-4 Turbo | 大上下文窗口（128K） |
| 实时对话机器人 | GPT-4o Mini | 快速响应、低延迟 |
| 成本敏感的内部工具 | GPT-4o Mini | 性价比高 |
| 多模态任务 | GPT-4o | 支持图像输入 |

### 非适用场景

- **离线环境**：需要网络连接，无法本地部署
- **完全隐私要求**：数据会发送到 OpenAI
- **超大规模调用**：需要考虑成本优化

### 与其他提供商的对比

| 特性 | OpenAI | Anthropic | Ollama |
|------|--------|-----------|--------|
| 生态系统 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| 中文优化 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 隐私性 | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 成本 | 中等 | 中等 | 免费 |
| 本地部署 | ❌ | ❌ | ✅ |

---

## 相关命令速查

| 命令 | 说明 |
|------|------|
| `openclaw onboard` | 运行配置向导 |
| `openclaw onboard --auth-choice openai-api-key` | 配置 API 密钥 |
| `openclaw models auth login --provider openai-codex` | 登录 Codex |
| `openclaw models list` | 列出可用模型 |
| `openclaw health` | 检查认证状态 |
| `openclaw usage --cost` | 查看成本统计 |

---

## 相关文档

- [AI 提供商概览](/zh-CN/providers/index) - 所有提供商对比
- [Anthropic 配置](/zh-CN/providers/anthropic) - Claude 配置
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [OAuth 概念](/zh-CN/concepts/oauth) - 认证机制详解
- [故障排查](/zh-CN/help/troubleshooting) - 常见问题解决

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：配置 OpenAI API 密钥并测试连接

**步骤**：
1. 获取 OpenAI API Key
2. 设置环境变量
3. 运行配置向导：`openclaw onboard`
4. 测试连接：`openclaw health`

**成功标准**：
- 能够成功调用 OpenAI 模型
- 查看使用统计：`openclaw usage`

### 练习 2：配置成本优化方案 ⭐⭐

**任务**：为不同任务配置不同模型

**步骤**：
1. 分析你的使用场景
2. 配置主模型（GPT-4o）和备用模型（GPT-4o Mini）
3. 设置使用限制
4. 监控成本变化

**成功标准**：
- 简单任务自动使用 GPT-4o Mini
- 复杂任务使用 GPT-4o
- 成本降低 50% 以上

### 练习 3：故障排查实战 ⭐⭐⭐

**任务**：诊断并解决认证问题

**场景**：突然收到 401 错误

**步骤**：
1. 收集错误信息
2. 按排查清单逐步检查
3. 定位问题原因
4. 解决问题并记录

**成功标准**：
- 能够独立诊断认证问题
- 有完整的排查流程文档

---

## 自检清单

完成本章节学习后，请自检以下能力：

### 概念理解
- [ ] 理解 API 密钥和 Codex 订阅的区别
- [ ] 知道模型引用的正确格式
- [ ] 理解速率限制的原理

### 动手能力
- [ ] 能够完成 OpenAI 的基础配置
- [ ] 能够配置多模型使用
- [ ] 能够设置成本限制

### 问题解决
- [ ] 能够解决认证失败问题
- [ ] 能够处理速率限制
- [ ] 能够优化使用成本
