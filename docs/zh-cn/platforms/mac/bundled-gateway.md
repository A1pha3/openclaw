---
summary: "macOS 上的网关运行时（外部 launchd 服务）"
read_when:
  - 打包 OpenClaw.app
  - 调试 macOS 网关 launchd 服务
  - 为 macOS 安装网关 CLI
title: "macOS 网关"
---

# macOS 上的网关（外部 launchd）

OpenClaw.app 不再内置 Node/Bun 或网关运行时。macOS 应用需要**外部**的 `openclaw` CLI 安装，不会将网关作为子进程启动，而是管理一个用户级 launchd 服务来保持网关运行（或者如果已经有本地网关在运行则连接到它）。

## 安装 CLI（本地模式必需）

你需要在 Mac 上安装 Node 22+，然后全局安装 `openclaw`：

```bash
npm install -g openclaw@<version>
```

macOS 应用的 **Install CLI** 按钮通过 npm/pnpm 运行相同的流程（不推荐使用 bun 运行网关运行时）。

## Launchd（网关作为 LaunchAgent）

标签：

- `bot.molt.gateway`（或 `bot.molt.<profile>`；旧版 `com.openclaw.*` 可能仍存在）

Plist 位置（用户级）：

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  （或 `~/Library/LaunchAgents/bot.molt.<profile>.plist`）

管理者：

- macOS 应用在本地模式下负责 LaunchAgent 的安装/更新。
- CLI 也可以安装它：`openclaw gateway install`。

行为：

- "OpenClaw Active" 启用/禁用 LaunchAgent。
- 退出应用**不会**停止网关（launchd 保持它存活）。
- 如果配置端口上已经有网关在运行，应用会连接到它而不是启动新的。

日志：

- launchd stdout/err：`/tmp/openclaw/openclaw-gateway.log`

## 版本兼容性

macOS 应用会检查网关版本与自身版本是否匹配。如果不兼容，更新全局 CLI 以匹配应用版本。

## 冒烟测试

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

然后：

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

## 常见 launchd 命令

```bash
# 重启网关服务
launchctl kickstart -k gui/$UID/bot.molt.gateway

# 卸载服务
launchctl bootout gui/$UID/bot.molt.gateway

# 查看服务状态
launchctl print gui/$UID/bot.molt.gateway
```
