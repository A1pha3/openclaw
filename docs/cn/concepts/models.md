---
read_when:
  - 添加或修改模型 CLI（models list/set/scan/aliases/fallbacks）
  - 更改模型回退行为或选择用户体验
  - 更新模型扫描探测（工具/图像）
summary: "模型 CLI 完整指南：模型选择、配置、别名、回退、扫描和故障排查"
title: "模型 CLI"
---

# 模型 CLI

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的模型选择机制
- [ ] 掌握模型配置的基本方法
- [ ] 使用 `/model` 命令在聊天中切换模型
- [ ] 配置模型别名和回退

### 进阶目标（建议掌握）

- [ ] 使用模型扫描发现新模型
- [ ] 配置图像模型的回退策略
- [ ] 理解模型白名单的作用
- [ ] 调试模型相关的认证问题

---

## 💡 模型选择机制

### 选择流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    模型选择流程                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  用户请求                                                                │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  1. 尝试主要模型                                                │   │
│  │     agents.defaults.model.primary                               │   │
│  │                                                                 │   │
│  │     ├─ 成功 ──► 使用该模型                                       │   │
│  │     └─ 失败 ──► 继续下一步                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  2. 尝试回退模型（按顺序）                                      │   │
│  │     agents.defaults.model.fallbacks[]                           │   │
│  │                                                                 │   │
│  │     ├─ 成功 ──► 使用该回退模型                                   │   │
│  │     └─ 全部失败 ──► 报告错误                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  提供商内部故障转移：                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  在切换到下一个模型之前，提供商会先尝试：                       │   │
│  │  • 认证配置文件轮换                                             │   │
│  │  • 冷却时间处理                                                 │   │
│  │                                                                 │   │
│  │  参见：[/concepts/model-failover](/concepts/model-failover) │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 配置层次

```
优先级（从高到低）：

1. 会话覆盖（/model 命令）
   │
2. 智能体配置（agents.list[].model）
   │
3. 默认配置（agents.defaults.model）
   │
4. 隐式默认（model: "main"）
```

---

## 🚀 快速开始

### 设置向导（推荐）

```bash
# 运行新手引导向导
openclaw onboard
```

支持配置的提供商：
- **OpenAI Code（Codex）** - OAuth
- **Anthropic** - API 密钥（推荐）或 setup-token
- **OpenRouter** - API 密钥
- **其他** - 参见提供商配置

### 手动配置

```json5
{
  agents: {
    defaults: {
      // 主要模型
      model: "anthropic/claude-sonnet-4-20250514",

      // 或使用完整格式
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: [
          "openai/gpt-4o",
          "anthropic/claude-haiku-3-5-20241022"
        ]
      }
    }
  }
}
```

---

## 📋 模型推荐

### 任务 vs 模型

| 任务 | 推荐模型 | 原因 |
|------|----------|------|
| **编程/工具调用** | GLM 系列 | 在编程和工具使用方面表现更好 |
| **写作/创意** | MiniMax 系列 | 在写作和氛围营造方面更强 |
| **通用任务** | Claude Sonnet | 平衡性能和成本 |
| **复杂推理** | Claude Opus | 最强推理能力 |
| **快速响应** | Claude Haiku | 低延迟，轻量任务 |

### 选择建议

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    模型选择指南                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  日常使用（平衡）：                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  claude-sonnet-4-20250514                                       │   │
│  │  • 良好的性能                                                   │   │
│  │  • 合理的成本                                                   │   │
│  │  • 200K 上下文窗口                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  代码开发：                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  glm-4-plus 或 claude-opus-4-20250514                           │   │
│  │  • 更好的代码理解                                               │   │
│  │  • 强大的工具使用                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  创意写作：                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  minimax-mini                                                 │   │
│  │  • 优秀的文本生成                                               │   │
│  │  • 丰富的风格变化                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  快速任务：                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  claude-haiku-3-5-20241022                                     │   │
│  │  • 低延迟                                                       │   │
│  │  • 低成本                                                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 配置详解

### 基础配置

```json5
{
  agents: {
    defaults: {
      // 简写形式
      model: "anthropic/claude-sonnet-4-20250514",

      // 完整形式
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: [
          "openai/gpt-4o",
          "anthropic/claude-haiku-3-5-20241022"
        ]
      },

      // 图像模型（仅当主要模型不支持图像时使用）
      imageModel: {
        primary: "openai/gpt-4o",
        fallbacks: ["anthropic/claude-opus-4-20250514"]
      },

      // 模型白名单（可选）
      models: {
        "anthropic/claude-sonnet-4-20250514": {
          alias: "Sonnet"
        },
        "anthropic/claude-opus-4-20250514": {
          alias: "Opus"
        }
      }
    }
  }
}
```

### 配置键说明

| 配置键 | 说明 | 必需 |
|--------|------|------|
| `model.primary` | 主要模型 | ✅ |
| `model.fallbacks[]` | 回退模型列表 | ❌ |
| `imageModel.primary` | 图像模型 | ❌ |
| `imageModel.fallbacks[]` | 图像回退 | ❌ |
| `models` | 白名单 + 别名 | ❌ |
| `models.providers` | 自定义提供商 | ❌ |

---

## 🔄 在聊天中切换模型

### `/model` 命令

```
# 显示可用模型
/model

# 列出所有模型
/model list

# 按编号选择
/model 3

# 直接指定模型
/model openai/gpt-4o

# 查看详细状态
/model status
```

### 命令说明

| 命令 | 说明 |
|------|------|
| `/model` | 显示紧凑的选择器（编号列表） |
| `/model list` | 列出所有可用模型 |
| `/model <#>` | 按编号选择模型 |
| `/model <ref>` | 直接指定模型（provider/model） |
| `/model status` | 显示详细状态和认证信息 |

### 引用格式

```
模型引用格式：provider/model

示例：
• anthropic/claude-sonnet-4-20250514
• openai/gpt-4o
• openrouter/moonshotai/kimi-k2

注意：
• 如果模型 ID 包含 /（OpenRouter），必须包含提供商前缀
• 省略提供商时使用默认提供商
• 别名可以直接使用（如 "Sonnet"）
```

---

## 📛 模型别名

### CLI 命令

```bash
# 列出别名
openclaw models aliases list

# 添加别名
openclaw models aliases add <别名> <provider/model>

# 删除别名
openclaw models aliases remove <别名>

# 示例
openclaw models aliases add sonnet anthropic/claude-sonnet-4-20250514
openclaw models aliases add haiku anthropic/claude-haiku-3-5-20241022
```

### 配置示例

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-20250514": {
          alias: "Sonnet"
        },
        "anthropic/claude-opus-4-20250514": {
          alias: "Opus"
        },
        "anthropic/claude-haiku-3-5-20241022": {
          alias: "Haiku"
        }
      }
    }
  }
}
```

### 使用别名

```
# 配置后可以使用别名
/model sonnet
/model opus
/model haiku
```

---

## 🛡️ 模型白名单

### "Model is not allowed" 错误

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    白名单机制                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  当设置 agents.defaults.models 时：                                     │
│                                                                         │
│  用户尝试选择白名单外的模型                                            │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  错误：Model "provider/model" is not allowed.                   │   │
│  │        Use /model to list available models.                    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  这发生在正常回复生成之前，所以看起来像"没有响应"                      │
│                                                                         │
│  解决方案：                                                              │
│  1. 将模型添加到白名单                                                  │
│  2. 清除白名单（删除 agents.defaults.models）                           │
│  3. 从 /model list 中选择可用模型                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 配置白名单

```json5
{
  agents: {
    defaults: {
      // 设置白名单
      models: {
        "anthropic/claude-sonnet-4-20250514": {
          alias: "Sonnet"
        },
        "anthropic/claude-opus-4-20250514": {
          alias: "Opus"
        },
        // 只允许这两个模型
      }
    }
  }
}
```

### 清除白名单

```json5
{
  agents: {
    defaults: {
      // 删除 models 配置即可清除白名单
      model: "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

---

## 🔁 模型回退

### CLI 命令

```bash
# 列出回退模型
openclaw models fallbacks list

# 添加回退
openclaw models fallbacks add <provider/model>

# 删除回退
openclaw models fallbacks remove <provider/model>

# 清空回退
openclaw models fallbacks clear

# 图像模型回退
openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
```

### 配置示例

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-20250514",
        fallbacks: [
          "openai/gpt-4o",
          "anthropic/claude-haiku-3-5-20241022"
        ]
      }
    }
  }
}
```

### 回退策略

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    回退策略                                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  推荐配置（平衡）：                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  primary: claude-sonnet-4-20250514                              │   │
│  │  fallbacks:                                                     │   │
│  │    - gpt-4o（不同提供商，故障转移）                             │   │
│  │    - claude-haiku（轻量，快速）                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  高可用配置：                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  primary: claude-sonnet-4-20250514                              │   │
│  │  fallbacks:                                                     │   │
│  │    - gpt-4o                                                     │   │
│  │    - claude-opus-4-20250514                                     │   │
│  │    - gemini-2.5-pro                                            │   │
│  │    - minimax-mini                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 模型扫描

### 扫描 OpenRouter 免费模型

```bash
# 基础扫描（需要 OpenRouter API 密钥）
openclaw models scan

# 仅元数据（不探测）
openclaw models scan --no-probe

# 交互式选择并设置默认值
openclaw models scan --set-default --set-image

# 筛选条件
openclaw models scan --min-params 100 --max-age-days 30
```

### 扫描标志

| 标志 | 说明 |
|------|------|
| `--no-probe` | 跳过实时探测 |
| `--min-params <b>` | 最小参数量（十亿） |
| `--max-age-days <d>` | 跳过较旧的模型 |
| `--provider <name>` | 提供商筛选 |
| `--max-candidates <n>` | 回退列表大小 |
| `--set-default` | 设置为主要模型 |
| `--set-image` | 设置为图像模型 |

### 排名标准

扫描结果按以下顺序排名：
1. 图像支持
2. 工具延迟
3. 上下文大小
4. 参数数量

---

## 📊 状态检查

### `models status`

```bash
# 显示状态
openclaw models status

# 简洁输出
openclaw models status --plain

# JSON 输出
openclaw models status --json

# 检查认证（自动化）
openclaw models status --check
```

### 输出包含

| 信息 | 说明 |
|------|------|
| 主要模型 | 当前配置的主要模型 |
| 回退模型 | 配置的回退列表 |
| 图像模型 | 配置的图像模型 |
| 认证状态 | OAuth 过期警告 |
| 缺失认证 | 需要配置的提供商 |

### 认证检查

```bash
# 检查认证状态
# 退出码：0=正常，1=缺失/过期，2=即将过期

if openclaw models status --check; then
  echo "All auth OK"
else
  echo "Auth check failed"
fi
```

---

## 🛠️ CLI 命令参考

### 模型管理

```bash
# 列出模型
openclaw models list [--all] [--local] [--provider <name>]

# 设置默认模型
openclaw models set <provider/model>

# 设置图像模型
openclaw models set-image <provider/model>

# 显示状态
openclaw models status
```

### 别名管理

```bash
# 列出别名
openclaw models aliases list

# 添加别名
openclaw models aliases add <alias> <provider/model>

# 删除别名
openclaw models aliases remove <alias>
```

### 回退管理

```bash
# 列出回退
openclaw models fallbacks list

# 添加回退
openclaw models fallbacks add <provider/model>

# 移除回退
openclaw models fallbacks remove <provider/model>

# 清空回退
openclaw models fallbacks clear
```

---

## 🐛 故障排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| "Model is not allowed" | 不在白名单中 | 添加到白名单或清除白名单 |
| "Missing auth" | 未配置认证 | 运行 `openclaw login` |
| 回退未触发 | 回退列表为空 | 添加回退模型 |
| 切换模型无效 | 模型 ID 格式错误 | 使用 provider/model 格式 |

### 调试技巧

```bash
# 1. 检查当前配置
openclaw models status

# 2. 列出可用模型
openclaw models list --all

# 3. 检查认证状态
openclaw models status --check

# 4. 查看详细日志
openclaw gateway --verbose
```

---

## 🎯 最佳实践

### 回退配置

| 实践 | 说明 |
|------|------|
| **跨提供商** | 使用不同提供商的回退 |
| **分层回退** | 主要 → 备用 → 轻量 |
| **定期检查** | 使用 `--check` 自动化监控 |

### 模型选择

| 场景 | 推荐 |
|------|------|
| 生产环境 | 主要 + 多个回退 |
| 开发测试 | 使用轻量模型节省成本 |
| 语音任务 | Haiku（快速） |

### 认证管理

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    认证最佳实践                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Anthropic：                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  推荐：API 密钥                                                  │   │
│  │  备选：claude setup-token（OAuth）                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenAI：                                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  使用：API 密钥                                                  │   │
│  │  存储：~/.openclaw/credentials/                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenRouter：                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  需要：OPENROUTER_API_KEY                                         │   │
│  │  用途：免费模型扫描                                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [模型故障转移](/concepts/model-failover) | 认证和回退详情 |
| [模型提供商](/concepts/model-providers) | 提供商配置 |
| [配置参考](/config/reference) | 完整配置选项 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置模型和回退 | ⭐⭐⭐⭐⭐ |
| 使用 /model 切换 | ⭐⭐⭐⭐⭐ |
| 扫描发现模型 | ⭐⭐⭐ |
| 调试认证问题 | ⭐⭐⭐ |

---

> **💡 专家提示**：使用 `openclaw models scan --set-default` 可以自动发现并配置免费的 OpenRouter 模型作为回退！
