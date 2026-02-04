---
title: "沙箱 vs 工具策略 vs 提权"
summary: "工具被阻止的原因：沙箱运行时、工具允许/拒绝策略和提权执行门禁"
read_when: "你遇到'沙箱监狱'或看到工具/提权拒绝，并想要知道确切需要修改的配置键。"
status: active
---

# 沙箱 vs 工具策略 vs 提权

OpenClaw 有三个相关但不同的控制：

1. **沙箱**（`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`）决定**工具在哪里运行**（Docker vs 主机）。
2. **工具策略**（`tools.*`、`tools.sandbox.tools.*`、`agents.list[].tools.*`）决定**哪些工具可用/允许**。
3. **提权**（`tools.elevated.*`、`agents.list[].tools.elevated.*`）是**仅限 exec 的逃生舱**，用于在沙箱化时在主机上运行。

## 快速调试

使用检查器查看 OpenClaw 实际在做什么：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

它会打印：

- 有效的沙箱模式/范围/工作区访问
- 会话当前是否沙箱化（main vs 非 main）
- 有效的沙箱工具允许/拒绝（以及它来自代理/全局/默认值）
- 提权门禁和修复密钥路径

## 沙箱：工具在哪里运行

沙箱化由 `agents.defaults.sandbox.mode` 控制：

- `"off"`：所有内容都在主机上运行。
- `"non-main"`：只有非 main 会话被沙箱化（群组/频道的常见"惊喜"）。
- `"all"`：所有内容都被沙箱化。

参见[沙箱化](/gateway/sandboxing)了解完整矩阵（范围、工作区挂载、镜像）。

### 绑定挂载（安全快速检查）

- `docker.binds` _穿透_沙箱文件系统：无论你挂载什么，在容器内都可见，且使用你设置的模式（`:ro` 或 `:rw`）。
- 如果省略模式，默认为读写；源码/机密信息首选 `:ro`。
- `scope: "shared"` 忽略每个代理的绑定（只有全局绑定适用）。
- 绑定 `/var/run/docker.sock` 实际上将主机控制权交给沙箱；只有故意这样做时才使用。
- 工作区访问（`workspaceAccess: "ro"`/`"rw"`）与绑定模式独立。

## 工具策略：哪些工具存在/可调用

两层很重要：

- **工具配置文件**：`tools.profile` 和 `agents.list[].tools.profile`（基础允许列表）
- **提供程序工具配置文件**：`tools.byProvider[provider].profile` 和 `agents.list[].tools.byProvider[provider].profile`
- **全局/每个代理工具策略**：`tools.allow`/`tools.deny` 和 `agents.list[].tools.allow`/`agents.list[].tools.deny`
- **提供程序工具策略**：`tools.byProvider[provider].allow/deny` 和 `agents.list[].tools.byProvider[provider].allow/deny`
- **沙箱工具策略**（仅在沙箱化时适用）：`tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` 和 `agents.list[].tools.sandbox.tools.*`

经验法则：

- `deny` 始终获胜。
- 如果 `allow` 非空，其他所有内容都被视为阻止。
- 工具策略是硬性停止：`/exec` 不能覆盖被拒绝的 `exec` 工具。
- `/exec` 只改变授权发送者的会话默认 exec；它不授予工具访问权限。
  提供程序工具键接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。

### 工具组（简写）

工具策略（全局、代理、沙箱）支持 `group:*` 条目，展开为多个工具：

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

可用的组：

- `group:runtime`：`exec`、`bash`、`process`
- `group:fs`：`read`、`write`、`edit`、`apply_patch`
- `group:sessions`：`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
- `group:memory`：`memory_search`、`memory_get`
- `group:ui`：`browser`、`canvas`
- `group:automation`：`cron`、`gateway`
- `group:messaging`：`message`
- `group:nodes`：`nodes`
- `group:openclaw`：所有内置 OpenClaw 工具（排除提供程序插件）

## 提权：仅限 exec 的"在主机上运行"

提权**不**授予额外工具；它只影响 `exec`。

- 如果你被沙箱化，`/elevated on`（或 `exec` 带有 `elevated: true`）在主机上运行（可能仍需要批准）。
- 使用 `/elevated full` 跳过会话的 exec 批准。
- 如果你已经直接运行，提权实际上是空操作（仍有门禁）。
- 提权**不是**技能范围的，**不**覆盖工具允许/拒绝。
- `/exec` 与提权分开。它只调整授权发送者的每个会话 exec 默认值。

门禁：

- 启用：`tools.elevated.enabled`（可选地还有 `agents.list[].tools.elevated.enabled`）
- 发送者允许列表：`tools.elevated.allowFrom.<provider>`（可选地还有 `agents.list[].tools.elevated.allowFrom.<provider>`）

参见[提权模式](/tools/elevated)。

## 常见的"沙箱监狱"修复

### "工具 X 被沙箱工具策略阻止"

修复密钥（任选其一）：

- 禁用沙箱：`agents.defaults.sandbox.mode=off`（或每个代理 `agents.list[].sandbox.mode=off`）
- 在沙箱内允许该工具：
  - 从 `tools.sandbox.tools.deny`（或每个代理 `agents.list[].tools.sandbox.tools.deny`）中移除
  - 或将其添加到 `tools.sandbox.tools.allow`（或每个代理允许）

### "我以为是 main，为什么被沙箱化了？"

在 `"non-main"` 模式下，群组/频道密钥**不是** main。使用主会话密钥（由 `sandbox explain` 显示）或将模式切换为 `"off"`。
