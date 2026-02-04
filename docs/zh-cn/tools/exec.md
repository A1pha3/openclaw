---
summary: "Exec 工具使用、stdin 模式和 TTY 支持"
read_when:
  - 使用或修改 exec 工具
  - 调试 stdin 或 TTY 行为
title: "Exec 工具"
---

# Exec 工具

在工作区中运行 shell 命令。支持通过 `process` 进行前台 + 后台执行。如果 `process` 不允许，`exec` 同步运行并忽略 `yieldMs`/`background`。后台会话按代理作用域；`process` 只能看到来自同一代理的会话。

## 参数

- `command`（必需）
- `workdir`（默认为 cwd）
- `env`（键/值覆盖）
- `yieldMs`（默认 10000）：延迟后自动后台
- `background`（布尔值）：立即后台
- `timeout`（秒，默认 1800）：到期时杀死
- `pty`（布尔值）：在可用时在伪终端中运行（仅限 TTY 的 CLI、编码代理、终端 UI）
- `host`（`sandbox | gateway | node`）：在哪里执行
- `security`（`deny | allowlist | full`）：`gateway`/`node` 的强制模式
- `ask`（`off | on-miss | always`）：`gateway`/`node` 的批准提示
- `node`（字符串）：`host=node` 的节点 id/名称
- `elevated`（布尔值）：请求提权模式（网关主机）；仅当提权解析为 `full` 时才强制 `security=full`

注意：

- `host` 默认为 `sandbox`。
- 当沙箱化关闭时忽略 `elevated`（执行已经在主机上运行）。
- `gateway`/`node` 批准由 `~/.openclaw/exec-approvals.json` 控制。
- `node` 需要配对的节点（配套应用或无头节点主机）。
- 如果多个节点可用，设置 `exec.node` 或 `tools.exec.node` 以选择一个。
- 在非 Windows 主机上，当设置时 exec 使用 `SHELL`；如果 `SHELL` 是 `fish`，它更喜欢 `PATH` 中的 `bash`（或 `sh`）以避免 fish 不兼容的脚本，然后回退到 `SHELL`（如果两者都不存在）。
- 重要：沙箱化**默认关闭**。如果沙箱化关闭，`host=sandbox` 直接在网关主机上运行（无容器）并且**不需要批准**。要要求批准，使用 `host=gateway` 并配置执行批准（或启用沙箱化）。

## 配置

- `tools.exec.notifyOnExit`（默认：true）：为 true 时，后台的 exec 会话在退出时加入队列一个系统事件并在退出时请求心跳。
- `tools.exec.approvalRunningNoticeMs`（默认：10000）：当批准门控的 exec 运行超过此时长时发出单个"运行中"通知（0 禁用）。
- `tools.exec.host`（默认：`sandbox`）
- `tools.exec.security`（默认：沙箱为 `deny`，网关 + node 未设置时为 `allowlist`）
- `tools.exec.ask`（默认：`on-miss`）
- `tools.exec.node`（默认：未设置）
- `tools.exec.pathPrepend`：要在 exec 运行的 `PATH` 前面添加的目录列表。
- `tools.exec.safeBins`：stdin-only 安全二进制文件，无需显式 allowlist 条目即可运行。

示例：

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH 处理

- `host=gateway`：将你的登录 shell `PATH` 合并到 exec 环境中（除非 exec 调用已经设置了 `env.PATH`）。守护进程本身仍然以最小 `PATH` 运行：
  - macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
  - Linux：`/usr/local/bin`、`/usr/bin`、`/bin`
- `host=sandbox`：在容器内运行 `sh -lc`（登录 shell），因此 `/etc/profile` 可能会重置 `PATH`。OpenClaw 通过内部环境变量在配置文件获取后预先添加 `env.PATH`（无 shell 插值）；`tools.exec.pathPrepend` 也适用于此。
- `host=node`：只发送你传递的 env 覆盖。如果 exec 调用已经设置了 `env.PATH`，则 `tools.exec.pathPrepend` 适用。无头节点主机仅在 `PATH` 预先添加节点主机 PATH 时接受 `PATH`（不替换）。macOS 节点完全删除 `PATH` 覆盖。

每个代理节点绑定（使用配置中的代理列表索引）：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

控制 UI：节点选项卡包括一个小的"Exec 节点绑定"面板，用于相同的设置。

## 会话覆盖（`/exec`）

使用 `/exec` 设置 `host`、`security`、`ask` 和 `node` 的**每个会话**默认值。发送不带参数的 `/exec` 以显示当前值。

示例：

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## 授权模型

`/exec` 仅对**授权发送者**（频道 allowlist/配对加上 `commands.useAccessGroups`）生效。它仅更新**会话状态**并不写入配置。要硬性禁用 exec，通过工具策略拒绝它（`tools.deny: ["exec"]` 或每个代理）。除非你明确设置 `security=full` 和 `ask=off`，否则主机批准仍然适用。

## 执行批准（配套应用 / 节点主机）

沙箱化代理可以在 `exec` 在网关或节点主机上运行之前要求每个请求批准。参见 [Exec approvals](/tools/exec-approvals) 了解策略、allowlist 和 UI 流程。

当需要批准时，exec 工具立即返回 `status: "approval-pending"` 和批准 ID。一旦批准（或拒绝 / 超时），网关发出系统事件（`Exec finished` / `Exec denied`）。如果命令在 `tools.exec.approvalRunningNoticeMs` 后仍在运行，发出单个 `Exec running` 通知。

## Allowlist + 安全 bin

Allowlist 强制执行**仅解析的二进制路径**（无 basename 匹配）。当 `security=allowlist` 时，shell 命令仅在每个管道段都在 allowlist 中或是一个安全 bin 时才自动允许。链接（`;`、`&&`、`||`）和中继在 allowlist 模式下被拒绝。

## 示例

前台：

```json
{ "tool": "exec", "command": "ls -la" }
```

后台 + 轮询：

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

发送按键（tmux 风格）：

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

提交（仅发送 CR）：

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

粘贴（默认加括号）：

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply_patch（实验性）

`apply_patch` 是 `exec` 的子工具，用于结构化多文件编辑。显式启用它：

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

注意：

- 仅适用于 OpenAI/OpenAI Codex 模型。
- 工具策略仍然适用；`allow: ["exec"]` 隐式允许 `apply_patch`。
- 配置位于 `tools.exec.applyPatch`。
