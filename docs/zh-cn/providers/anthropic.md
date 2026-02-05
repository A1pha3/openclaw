---
summary: "Anthropic Claude API 密钥和 setup-token 配置完全指南，包含认证机制、最佳实践和深度故障排查"
read_when:
  - 你想在 OpenClaw 中使用 Anthropic Claude 模型
  - 你想使用 setup-token 而不是 API 密钥
  - 你需要了解 Anthropic 特有的认证机制
title: "Anthropic Claude 完全配置指南"
---

# Anthropic Claude 完全配置指南

本文档详细介绍 OpenClaw 中 Anthropic Claude 的两种认证方式：API 密钥和 setup-token。完成本指南后，你将深入理解 Anthropic 的认证机制，掌握最佳配置实践，并能够独立解决认证相关问题。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 Anthropic API 密钥和 setup-token 的区别
- [ ] 能够完成 API 密钥的基础配置
- [ ] 掌握 Claude 模型引用格式
- [ ] 理解提示缓存机制

### 进阶目标（建议掌握）⭐⭐

- [ ] 根据需求选择合适的认证方式
- [ ] 配置多账户管理
- [ ] 理解提示缓存 TTL 配置
- [ ] 能够诊断认证相关问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计企业级多账户架构
- [ ] 优化提示缓存策略
- [ ] 建立完整的故障排查体系

---

## 核心概念解析

### Anthropic 与 OpenAI 的关键差异

| 特性 | Anthropic | OpenAI |
|------|-----------|--------|
| 认证方式 | API Key + setup-token | API Key + OAuth |
| 提示缓存 | 原生支持（扩展缓存 TTL） | 基础支持 |
| 上下文窗口 | 最大 200K tokens | 最大 128K tokens |
| 定价模式 | 按 token 计费 | 按 token 计费 |

### 为什么选择 Anthropic

**Anthropic 的核心优势**：

1. **长上下文处理**：Claude 在长文档分析、代码库理解方面表现卓越
2. **指令遵循**：Claude 更准确地遵循复杂指令
3. **安全性**：内置更严格的安全过滤机制
4. **提示缓存**：显著降低长上下文场景的成本

**适用场景**：
- 长文档分析（>100K tokens）
- 复杂代码库理解
- 需要高精度指令遵循的任务
- 多轮复杂对话

---

## 认证方式详解

### 方式一：Anthropic API 密钥

**原理**：使用 Anthropic 控制台颁发的 API 密钥进行身份验证。

**获取步骤**：

1. 访问 [Anthropic Console](https://console.anthropic.com/)
2. 登录你的 Anthropic 账户
3. 导航至「API Keys」页面
4. 点击「Create Key」
5. 设置密钥名称（建议标注用途）
6. **立即复制并安全保存**（密钥只显示一次）

**配置步骤**：

```bash
# 方式一：使用向导配置
openclaw onboard
# 选择：Anthropic API 密钥

# 方式二：非交互式设置
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

**配置文件**：

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-api03-..." },
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-20250514" }
    }
  }
}
```

### 方式二：Claude setup-token

**原理**：使用 Claude Code CLI 生成的 setup-token 进行认证，适用于 Claude 订阅用户。

**核心概念**：

setup-token 是 Claude Code CLI 特有的认证机制，允许你使用 Claude 订阅访问 OpenClaw，而无需单独的 API 密钥。

**与 API 密钥的区别**：

| 特性 | API 密钥 | setup-token |
|------|---------|-------------|
| 来源 | Anthropic Console | Claude Code CLI |
| 定价 | 按使用量付费 | 使用 Claude 订阅 |
| 刷新 | 手动管理 | 自动刷新 |
| 适用 | 所有用户 | Claude Code 用户 |

**获取 setup-token**：

```bash
# 确保已安装 Claude Code CLI
# 安装方式：https://claude.com/cli

# 生成 setup-token（任意机器）
claude setup-token
```

此命令会：
1. 打开浏览器进行身份验证
2. 生成临时 token
3. 输出 token 字符串

**配置步骤**：

```bash
# 方式一：在向导中配置
openclaw onboard --auth-choice setup-token
# 按提示粘贴 setup-token

# 方式二：直接运行认证
openclaw models auth setup-token --provider anthropic

# 方式三：粘贴已生成的 token
openclaw models auth paste-token --provider anthropic
```

**配置文件**：

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-20250514" }
    }
  }
}
```

**重要说明**：
- setup-token 通常有效期为 30 天
- 需要在 Gateway 所在机器上配置
- 如果使用不同机器生成的 token，需要使用 `paste-token` 命令

---

## 提示缓存机制

### 为什么需要提示缓存

在长对话场景中，重复发送相同的系统提示会浪费大量 tokens。Anthropic 的提示缓存机制允许缓存重复的提示内容，显著降低使用成本。

**成本节省示例**：

| 场景 | 无缓存成本 | 有缓存成本 | 节省比例 |
|------|-----------|-----------|----------|
| 1000 次请求（系统提示 10K tokens） | $50.00 | $5.00 | 90% |
| 500 次请求（系统提示 50K tokens） | $125.00 | $12.50 | 90% |

### 缓存 TTL 配置

**默认行为**：
OpenClaw **不会**覆盖 Anthropic 的默认缓存 TTL，除非你明确设置。

**配置方法**：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-20250514": {
          params: {
            cacheControlTtl: "5m"  // 5 分钟缓存
            // 可选值：1m, 5m, 10m, 30m, 1h
          }
        }
      }
    }
  }
}
```

### 缓存策略设计

**策略一：短 TTL（快速失效）**

```json5
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-20250514": {
          "params": {
            "cacheControlTtl": "1m"
          }
        }
      }
    }
  }
}
```

适用场景：频繁变更的系统提示

**策略二：长 TTL（最大节省）**

```json5
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-20250514": {
          "params": {
            "cacheControlTtl": "1h"
          }
        }
      }
    }
  }
}
```

适用场景：稳定的系统提示，最大化成本节省

---

## 多账户管理

### 企业级认证架构

在企业环境中，你可能需要管理多个 Anthropic 账户：

```json5
{
  "auth": {
    "profiles": {
      "anthropic:personal@example.com": {
        "provider": "anthropic",
        "mode": "oauth",
        "email": "personal@example.com"
      },
      "anthropic:work@example.com": {
        "provider": "anthropic",
        "mode": "oauth",
        "email": "work@example.com"
      }
    },
    "order": {
      "anthropic": [
        "anthropic:personal@example.com",
        "anthropic:work@example.com"
      ]
    }
  }
}
```

### 账户切换策略

```json5
{
  "agents": {
    "list": [
      {
        "id": "dev-agent",
        "model": "anthropic/claude-sonnet-4-20250514",
        "auth": {
          "profile": "anthropic:dev@example.com"
        }
      },
      {
        "id": "prod-agent",
        "model": "anthropic/claude-opus-4-20250514",
        "auth": {
          "profile": "anthropic:prod@example.com"
        }
      }
    ]
  }
}
```

---

## 故障排查指南

### 故障排查思维模型

```
1. 收集信息
   └── 错误消息、时间戳、环境信息
   
2. 分析症状
   └── 401 = 认证问题
   └── 429 = 速率限制
   └── 5xx = 服务端问题
   
3. 定位根因
   └── 认证问题：Key/Token 是否有效
   └── 速率限制：是否超出配额
   └── 服务问题：是否全局故障
   
4. 实施修复
   └── 对症下药
   
5. 验证解决
   └── 确认问题已解决
```

### 常见问题与解决方案

#### 问题一：401 错误 / 令牌突然无效

**症状**：
```
Error: Invalid API Key or unauthorized
Status: 401
Message: "The API key is invalid"
```

**排查流程**：

```bash
# 1. 检查认证状态
openclaw models status --provider anthropic

# 2. 验证 API Key 是否设置
echo $ANTHROPIC_API_KEY

# 3. 直接测试 API 访问
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  https://api.anthropic.com/v1/messages \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

**可能原因与解决方案**：

| 原因 | 解决方案 |
|------|----------|
| API Key 已过期 | 在 Anthropic Console 创建新 Key |
| setup-token 过期 | 重新运行 `claude setup-token` |
| 账户订阅过期 | 续订 Anthropic 订阅 |
| Key 被撤销 | 创建新 Key 并更新配置 |

**setup-token 特定问题**：

```bash
# 如果 Claude CLI 登录在不同机器上
# 在 Gateway 主机上粘贴 token
openclaw models auth paste-token --provider anthropic

# 重新生成 token
claude setup-token
# 粘贴到 Gateway 主机
```

#### 问题二：找不到提供商 API 密钥

**症状**：
```
Error: No API key found for provider "anthropic"
```

**排查流程**：

```bash
# 1. 检查配置文件
openclaw config get models.providers.anthropic

# 2. 查看所有认证配置文件
openclaw models status --json

# 3. 检查环境变量
env | grep ANTHROPIC
```

**解决方案**：

认证**按代理进行**，新代理不会自动继承主代理的密钥。

```bash
# 为当前代理配置认证
openclaw onboard --auth-choice anthropic-api-key
# 或
openclaw models auth paste-token --provider anthropic

# 验证状态
openclaw models status
```

#### 问题三：找不到配置文件凭据

**症状**：
```
Error: No credentials found for profile "anthropic:default"
```

**排查流程**：

```bash
# 1. 查看活跃的认证配置文件
openclaw models status --json | jq '.auth.profiles'

# 2. 检查配置文件路径
ls -la ~/.openclaw/

# 3. 查看详细错误
openclaw models status --verbose
```

**解决方案**：

```bash
# 重新运行向导，为默认配置文件添加凭据
openclaw onboard --auth-choice anthropic-api-key

# 或手动粘贴凭据
openclaw models auth paste-token --provider anthropic
```

#### 问题四：认证配置文件不可用

**症状**：
```
Error: No available authentication profiles (all in cooldown/unavailable)
Status: 503
```

**排查流程**：

```bash
# 1. 查看认证状态详情
openclaw models status --json | jq '.auth'

# 2. 检查不可用原因
openclaw models status --json | jq '.auth.unusableProfiles'
```

**解决方案**：

```bash
# 等待冷却结束，或添加新的认证配置文件
openclaw models auth paste-token --provider anthropic

# 配置多个认证配置文件实现高可用
openclaw models auth login --provider anthropic --email second@example.com
```

#### 问题五：OAuth 令牌刷新失败

**症状**：
```
Error: OAuth token refresh failed for Anthropic Claude subscription
Status: 401
```

**原因分析**：
- Claude 订阅已过期或被撤销
- OAuth 配置已更改
- 网络问题导致刷新失败

**解决方案**：

```bash
# 1. 重新生成 setup-token
claude setup-token

# 2. 在 Gateway 主机上粘贴新 token
openclaw models auth paste-token --provider anthropic

# 3. 验证订阅状态
claude subscription status
```

---

## 适用场景分析

### 推荐使用 Anthropic 的场景

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| 长文档分析（>100K tokens） | Claude Opus 4 | 最大 200K 上下文窗口 |
| 复杂代码库理解 | Claude Opus 4 | 出色的代码理解能力 |
| 需要高精度指令遵循 | Claude Sonnet 4 | 平衡性能与速度 |
| 成本敏感的长对话 | Claude Haiku 3 + 缓存 | 低成本高速响应 |
| 多模态任务 | Claude Sonnet 4 | 支持图像理解 |

### 不适用场景

- **离线环境**：需要网络连接
- **极端成本敏感**：相比开源模型仍较贵
- **实时性要求极高**：可能有额外延迟

### 与其他提供商的对比

| 特性 | Anthropic | OpenAI | Ollama |
|------|-----------|--------|--------|
| 最大上下文 | 200K | 128K | 受本地硬件限制 |
| 提示缓存 | ✅ 原生支持 | 基础 | ❌ |
| 中文能力 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 代码能力 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 隐私性 | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 定价 | 中等 | 中等 | 免费 |

---

## 成本优化策略

### 策略一：模型分级使用

```json5
{
  "agents": {
    "list": [
      {
        "id": "complex-tasks",
        "model": "anthropic/claude-opus-4-20250514"
      },
      {
        "id": "simple-tasks",
        "model": "anthropic/claude-haiku-3-5-20241022"
      }
    ]
  }
}
```

### 策略二：提示缓存优化

```json5
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-opus-4-20250514": {
          "params": {
            "cacheControlTtl": "30m"
          }
        },
        "anthropic/claude-sonnet-4-20250514": {
          "params": {
            "cacheControlTtl": "1h"
          }
        }
      }
    }
  }
}
```

### 策略三：回退机制

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "anthropic/claude-sonnet-4-20250514",
          "anthropic/claude-haiku-3-5-20241022",
          "openai/gpt-4o"
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
| `openclaw onboard` | 运行配置向导 |
| `openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"` | 非交互式配置 API Key |
| `openclaw models auth setup-token --provider anthropic` | 配置 setup-token |
| `openclaw models auth paste-token --provider anthropic` | 粘贴已有 token |
| `openclaw models status` | 查看认证状态 |
| `openclaw models status --json` | JSON 格式详细状态 |
| `claude setup-token` | Claude CLI 生成 token |

---

## 相关文档

- [AI 提供商概览](/zh-CN/providers/index) - 所有提供商对比
- [OpenAI 配置](/zh-CN/providers/openai) - GPT 配置
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [OAuth 概念](/zh-CN/concepts/oauth) - 认证机制详解
- [故障排查](/zh-CN/gateway/troubleshooting) - 完整故障排查指南
- [常见问题](/zh-CN/help/faq) - FAQ

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：配置 Anthropic API 密钥

**步骤**：
1. 获取 Anthropic API Key
2. 设置环境变量
3. 运行配置向导
4. 验证连接：`openclaw health`

**成功标准**：能够成功调用 Claude 模型

### 练习 2：配置 setup-token 认证 ⭐⭐

**任务**：使用 Claude Code CLI 的 setup-token

**步骤**：
1. 安装 Claude Code CLI
2. 运行 `claude setup-token`
3. 粘贴 token 到 OpenClaw
4. 验证认证状态

**成功标准**：使用 Claude 订阅成功调用模型

### 练习 3：优化提示缓存 ⭐⭐⭐

**任务**：配置提示缓存降低长期成本

**步骤**：
1. 分析系统提示大小
2. 配置合适的缓存 TTL
3. 监控缓存命中率
4. 调整配置优化效果

**成功标准**：缓存命中率 > 80%，成本降低 > 50%

### 练习 4：故障排查实战 ⭐⭐⭐

**任务**：诊断 401 认证错误

**步骤**：
1. 模拟认证失败场景
2. 按排查流程逐步检查
3. 定位问题原因
4. 修复并记录

**成功标准**：能够独立诊断各类认证问题

---

## 自检清单

### 概念理解
- [ ] 理解 API Key 和 setup-token 的区别
- [ ] 知道提示缓存的工作原理
- [ ] 理解多账户管理机制

### 动手能力
- [ ] 能够完成 Anthropic 的基础配置
- [ ] 能够配置多账户
- [ ] 能够调整提示缓存 TTL

### 问题解决
- [ ] 能够诊断 401 认证错误
- [ ] 能够处理令牌刷新失败
- [ ] 能够解决配置文件问题
