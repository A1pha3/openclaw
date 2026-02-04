---
summary: "通过 API 密钥或 setup-token 在 OpenClaw 中使用 Anthropic Claude"
read_when:
  - 你想在 OpenClaw 中使用 Anthropic 模型
  - 你想使用 setup-token 而不是 API 密钥
title: "Anthropic"
---

# Anthropic（Claude）

Anthropic 构建了 **Claude** 模型系列并通过 API 提供访问。在 OpenClaw 中，你可以通过 API 密钥或 **setup-token** 进行认证。

## 选项 A：Anthropic API 密钥

**最适合：** 标准 API 访问和按使用量计费。在 Anthropic 控制台中创建你的 API 密钥。

### CLI 设置

```bash
openclaw onboard
# 选择：Anthropic API 密钥

# 或非交互式
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 配置片段

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } },
}
```

## 提示缓存（Anthropic API）

OpenClaw **不会**覆盖 Anthropic 的默认缓存 TTL，除非你设置它。这**仅适用于 API**；订阅认证不遵守 TTL 设置。

要按模型设置 TTL，在模型的 `params` 中使用 `cacheControlTtl`：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" }, // 或 "1h"
        },
      },
    },
  },
}
```

OpenClaw 为 Anthropic API 请求包含 `extended-cache-ttl-2025-04-11` 测试版标志；如果你覆盖提供商标头，请保留它（请参阅 [/gateway/configuration](/gateway/configuration)）。

## 选项 B：Claude setup-token

**最适合：** 使用你的 Claude 订阅。

### 在哪里获取 setup-token

setup-token 由 **Claude Code CLI** 创建，而不是 Anthropic 控制台。你可以在**任何机器**上运行：

```bash
claude setup-token
```

将令牌粘贴到 OpenClaw 中（向导：**Anthropic 令牌（粘贴 setup-token）**），或在 Gateway 主机上运行：

```bash
openclaw models auth setup-token --provider anthropic
```

如果你在不同的机器上生成了令牌，粘贴它：

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI 设置

```bash
# 在引导期间粘贴 setup-token
openclaw onboard --auth-choice setup-token
```

### 配置片段

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } },
}
```

## 说明

- 使用 `claude setup-token` 生成 setup-token 并粘贴它，或在 Gateway 主机上运行 `openclaw models auth setup-token`。
- 如果你在 Claude 订阅上看到 "OAuth token refresh failed …"，请使用 setup-token 重新认证。请参阅 [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription)。
- 认证详情 + 重用规则请参阅 [/concepts/oauth](/concepts/oauth)。

## 故障排除

**401 错误 / 令牌突然无效**

- Claude 订阅认证可能过期或被撤销。重新运行 `claude setup-token` 并粘贴到 **Gateway 主机**。
- 如果 Claude CLI 登录在不同的机器上，在 Gateway 主机上使用 `openclaw models auth paste-token --provider anthropic`。

**找不到提供商 "anthropic" 的 API 密钥**

- 认证**按代理进行**。新代理不会继承主代理的密钥。
- 重新为该代理运行引导，或在 Gateway 主机上粘贴 setup-token / API 密钥，然后使用 `openclaw models status` 验证。

**找不到配置文件 `anthropic:default` 的凭据**

- 运行 `openclaw models status` 查看哪个认证配置文件处于活动状态。
- 重新运行引导，或为该配置文件粘贴 setup-token / API 密钥。

**没有可用的认证配置文件（全部在冷却/不可用）**

- 检查 `openclaw models status --json` 中的 `auth.unusableProfiles`。
- 添加另一个 Anthropic 配置文件或等待冷却。

更多内容请参阅 [/gateway/troubleshooting](/gateway/troubleshooting) 和 [/help/faq](/help/faq)。
