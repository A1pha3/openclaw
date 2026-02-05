---
summary: "通义千问 Qwen OAuth 完全配置指南，包含免费层使用、模型配置和认证管理"
read_when:
  - 你想与 OpenClaw 一起使用 Qwen
  - 你想要免费层 OAuth 访问 Qwen Coder
  - 你需要了解 Qwen 国产模型的特点
title: "Qwen 通义千问完全配置指南"
---

# Qwen 通义千问完全配置指南

本文档详细介绍如何在 OpenClaw 中配置和使用通义千问（Qwen）AI 模型。完成本指南后，你将掌握 Qwen OAuth 认证流程，理解国产模型的优势，并能够充分利用免费额度。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 Qwen 和 Qwen Coder 的特点
- [ ] 能够完成 OAuth 认证配置
- [ ] 掌握 Qwen 模型的引用格式
- [ ] 知道如何查看可用模型

### 进阶目标（建议掌握）⭐⭐

- [ ] 充分利用免费额度
- [ ] 配置多模型切换
- [ ] 理解国产模型的使用场景
- [ ] 能够诊断认证问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计国产模型组合策略
- [ ] 优化成本和使用效率
- [ ] 建立多账户管理方案

---

## 核心概念解析

### 为什么选择 Qwen

**Qwen（通义千问）核心优势**：

| 特性 | 说明 | 适用场景 |
|------|------|----------|
| **免费额度** | 每日 2000 次请求 | 个人使用、开发测试 |
| **中文优化** | 原生中文理解能力 | 中文场景 |
| **代码能力** | Qwen Coder 专用优化 | 代码辅助 |
| **国产模型** | 数据合规、无跨境 | 企业合规需求 |
| **快速响应** | 低延迟、高并发 | 实时对话 |

**与海外模型对比**：

| 特性 | Qwen | Claude | GPT-4 |
|------|------|--------|-------|
| 中文能力 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 代码能力 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 免费额度 | ✅ 每日 2000 | ❌ | ❌ |
| 隐私合规 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| 响应速度 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

### Qwen 产品线

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| Qwen Coder | 代码专用优化 | 代码生成、调试 |
| Qwen Vision | 多模态理解 | 图文分析 |
| Qwen Max | 综合能力最强 | 复杂对话 |
| Qwen Plus | 平衡性能 | 日常任务 |

---

## OAuth 认证配置

### 启用插件

**必须步骤**：Qwen OAuth 需要启用专属插件。

```bash
openclaw plugins enable qwen-portal-auth
```

启用后需要**重启 Gateway**：

```bash
# 重启 Gateway 以加载插件
openclaw gateway restart
```

### OAuth 认证流程

**原理**：使用 Qwen 设备代码 OAuth 进行身份验证，无需 API 密钥。

**配置步骤**：

```bash
# 运行 OAuth 登录
openclaw models auth login --provider qwen-portal --set-default
```

此命令会：
1. 输出设备代码和验证 URL
2. 打开浏览器完成身份验证
3. 将凭据保存到配置文件
4. 创建 `qwen` 别名方便使用

**手动验证**：

```bash
# 1. 访问显示的 URL
# https://www.qwen.ai/activate

# 2. 输入显示的设备代码
# 例如：ABCD-1234-EFGH-5678

# 3. 登录你的 Qwen 账户

# 4. 完成验证
```

### 配置文件结构

认证成功后，`models.json` 会生成以下配置：

```json5
{
  "models": {
    "providers": {
      "qwen-portal": {
        "baseUrl": "https://portal.qwen.ai/v1",
        "accessToken": "...",
        "refreshToken": "...",
        "expiresAt": "..."
      }
    }
  },
  "auth": {
    "profiles": {
      "qwen-portal:default": {
        "provider": "qwen-portal",
        "mode": "oauth",
        "email": "your@email.com"
      }
    }
  }
}
```

### Qwen Code CLI 凭据同步

**如果已使用 Qwen Code CLI**：

OpenClaw 会自动同步 `~/.qwen/oauth_creds.json` 中的凭据。

```bash
# 检查 Qwen Code CLI 是否已登录
qwen whoami

# 如果已登录，OpenClaw 会自动同步凭据
# 但仍需要运行登录命令创建提供商条目
openclaw models auth login --provider qwen-portal --set-default
```

---

## 模型配置详解

### 模型引用格式

| 模型类型 | 引用格式 |
|----------|----------|
| 代码模型 | `qwen-portal/coder-model` |
| 视觉模型 | `qwen-portal/vision-model` |

### 切换模型

**命令行方式**：

```bash
# 切换到代码模型
openclaw models set qwen-portal/coder-model

# 切换到视觉模型
openclaw models set qwen-portal/vision-model
```

**配置文件方式**：

```json5
{
  "agents": {
    "defaults": {
      "model": "qwen-portal/coder-model"
    }
  }
}
```

### 推荐配置

**基础配置**：

```json5
{
  "agents": {
    "defaults": {
      "model": "qwen-portal/coder-model"
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
        "primary": "qwen-portal/coder-model",
        "fallbacks": [
          "anthropic/claude-sonnet-4-20250514",
          "openai/gpt-4o"
        ]
      }
    }
  }
}
```

---

## 免费额度管理

### 额度说明

| 配额类型 | 数量 | 说明 |
|----------|------|------|
| 每日请求 | 2000 次 | 重置周期：自然日 |
| 速率限制 | 需遵守 Qwen 限制 | 具体数值见仪表板 |

### 监控使用情况

```bash
# 查看 Qwen 使用情况
# 访问：https://portal.qwen.ai/usage

# OpenClaw 暂不支持直接查看 Qwen 配额
```

### 额度优化策略

**策略一：任务分级**

```json5
{
  "agents": {
    "list": [
      {
        "id": "quick-tasks",
        "model": "qwen-portal/coder-model"
      },
      {
        "id": "complex-tasks",
        "model": "anthropic/claude-opus-4-20250514"
      }
    ]
  }
}
```

**策略二：智能切换**

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "qwen-portal/coder-model",
        "fallbacks": [
          "anthropic/claude-sonnet-4-20250514"
        ]
      }
    }
  }
}
```

---

## 故障排查指南

### 常见问题与解决方案

#### 问题一：插件未启用

**症状**：
```
Error: Provider "qwen-portal" not found
```

**排查步骤**：

```bash
# 1. 检查插件状态
openclaw plugins list | grep qwen

# 2. 启用插件
openclaw plugins enable qwen-portal-auth

# 3. 重启 Gateway
openclaw gateway restart
```

#### 问题二：OAuth 失败

**症状**：
```
Error: OAuth authentication failed
Status: 401
```

**排查步骤**：

```bash
# 1. 检查网络连接
curl -I https://portal.qwen.ai

# 2. 验证 Qwen 账户
# 访问：https://qwen.ai/login

# 3. 检查令牌是否过期
openclaw models status --json | jq '.auth.profiles'
```

**解决方案**：

```bash
# 重新运行 OAuth 登录
openclaw models auth login --provider qwen-portal --set-default
```

#### 问题三：令牌刷新失败

**症状**：
```
Error: Token refresh failed
Status: 401
```

**原因**：
- 访问被撤销
- 账户被暂停
- 网络问题

**解决方案**：

```bash
# 重新认证
openclaw models auth login --provider qwen-portal --set-default
```

#### 问题四：找不到模型

**症状**：
```
Error: Model not found
Status: 404
```

**排查步骤**：

```bash
# 1. 查看可用模型
openclaw models list | grep qwen

# 2. 检查模型名称
# 正确：qwen-portal/coder-model
# 错误：qwen/coder-model
```

#### 问题五：速率限制

**症状**：
```
Error: Rate limit exceeded
Status: 429
```

**原因**：
- 超过每日 2000 次请求
- 触发 Qwen 速率限制

**解决方案**：

```bash
# 1. 等待自然日重置
# 或
# 2. 配置回退到其他提供商
```

---

## 适用场景分析

### 推荐使用 Qwen 的场景

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| 中文对话 | Qwen Coder | 原生中文理解 |
| 代码辅助 | Qwen Coder | 专用代码优化 |
| 成本敏感 | Qwen 免费层 | 每日 2000 次 |
| 国产合规 | Qwen Portal | 数据合规 |
| 开发测试 | Qwen 免费层 | 免费实验 |

### 不适用场景

- **复杂推理任务**：建议使用 Claude Opus
- **高并发生产环境**：考虑企业版
- **超长文档**：建议使用 Claude 200K

### 组合使用策略

```json5
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "qwen-portal": {
        // OAuth 自动配置
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "qwen-portal/coder-model",
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
| `openclaw plugins enable qwen-portal-auth` | 启用插件 |
| `openclaw gateway restart` | 重启 Gateway |
| `openclaw models auth login --provider qwen-portal --set-default` | OAuth 登录 |
| `openclaw models set qwen-portal/coder-model` | 切换模型 |
| `openclaw models list | grep qwen` | 查看 Qwen 模型 |

---

## 相关文档

- [AI 提供商概览](/zh-CN/providers/index) - 所有提供商对比
- [OpenAI 配置](/zh-CN/providers/openai) - GPT 配置
- [Anthropic 配置](/zh-CN/providers/anthropic) - Claude 配置
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [OAuth 概念](/zh-CN/concepts/oauth) - 认证机制
- [故障排查](/zh-CN/help/troubleshooting) - 常见问题解决

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：配置 Qwen OAuth

**步骤**：
1. 启用插件
2. 重启 Gateway
3. 运行 OAuth 登录
4. 测试模型调用

**成功标准**：能够使用 Qwen 免费层

### 练习 2：配置多模型切换 ⭐⭐

**任务**：配置 Qwen 和其他提供商的回退

**步骤**：
1. 配置 Qwen OAuth
2. 配置备用提供商（如 Anthropic）
3. 配置回退链
4. 测试回退机制

**成功标准**：Qwen 不可用时自动切换

### 练习 3：成本优化实践 ⭐⭐⭐

**任务**：最大化利用 Qwen 免费额度

**步骤**：
1. 分析任务类型
2. 配置任务分级
3. 监控使用情况
4. 优化调用策略

**成功标准**：免费完成大部分任务

---

## 自检清单

### 概念理解
- [ ] 理解 Qwen 的产品线
- [ ] 知道 OAuth 认证流程
- [ ] 理解免费额度的限制

### 动手能力
- [ ] 能够完成 Qwen OAuth 配置
- [ ] 能够切换不同模型
- [ ] 能够配置回退机制

### 问题解决
- [ ] 能够诊断插件问题
- [ ] 能够处理认证失败
- [ ] 能够优化使用额度
