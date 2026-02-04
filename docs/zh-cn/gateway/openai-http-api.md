---
summary: "从网关暴露 OpenAI 兼容的 /v1/chat/completions HTTP 端点"
read_when:
  - "集成期望 OpenAI Chat Completions 的工具"
title: "OpenAI Chat Completions"
---

# OpenAI Chat Completions（HTTP）

OpenClaw 的网关可以提供一个小型的 OpenAI 兼容 Chat Completions 端点。

此端点**默认禁用**。首先在配置中启用它。

- `POST /v1/chat/completions`
- 与网关相同的端口（WS + HTTP 多路复用）：`http://<gateway-host>:<port>/v1/chat/completions`

在底层，请求作为正常的网关代理运行执行（与 `openclaw agent` 相同的代码路径），因此路由/权限/配置与你的网关匹配。

## 认证

使用网关认证配置。发送 bearer 令牌：

- `Authorization: Bearer <token>`

注意：

- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。

## 选择代理

不需要自定义标头：在 OpenAI `model` 字段中编码代理 ID：

- `model: "openclaw:<agentId>"`（例如：`"openclaw:main"`、`"openclaw:beta"`）
- `model: "agent:<agentId>"`（别名）

或者通过标头定位特定的 OpenClaw 代理：

- `x-openclaw-agent-id: <agentId>`（默认：`main`）

高级：

- `x-openclaw-session-key: <sessionKey>` 完全控制会话路由。

## 启用端点

设置 `gateway.http.endpoints.chatCompletions.enabled` 为 `true`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## 禁用端点

设置 `gateway.http.endpoints.chatCompletions.enabled` 为 `false`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## 会话行为

默认情况下，端点是**每个请求无状态的**（每次调用生成新的会话键）。

如果请求包含 OpenAI `user` 字符串，网关从中派生稳定的会话键，因此重复调用可以共享代理会话。

## 流式传输（SSE）

设置 `stream: true` 以接收服务器发送的事件（SSE）：

- `Content-Type: text/event-stream`
- 每个事件行是 `data: <json>`
- 流以 `data: [DONE]` 结束

## 示例

非流式：

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

流式：

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
