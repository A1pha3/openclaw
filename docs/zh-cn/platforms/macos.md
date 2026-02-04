---
summary: "OpenClaw macOS 伴侣应用（菜单栏 + 网关代理）"
read_when:
  - 实现 macOS 应用功能
  - 更改网关生命周期或 macOS 上的节点桥接
title: "macOS 应用"
---

# OpenClaw macOS 伴侣应用（菜单栏 + 网关代理）

macOS 应用是 OpenClaw 的**菜单栏伴侣**。它管理权限、在本地管理/连接网关（launchd 或手动），并将 macOS 功能作为节点暴露给 agent。

## 功能概述

- 在菜单栏显示原生通知和状态
- 拥有 TCC 提示（通知、辅助功能、屏幕录制、麦克风、语音识别、自动化/AppleScript）
- 运行或连接网关（本地或远程）
- 暴露 macOS 专属工具（Canvas、相机、屏幕录制、`system.run`）
- 在**远程**模式下启动本地节点宿主服务（launchd），在**本地**模式下停止它
- 可选托管 **PeekabooBridge** 用于 UI 自动化
- 应要求通过 npm/pnpm 安装全局 CLI（`openclaw`）（网关运行时不推荐 bun）

## 本地 vs 远程模式

| 模式 | 说明 |
|------|------|
| **本地**（默认） | 应用连接到正在运行的本地网关（如果存在）；否则通过 `openclaw gateway install` 启用 launchd 服务 |
| **远程** | 应用通过 SSH/Tailscale 连接到远程网关，不启动本地进程。应用启动本地**节点宿主服务**以便远程网关可以访问此 Mac |

## Launchd 控制

应用管理每用户 LaunchAgent，标签为 `bot.molt.gateway`（使用 `--profile`/`OPENCLAW_PROFILE` 时为 `bot.molt.<profile>`；旧版 `com.openclaw.*` 仍会卸载）。

```bash
# 重启服务
launchctl kickstart -k gui/$UID/bot.molt.gateway

# 停止服务
launchctl bootout gui/$UID/bot.molt.gateway
```

如果 LaunchAgent 未安装，从应用启用或运行 `openclaw gateway install`。

## 节点功能（mac）

macOS 应用将自身呈现为节点。常用命令：

| 类别 | 命令 |
|------|------|
| Canvas | `canvas.present`、`canvas.navigate`、`canvas.eval`、`canvas.snapshot`、`canvas.a2ui.*` |
| 相机 | `camera.snap`、`camera.clip` |
| 屏幕 | `screen.record` |
| 系统 | `system.run`、`system.notify` |

节点报告 `permissions` 映射，以便 agent 可以决定允许什么。

### 节点服务 + 应用 IPC

- 当无头节点宿主服务运行时（远程模式），它作为节点连接到网关 WS
- `system.run` 在 macOS 应用中执行（UI/TCC 上下文），通过本地 Unix 套接字；提示和输出留在应用内

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

## Exec 审批（system.run）

`system.run` 由 macOS 应用中的 **Exec 审批**控制（设置 → Exec 审批）。安全设置 + 询问 + 白名单存储在 Mac 本地：

```
~/.openclaw/exec-approvals.json
```

示例配置：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

注意事项：
- `allowlist` 条目是已解析二进制路径的 glob 模式
- 在提示中选择"始终允许"会将该命令添加到白名单
- `system.run` 环境覆盖会被过滤（删除 `PATH`、`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`）然后与应用环境合并

## 深度链接

应用注册 `openclaw://` URL scheme 用于本地操作。

### `openclaw://agent`

触发网关 `agent` 请求。

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

查询参数：

| 参数 | 说明 |
|------|------|
| `message` | **必需** |
| `sessionKey` | 可选 |
| `thinking` | 可选 |
| `deliver` / `to` / `channel` | 可选 |
| `timeoutSeconds` | 可选 |
| `key` | 可选的无人值守模式密钥 |

安全：
- 没有 `key` 时，应用提示确认
- 有有效 `key` 时，运行是无人值守的（用于个人自动化）

## 入门流程（典型）

1. 安装并启动 **OpenClaw.app**
2. 完成权限检查清单（TCC 提示）
3. 确保**本地**模式激活且网关正在运行
4. 如果需要终端访问，安装 CLI

## 构建和开发工作流（原生）

```bash
cd apps/macos && swift build
swift run OpenClaw  # 或 Xcode
# 打包应用
scripts/package-mac-app.sh
```

## 调试网关连接（macOS CLI）

使用调试 CLI 执行与 macOS 应用相同的网关 WebSocket 握手和发现逻辑，无需启动应用。

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

### 连接选项

| 选项 | 说明 |
|------|------|
| `--url <ws://host:port>` | 覆盖配置 |
| `--mode <local\|remote>` | 从配置解析（默认：配置或本地） |
| `--probe` | 强制新鲜健康探测 |
| `--timeout <ms>` | 请求超时（默认：`15000`） |
| `--json` | 结构化输出用于比较 |

### 发现选项

| 选项 | 说明 |
|------|------|
| `--include-local` | 包含会被过滤为"本地"的网关 |
| `--timeout <ms>` | 整体发现窗口（默认：`2000`） |
| `--json` | 结构化输出用于比较 |

提示：与 `openclaw gateway discover --json` 比较，看 macOS 应用的发现管道（NWBrowser + tailnet DNS-SD 回退）是否与 Node CLI 基于 `dns-sd` 的发现不同。

## 远程连接管道（SSH 隧道）

当 macOS 应用在**远程**模式下运行时，它打开 SSH 隧道，使本地 UI 组件可以像访问 localhost 一样与远程网关通信。

### 控制隧道（网关 WebSocket 端口）

- **目的**：健康检查、状态、Web Chat、配置和其他控制平面调用
- **本地端口**：网关端口（默认 `18789`），始终稳定
- **远程端口**：远程主机上的同一网关端口
- **行为**：无随机本地端口；应用重用现有健康隧道或在需要时重启
- **SSH 形式**：`ssh -N -L <local>:127.0.0.1:<remote>`，带 BatchMode + ExitOnForwardFailure + keepalive 选项
- **IP 报告**：SSH 隧道使用回环，所以网关会看到节点 IP 为 `127.0.0.1`。如果想显示真实客户端 IP，使用 **Direct (ws/wss)** 传输（详见 [macOS 远程访问](/zh-cn/platforms/mac/remote)）

## 相关文档

- [网关运维手册](/zh-cn/gateway)
- [网关（macOS）](/zh-cn/platforms/mac/bundled-gateway)
- [macOS 权限](/zh-cn/platforms/mac/permissions)
- [Canvas](/zh-cn/platforms/mac/canvas)
