---
summary: "OpenClaw 如何轮换认证配置文件并在模型之间回退"
read_when:
  - 诊断认证配置文件轮换、冷却时间或模型回退行为
  - 更新认证配置文件或模型的回退规则
title: "模型故障转移"
---

# 模型故障转移

OpenClaw 以两个阶段处理故障：

1. **当前提供程序内的认证配置文件轮换**。
2. **模型回退**到 `agents.defaults.model.fallbacks` 中的下一个模型。

本文档解释运行时规则和支持它们的数据。

## 认证存储（密钥 + OAuth）

OpenClaw 对 API 密钥和 OAuth 令牌使用**认证配置文件**。

- 机密信息位于 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`（旧版：`~/.openclaw/agent/auth-profiles.json`）。
- 配置 `auth.profiles` / `auth.order` 仅为**元数据 + 路由**（无机密信息）。
- 旧版仅导入 OAuth 文件：`~/.openclaw/credentials/oauth.json`（在首次使用时导入到 `auth-profiles.json`）。

更多详情：[/concepts/oauth](/concepts/oauth)

凭证类型：

- `type: "api_key"` → `{ provider, key }`
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（+ 某些提供程序的 `projectId`/`enterpriseUrl`）

## 配置文件 ID

OAuth 登录创建不同的配置文件，以便多个账户可以共存。

- 默认：当没有电子邮件可用时为 `provider:default`。
- 带有电子邮件的 OAuth：`provider:<email>`（例如 `google-antigravity:user@gmail.com`）。

配置文件位于 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` 中的 `profiles` 下。

## 轮换顺序

当提供程序有多个配置文件时，OpenClaw 按以下顺序选择：

1. **显式配置**：`auth.order[provider]`（如果设置）。
2. **配置的配置文件**：`auth.profiles` 按提供程序过滤。
3. **存储的配置文件**：提供程序的 `auth-profiles.json` 中的条目。

如果没有配置显式顺序，OpenClaw 使用轮询顺序：

- **主密钥：** 配置文件类型（**OAuth 在 API 密钥之前**）。
- **次密钥：** `usageStats.lastUsed`（在每种类型中最旧优先）。
- **冷却/禁用的配置文件** 移动到最后，按最早到期排序。

### 会话粘性（缓存友好）

OpenClaw **将选定的认证配置文件固定到每个会话**以保持提供程序缓存温暖。
它**不会在每次请求时轮换**。固定的配置文件被重复使用，直到：

- 会话被重置（`/new` / `/reset`）
- 压缩完成（压缩计数增加）
- 配置文件处于冷却/禁用状态

通过 `/model …@<profileId>` 手动选择设置**用户覆盖**用于该会话，并且在新的会话开始之前不会自动轮换。

自动固定的配置文件（由会话路由器选择）被视为**偏好**；它们首先被尝试，但 OpenClaw 可能在速率限制/超时时轮换到另一个配置文件。用户固定的配置文件保持锁定到该配置文件；如果它失败并且配置了模型回退，OpenClaw 会移动到下一个模型而不是切换配置文件。

### 为什么 OAuth 可能"看起来丢失"

如果你有同一个提供程序的 OAuth 配置文件和 API 密钥配置文件，轮询可能在消息之间在它们之间切换，除非固定。要强制单个配置文件：

- 使用 `auth.order[provider] = ["provider:profileId"]` 固定，或
- 通过 `/model …` 使用配置文件覆盖（当你的 UI/聊天表面支持时）使用每个会话覆盖。

## 冷却时间

当配置文件因认证/速率限制错误失败时（或看起来像速率限制的超时），OpenClaw 将其标记为冷却并移动到下一个配置文件。格式/无效请求错误（例如 Cloud Code Assist 工具调用 ID 验证失败）被视为值得故障转移，并使用相同的冷却时间。

冷却时间使用指数退避：

- 1 分钟
- 5 分钟
- 25 分钟
- 1 小时（上限）

状态存储在 `auth-profiles.json` 中的 `usageStats` 下：

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## 计费禁用

计费/信用失败（例如"积分不足" / "信用余额太低"）被视为值得故障转移，但它们通常不是瞬态的。OpenClaw 不是短冷却时间，而是将配置文件标记为**禁用**（带有更长的退避）并轮换到下一个配置文件/提供程序。

状态存储在 `auth-profiles.json` 中：

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

默认值：

- 计费退避从 **5 小时** 开始，每次计费失败加倍，上限为 **24 小时**。
- 如果配置文件 24 小时没有失败，退避计数器重置（可配置）。

## 模型回退

如果提供程序的所有配置文件都失败，OpenClaw 会移动到 `agents.defaults.model.fallbacks` 中的下一个模型。这适用于认证失败、速率限制和耗尽配置文件轮换的超时（其他错误不会推进回退）。

当运行以模型覆盖开始时（钩子或 CLI），回退仍然在尝试任何配置的回退后在 `agents.defaults.model.primary` 结束。

## 相关配置

参见 [网关配置](/gateway/configuration) 了解：

- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel` 路由

参见 [Models](/concepts/models) 了解更广泛的模型选择和回退概览。
