---
summary: "通过网关 HTTP 端点直接调用单个工具"
read_when:
  - 不运行完整代理轮次地调用工具
  - 构建需要工具策略强制的自动化
title: "工具调用 API"
---

# 工具调用（HTTP）

OpenClaw 的网关暴露了一个简单的 HTTP 端点，用于直接调用单个工具。它始终启用，但受网关认证和工具策略限制。

- `POST /tools/invoke`
- 与网关相同的端口（WS + HTTP 复用）：`http://<gateway-host>:<port>/tools/invoke`

默认最大 payload 大小为 2 MB。

## 认证

使用网关认证配置。发送 bearer 令牌：

- `Authorization: Bearer <token>`

注意：

- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。

## 请求体

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

字段：

- `tool`（字符串，必填）：要调用的工具名称。
- `action`（字符串，可选）：如果工具 schema 支持 `action` 且 args payload 省略了它，则映射到 args。
- `args`（对象，可选）：工具特定参数。
- `sessionKey`（字符串，可选）：目标会话密钥。如果省略或为 `"main"`，网关使用配置的 main 会话密钥（尊重 `session.mainKey` 和默认代理，或全局作用域中的 `global`）。
- `dryRun`（布尔值，可选）：为未来使用保留；当前忽略。

## 策略 + 路由行为

工具可用性通过与网关代理使用的相同策略链进行过滤：

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- 组策略（如果会话密钥映射到组或频道）
- 子代理策略（使用子代理会话密钥调用时）

如果工具不被策略允许，端点返回 **404**。

为了帮助组策略解析上下文，你可以选择设置：

- `x-openclaw-message-channel: <channel>`（示例：`slack`、`telegram`）
- `x-openclaw-account-id: <accountId>`（当存在多个账户时）

## 响应

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }`（无效请求或工具错误）
- `401` → 未授权
- `404` → 工具不可用（未找到或不在允许列表中）
- `405` → 方法不允许

## 示例

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
