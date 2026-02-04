---
summary: "网关拥有的节点配对（选项 B）用于 iOS 和其他远程节点"
read_when:
  - "实现没有 macOS UI 的节点配对审批"
  - "添加 CLI 流程以批准远程节点"
  - "使用节点管理扩展网关协议"
title: "网关拥有的配对"
---

# 网关拥有的配对（选项 B）

在网关拥有的配对中，**网关**是关于哪些节点被允许加入的事实来源。UI（macOS 应用、未来的客户端）只是批准或拒绝待处理请求的前端。

**重要：** WS 节点在 `connect` 期间使用**设备配对**（角色 `node`）。`node.pair.*` 是一个独立的配对存储，并且**不**控制 WS 握手。只有明确调用 `node.pair.*` 的客户端使用此流程。

## 概念

- **待处理请求**：节点要求加入；需要批准。
- **已配对节点**：已批准并颁发认证令牌的节点。
- **传输**：网关 WS 端点转发请求但不决定成员资格。（旧版 TCP 桥接支持已弃用/移除。）

## 配对如何工作

1. 节点连接到网关 WS 并请求配对。
2. 网关存储**待处理请求**并发出 `node.pair.requested`。
3. 你批准或拒绝请求（CLI 或 UI）。
4. 批准时，网关颁发**新令牌**（重新配对时令牌会轮换）。
5. 节点使用令牌重新连接，现在处于"配对"状态。

待处理请求在 **5 分钟**后自动过期。

## CLI 工作流程（适合无头）

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` 显示配对/连接的节点及其功能。

## API 表面（网关协议）

事件：

- `node.pair.requested` — 当创建新的待处理请求时发出。
- `node.pair.resolved` — 当请求被批准/拒绝/过期时发出。

方法：

- `node.pair.request` — 创建或重用待处理请求。
- `node.pair.list` — 列出待处理 + 已配对的节点。
- `node.pair.approve` — 批准待处理请求（颁发令牌）。
- `node.pair.reject` — 拒绝待处理请求。
- `node.pair.verify` — 验证 `{ nodeId, token }`。

注意：

- `node.pair.request` 每个节点是幂等的：重复调用返回相同的待处理请求。
- 批准**总是**生成新令牌；不会从 `node.pair.request` 返回任何令牌。
- 请求可以包含 `silent: true` 作为自动批准流程的提示。

## 自动批准（macOS 应用）

macOS 应用可以选择性地尝试**静默批准**当：

- 请求被标记为 `silent`，并且
- 应用可以使用相同用户验证到网关主机的 SSH 连接。

如果静默批准失败，它会回退到正常的"批准/拒绝"提示。

## 存储（本地，私有）

配对状态存储在网关状态目录下（默认 `~/.openclaw`）：

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

如果你覆盖 `OPENCLAW_STATE_DIR`，`nodes/` 文件夹会随之移动。

安全说明：

- 令牌是机密；将 `paired.json` 视为敏感内容。
- 轮换令牌需要重新批准（或删除节点条目）。

## 传输行为

- 传输是**无状态的**；它不存储成员资格。
- 如果网关离线或配对被禁用，节点无法配对。
- 如果网关处于远程模式，配对仍然针对远程网关的存储进行。
