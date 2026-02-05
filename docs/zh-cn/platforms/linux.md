---
summary: "OpenClaw Linux 平台完整指南：安装部署、systemd 服务管理、运行时配置与故障排查"
read_when:
  - 在 Linux 服务器或工作站上部署 OpenClaw
  - 理解 systemd 用户服务与系统服务的区别和适用场景
  - 配置网关开机自启和故障恢复
  - 排查 Linux 平台特有的问题
title: "Linux 应用"
---

# OpenClaw Linux 平台完整指南

本指南全面介绍 OpenClaw 在 Linux 平台上的部署、配置和运维管理。Linux 是运行 OpenClaw 网关的**首选平台**，因为它提供了稳定的后台运行能力、完整的工具链兼容性和最佳的资源效率。阅读本指南后，你将能够独立完成 Linux 平台的完整部署，理解 systemd 服务管理机制，并掌握常见问题的诊断与解决方案。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 **Linux 作为网关宿主平台**的核心优势和技术原因
- [ ] 掌握 **Node.js 运行时**的版本要求和安装方法
- [ ] 完成 **最小化安装**：安装 CLI → 配置网关 → 验证运行
- [ ] 理解 **systemd 用户服务**的工作原理和生命周期管理

### 进阶目标（建议掌握）

- [ ] 能够配置 **systemd 系统服务**以适应服务器环境
- [ ] 掌握 **systemd 用户服务 vs 系统服务**的选择依据和转换方法
- [ ] 能够设计 **高可用部署方案**（服务监控、自动重启、资源限制）
- [ ] 理解 **进程守护**机制和**优雅重启**的实现方式

### 专家目标（挑战）

- [ ] 能够为不同 Linux 发行版**定制安装包**
- [ ] 设计 **多网关场景**的资源调度和安全隔离方案
- [ ] 掌握 **容器化部署**的最佳实践（Docker、Nix）
- [ ] 能够排查和解决 **复杂的进程管理问题**

---

## 第一部分：核心概念与架构设计

### 为什么 Linux 是首选网关平台

在选择网关运行平台时，理解各操作系统的特性差异有助于做出正确的架构决策。Linux 作为服务器操作系统的事实标准，在运行后台服务方面具有天然优势。

#### 平台特性对比

| 特性 | Linux | macOS | Windows |
|------|-------|-------|---------|
| **后台服务稳定性** | 优秀（多年运行不重启） | 良好（需用户登录） | 一般（需用户登录） |
| **工具链兼容性** | 完整（原生支持） | 良好（需 Homebrew） | 部分（WSL 推荐） |
| **资源占用** | 极低（可小于 100MB） | 中等（GUI 开销） | 中等（GUI 开销） |
| **远程管理** | SSH 原生支持 | 需额外配置 | RDP/SSH |
| **进程守护** | systemd 成熟方案 | launchd 可用 | NSSM/服务 |
| **容器支持** | 原生 | Docker Desktop | Docker Desktop |

#### 设计权衡分析

**为什么 Linux 适合运行网关**：

```
┌─────────────────────────────────────────────────────────────┐
│                    Linux 网关架构优势                        │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    稳定性层                          │   │
│   │   ├── systemd 确保服务意外退出后自动重启             │   │
│   │   ├── 开机自启无需用户登录                          │   │
│   │   └── 资源限制防止服务失控                          │   │
│   └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    兼容性层                          │   │
│   │   ├── 完整 POSIX 兼容                               │   │
│   │   ├── 所有技能工具原生运行                          │   │
│   │   └── Node.js 性能优化最佳                          │   │
│   └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    可管理性层                        │   │
│   │   ├── SSH 远程管理                                  │   │
│   │   ├── 日志集中管理                                  │   │
│   │   └── 资源监控完善                                  │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**为什么不建议使用 Bun 运行网关**：根据 OpenClaw 团队测试，Bun 在处理某些特定渠道（如 WhatsApp、Telegram）时存在兼容性问题。这些问题可能源于 Bun 的 Node.js 兼容层实现细节。Node.js 是经过充分测试的推荐运行时。

### 核心术语解析

| 术语 | 英文原文 | 说明 |
|------|----------|------|
| **systemd** | systemd | Linux 系统和服务管理器 |
| **systemd user** | systemd user | 按用户运行的服务管理 |
| **systemd system** | systemd system | 系统级服务管理 |
| **daemon** | daemon | 后台运行的守护进程 |
| **service unit** | service unit | systemd 的服务配置文件 |
| **socket unit** | socket unit | systemd 的套接字配置文件 |

---

## 第二部分：渐进式学习路径

### 阶段一：入门基础 ⭐

**目标**：完成最小化安装，实现网关的基本运行

#### 1.1 环境准备

**为什么需要环境准备**：Linux 环境多样，不同发行版的包管理器和系统配置存在差异。标准化环境可以减少后续问题。

**Node.js 版本要求**：OpenClaw 要求 **Node.js 22 或更高版本**。这是因为：
- 新版本提供更好的性能和安全修复
- OpenClaw 使用了较新的 JavaScript 特性
- 旧版本可能存在兼容性问题

**安装 Node.js**：

```bash
# 方法一：使用 nvm（推荐，灵活切换版本）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22

# 方法二：使用 n（全局安装）
sudo npm install -g n
sudo n 22

# 方法三：使用发行版包管理器（版本可能较旧）
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Fedora
sudo dnf module enable nodejs:22
sudo dnf install -y nodejs
```

**验证安装**：

```bash
node --version  # 应显示 v22.x.x 或更高
npm --version   # 应显示 10.x.x 或更高
```

#### 1.2 安装 OpenClaw CLI

**为什么使用全局安装**：全局安装使 `openclaw` 命令在系统 PATH 中可用，便于从任何位置执行。

**安装命令**：

```bash
# 使用 npm（推荐）
npm install -g openclaw@latest

# 或使用 pnpm
pnpm add -g openclaw@latest

# 验证安装
openclaw --version
```

**安装位置说明**：

```
全局安装路径：
├── npm：/usr/local/lib/node_modules/openclaw
├── pnpm：/usr/local/lib/node_modules/openclaw
└── 命令链接：/usr/local/bin/openclaw
```

#### 1.3 首次启动与验证

**快速启动流程**：

```bash
# 1. 启动网关（默认端口 18789）
openclaw gateway --port 18789

# 2. 验证网关运行状态
openclaw gateway status

# 3. 测试 WebSocket 连接
curl -s http://127.0.0.1:18789/health
# 预期输出：{"status":"ok"}
```

**预期结果**：

```
┌────────────────────────────────────────────────────────┐
│                    网关启动验证                        │
│                                                         │
│   $ openclaw gateway status                           │
│   Status: running                                     │
│   Port: 18789                                         │
│   Bind: 127.0.0.1                                     │
│   PID: 12345                                          │
│                                                         │
│   $ curl http://127.0.0.1:18789/health                │
│   {"status":"ok"}                                     │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

### 阶段二：核心功能 ⭐⭐

**目标**：掌握 systemd 服务配置，实现开机自启和后台运行

#### 2.1 systemd 用户服务配置

**为什么需要 systemd 服务**：命令行启动的进程在会话结束后会被终止。systemd 服务确保网关在后台持续运行，即使没有用户登录。

**服务安装**：

```bash
# 方法一：使用向导安装（推荐）
openclaw onboard --install-daemon

# 方法二：手动安装
openclaw gateway install

# 方法三：使用配置向导
openclaw configure
# 选择 "Gateway service" 选项
```

**服务文件结构**（自动生成）：

```ini
# ~/.config/systemd/user/openclaw-gateway.service

[Unit]
Description=OpenClaw Gateway (profile: default)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment="OPENCLAW_PROFILE=default"
MemoryMax=500M
MemoryHigh=400M
CPUQuota=200%

[Install]
WantedBy=default.target
```

**关键配置项解析**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `Type` | simple | 简单进程（默认类型） |
| `Restart` | always | 退出后总是重启 |
| `RestartSec` | 5 | 重启前等待 5 秒 |
| `MemoryMax` | 500M | 最大内存限制 |
| `MemoryHigh` | 400M | 内存高压阈值 |
| `CPUQuota` | 200% | CPU 使用上限 |

#### 2.2 服务生命周期管理

**基础管理命令**：

```bash
# 查看服务状态
systemctl --user status openclaw-gateway

# 启动服务
systemctl --user start openclaw-gateway

# 停止服务
systemctl --user stop openclaw-gateway

# 重启服务
systemctl --user restart openclaw-gateway

# 查看服务日志
journalctl --user -u openclaw-gateway -f

# 查看实时日志（带过滤）
journalctl --user -u openclaw-gateway --since "5 minutes ago"

# 禁用自启动
systemctl --user disable openclaw-gateway

# 启用自启动
systemctl --user enable openclaw-gateway
```

**服务状态解读**：

```
● openclaw-gateway.service - OpenClaw Gateway
     Loaded: loaded (/home/user/.config/systemd/user/openclaw-gateway.service; enabled)
     Active: active (running) since Thu 2024-01-01 00:00:00 CST; 1 day ago
   Main PID: 12345 (node)
      Tasks: 15 (limit: 32768)
     Memory: 150.0M (max: 500.0M)
        CPU: 2h 30m
   CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/openclaw-gateway.service
```

| 字段 | 值 | 含义 |
|------|-----|------|
| `Loaded` | loaded | 配置文件已加载 |
| `Active` | active (running) | 服务正在运行 |
| `Main PID` | 12345 | 主进程 PID |
| `Memory` | 150.0M | 当前内存使用 |
| `CPU` | 2h 30m | 累计 CPU 时间 |

#### 2.3 日志管理最佳实践

**日志查看技巧**：

```bash
# 实时跟踪日志
journalctl --user -u openclaw-gateway -f

# 查看最近 100 行日志
journalctl --user -u openclaw-gateway -n 100

# 按时间过滤
journalctl --user -u openclaw-gateway --since "2024-01-01" --until "2024-01-02"

# 包含错误信息的日志
journalctl --user -u openclaw-gateway -p err

# 导出日志到文件
journalctl --user -u openclaw-gateway > openclaw.log
```

**日志轮转配置**：

```bash
# 创建日志轮转配置
sudo nano /etc/logrotate.d/openclaw

# 配置内容：
/home/*/.local/share/openclaw/logs/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
```

---

### 阶段三：高级配置 ⭐⭐⭐

**目标**：掌握系统服务配置、故障恢复和多服务管理

#### 3.1 systemd 系统服务

**为什么需要系统服务**：用户服务依赖于用户会话。在某些场景下（如无头服务器、远程 VPS），系统服务更为合适。

**系统服务 vs 用户服务**：

| 特性 | 用户服务 | 系统服务 |
|------|----------|----------|
| **权限范围** | 当前用户 | 系统级 |
| **启动时机** | 用户登录后 | 系统启动时 |
| **配置位置** | `~/.config/systemd/user/` | `/etc/systemd/system/` |
| **管理命令** | `systemctl --user` | `sudo systemctl` |
| **适用场景** | 个人工作站 | 服务器 |
| **权限要求** | 无需 sudo | 需要 root |

**创建系统服务**：

```bash
# 1. 创建服务文件
sudo nano /etc/systemd/system/openclaw-gateway.service

# 2. 服务配置内容：
[Unit]
Description=OpenClaw Gateway Service
After=network.target

[Service]
Type=simple
User=openclaw
Group=openclaw
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment="OPENCLAW_PROFILE=default"

# 资源限制
MemoryMax=1G
MemoryHigh=800M
CPUQuota=150%
NoNewPrivileges=true
ProtectSystem=strict
ReadWritePaths=/home/openclaw/.openclaw

# 日志配置
StandardOutput=journal
StandardError=journal
SyslogIdentifier=openclaw-gateway

[Install]
WantedBy=multi-user.target

# 3. 重新加载 systemd
sudo systemctl daemon-reload

# 4. 启用并启动服务
sudo systemctl enable openclaw-gateway
sudo systemctl start openclaw-gateway
```

**安全强化配置**：

```ini
# 系统服务安全配置
[Service]
# 权限隔离
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true

# 资源限制
MemoryMax=500M
MemoryHigh=400M
CPUQuota=100%

# 文件系统保护
ReadOnlyPaths=/
ReadWritePaths=/home/openclaw/.openclaw

# 能力限制
CapabilityBoundingSet=
AmbientCapabilities=
```

#### 3.2 多配置文件管理

**使用 profile 管理多实例**：

```
~/.config/systemd/user/
├── openclaw-gateway.service          # 默认配置
├── openclaw-gateway-work.service    # 工作配置
└── openclaw-gateway-dev.service    # 开发配置
```

**profile 配置示例**：

```bash
# 安装带 profile 的服务
openclaw gateway install --profile work

# 启动特定 profile
systemctl --user start openclaw-gateway-work

# 查看所有 profile
systemctl --user list-units | grep openclaw
```

#### 3.3 故障恢复配置

**优雅重启策略**：

```ini
[Service]
# 重启策略
Restart=always
RestartSec=5

# 重启次数限制（避免无限重启）
StartLimitBurst=5
StartLimitIntervalSec=60

# 重启前的状态检查
ExecStartPre=/bin/sleep 2
ExecStartPost=/bin/bash -c 'until curl -s http://127.0.0.1:18789/health; do sleep 1; done'
```

**健康检查脚本**：

```bash
#!/bin/bash
# /usr/local/bin/openclaw-healthcheck

HEALTH_URL="http://127.0.0.1:18789/health"
TIMEOUT=5

if curl -s --max-time $TIMEOUT $HEALTH_URL | grep -q "ok"; then
    exit 0
else
    exit 1
fi
```

---

### 阶段四：专家技巧 ⭐⭐⭐⭐

**目标**：掌握容器化部署、性能调优和复杂场景处理

#### 4.1 Docker 部署

**为什么使用 Docker**：Docker 提供一致的运行环境、简化的依赖管理和便捷的部署流程。

**Dockerfile 示例**：

```dockerfile
FROM node:22-alpine

LABEL maintainer="user@example.com"
LABEL description="OpenClaw Gateway"

# 安装 CLI
RUN npm install -g openclaw@latest

# 创建非 root 用户
RUN addgroup -g 1000 openclaw && \
    adduser -u 1000 -G openclaw -s /bin/sh -D openclaw

# 切换到非 root 用户
USER openclaw

# 暴露端口
EXPOSE 18789

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -s http://127.0.0.1:18789/health || exit 1

# 启动命令
CMD ["openclaw", "gateway", "--port", "18789"]
```

**docker-compose 配置**：

```yaml
version: '3.8'

services:
  openclaw-gateway:
    build: .
    container_name: openclaw-gateway
    ports:
      - "18789:18789"
    volumes:
      - ~/.openclaw:/home/openclaw/.openclaw
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:18789/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

#### 4.2 Nix 部署

**为什么使用 Nix**：Nix 提供声明式配置、不可变基础设施和完美的环境重现性。

**shell.nix 示例**：

```nix
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
  name = "openclaw-env";
  version = "1.0";

  buildInputs = [
    pkgs.nodejs-22_x
    pkgs.openclaw
  ];

  shellHook = ''
    export OPENCLAW_HOME=$HOME/.openclaw
    export PATH=$PATH:$HOME/.local/bin
  '';
}
```

**部署配置**：

```nix
# deployment.nix
{ config, pkgs, ... }:

{
  services.openclaw-gateway = {
    enable = true;
    profile = "default";
    port = 18789;
    user = "openclaw";
    group = "openclaw";
    environmentFile = "/etc/openclaw/env";
    restartPolicy = "always";
    wantedBy = [ "multi-user.target" ];
  };
}
```

#### 4.3 性能监控与调优

**资源监控命令**：

```bash
# 实时资源使用
htop

# 内存使用详情
ps aux | grep openclaw
cat /proc/$(pgrep -f "openclaw gateway")/status

# 网络连接
ss -ltnp | grep 18789

# 磁盘 I/O
iostat -x 1
```

**性能调优参数**：

```ini
[Service]
# 内存优化
MemoryMax=500M
MemoryHigh=400M
MemoryAccounting=true

# CPU 优化
CPUAffinity=0-1
CPUQuota=150%

# 进程优化
Nice=-10
CPUSchedulingPolicy=other
```

---

## 第三部分：专家思维模型

### 思维模型一：服务分层架构

**核心思想**：将系统服务分为多个层次，每个层次有明确的职责和边界。

```
┌─────────────────────────────────────────────────────────────┐
│                       服务分层模型                           │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    接入层                            │   │
│   │   ├── 负载均衡器（如需要）                           │   │
│   │   ├── SSL/TLS 终端                                   │   │
│   │   └── 连接速率限制                                   │   │
│   └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    网关层                            │   │
│   │   ├── WebSocket 处理                                 │   │
│   │   ├── 认证与授权                                     │   │
│   │   └── 会话管理                                      │   │
│   └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    业务层                            │   │
│   │   ├── 渠道连接                                       │   │
│   │   ├── Agent 运行时                                   │   │
│   │   └── 工具执行                                       │   │
│   └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    基础设施层                        │   │
│   │   ├── systemd 进程守护                               │   │
│   │   ├── 日志系统                                      │   │
│   │   └── 监控告警                                      │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 思维模型二：故障域划分

**核心思想**：识别系统中的故障域，设计隔离策略防止级联故障。

```
┌─────────────────────────────────────────────────────────────┐
│                      故障域隔离模型                           │
│                                                              │
│   故障域 1：进程级                                            │
│   ┌───────────────────────────────────────────────────┐     │
│   │  单个服务异常 → 重启该服务                         │     │
│   │  影响范围：仅限该服务                              │     │
│   │  保护机制：systemd Restart                        │     │
│   └───────────────────────────────────────────────────┘     │
│                                                              │
│   故障域 2：用户级                                            │
│   ┌───────────────────────────────────────────────────┐     │
│   │  用户会话断开 → 用户服务停止                        │     │
│   │  影响范围：该用户的所有服务                         │     │
│   │  保护机制：使用系统服务                             │     │
│   └───────────────────────────────────────────────────┘     │
│                                                              │
│   故障域 3：系统级                                            │
│   ┌───────────────────────────────────────────────────┐     │
│   │  系统重启 → 所有服务停止                            │     │
│   │  影响范围：整个系统                                │     │
│   │  保护机制：systemd WantedBy=multi-user.target      │     │
│   └───────────────────────────────────────────────────┘     │
│                                                              │
│   故障域 4：硬件级                                            │
│   ┌───────────────────────────────────────────────────┐     │
│   │  硬件故障 → 系统不可用                              │     │
│   │  影响范围：整个机器                                │     │
│   │  保护机制：高可用集群                              │     │
│   └───────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 思维模型三：演进式部署

**核心思想**：采用渐进式变更策略，降低部署风险。

```
部署决策流程：

变更请求？
│
├── 配置变更？
│   ├── 热更新？ → systemd reload
│   └── 需要重启？ → 滚动重启
│
├── 代码更新？
│   ├── 影子模式测试？
│   │   ├── 通过 → 全量部署
│   │   └── 回滚
│   └── 蓝绿部署？
│       ├── 新版本健康 → 切换流量
│       └── 回滚旧版本
│
└── 重大升级？
    ├── 备份当前状态
    ├── 在测试环境验证
    ├── 制定回滚计划
    └── 执行升级
```

---

## 第四部分：适用场景分析

### 场景一：个人 VPS 部署

**典型用户**：拥有个人 VPS 的开发者

**推荐配置**：

```
┌─────────────────────────────────────────────────────────────┐
│                    个人 VPS 部署架构                          │
│                                                              │
│   用户机器              网络               VPS               │
│   ┌─────────┐        ┌─────────┐      ┌─────────────┐      │
│   │  Mac/   │────SSH──▶│  Tailscale│─────▶│ OpenClaw   │      │
│   │ Windows │  隧道   │  /VPN   │      │  Gateway    │      │
│   └─────────┘        └─────────┘      └─────────────┘      │
│                                              │              │
│                                              ▼              │
│                                        ┌─────────────┐    │
│                                        │  本地 CLI   │    │
│                                        │  连接       │    │
│                                        └─────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**部署步骤**：

```bash
# 1. SSH 连接到 VPS
ssh user@vps-host

# 2. 安装 OpenClaw
npm install -g openclaw@latest

# 3. 安装系统服务
sudo openclaw gateway install

# 4. 配置自动更新
# 编辑 crontab
crontab -e
# 添加：0 4 * * * npm install -g openclaw@latest

# 5. 设置 SSH 隧道（本地）
ssh -N -L 18789:127.0.0.1:18789 user@vps-host &
```

### 场景二：团队协作环境

**典型用户**：小团队共享 OpenClaw 实例

**推荐配置**：

```
安全考量：
├── 用户隔离：创建专用用户
├── 权限控制：基于 SSH 密钥的访问控制
├── 资源限制：为每个实例设置资源配额
└── 日志审计：启用详细日志记录
```

**配置示例**：

```ini
# /etc/systemd/system/openclaw-team.service
[Unit]
Description=OpenClaw Team Gateway
After=network.target

[Service]
Type=simple
User=openclaw-team
Group=openclaw-team
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=10

# 资源限制
MemoryMax=2G
MemoryHigh=1.5G
CPUQuota=200%
TasksMax=50

# 安全配置
NoNewPrivileges=true
ProtectSystem=strict
ReadWritePaths=/var/lib/openclaw-team

[Install]
WantedBy=multi-user.target
```

### 场景三：高可用部署

**典型用户**：对可用性有严格要求的企业用户

**推荐架构**：

```
┌─────────────────────────────────────────────────────────────┐
│                      高可用部署架构                           │
│                                                              │
│                    ┌─────────────┐                         │
│                    │   负载均衡   │                         │
│                    │   (HAProxy)  │                         │
│                    └──────┬──────┘                         │
│                           │                                │
│           ┌────────────────┼────────────────┐             │
│           ▼                ▼                ▼             │
│      ┌─────────┐      ┌─────────┐      ┌─────────┐      │
│      │ Node 1  │      │ Node 2  │      │ Node 3  │      │
│      │ (主)    │      │ (备)    │      │ (备)    │      │
│      └─────────┘      └─────────┘      └─────────┘      │
│           │                │                │             │
│           └────────────────┼────────────────┘             │
│                           ▼                                │
│                    ┌─────────────┐                        │
│                    │  共享存储   │                        │
│                    │ (NFS/S3)   │                        │
│                    └─────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 第五部分：故障排查指南

### 排查流程图

```
问题：Linux 网关无法正常工作
│
├── 症状：服务无法启动
│   │
│   ├── 检查 1：服务状态
│   │   └── systemctl --user status openclaw-gateway
│   │       │
│   │       ├── failed → 查看错误日志
│   │       │   └── journalctl --user -u openclaw-gateway -e
│   │       └── not found → 重新安装服务
│   │           └── openclaw gateway install
│   │
│   └── 检查 2：进程依赖
│       ├── Node.js 是否安装？
│       │   └── node --version
│       │
│       └── 端口是否被占用？
│           └── ss -ltnp | grep 18789
│
├── 症状：服务频繁重启
│   │
│   ├── 检查 1：崩溃日志
│   │   └── journalctl --user -u openclaw-gateway --since "1 hour ago"
│   │
│   └── 检查 2：资源限制
│       └── cat /sys/fs/cgroup/user.slice/user-1000.slice/openclaw-gateway.service/memory.max
│
├── 症状：无法从远程连接
│   │
│   ├── 检查 1：网络连通性
│   │   ├── 本地测试
│   │   │   └── curl http://127.0.0.1:18789/health
│   │   │
│   │   └── 远程测试
│   │       └── curl http://<vps-ip>:18789/health
│   │
│   └── 检查 2：防火墙
│       └── sudo ufw status
│           │
│           ├── 18789 端口是否允许？
│           └── sudo ufw allow 18789/tcp
│
└── 症状：资源使用异常
    │
    ├── 检查 1：内存使用
    │   └── systemctl --user status openclaw-gateway
    │       └── Memory: xxx (max: 500M)
    │
    └── 检查 2：CPU 使用
        └── top -p $(pgrep -f "openclaw gateway")
```

### 常见错误与解决方案

| 错误类型 | 症状 | 解决方案 |
|----------|------|----------|
| **端口被占用** | "Address already in use" | 找到并停止占用进程，或更改端口 |
| **权限不足** | "Permission denied" | 检查用户权限，使用 sudo |
| **Node 版本过低** | 兼容性问题 | 安装 Node 22+ |
| **systemd 不可用** | "systemd not available" | 使用前台运行模式 |
| **内存不足** | OOM Killer | 增加内存限制或优化配置 |
| **磁盘空间不足** | 无法写入日志 | 清理磁盘空间 |

### 诊断命令速查表

```bash
# 服务状态
systemctl --user status openclaw-gateway

# 查看日志
journalctl --user -u openclaw-gateway -f

# 查看进程
ps aux | grep openclaw

# 查看端口
ss -ltnp | grep 18789

# 查看资源
systemctl --user status openclaw-gateway

# 查看依赖
ldd $(which node)

# 测试健康
curl http://127.0.0.1:18789/health

# 测试远程连接
curl http://<server-ip>:18789/health

# 检查防火墙
sudo ufw status

# 检查系统日志
dmesg | grep openclaw
```

---

## 第六部分：最佳实践总结

### 安装最佳实践

```markdown
### 1. Node.js 安装

**推荐使用 nvm**（版本管理灵活）：
```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 安装并使用 Node 22
nvm install 22
nvm use 22
```

### 2. CLI 安装

**使用 npm 全局安装**：
```bash
npm install -g openclaw@latest
```

### 3. 服务安装

**推荐使用向导**：
```bash
openclaw onboard --install-daemon
```
```

### 配置最佳实践

```markdown
### 1. systemd 服务配置

**资源限制配置**：
```ini
[Service]
MemoryMax=500M
MemoryHigh=400M
CPUQuota=150%
Restart=always
RestartSec=5
```

### 2. 日志管理

**日志轮转配置**：
```bash
# /etc/logrotate.d/openclaw
/home/*/.local/share/openclaw/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
}
```

### 3. 监控告警

**资源监控脚本**：
```bash
#!/bin/bash
# /usr/local/bin/openclaw-monitor

MEM=$(systemctl --user status openclaw-gateway | grep Memory | awk '{print $2}')
if [ "${MEM//M}" -gt 450 ]; then
    echo "Warning: High memory usage: $MEM"
fi
```
```

### 安全最佳实践

```markdown
### 1. 权限最小化

**创建专用服务用户**：
```bash
sudo useradd -r -s /sbin/nologin openclaw
```

### 2. 文件系统保护

**systemd 保护指令**：
```ini
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
```

### 3. 网络安全

**防火墙配置**：
```bash
# 只允许本地访问（推荐）
sudo ufw allow from 127.0.0.1 to any port 18789

# 或限制特定 IP
sudo ufw allow from 192.168.1.0/24 to any port 18789
```
```

---

## 推荐 Linux 发行版

| 发行版 | 支持状态 | 推荐程度 | 备注 |
|--------|----------|----------|------|
| **Ubuntu 22.04+** | 完全支持 | ⭐⭐⭐⭐⭐ | 首选，社区测试最多 |
| **Debian 12+** | 完全支持 | ⭐⭐⭐⭐ | 稳定可靠，企业首选 |
| **Fedora 38+** | 支持 | ⭐⭐⭐ | 软件包较新 |
| **Arch Linux** | 支持 | ⭐⭐⭐ | 滚动更新，需自行维护 |
| **Alpine Linux** | 部分支持 | ⭐⭐⭐ | Docker 镜像常用 |
| **RHEL/CentOS 8+** | 支持 | ⭐⭐⭐ | 企业环境适用 |

---

## 相关文档

- [快速开始](/zh-cn/start/getting-started)
- [安装与更新](/zh-cn/install/updating)
- [网关操作手册](/zh-cn/gateway)
- [配置参考](/zh-cn/gateway/configuration)
- [Docker 部署](/zh-cn/install/docker)
- [Nix 安装](/zh-cn/install/nix)
- [exe.dev VM 运维指南](/zh-cn/platforms/exe-dev)
- [远程网关访问](/zh-cn/gateway/remote)
