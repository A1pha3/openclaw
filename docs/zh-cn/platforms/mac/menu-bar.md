---
summary: "菜单栏状态逻辑及向用户展示的内容"
read_when:
  - 调整 mac 菜单 UI 或状态逻辑
title: "菜单栏"
---

# 菜单栏状态逻辑

## 显示内容

- 我们在菜单栏图标和菜单第一行状态中展示当前助手工作状态。
- 工作进行时健康状态会隐藏；当所有会话空闲时恢复显示。
- 菜单中的"节点"区块仅列出**设备**（通过 `node.list` 配对的节点），不包括客户端/在线状态条目。
- 当提供商使用快照可用时，Context 下方会出现"使用量"部分。

## 状态模型

- 会话：事件带有 `runId`（每次运行）以及负载中的 `sessionKey`。"main" 会话的键是 `main`；如果不存在，我们回退到最近更新的会话。
- 优先级：main 总是优先。如果 main 活跃，立即显示其状态。如果 main 空闲，显示最近活跃的非 main 会话。我们不会在活动中途切换；只有当前会话变为空闲或 main 变为活跃时才切换。
- 活动类型：
  - `job`：高级命令执行（`state: started|streaming|done|error`）。
  - `tool`：`phase: start|result` 带有 `toolName` 和 `meta/args`。

## IconState 枚举（Swift）

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)`（调试覆盖）

### ActivityKind → 图标

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- 默认 → 🛠️

### 视觉映射

- `idle`：正常小龙虾。
- `workingMain`：带图标徽章，完整色调，腿部"工作"动画。
- `workingOther`：带图标徽章，低饱和色调，无跑动。
- `overridden`：无论活动如何都使用选定的图标/色调。

## 状态行文本（菜单）

- 工作进行时：`<会话角色> · <活动标签>`
  - 示例：`Main · exec: pnpm test`，`Other · read: apps/macos/Sources/OpenClaw/AppState.swift`。
- 空闲时：回退到健康摘要。

## 事件摄取

- 来源：控制通道 `agent` 事件（`ControlChannel.handleAgentEvent`）。
- 解析字段：
  - `stream: "job"` 带 `data.state` 用于启动/停止。
  - `stream: "tool"` 带 `data.phase`、`name`、可选 `meta`/`args`。
- 标签：
  - `exec`：`args.command` 的第一行。
  - `read`/`write`：缩短的路径。
  - `edit`：路径加上从 `meta`/diff 计数推断的更改类型。
  - 回退：工具名称。

## 调试覆盖

- 设置 ▸ 调试 ▸ "图标覆盖"选择器：
  - `System (auto)`（默认）
  - `Working: main`（按工具类型）
  - `Working: other`（按工具类型）
  - `Idle`
- 通过 `@AppStorage("iconOverride")` 存储；映射到 `IconState.overridden`。

## 测试清单

- 触发 main 会话任务：验证图标立即切换且状态行显示 main 标签。
- main 空闲时触发非 main 会话任务：图标/状态显示非 main；保持稳定直到完成。
- 其他活跃时启动 main：图标立即切换到 main。
- 快速工具连发：确保徽章不闪烁（工具结果的 TTL 宽限期）。
- 所有会话空闲后健康行重新出现。
