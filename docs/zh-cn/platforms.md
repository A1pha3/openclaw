---
summary: "部署平台完整指南——在 macOS、Linux、Windows WSL2、Docker、Fly.io、Hetzner 等平台上运行 OpenClaw，包含安装步骤、配置要求和最佳实践"
read_when:
  - 选择 OpenClaw 的部署平台
  - 在特定操作系统上安装 OpenClaw
  - 配置容器化或云端部署
  - 了解不同平台的特性和限制
title: "部署平台"
---

# 💻 部署平台

OpenClaw 作为一款跨平台的个人 AI 助手，支持在多种操作系统和部署环境中运行。不同的部署平台具有各自的优势和适用场景，选择合适的平台是成功部署的第一步。本文档将详细介绍各个平台的安装流程、配置要求和最佳实践，帮助你在最适合的环境中运行 OpenClaw。

> **部署决策的重要性**
>
> 部署平台的选择直接影响 OpenClaw 的性能表现、功能完整性和运维复杂度。macOS 提供了最完整的功能支持，包括原生 iMessage 和语音功能；Linux 是服务器部署的首选，稳定且资源占用低；Windows 需要通过 WSL2 运行；Docker 则提供了最大的部署灵活性。选择时请考虑你的实际需求、技术能力和资源条件。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解各平台的特点、优势和限制
- 掌握在目标平台上安装 OpenClaw 的完整流程
- 完成基础配置并验证安装成功
- 理解平台特定的功能差异和配置要求

### 进阶目标（建议掌握）

- 针对生产环境进行优化配置
- 实现跨平台部署和迁移
- 配置高可用和负载均衡方案
- 处理平台特定的故障和问题

### 专家目标（挑战）

- 开发自定义平台适配层
- 实现定制化的容器化部署
- 设计多区域分布式部署架构

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速体验** | 首次部署，想快速体验 | 30 分钟 | 选择简单平台，完成基础安装 |
| **个人使用** | 个人用户，需要完整功能 | 1 小时 | macOS 或 Linux 完整配置 |
| **团队部署** | 团队共享使用 | 2 小时 | 服务器部署、用户管理 |
| **生产环境** | 生产级部署要求 | 4 小时 | 高可用、监控、备份 |

---

## 🎛️ 平台对比与选择

在选择部署平台之前，理解各平台的特点和适用场景至关重要。以下从多个维度进行详细对比：

### 平台特性矩阵

| 特性 | macOS | Linux | Windows WSL2 | Docker | Fly.io |
|------|-------|-------|--------------|--------|--------|
| **功能完整度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **安装复杂度** | 简单 | 简单 | 中等 | 简单 | 简单 |
| **资源消耗** | 中等 | 低 | 中等 | 中到高 | 按需付费 |
| **稳定性** | 高 | 非常高 | 高 | 高 | 高 |
| **远程访问** | 需配置 | 原生支持 | 需配置 | 需配置 | 原生支持 |
| **iMessage 支持** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **语音功能** | ✅ | ⚠️ 有限 | ❌ | ❌ | ❌ |
| **适合场景** | 个人使用 | 服务器部署 | Windows 用户 | 容器化部署 | 边缘部署 |

### 平台选择决策树

```
我的主要使用场景是什么？
├── 个人日常使用
│   ├── 使用 macOS → 选择 macOS（功能最全）
│   └── 使用 Windows → 选择 WSL2
│
├── 服务器部署
│   ├── 有 Linux 服务器 → 选择 Linux
│   └── 想简化部署 → 选择 Docker
│
├── 追求最大灵活性
│   ├── 熟悉容器 → 选择 Docker
│   └── 想要全球部署 → 选择 Fly.io
│
└── 开发和测试
        → 选择 Docker（易于清理和重建）
```

---

## 🍎 macOS 平台

macOS 是 OpenClaw 功能最完整的运行平台，提供原生支持的所有特性，包括 iMessage、语音功能和系统集成。

### 为什么选择 macOS

macOS 作为部署平台有几个独特优势：首先是**功能完整性**——iMessage 集成、语音合成、系统通知等特性只有在 macOS 上才能完全实现；其次是**用户体验**——菜单栏应用提供了便捷的快捷操作和控制界面；第三是**开发友好**——对于开发者而言，macOS 与 Linux 环境相近，便于本地开发和测试。

### 系统要求

**最低要求**：macOS 12 (Monterey) 或更新版本；4GB 可用内存；2GB 可用磁盘空间；稳定的网络连接。

**推荐配置**：macOS 14 (Sonoma) 或更新版本；8GB 或更多内存；10GB 或更多可用空间；SSD 存储。

### 安装流程

**方法一：Homebrew 安装（推荐）**

```bash
# 如果尚未安装 Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 OpenClaw
brew install openclaw/openclaw/openclaw

# 启动引导程序
openclaw onboard --install-daemon
```

**方法二：安装脚本**

```bash
# 使用官方安装脚本
curl -fsSL https://openclaw.ai/install.sh | bash

# 启动服务
openclaw onboard --install-daemon
```

**方法三：手动安装**

```bash
# 下载最新版本
curl -L https://github.com/openclaw/openclaw/releases/latest/download/openclaw-universal.dmg -o openclaw.dmg

# 挂载并安装
hdiutil attach openclaw.dmg
cp -R "/Volumes/OpenClaw/OpenClaw.app" /Applications/

# 启动引导程序
/Applications/OpenClaw.app/Contents/MacOS/openclaw onboard --install-daemon
```

### macOS 特有功能配置

**iMessage 配置**

macOS 版本支持原生 iMessage 集成，这是其他平台无法提供的独特功能：

```bash
# 验证 iMessage 连接状态
openclaw channels status imessage

# 配置 iMessage 渠道
openclaw channels login imessage
```

配置 iMessage 需要满足几个前提条件：首先，你的 Mac 必须登录 Apple ID 并启用 iMessage；其次，需要在系统偏好设置中允许 OpenClaw 访问信息应用；最后，确保防火墙和安全设置不会阻止本地通信。

**语音输出配置**

OpenClaw 在 macOS 上支持高质量的语音合成输出：

```bash
# 查看可用的语音包
openclaw voices list

# 配置默认语音
openclaw config set voice.provider "system"
openclaw config set voice.voice "Samantha"
openclaw config set voice.rate 1.0
```

macOS 内置的语音引擎提供了自然流畅的语音合成效果，你可以在系统设置中预览和选择不同的语音包。语速和音调也可以根据个人喜好进行调整。

**菜单栏应用**

macOS 版本的 OpenClaw 提供了便捷的菜单栏应用，提供快速访问和控制功能：

- **状态指示器**——显示当前 Gateway 连接状态和消息通知
- **快捷操作**——快速发送消息、启动对话、切换代理
- **健康监控**——实时显示系统资源使用情况
- **偏好设置**——一键访问常用配置选项

通过菜单栏应用，你可以在不使用浏览器的情况下完成大多数日常操作。

### 故障排查

**问题一：Gatekeeper 阻止运行**

症状：安装后双击应用提示"无法打开，因为 Apple 无法检查是否包含恶意软件"。

解决方案：前往"系统设置" > "隐私与安全性"，找到"安全性"部分，点击"仍要打开"；或者在终端中执行 `xattr -cr /Applications/OpenClaw.app`。

**问题二：iMessage 无法连接**

症状：iMessage 渠道显示连接中但无法收发消息。

解决方案：首先检查 Apple ID 登录状态，确保 iMessage 已激活；其次验证系统权限，在"系统设置" > "隐私与安全性"中确保 OpenClaw 有信息访问权限；最后重启信息应用 `killall Messages` 然后重试。

**问题三：内存占用过高**

症状：OpenClaw 运行一段时间后内存占用持续增长。

解决方案：定期重启 Gateway 服务；减少同时连接的渠道数量；限制会话历史长度；检查是否有内存泄漏（通过 `openclaw doctor` 诊断）。

---

## 🐧 Linux 平台

Linux 是 OpenClaw 服务器部署的首选平台，稳定、高效、资源占用低。大多数云服务器都运行 Linux，非常适合长期运行的 AI 助手服务。

### 为什么选择 Linux

Linux 作为服务器平台的优势在于：**稳定性**——Linux 服务器可以长时间运行而不需要重启，适合需要 24/7 运行的服务；**资源效率**——相比桌面操作系统，Linux 的资源占用更低，更多资源可以用于 AI 推理；**远程管理**——通过 SSH 可以完全远程管理服务器；**生态完善**——丰富的服务器运维工具和脚本支持。

### 系统要求

**最低要求**：Ubuntu 20.04 LTS 或等效发行版；2GB RAM；1GB 可用磁盘空间；Node.js 22+。

**推荐配置**：Ubuntu 22.04 LTS 或更新版本；4GB 或更多 RAM；10GB 或更多可用空间；SSD 存储；稳定的公网 IP。

### 安装流程

**方法一：APT 安装（Ubuntu/Debian）**

```bash
# 添加 OpenClaw 软件源
echo 'deb [trusted=yes] https://apt.openclaw.ai/ stable main' | \
  sudo tee /etc/apt/sources.list.d/openclaw.list

# 更新并安装
sudo apt update
sudo apt install openclaw

# 启动引导程序
sudo openclaw onboard --install-daemon
```

**方法二：安装脚本**

```bash
# 使用官方安装脚本（自动检测系统）
curl -fsSL https://openclaw.ai/install.sh | bash

# 启动服务
openclaw onboard --install-daemon
```

**方法三：从源码安装（开发者）**

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖
pnpm install

# 构建项目
pnpm build

# 运行
pnpm openclaw onboard --install-daemon
```

### systemd 服务配置

在 Linux 上，建议将 OpenClaw 配置为 systemd 服务，确保开机自启动和稳定运行：

```bash
# 创建服务文件
sudo nano /etc/systemd/system/openclaw.service
```

服务文件内容：

```ini
[Unit]
Description=OpenClaw Personal AI Assistant
After=network.target

[Service]
Type=simple
User=username
WorkingDirectory=/home/username
ExecStart=/usr/bin/openclaw gateway run
Restart=always
RestartSec=10
Environment=NODE_ENV=production
EnvironmentFile=/etc/openclaw/environment

# 日志配置
StandardOutput=journal
StandardError=journal
SyslogIdentifier=openclaw

# 资源限制
MemoryMax=2G
CPUQuota=80%

[Install]
WantedBy=multi-user.target
```

启用和管理服务：

```bash
# 重新加载配置
sudo systemctl daemon-reload

# 启用开机自启动
sudo systemctl enable openclaw

# 启动服务
sudo systemctl start openclaw

# 查看状态
sudo systemctl status openclaw

# 查看日志
sudo journalctl -u openclaw -f
```

### Nginx 反向代理配置

为了通过域名访问 Web 界面，建议配置 Nginx 反向代理：

```nginx
# /etc/nginx/sites-available/openclaw
server {
    listen 80;
    server_name openclaw.example.com;

    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name openclaw.example.com;

    # SSL 证书配置
    ssl_certificate /etc/letsencrypt/live/openclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.example.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # OpenClaw WebSocket 和 Web 界面
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 健康检查端点
    location /health {
        proxy_pass http://127.0.0.1:18789/health;
    }
}
```

### 故障排查

**问题一：端口被占用**

症状：启动 Gateway 时提示端口 18789 已被占用。

解决方案：使用 `sudo lsof -i :18789` 查找占用端口的进程；如果确定可以停止该进程，使用 `sudo kill <PID>` 终止；或者修改 OpenClaw 端口配置 `openclaw config set gateway.port 18790`。

**问题二：Node.js 版本过低**

症状：启动时提示"Node.js version 22+ required"。

解决方案：使用 `node -v` 检查当前版本；使用 `nvm` 安装新版本 `nvm install 22 && nvm use 22`；或者通过包管理器安装最新版本。

**问题三：外网无法访问**

症状：内网可以访问，但外网无法连接 Web 界面。

解决方案：检查防火墙设置 `sudo ufw status`；开放端口 `sudo ufw allow 443/tcp`；检查云服务器安全组配置；验证 Nginx 配置正确并重启服务。

---

## 🪟 Windows 平台

Windows 用户需要通过 WSL2（Windows Subsystem for Linux 2）运行 OpenClaw。原生 Windows 支持目前仍在规划中，WSL2 提供了最佳兼容性和性能。

### 为什么选择 WSL2

WSL2 是 Microsoft 提供的 Linux 虚拟机环境，在 Windows 上提供了几乎完整的 Linux 功能。相比传统虚拟机，WSL2 启动更快、资源占用更低、与 Windows 文件系统集成更好。对于不想使用虚拟机但又需要 Linux 环境的 Windows 用户，WSL2 是最佳选择。

### 系统要求

**最低要求**：Windows 10 22H2 或更新版本；启用 WSL2 功能；4GB RAM 分配给 WSL2；1GB 可用磁盘空间。

**推荐配置**：Windows 11；启用 WSL2 和虚拟机平台；8GB 或更多 RAM 分配给 WSL2；SSD 存储。

### 安装流程

**步骤一：启用 WSL2 功能**

```powershell
# 以管理员身份运行 PowerShell
# 启用 WSL 和虚拟机平台
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 重启计算机
Restart-Computer
```

**步骤二：安装 WSL2 发行版**

```powershell
# 安装 Ubuntu（推荐）
wsl --install -d Ubuntu

# 或者安装其他发行版
wsl --install -d Debian
wsl --install -d Fedora
```

**步骤三：在 WSL2 中安装 OpenClaw**

```bash
# 进入 WSL2 终端
wsl -d Ubuntu

# 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 启动引导程序
openclaw onboard --install-daemon
```

**步骤四：配置端口转发（可选）**

如果需要从 Windows 访问 WSL2 中运行的 OpenClaw：

```powershell
# 以管理员身份运行 PowerShell
# 添加端口转发规则
netsh interface portproxy add v4tov4 listenport=18789 listenaddress=0.0.0.0 connectport=18789 connectaddress=127.0.0.1

# 验证转发规则
netsh interface portproxy show all
```

### WSL2 与 Windows 集成

**文件访问**

WSL2 中的文件可以通过以下路径在 Windows 资源管理器中访问：

```
\\wsl$\Ubuntu\home\<username>\.openclaw\
```

**从 Windows 启动**

创建 Windows 快捷方式启动 OpenClaw：

```
目标：C:\Windows\System32\wsl.exe -d Ubuntu /usr/bin/openclaw gateway run
工作目录：C:\Users\<username>
```

**IDE 集成**

推荐使用 VS Code 的 WSL 扩展进行开发：

```bash
# 在 WSL2 中
code .
```

### 故障排查

**问题一：WSL2 不支持**

症状：安装过程中提示 WSL2 不支持。

解决方案：检查 BIOS/UEFI 设置，确保虚拟化技术已启用；在 Windows 功能中确保"虚拟机平台"已启用；更新到 Windows 11 或 Windows 10 22H2+。

**问题二：内存占用过高**

症状：WSL2 占用过多 Windows 内存。

解决方案：创建 WSL2 配置文件 `%USERPROFILE%\.wslconfig`：

```ini
[wsl2]
memory=4GB
processors=4
swap=2GB
```

然后重启 WSL2：`wsl --shutdown` 然后重新启动。

---

## 🐳 Docker 容器部署

Docker 提供了最大化的部署灵活性，适用于任何支持 Docker 的环境。

### 为什么选择 Docker

Docker 部署的优势在于：**环境一致性**——开发和生产环境保持一致，消除"在我机器上能运行"的问题；**隔离性**——容器之间相互隔离，不会影响主机系统；**可移植性**——一次构建，到处运行；**易于管理**——使用 Docker Compose 可以轻松管理多容器部署。

### 系统要求

**最低要求**：Docker Engine 20.10+ 或 Docker Desktop；2GB 可用内存；1GB 可用磁盘空间。

**推荐配置**：Docker Desktop 4.0+ 或 Docker Engine 24+；4GB 或更多内存；10GB 或更多磁盘空间。

### 安装流程

**方法一：快速启动**

```bash
# 拉取最新镜像
docker pull openclaw/openclaw:latest

# 运行容器
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  openclaw/openclaw:latest
```

**方法二：Docker Compose（推荐）**

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "18789:18789"
    volumes:
      - ~/.openclaw:/root/.openclaw
      - ./data:/data
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "openclaw", "health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

启动和管理：

```bash
# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f

# 停止服务
docker compose down

# 更新镜像
docker compose pull
docker compose up -d
```

### 数据持久化

OpenClaw 的数据需要持久化存储，确保容器重启后数据不丢失：

| 数据类型 | 挂载路径 | 说明 |
|----------|----------|------|
| 配置目录 | `~/.openclaw` | 配置文件、凭证、密钥 |
| 工作区 | `~/.openclaw/workspace` | 用户工作区文件 |
| 数据目录 | `./data` | 日志、缓存、临时文件 |

### 生产配置

```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "127.0.0.1:18789:18789"
    volumes:
      - ~/.openclaw:/root/.openclaw
      - openclaw_data:/data
    environment:
      - NODE_ENV=production
      - TZ=Asia/Shanghai
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

volumes:
  openclaw_data:
```

### 故障排查

**问题一：容器无法启动**

症状：容器启动后立即退出或重启循环。

解决方案：查看容器日志 `docker logs openclaw`；检查配置文件是否有语法错误；验证挂载目录权限。

**问题二：端口冲突**

症状：提示端口已被占用。

解决方案：修改本地端口映射 `-p 18790:18789`；或停止占用端口的容器。

---

## ☁️ 云端部署

### Fly.io 边缘部署

Fly.io 提供了全球边缘部署能力，适合需要低延迟访问的用户。

**快速部署**：

```bash
# 安装 Fly CLI
curl -L https://fly.io/install.sh | sh

# 登录
fly auth login

# 启动 OpenClaw
fly launch

# 配置持久卷
fly volumes create openclaw_data -s 10
```

**fly.toml 配置**：

```toml
app = "openclaw"
primary_region = "iad"

[build]

[http_service]
  internal_port = 18789
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true

[[mounts]]
  source = "openclaw_data"
  destination = "/root/.openclaw"

[vm]
  size = "shared-cpu-1x"
  memory = "1gb"
  cpus = 1
```

### Hetzner 服务器部署

Hetzner 提供了高性价比的专用服务器选择：

```bash
# 通过 SSH 连接到服务器
ssh root@<server-ip>

# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 运行 OpenClaw
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  -v /mnt/storage:/data \
  openclaw/openclaw:latest
```

---

## 📋 平台选择决策指南

### 场景一：个人日常使用

**推荐平台**：macOS（如果使用 Mac）或 Linux（如果有服务器）

选择理由：macOS 提供最完整的功能，包括 iMessage 和语音功能；Linux 在服务器上运行稳定，可以 24/7 在线。

### 场景二：团队协作服务

**推荐平台**：Linux 服务器或 Docker

选择理由：服务器可以支持多用户同时访问，Docker 便于团队成员共享相同配置。

### 场景三：开发和测试环境

**推荐平台**：Docker

选择理由：容器便于快速创建和销毁，不会污染主机环境；团队成员可以使用相同的容器镜像。

### 场景四：移动场景

**推荐平台**：macOS + iOS 节点 或 Android 节点

选择理由：移动节点让 AI 助手可以随时随地使用。

---

## 🛠️ 故障排查通用指南

### 日志收集

无论在哪个平台遇到问题，首先收集日志：

```bash
# 获取最近 100 行日志
openclaw logs --tail 100

# 获取错误日志
openclaw logs --level error

# 导出日志文件
openclaw logs --export openclaw-logs.txt
```

### 健康检查

使用 `openclaw doctor` 进行全面系统检查：

```bash
# 运行诊断
openclaw doctor

# 详细诊断模式
openclaw doctor --verbose
```

### 常见问题快速参考

| 问题 | 症状 | 快速解决方案 |
|------|------|--------------|
| Gateway 无法启动 | 端口占用或连接拒绝 | `lsof -i :18789` |
| 渠道连接失败 | 凭证错误或超时 | `openclaw doctor --channel` |
| AI 无响应 | 代理卡住或超时 | `openclaw agent restart` |
| 内存不足 | OOM 或系统变慢 | 增加内存或限制并发 |

---

## 📖 相关文档

- [安装指南](/zh-CN/start/installation)——详细安装步骤
- [配置参考](/zh-CN/config/reference)——完整配置选项
- [运维指南](/zh-CN/operations)——生产环境运维
- [故障排除](/zh-CN/help/troubleshooting)——问题解决指南
