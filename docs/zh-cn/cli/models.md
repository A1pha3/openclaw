---
summary: "`openclaw models` 命令参考（状态/列表/设置/扫描、别名、回退、认证）"
read_when:
  - 想要更改默认模型或查看提供商认证状态
  - 想要扫描可用模型/提供商和调试认证配置
title: "models"
---

# `openclaw models`

模型发现、扫描和配置（默认模型、回退策略、认证配置）。

## 为什么需要这个命令

模型管理是 OpenClaw 的核心功能：

- **灵活选择**：轻松切换不同的 AI 模型
- **回退策略**：主模型不可用时自动切换备用
- **认证管理**：统一管理多个提供商的认证
- **成本优化**：根据任务选择合适的模型

## 相关链接

- 提供商 + 模型：[Models](/zh-cn/providers/models)
- 提供商认证设置：[Getting started](/zh-cn/start/getting-started)

## 常用命令

```bash
# 查看模型状态
openclaw models status

# 列出可用模型
openclaw models list

# 设置默认模型
openclaw models set <model-or-alias>

# 扫描模型
openclaw models scan
```

## 模型状态

```bash
openclaw models status
```

显示已解析的默认模型/回退策略以及认证概览。当提供商使用快照可用时，OAuth/token 状态部分包含使用量头信息。

### 选项

| 选项 | 说明 |
|------|------|
| `--json` | JSON 输出 |
| `--plain` | 纯文本输出 |
| `--check` | 检查认证（退出码 1=过期/缺失，2=即将过期） |
| `--probe` | 实时探测已配置的认证配置 |
| `--probe-provider <name>` | 探测指定提供商 |
| `--probe-profile <id>` | 探测指定配置（可重复或逗号分隔） |
| `--probe-timeout <ms>` | 探测超时 |
| `--probe-concurrency <n>` | 探测并发数 |
| `--probe-max-tokens <n>` | 探测最大 token 数 |
| `--agent <id>` | 指定代理 ID |

**注意**：探测是真实请求，可能消耗 token 并触发速率限制。

## 设置模型

```bash
openclaw models set <model-or-alias>
```

接受 `provider/model` 格式或别名：

```bash
# 使用完整格式
openclaw models set anthropic/claude-opus-4-5

# 使用别名
openclaw models set opus

# OpenRouter 风格（包含 /）
openclaw models set openrouter/moonshotai/kimi-k2
```

**解析规则**：

- 模型引用按**第一个** `/` 分割
- 如果模型 ID 包含 `/`（OpenRouter 风格），需包含提供商前缀
- 省略提供商时，视为别名或默认提供商的模型

## 别名和回退

```bash
# 列出别名
openclaw models aliases list

# 列出回退策略
openclaw models fallbacks list
```

### 别名配置

```json5
{
  models: {
    aliases: {
      opus: "anthropic/claude-opus-4-5",
      sonnet: "anthropic/claude-sonnet-4",
      gpt4: "openai/gpt-4o",
    },
  },
}
```

### 回退配置

```json5
{
  models: {
    fallbacks: [
      "anthropic/claude-opus-4-5",
      "openai/gpt-4o",
      "anthropic/claude-sonnet-4",
    ],
  },
}
```

## 认证管理

```bash
# 添加认证配置
openclaw models auth add

# 登录提供商
openclaw models auth login --provider <id>

# 设置 token
openclaw models auth setup-token
openclaw models auth paste-token
```

### 登录流程

`models auth login` 运行提供商插件的认证流程（OAuth/API key）。使用 `openclaw plugins list` 查看已安装的提供商。

### Token 管理

| 命令 | 说明 |
|------|------|
| `setup-token` | 提示输入 setup-token（在任意机器上用 `claude setup-token` 生成） |
| `paste-token` | 接受从其他地方生成的 token 字符串 |

## 状态输出示例

```
Models Status
─────────────
Default: anthropic/claude-opus-4-5
Fallbacks: openai/gpt-4o, anthropic/claude-sonnet-4

Provider Auth Status:
  anthropic
    ✓ OAuth (claude.ai) - expires in 29 days
      Usage: 45/100 requests today
  openai
    ✓ API Key - valid
      Usage: 12,450 / 100,000 tokens

Aliases:
  opus → anthropic/claude-opus-4-5
  sonnet → anthropic/claude-sonnet-4
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 模型未找到 | 拼写错误或未安装提供商 | `openclaw models list` 查看可用模型 |
| 认证失败 | Token 过期 | `openclaw models auth login --provider <id>` |
| 回退不工作 | 配置错误 | 检查 `models.fallbacks` 配置 |
| 探测超时 | 网络问题 | 增加 `--probe-timeout` |

## 最佳实践

1. **配置回退**：始终设置至少一个备用模型
2. **使用别名**：简化常用模型的引用
3. **定期检查认证**：运行 `models status --check`
4. **监控使用量**：注意速率限制和配额
