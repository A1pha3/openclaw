---
summary: "OpenClaw 存在条目如何生成、合并和显示"
read_when:
  - 调试实例选项卡
  - 调查重复或陈旧的实例行
  - 更改网关 WS 连接或系统事件信标
title: "在线状态"
---

# 在线状态

OpenClaw "在线状态"是对以下内容的轻量级、尽力而为的视图：

- **网关**本身，以及
- **连接到网关的客户端**（mac 应用、WebChat、CLI 等）

在线状态主要用于渲染 macOS 应用的**实例**选项卡，并提供快速操作员可见性。

## 在线状态字段（显示什么）

在线状态条目是带有以下字段的结构化对象：

- `instanceId`（可选但强烈推荐）：稳定的客户端身份（通常来自 `connect.client.instanceId`）
- `host`：人类友好的主机名
- `ip`：尽力而为的 IP 地址
- `version`：客户端版本字符串
- `deviceFamily` / `modelIdentifier`：硬件提示
- `mode`：`ui`、`webchat`、`cli`、`backend`、`probe`、`test`、`node`、...
- `lastInputSeconds`："自上次用户输入以来的秒数"（如果已知）
- `reason`：`self`、`connect`、`node-connected`、`periodic`、...
- `ts`：最后更新戳记（自 epoch 以来的毫秒）

## 生成器（在线状态来自哪里）

在线状态条目由多个来源生成并**合并**。

### 1）网关自条目

网关总是在启动时植入一个"自"条目，以便 UI 在任何客户端连接之前显示网关主机。

### 2）WebSocket 连接

每个 WS 客户端以 `connect` 请求开始。成功握手后，网关为该连接 upsert 一个在线状态条目。

#### 为什么一次性 CLI 命令不会显示

CLI 通常连接进行短的一次性命令。为了避免垃圾邮件实例列表，`client.mode === "cli"` **不会**转换为在线状态条目。

### 3）`system-event` 信标

客户端可以通过 `system-event` 方法发送更丰富的定期信标。mac 应用使用它来报告主机名、IP 和 `lastInputSeconds`。

### 4）节点连接（角色：node）

当节点通过网关 WebSocket 以 `role: node` 连接时，网关为该节点 upsert 一个在线状态条目（与其他 WS 客户端相同的流程）。

## 合并 + 去重规则（为什么 `instanceId` 很重要）

在线状态条目存储在单个内存映射中：

- 条目由**在线状态密钥**键控。
- 最好的密钥是稳定的 `instanceId`（来自 `connect.client.instanceId`），它可以在重启后存活。
- 密钥不区分大小写。

如果客户端重新连接而没有稳定的 `instanceId`，它可能会显示为**重复**行。

## TTL 和有界大小

在线状态有意是短暂的：

- **TTL：** 超过 5 分钟的条目被修剪
- **最大条目数：** 200（最旧的先丢弃）

这使列表保持新鲜并避免无限制的内存增长。

## 远程/隧道警告（loopback IP）

当客户端通过 SSH 隧道 / 本地端口转发连接时，网关可能将远程地址视为 `127.0.0.1`。为了避免覆盖客户端报告的良好 IP，忽略环回远程地址。

## 消费者

### macOS 实例选项卡

macOS 应用渲染 `system-presence` 的输出，并根据最后更新的年龄应用小的状态指示器（活动/空闲/陈旧）。

## 调试提示

- 要查看原始列表，对网关调用 `system-presence`。
- 如果你看到重复项：
  - 确认客户端在握手中发送稳定的 `client.instanceId`
  - 确认定期信标使用相同的 `instanceId`
  - 检查连接派生的条目是否缺少 `instanceId`（重复是预期的）
