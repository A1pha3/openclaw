---
summary: "在 OpenClaw 中使用 Qwen OAuth（免费层）"
read_when:
  - 你想与 OpenClaw 一起使用 Qwen
  - 你想要免费层 OAuth 访问 Qwen Coder
title: "Qwen"
---

# Qwen

Qwen 为 Qwen Coder 和 Qwen Vision 模型提供免费层 OAuth 流程（2000 次请求/天，受 Qwen 速率限制）。

## 启用插件

```bash
openclaw plugins enable qwen-portal-auth
```

启用后重启 Gateway。

## 认证

```bash
openclaw models auth login --provider qwen-portal --set-default
```

这会运行 Qwen 设备代码 OAuth 流程并将提供商条目写入你的 `models.json`（加上用于快速切换的 `qwen` 别名）。

## 模型 ID

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

使用以下命令切换模型：

```bash
openclaw models set qwen-portal/coder-model
```

## 重用 Qwen Code CLI 登录

如果你已经使用 Qwen Code CLI 登录，OpenClaw 会在加载认证存储时从 `~/.qwen/oauth_creds.json` 同步凭据。你仍然需要一个 `models.providers.qwen-portal` 条目（使用上面的登录命令创建一个）。

## 说明

- 令牌自动刷新；如果刷新失败或访问被撤销，请重新运行登录命令。
- 默认基础 URL：`https://portal.qwen.ai/v1`（如果 Qwen 提供不同的端点，用 `models.providers.qwen-portal.baseUrl` 覆盖）。
- 有关提供商范围的规则，请参阅 [模型提供商](/concepts/model-providers)。
