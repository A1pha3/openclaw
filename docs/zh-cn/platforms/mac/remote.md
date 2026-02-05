---
summary: "macOS 应用通过 SSH 控制远程 OpenClaw 网关的完整配置与使用指南"
read_when:
  - 设置或调试远程 mac 控制
  - 配置 SSH 隧道或直接 WebSocket 连接
  - 实现跨网络网关访问
  - 理解远程传输模式与安全配置
title: "远程访问配置"
---

# 远程访问配置指南

本章节详细解析 OpenClaw macOS 应用如何通过 SSH 控制远程 OpenClaw 网关，涵盖 SSH 隧道模式、直接 WebSocket 模式、远程传输机制、安全配置与故障排查。通过学习本章节，你将能够在任何网络环境下通过 macOS 菜单栏应用控制远程运行的网关，实现健康检查、语音唤醒转发和 WebChat 等完整功能。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 **远程访问架构**的两种模式（SSH 隧道与直接连接）
- [ ] 配置 macOS 应用连接远程网关
- [ ] 执行远程健康检查与状态监控
- [ ] 掌握远程模式下的安全配置要点

### 进阶目标（建议掌握）

- [ ] 设计安全的 SSH 认证策略（密钥认证与 Tailscale）
- [ ] 优化远程连接性能（带宽与延迟考虑）
- [ ] 配置多远程网关切换
- [ ] 实现远程节点的权限管理

### 专家目标（挑战）

- [ ] 构建高可用远程访问架构（多网关故障转移）
- [ ] 实现企业级安全审计与访问控制
- [ ] 设计自动化部署与配置管理
- [ ] 集成外部监控系统

---

## 第一部分：架构设计原理

### 为什么需要远程访问

OpenClaw 作为个人 AI 助手，可能运行在不同位置：

```
┌─────────────────────────────────────────────────────────────┐
│                    典型部署场景                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   场景一：本地运行                                             │
│   ┌─────────────────────┐                                   │
│   │   MacBook Pro       │                                   │
│   │   + OpenClaw.app    │                                   │
│   │   + 网关本地运行    │                                   │
│   │   + 渠道连接本地    │                                   │
│   └─────────────────────┘                                   │
│                                                              │
│   场景二：远程服务器运行                                       │
│   ┌─────────────────────┐     ┌─────────────────────┐      │
│   │   MacBook Pro       │ SSH │  Linux 服务器       │      │
│   │   + OpenClaw.app    │────→│   + 网关远程运行    │      │
│   │   + 远程控制       │     │   + 渠道连接远程    │      │
│   └─────────────────────┘     └─────────────────────┘      │
│                                                              │
│   场景三：混合模式                                             │
│   ┌─────────────────────┐     ┌─────────────────────┐      │
│   │   iPhone + OpenClaw │     │   Mac Mini          │      │
│   │   + 节点模式       │────→│   + 网关运行        │      │
│   └─────────────────────┘     └─────────────────────┘      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**远程访问的价值**：

- **性能分离**：将计算密集型网关运行在服务器，本地仅做控制
- **持续在线**：服务器可 7x24 小时运行，消息不遗漏
- **渠道灵活性**：服务器可绑定的端口更灵活，适合企业网络环境
- **资源共享**：同一网关可服务多个客户端（macOS、iOS、Android）

### 两种传输模式

macOS 应用支持两种远程传输模式：

| 维度 | SSH 隧道模式 | 直接 (ws/wss) 模式 |
|------|-------------|-------------------|
| **连接方式** | `ssh -N -L` 端口转发 | 直接 WebSocket 连接 |
| **安全性** | SSH 加密隧道 | 依赖网关认证 |
| **配置复杂度** | 需要 SSH 访问 | 更简单 |
| **适用场景** | 企业内网、无公网 IP | 公网或 Tailscale |

**SSH 隧道模式详解**：

```
┌─────────────────────────────────────────────────────────────┐
│                  SSH 隧道模式架构                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────────────┐                                   │
│   │   macOS 应用         │                                   │
│   │   + WebChat UI      │                                   │
│   │   + 控制面板        │                                   │
│   └──────────┬──────────┘                                   │
│              │                                                │
│              │ SSH -N -L 18789:127.0.0.1:18789              │
│              │ (本地端口转发)                                 │
│              ▼                                                │
│   ┌─────────────────────┐                                   │
│   │   SSH 隧道          │                                   │
│   │   (加密通道)        │                                   │
│   └──────────┬──────────┘                                   │
│              │                                                │
│              │ ssh user@remote-host                          │
│              ▼                                                │
│   ┌─────────────────────┐                                   │
│   │   远程主机          │                                   │
│   │   + OpenClaw 网关  │                                   │
│   │   + 渠道服务       │                                   │
│   └─────────────────────┘                                   │
│                                                              │
│   关键特点：                                                  │
│   ├── 网关看到的所有连接来自 127.0.0.1                       │
│   ├── 所有流量通过 SSH 加密                                   │
│   └── 不需要公网暴露网关端口                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**直接 (ws/wss) 模式详解**：

```
┌─────────────────────────────────────────────────────────────┐
│                  直接连接模式架构                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────────────┐                                   │
│   │   macOS 应用         │                                   │
│   │   + WebChat UI      │                                   │
│   │   + 控制面板        │                                   │
│   └──────────┬──────────┘                                   │
│              │                                                │
│              │ ws://gateway.example.ts.net:18789             │
│              │ (直接 WebSocket 连接)                         │
│              ▼                                                │
│   ┌─────────────────────┐     ┌─────────────────────┐      │
│   │   Tailscale/公网   │     │   远程主机          │      │
│   │   + Serve/Funnel   │────→│   + OpenClaw 网关  │      │
│   │   + HTTPS 终结     │     │   + 渠道服务       │      │
│   └─────────────────────┘     └─────────────────────┘      │
│                                                              │
│   关键特点：                                                  │
│   ├── 网关看到真实的客户端 IP                                 │
│   ├── 需要配置网关认证（令牌或密码）                           │
│   └── Tailscale 可实现零配置内网穿透                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第二部分：远程主机配置

### 前置条件

在配置 macOS 应用之前，需确保远程主机满足以下条件：

**必需软件**：

```bash
# 1. 安装 Node.js 22+
node --version  # 应显示 v22.x.x

# 2. 安装 pnpm
pnpm --version  # 应显示 9.x.x

# 3. 克隆并构建 OpenClaw（如果从源码运行）
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build

# 4. 全局链接 OpenClaw CLI
pnpm link --global

# 5. 验证 openclaw 命令
openclaw --version
```

**PATH 配置**：

确保 `openclaw` 在非交互式 shell 的 PATH 中：

```bash
# 添加到 /etc/paths（所有用户）
echo "/usr/local/bin" | sudo tee /etc/paths/openclaw
echo "/opt/homebrew/bin" | sudo tee /etc/paths/homebrew

# 或添加到 shell 配置文件
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc

# 符号链接到标准路径（推荐）
sudo ln -sf /usr/local/bin/openclaw /usr/local/bin/openclaw
```

**SSH 访问配置**：

```bash
# 1. 确保 SSH 服务已安装
# macOS: 系统设置 → 共享 → 远程登录

# 2. 生成 SSH 密钥（如果尚未生成）
ssh-keygen -t ed25519 -C "your-email@example.com"

# 3. 将公钥复制到远程主机
ssh-copy-id user@remote-host

# 4. 验证密钥登录
ssh user@remote-host "openclaw --version"
```

### Tailscale 配置（推荐）

Tailscale 提供零配置的 VPN 能力，适合远程访问场景：

```
Tailscale 优势：

┌─────────────────────────────────────────────────────────────┐
│                    Tailscale 核心优势                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 零配置组网：自动发现节点，无需端口映射                      │
│  2. WireGuard 加密：内核级性能，军事级安全                    │
│  3. 魔法网络：即使设备在不同网络也能直连                      │
│  4. Tailnet 隔离：企业级安全边界                             │
│  5. 移动性好：自动切换网络，IP 保持                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**安装与配置**：

```bash
# 1. 在远程主机安装 Tailscale
# macOS: brew install tailscale
# Linux: curl -fsSL https://tailscale.com/install.sh | sh

# 2. 启动并认证
sudo tailscale up

# 3. 浏览器打开认证链接完成登录

# 4. 获取设备 IP
tailscale ip -4

# 输出示例：
# 100.x.x.x  # Tailscale Magic DNS IP
```

---

## 第三部分：macOS 应用配置

### 基础配置步骤

1. 打开 **OpenClaw 应用**
2. 进入 **Settings → General**
3. 在 **OpenClaw runs** 下选择模式

**模式选择**：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **Local (this Mac)** | 本地运行 | 开发测试、日常使用 |
| **Remote over SSH** | SSH 隧道连接 | 企业内网、无公网 IP |
| **Remote direct (ws/wss)** | 直接 WebSocket | 公网访问、Tailscale |

### SSH 隧道模式配置

```
设置 → General → OpenClaw runs → Remote over SSH
```

**配置字段**：

| 字段 | 说明 | 示例 |
|------|------|------|
| **Transport** | 传输模式 | SSH tunnel |
| **SSH target** | SSH 连接目标 | user@host 或 user@host:port |
| **Identity file** | SSH 密钥路径 | ~/.ssh/id_ed25519 |
| **Project root** | 远程仓库路径 | ~/Projects/openclaw |
| **CLI path** | openclaw 可执行路径 | /usr/local/bin/openclaw |

**SSH target 格式**：

```
标准格式：user@hostname
带端口：user@hostname:2222
Tailscale：user@device-name.tailnet.ts.net
```

**完整配置示例**：

```
┌─────────────────────────────────────────────────────────────┐
│              SSH 隧道模式配置示例                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Transport:    SSH tunnel                                 │
│   SSH target:   admin@gateway-server.tailnet.ts.net:22     │
│   Identity:     ~/.ssh/id_ed25519                          │
│   Project root: /home/admin/openclaw                       │
│   CLI path:     /usr/local/bin/openclaw                   │
│                                                              │
│   [Test remote]  [Connect]                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 直接连接模式配置

```
设置 → General → OpenClaw runs → Remote direct (ws/wss)
```

**配置字段**：

| 字段 | 说明 | 示例 |
|------|------|------|
| **Transport** | 传输模式 | Direct (ws/wss) |
| **Gateway URL** | 网关 WebSocket 地址 | wss://gateway.example.ts.net |

**Gateway URL 格式**：

```
Tailscale Serve: wss://gateway-server.tailnet.ts.net
公网 HTTPS:      wss://gateway.example.com
局域网 WS:       ws://192.168.1.100:18789
本地回环:        ws://127.0.0.1:18789
```

### 连接测试

**Test Remote 按钮**：

点击 **Test remote** 验证配置正确性：

```bash
# 测试命令（在远程主机执行）
openclaw status --json

# 成功输出示例：
# {"status":"running","version":"2.5.0","port":18789}

# 失败可能原因：
# ├── SSH 连接失败（认证或网络问题）
# ├── PATH 问题（exit 127）
# └── 网关未运行
```

**连接状态指示**：

| 状态 | 表现 |
|------|------|
| **已连接** | 菜单栏显示绿色状态点 |
| **连接中** | 菜单栏显示橙色状态点 |
| **错误** | 菜单栏显示红色状态点 + 错误信息 |

---

## 第四部分：功能特性

### 健康检查

远程模式下的健康检查通过 SSH 隧道或直接连接执行：

```bash
# 等效命令
openclaw health --json

# 输出示例（远程）：
# {
#   "status": "healthy",
#   "gateway": "running",
#   "whatsapp": "connected",
#   "telegram": "offline",
#   "uptime": "2h30m"
# }
```

### WebChat

**SSH 隧道模式**：

WebChat 通过转发的 WebSocket 控制端口连接：

```
┌─────────────────────────────────────────────────────────────┐
│                  SSH 隧道 WebChat 流程                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   macOS WebChat UI                                           │
│        │                                                     │
│        │ localhost:18789 (转发的本地端口)                      │
│        ▼                                                     │
│   SSH 隧道                                                   │
│        │                                                     │
│        │ remote-host:18789 (网关实际端口)                      │
│        ▼                                                     │
│   OpenClaw 网关                                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**直接连接模式**：

WebChat 直接连接到配置的 Gateway URL：

```
┌─────────────────────────────────────────────────────────────┐
│                  直接连接 WebChat 流程                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   macOS WebChat UI                                           │
│        │                                                     │
│        │ wss://gateway-server.ts.net:18789                  │
│        │ (直接 WebSocket 连接)                               │
│        ▼                                                     │
│   Tailscale/公网                                             │
│        │                                                     │
│        │ gateway-server.ts.net:18789                          │
│        ▼                                                     │
│   OpenClaw 网关                                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 语音唤醒转发

**自动转发机制**：

当 macOS 应用检测到语音唤醒时，转录内容自动通过远程连接转发：

```
┌─────────────────────────────────────────────────────────────┐
│                  语音唤醒转发流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. 麦克风捕获音频                                           │
│           │                                                  │
│           ▼                                                  │
│   2. 本地语音识别（离线 ASR）                                 │
│           │                                                  │
│           ▼                                                  │
│   3. 文本通过远程连接转发                                     │
│           │                                                  │
│           │ SSH 隧道或直接 WS                                │
│           ▼                                                  │
│   4. 远程网关接收并发送给助手                                 │
│           │                                                  │
│           ▼                                                  │
│   5. 助手响应通过相同通道返回                                  │
│                                                              │
│   关键点：                                                   │
│   ├── 无需单独配置转发器                                      │
│   ├── 转录在本地完成                                         │
│   └── 网络传输的是文本而非音频                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 权限管理

远程主机需要与本地相同的 TCC 权限（如果需要执行本地操作）：

| 权限 | 本地需求 | 远程需求 |
|------|---------|----------|
| Accessibility | 控制本地应用 | 控制远程应用（如果 SSH 隧道包含 X11 转发） |
| ScreenCapture | 截图本地屏幕 | 截图远程屏幕（需额外配置） |
| Microphone | 语音输入 | 本地麦克风（语音唤醒在本地处理） |
| Notifications | 本地通知 | 远程主机发送通知 |

---

## 第五部分：安全配置

### SSH 安全最佳实践

```
SSH 安全配置清单：

┌─────────────────────────────────────────────────────────────┐
│                    必要措施                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ✅ 使用密钥认证（禁用密码登录）                               │
│  ✅ 限制允许登录的用户                                        │
│  ✅ 使用非标准端口（可选）                                     │
│  ✅ 启用 SSH 压缩（提升低带宽环境性能）                       │
│  ✅ 配置连接超时                                              │
│                                                              │
├─────────────────────────────────────────────────────────────┐
│                    推荐措施                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📋 启用双因素认证                                           │
│  📋 使用 Fail2ban 防止暴力破解                                │
│  📋 配置 IP 白名单                                           │
│  📋 启用 SSH 代理转发（如果需要）                             │
│  📋 使用 Tailscale 作为传输层（零信任）                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**禁用密码认证**（远程主机 `/etc/ssh/sshd_config`）：

```bash
# 编辑 sshd_config
sudo nano /etc/ssh/sshd_config

# 添加或修改：
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# 重启 SSH 服务
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load /System/Library/LaunchDaemons/ssh.plist
```

### 网关认证配置

当使用直接连接模式时，需要配置网关认证：

**令牌认证**：

```json
// ~/.openclaw/openclaw.json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "your-secure-token-here"
    }
  }
}
```

**密码认证**：

```json
// ~/.openclaw/openclaw.json
{
  "gateway": {
    "auth": {
      "mode": "password",
      "username": "admin",
      "password": "your-secure-password"
    }
  }
}
```

### Tailscale 安全策略

```
Tailscale ACL 配置示例：

{
  "acls": [
    // 允许同一 Tailnet 内设备互联
    {"action": "accept", "users": ["*"], "ports": ["*:*"]},
    
    // 限制网关访问（仅 macOS 应用）
    {
      "action": "accept",
      "users": ["user@email.com"],
      "ports": ["gateway-server:18789"]
    }
  ],
  "ssh": [
    {
      "action": "accept",
      "users": ["user@email.com"],
      "src":    ["*"],
      "dst":    ["gateway-server"],
      "period": {"start": "00:00", "end": "23:59"}
    }
  ]
}
```

---

## 第六部分：故障排查指南

### 问题分类与诊断流程

```
远程连接问题判定树：

├── SSH 连接失败
│   ├── 认证失败 → 检查密钥配置
│   ├── 网络不可达 → 检查网络/VPN
│   └── 主机不可达 → 检查 DNS/防火墙
│
├── 测试远程失败
│   ├── exit 127 → PATH 问题
│   ├── 连接超时 → 网络/防火墙
│   └── 网关未运行 → 远程主机问题
│
├── WebChat 卡住
│   ├── WS 连接失败 → 检查端口转发
│   ├── 认证失败 → 检查令牌/密码
│   └── 网关不健康 → 远程健康检查
│
└── 语音唤醒不工作
    ├── 本地语音识别问题 → 检查本地麦克风
    ├── 文本转发失败 → 检查远程连接
    └── 助手无响应 → 检查网关状态
```

### 常见故障与解决方案

**故障一：exit 127 / not found**

**症状**：`Test remote` 失败，输出 `exit 127`

**原因**：`openclaw` 不在非交互式 shell 的 PATH 中

**诊断**：

```bash
# 在远程主机测试
ssh user@remote-host "which openclaw"

# 测试 PATH
ssh user@remote-host "echo \$PATH"
```

**解决方案**：

```bash
# 方案一：添加到 /etc/paths
echo "/usr/local/bin" | sudo tee /etc/paths/openclaw

# 方案二：符号链接到标准路径
sudo ln -sf /usr/local/bin/openclaw /usr/local/bin/openclaw

# 方案三：配置 SSH 命令前缀
# 在 macOS 应用的高级设置中配置：
# SSH command prefix: "PATH=/usr/local/bin:\$PATH"
```

**故障二：健康探测失败**

**症状**：`Test remote` 成功但健康状态显示异常

**诊断**：

```bash
# 手动测试远程网关
ssh user@remote-host "openclaw status --json"

# 检查网关日志
ssh user@remote-host "tail -f /tmp/openclaw/openclaw-gateway.log"
```

**解决方案**：

```bash
# 1. 确保网关正在运行
ssh user@remote-host "openclaw gateway status"
ssh user@remote-host "openclaw gateway run"

# 2. 检查端口监听
ssh user@remote-host "lsof -nP -iTCP:18789 -sTCP:LISTEN"

# 3. 检查 Baileys 登录状态
ssh user@remote-host "openclaw channels status whatsapp"
```

**故障三：WebChat 卡住**

**症状**：WebChat 界面显示连接中但不更新

**诊断**：

```bash
# 检查 WebSocket 连接
# 浏览器开发者工具 → Network → WS

# 测试本地端口转发
ssh -N -L 18789:127.0.0.1:18789 user@remote-host &
sleep 2
curl -I http://127.0.0.1:18789
```

**解决方案**：

```
方案一（SSH 隧道）：
1. 确认 SSH 隧道正确建立
2. 检查转发的本地端口
3. 尝试断开重连

方案二（直接连接）：
1. 确认 Gateway URL 正确
2. 检查网关认证配置
3. 验证 Tailscale/公网可达性
```

**故障四：节点 IP 显示 127.0.0.1**

**症状**：远程网关显示连接的节点 IP 为 `127.0.0.1`

**原因**：SSH 隧道的预期行为（所有连接通过隧道转发）

**说明**：这是正常行为，不是故障

**解决方案**：

如果需要网关看到真实客户端 IP，切换到 **Direct (ws/wss)** 模式。

---

## 适用场景与最佳实践

### 推荐使用场景

**场景一：服务器托管**

```
需求：在公网服务器运行网关，本地通过菜单栏控制

推荐配置：
├── 传输模式：SSH 隧道 或 Tailscale Serve
├── 认证：SSH 密钥 + 网关令牌
└── 优势：安全隔离，零公网暴露
```

**场景二：办公室-居家办公**

```
需求：在办公室服务器运行网关，居家通过 VPN 访问

推荐配置：
├── 传输模式：Tailscale
├── 认证：Tailscale 零信任网络
└── 优势：跨网络透明访问
```

**场景三：开发测试**

```
需求：在远程开发服务器测试功能

推荐配置：
├── 传输模式：SSH 隧道
├── 认证：SSH 密钥
└── 优势：与本地开发无缝切换
```

### 性能优化建议

| 场景 | 优化建议 |
|------|----------|
| **低带宽** | 启用 SSH 压缩：`ssh -C` |
| **高延迟** | 调整 TCP 超时配置 |
| **大文件传输** | 使用直接连接模式 |
| **频繁断开** | 配置 SSH 连接保活 |

---

## 章节总结

本章节全面介绍了 OpenClaw macOS 应用的远程访问配置与使用。通过学习，你应该已经掌握：

1. **架构设计**：理解 SSH 隧道与直接连接两种模式的原理
2. **远程配置**：完成 macOS 应用与远程网关的连接配置
3. **功能使用**：使用健康检查、WebChat、语音唤醒等远程功能
4. **安全配置**：实施 SSH 认证与网关访问控制
5. **故障排查**：建立系统化的远程连接问题诊断流程

后续建议结合 [网关配置](/zh-cn/gateway/configuration) 与 [安全指南](/zh-cn/gateway/security) 章节，完善远程运维知识体系。

相关文档：[远程网关访问](/zh-cn/gateway/remote)、[Tailscale 配置](/zh-cn/gateway/tailscale)
