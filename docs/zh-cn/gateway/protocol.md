---
summary: "网关 WebSocket 协议：握手、帧、版本控制"
read_when:
  - "实现或更新网关 WS 客户端"
  - "调试协议不匹配或连接失败"
  - "重新生成协议架构/模型"
title: "网关协议"
---

# 网关协议（WebSocket）

网关 WS 协议是 OpenClaw 的**单一控制平面 + 节点传输**。所有客户端（CLI、web UI、macOS 应用、iOS/Android 节点、无头节点）通过 WebSocket 连接，并在握手时声明其**角色** + **范围**。

## 传输

- WebSocket，带有 JSON 负载的文本帧。
- 第一帧**必须**是 `connect` 请求。

## 握手（连接）

网关 → 客户端（连接前挑战）：

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

客户端 → 网关：

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

网关 → 客户端：

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

当颁发设备令牌时，`hello-ok` 还包括：

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### 节点示例

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## 帧格式

- **请求**: `{type:"req", id, method, params}`
- **响应**: `{type:"res", id, ok, payload|error}`
- **事件**: `{type:"event", event, payload, seq?, stateVersion?}`

有副作用的方法需要**幂等键**（见架构）。

## 角色 + 范围

### 角色

- `operator` = 控制平面客户端（CLI/UI/自动化）。
- `node` = 能力主机（camera/screen/canvas/system.run）。

### 范围（操作员）

常见范围：

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

### 能力/命令/权限（节点）

节点在连接时声明能力声明：

- `caps`：高级能力类别。
- `commands`：调用的命令允许列表。
- `permissions`：细粒度切换（例如 `screen.record`、`camera.capture`）。

网关将这些视为**声明**并强制执行服务器端允许列表。

## 存在

- `system-presence` 返回按设备身份键入的条目。
- 存在条目包括 `deviceId`、`roles` 和 `scopes`，因此 UI 可以为每个设备显示单行，即使它同时连接为**操作员**和**节点**。

### 节点辅助方法

- 节点可以调用 `skills.bins` 来获取当前技能可执行文件列表以进行自动允许检查。

## 执行审批

- 当执行请求需要审批时，网关广播 `exec.approval.requested`。
- 操作员客户端通过调用 `exec.approval.resolve` 来解决（需要 `operator.approvals` 范围）。

## 版本控制

- `PROTOCOL_VERSION` 位于 `src/gateway/protocol/schema.ts`。
- 客户端发送 `minProtocol` + `maxProtocol`；服务器拒绝不匹配。
- 架构 + 模型从 TypeBox 定义生成：
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

## 认证

- 如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token`），`connect.params.auth.token` 必须匹配，否则套接字关闭。
- 配对后，网关颁发**设备令牌**，范围为连接角色 + 范围。它在 `hello-ok.auth.deviceToken` 中返回，客户端应持久化它以供将来连接。
- 设备令牌可以通过 `device.token.rotate` 和 `device.token.revoke` 轮换/撤销（需要 `operator.pairing` 范围）。

## 设备身份 + 配对

- 节点应包含稳定的设备身份（`device.id`），派生自密钥对指纹。

...
