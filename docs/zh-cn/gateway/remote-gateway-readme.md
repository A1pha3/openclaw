---
summary: "OpenClaw.app 通过 SSH 隧道连接到远程网关的设置"
read_when: "通过 SSH 将 macOS 应用连接到远程网关"
title: "远程网关设置"
---

# 使用远程网关运行 OpenClaw.app

OpenClaw.app 使用 SSH 隧道连接到远程网关。本指南展示如何设置它。

## 概览

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端机器                              │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (本地端口)             │
│                     │                                        │
│                     ▼                                        │
│  SSH 隧道 ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         远程机器                               │
│                                                              │
│  网关 WebSocket ──► ws://127.0.0.1:18789 ──►                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 快速设置

### 步骤 1：添加 SSH 配置

编辑 `~/.ssh/config` 并添加：

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # 例如：172.27.187.184
    User <REMOTE_USER>            # 例如：jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

将 `<REMOTE_IP>` 和 `<REMOTE_USER>` 替换为你的值。

### 步骤 2：复制 SSH 密钥

将公钥复制到远程机器（输入一次密码）：

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### 步骤 3：设置网关令牌

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### 步骤 4：启动 SSH 隧道

```bash
ssh -N remote-gateway &
```

### 步骤 5：重启 OpenClaw.app

```bash
# 退出 OpenClaw.app (⌘Q)，然后重新打开：
open /path/to/OpenClaw.app
```

应用现在将通过 SSH 隧道连接到远程网关。

---

## 登录时自动启动隧道

要使 SSH 隧道在登录时自动启动，请创建一个 Launch Agent。

### 创建 PLIST 文件

将其保存为 `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### 加载 Launch Agent

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

隧道现在将：

- 登录时自动启动
- 如果崩溃则重启
- 在后台保持运行

旧版本注意：如果存在遗留的 `com.openclaw.ssh-tunnel` LaunchAgent，请将其删除。

---

## 故障排除

**检查隧道是否运行：**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**重启隧道：**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**停止隧道：**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

---

## 工作原理

| 组件                            | 功能                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `LocalForward 18789 127.0.0.1:18789` | 将本地端口 18789 转发到远程端口 18789                        |
| `ssh -N`                        | SSH 不执行远程命令（仅端口转发）                              |
| `KeepAlive`                     | 自动重启崩溃的隧道                                           |
| `RunAtLoad`                     | 代理加载时启动隧道                                           |

OpenClaw.app 连接到客户端机器上的 `ws://127.0.0.1:18789`。SSH 隧道将该连接转发到运行网关的远程机器上的端口 18789。
