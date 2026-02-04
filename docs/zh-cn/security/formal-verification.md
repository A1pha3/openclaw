---
title: 形式化验证（安全模型）
summary: OpenClaw 最高风险路径的机器检查安全模型
permalink: /security/formal-verification/
---

# 形式化验证（安全模型）

本文档跟踪 OpenClaw 的**形式化安全模型**（目前使用 TLA+/TLC；根据需要会增加更多）。

> 注意：一些旧链接可能引用的是之前的项目名称。

**目标（北极星）：** 在明确假设下，提供机器检查的论证，证明 OpenClaw 强制执行其预期的安全策略（授权、会话隔离、工具门控和错误配置安全性）。

**目前的定义：** 可执行的、攻击者驱动的**安全回归套件**：

- 每个声明都在有限状态空间上有一个可运行的模型检查。
- 许多声明都有一个配对的**负模型**，为真实的错误类生成反例跟踪。

**目前不包括：** 证明"OpenClaw 在所有方面都是安全的"或完整 TypeScript 实现是正确的。

## 模型所在位置

模型在单独的仓库中维护：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

## 重要注意事项

- 这些是**模型**，不是完整的 TypeScript 实现。模型和代码之间可能存在差异。
- 结果受 TLC 探索的状态空间限制；"绿色"并不意味着在建模的假设和范围之外的安全性。
- 一些声明依赖于明确的环境假设（例如，正确的部署、正确的配置输入）。

## 复现结果

目前，结果通过在本地克隆模型仓库并运行 TLC 来复现（见下文）。未来的迭代可以提供：

- CI 运行的模型和公共产物（反例跟踪、运行日志）
- 为小的、有界的检查提供托管的"运行此模型"工作流

入门指南：

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 需要 Java 11+（TLC 在 JVM 上运行）。
# 仓库 vendored 了一个固定的 `tla2tools.jar`（TLA+ 工具）并提供了 `bin/tlc` + Make 目标。

make <target>
```

### 网关暴露和开放网关错误配置

**声明：** 在没有认证的情况下绑定到 loopback 以外的范围可能会导致远程妥协的可能性增加；令牌/密码阻止未认证攻击者（根据模型假设）。

- 绿色运行：
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 红色（预期）：
  - `make gateway-exposure-v2-negative`

另见：模型仓库中的 `docs/gateway-exposure-matrix.md`。

### Nodes.run 管道（最高风险能力）

**声明：** `nodes.run` 需要（a）节点命令允许列表加上声明的命令，以及（b）配置时的实时批准；批准被令牌化以防止重放（在模型中）。

- 绿色运行：
  - `make nodes-pipeline`
  - `make approvals-token`
- 红色（预期）：
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### 配对存储（DM 门控）

**声明：** 配对请求遵守 TTL 和待处理请求上限。

- 绿色运行：
  - `make pairing`
  - `make pairing-cap`
- 红色（预期）：
  - `make pairing-negative`
  - `make pairing-cap-negative`

### 入口门控（提及 + 控制命令绕过）

**声明：** 在需要提及的群组上下文中，未经授权的"控制命令"无法绕过提及门控。

- 绿色：
  - `make ingress-gating`
- 红色（预期）：
  - `make ingress-gating-negative`

### 路由/会话密钥隔离

**声明：** 来自不同对等方的 DM 不会合并到同一会话中，除非明确链接/配置。

- 绿色：
  - `make routing-isolation`
- 红色（预期）：
  - `make routing-isolation-negative`

## v1++：额外的有界模型（并发、重试、跟踪正确性）

这些是后续模型，围绕现实世界的失败模式（非原子更新、重试和消息扇出）收紧保真度。

### 配对存储并发 / 幂等性

**声明：** 配对存储应该在交错执行下强制执行 `MaxPending` 和幂等性（即"检查然后写入"必须是原子的/锁定的；刷新不应创建重复项）。

这意味着：

- 在并发请求下，您不能超过通道的 `MaxPending`。
- 对相同的 `(channel, sender)` 的重复请求/刷新不应创建重复的实时待处理行。

- 绿色运行：
  - `make pairing-race`（原子/锁定的上限检查）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 红色（预期）：
  - `make pairing-race-negative`（非原子的 begin/commit 上限竞争）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### 入口跟踪关联 / 幂等性

**声明：** 摄取应该在扇出时保持跟踪关联，并且在提供者重试时是幂等的。

这意味着：

- 当一个外部事件变成多个内部消息时，每个部分都保持相同的跟踪/事件身份。
- 重试不会导致双重处理。
- 如果提供者事件 ID 缺失，回退到安全密钥（例如，跟踪 ID）以避免丢弃不同事件。

- 绿色：
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 红色（预期）：
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### 路由 dmScope 优先级 + identityLinks

**声明：** 路由必须默认保持 DM 会话隔离，并且只有在明确配置时才合并会话（通道优先级 + 身份链接）。

这意味着：

- 通道特定的 dmScope 覆盖必须优先于全局默认值。
- identityLinks 应该只在明确的链接组内合并，而不是在不相关的对等方之间合并。

- 绿色：
  - `make routing-precedence`
  - `make routing-identitylinks`
- 红色（预期）：
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
