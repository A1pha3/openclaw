---
summary: 介绍在各种环境中部署 OpenClaw，涵盖部署模式、本地部署、Docker 部署、云平台、VPS 部署、远程访问和高可用部署
read_when:
  - 部署 OpenClaw 到生产环境时
  - 选择部署方式时
  - 配置远程访问或高可用时
title: 部署指南
---

# 部署指南

本指南详细说明如何在各种生产环境中部署 OpenClaw。在开始部署之前，请确保您已阅读并理解[运维手册](/zh-CN/operations/index)中的架构设计和运维原则。部署是运维工作的起点，正确的部署方案将为后续的稳定运行奠定坚实基础。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 根据业务需求和技术条件选择最适合的部署模式
- 独立完成从环境准备到服务上线的完整部署流程
- 配置安全的远程访问机制
- 实施高可用部署方案（针对大规模场景）
- 建立部署验证和回滚机制

### 适用场景

| 场景 | 推荐部署方式 | 预期学习时间 |
|------|-------------|-------------|
| 个人使用/开发测试 | 本地部署 | 30分钟 |
| 小团队生产环境 | 本地部署 + systemd/launchd | 1小时 |
| 需要容器化的环境 | Docker 部署 | 1-2小时 |
| 无本地服务器条件 | VPS 部署 | 1-2小时 |
| 大规模/高可用需求 | 多实例/主备部署 | 3-4小时 |

## 部署模式概览

### 模式选择的核心考量因素

选择部署模式时，需要综合评估以下维度：

**数据敏感度评估**：OpenClaw 处理的是用户的即时通讯数据，其中可能包含敏感的个人信息。部署模式的选择直接影响数据的存储位置和传输路径。在评估数据敏感度时，需要考虑以下几个层面：首先是渠道凭证的安全性，不同渠道的凭证存储方式不同，WhatsApp 的会话文件需要本地持久化，而 OAuth 令牌可以集中管理；其次是会话数据的隐私保护，代理会话中可能包含用户的对话历史和偏好设置；最后是配置信息的泄露风险，包括 API 密钥、令牌等敏感配置。

**运维能力匹配**：不同部署模式对运维人员的技术能力有不同要求。本地部署方案相对简单，主要依赖操作系统自带的服务管理功能；Docker 部署需要容器技术的基本知识；云平台部署则需要了解各平台的特定配置和限制。选择超出团队能力的部署方式会增加运维复杂度和故障风险。

**资源成本权衡**：部署模式的成本结构差异显著。本地部署主要是一次性硬件成本；VPS 部署是持续性的月度费用；云平台部署的成本则取决于流量和使用模式。在选择时需要综合考虑初始投入和长期运营成本。

### 部署模式对比矩阵

| 维度 | 本地部署 | Docker 部署 | VPS 部署 | 云平台部署 |
|------|---------|-------------|---------|------------|
| 数据控制 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| 部署复杂度 | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 运维难度 | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| 成本可控性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| 扩展能力 | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 适用规模 | 个人/小团队 | 小团队/多环境 | 中小团队 | 中大型团队 |

## 本地部署

### 环境准备

**系统要求详解**：

在开始部署之前，需要确保目标系统满足以下要求。这些要求基于 OpenClaw 的技术架构和实际运行需求制定：

| 组件 | 最低要求 | 推荐配置 | 说明 |
|------|---------|---------|------|
| 操作系统 | macOS 12+ / Ubuntu 20.04+ / Debian 11+ | 最新稳定版 | 推荐使用 LTS 版本以获得更好的稳定性 |
| CPU | 1 核心 | 2+ 核心 | 核心数量影响并行处理能力 |
| 内存 | 1GB | 2GB+ | 代理运行时需要一定内存空间 |
| 存储 | 10GB 可用空间 | 50GB+ SSD | 主要用于日志和会话存储 |
| Node.js | >= 22.12.0 | 22.x 最新版 | 严格版本要求，不支持 21.x |

**Node.js 安装验证**：

```bash
# 检查当前版本
node --version

# 需要输出 >= 22.12.0，例如：
# v22.12.0

# 如果版本不满足要求，使用 nvm 安装
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc  # 或 ~/.zshrc
nvm install 22
nvm use 22
nvm alias default 22

# 验证安装
node --version
npm --version
```

### 系统服务安装

**推荐方式：使用向导安装服务**

OpenClaw 提供了交互式向导来简化服务安装过程：

```bash
# 运行安装向导（推荐）
openclaw onboard --install-daemon

# 向导会依次引导您完成：
# 1. 环境检查（Node.js 版本、依赖项）
# 2. 配置创建（配置文件生成）
# 3. 服务安装（systemd/launchd）
# 4. 启动验证（健康检查）
```

**手动安装 systemd 服务（Linux）**：

对于需要精细控制的场景，可以手动创建 systemd 服务文件：

```bash
# 1. 创建服务目录
mkdir -p ~/.config/systemd/user

# 2. 创建服务文件
cat > ~/.config/systemd/user/openclaw-gateway.service << 'EOF'
[Unit]
Description=OpenClaw Personal AI Assistant Gateway
Documentation=https://docs.openclaw.ai/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/openclaw gateway run \
    --bind loopback \
    --port 18789 \
    --verbose
Restart=always
RestartSec=10
RestartPreventExitStatus=100  # 避免无限重启循环

# 资源限制
MemoryMax=1.5G
MemoryHigh=1.2G
CPUQuota=80%

# 日志配置
StandardOutput=journal
StandardError=journal
SyslogIdentifier=openclaw-gateway

# 环境变量（如需要）
Environment=CLAWDBOT_GATEWAY_TOKEN=your_secure_token_here
Environment=ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}

# 安全加固
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=~/.clawdbot

[Install]
WantedBy=default.target
EOF

# 3. 重新加载 systemd
systemctl --user daemon-reload

# 4. 启用开机自启动
systemctl --user enable openclaw-gateway

# 5. 启动服务
systemctl --user start openclaw-gateway

# 6. 验证状态
systemctl --user status openclaw-gateway
```

**手动安装 launchd 服务（macOS）**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.gateway</string>

    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>gateway</string>
        <string>run</string>
        <string>--bind</string>
        <string>loopback</string>
        <string>--port</string>
        <string>18789</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
        <key>Crashed</key>
        <true/>
    </dict>

    <key>ProcessType</key>
    <string>Background</string>

    <key>StandardOutPath</key>
    <string>/tmp/openclaw/openclaw.log</string>

    <key>StandardErrorPath</key>
    <string>/tmp/openclaw/openclaw-error.log</string>

    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin</string>
        <key>CLAWDBOT_GATEWAY_TOKEN</key>
        <string>your_secure_token_here</string>
    </dict>

    <key>SoftResourceLimits</key>
    <dict>
        <key>NumberOfFiles</key>
        <integer>1024</key>
        <key>StackSize</key>
        <integer>8388608</integer>  <!-- 8MB -->
    </dict>
</dict>
</plist>
```

### 手动运行（调试场景）

对于需要排查问题或进行开发的场景，可以在前台手动运行：

```bash
# 前台运行（开发/调试模式）
openclaw gateway --port 18789 --verbose

# 详细日志模式
openclaw gateway --port 18789 --verbose --log-level debug

# 指定配置文件
openclaw gateway --config /path/to/custom/config.json --verbose

# 查看帮助信息
openclaw gateway --help
```

## Docker 部署

### Docker 部署的适用场景

在选择 Docker 部署之前，需要评估其适用性。以下场景适合使用 Docker：

**适合 Docker 部署的场景**：

- 开发/测试环境需要快速创建和销毁
- 多环境部署需要一致性保障（开发、测试、预发布）
- 团队成员技术栈以容器为主
- 需要与现有容器编排平台集成
- 对资源隔离有较高要求

**不适合 Docker 部署的场景**：

- 个人使用场景，本地部署更简单高效
- 对性能要求极高的场景，容器会有额外开销
- 需要频繁访问本地硬件资源（如 USB 设备）
- 渠道凭证需要本地持久化且安全性要求极高

### 快速启动

**使用 Docker Run 快速启动**：

```bash
# 创建数据目录
mkdir -p ~/.clawdbot

# 运行容器
docker run -d \
    --name openclaw \
    --restart unless-stopped \
    -p 127.0.0.1:18789:18789 \
    -v ~/.clawdbot:/root/.clawdbot \
    -e ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY} \
    -e CLAWDBOT_GATEWAY_TOKEN=${GATEWAY_TOKEN} \
    ghcr.io/openclaw/openclaw:latest \
    gateway --port 18789

# 验证运行状态
docker ps | grep openclaw

# 查看日志
docker logs -f openclaw

# 健康检查
curl http://127.0.0.1:18789/health
```

### docker-compose 部署

**生产级 docker-compose 配置**：

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: ghcr.io/openclaw/openclaw:latest
    container_name: openclaw-gateway
    ports:
      # 严格限制仅本地访问
      - "127.0.0.1:18789:18789"
      # Web UI 端口（如果需要）
      - "127.0.0.1:18793:18793"
    volumes:
      # 持久化数据
      - openclaw-data:/root/.clawdbot
      # 可选：映射配置文件
      # - ./openclaw.json:/root/.clawdbot/openclaw.json:ro
    environment:
      # 必需的环境变量
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - CLAWDBOT_GATEWAY_TOKEN=${GATEWAY_TOKEN}
      # 可选：其他 API 密钥
      # - OPENAI_API_KEY=${OPENAI_API_KEY}
    command: >
      gateway
      --port 18789
      --bind 127.0.0.1
      --verbose
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "openclaw", "health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    # 安全配置
    user: "1000:1000"  # 非 root 用户运行
    read_only: true    # 只读根文件系统
    tmpfs:
      - /tmp:size=100M,mode=1777
    # 资源限制
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
    # 日志配置
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "3"

volumes:
  openclaw-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/openclaw  # 改为实际数据路径
```

**启动和管理**：

```bash
# 创建数据目录（宿主机上）
sudo mkdir -p /data/openclaw
sudo chown 1000:1000 /data/openclaw

# 启动服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 健康检查
docker-compose exec openclaw openclaw health

# 重启服务
docker-compose restart openclaw

# 停止服务
docker-compose down
```

### 构建自定义镜像

对于需要预装配置或插件的场景，可以构建自定义镜像：

```dockerfile
# Dockerfile.custom
FROM ghcr.io/openclaw/openclaw:latest

# 设置工作目录
WORKDIR /root/.clawdbot

# 复制配置文件
COPY openclaw.json ./

# 复制初始技能（如果有）
COPY skills/ ./skills/

# 安装插件
RUN openclaw plugins install @openclaw/example-plugin

# 设置非 root 用户
RUN chown -R 1000:1000 /root/.clawdbot

# 切换用户
USER 1000:1000

# 默认命令
CMD ["gateway", "--port", "18789", "--bind", "127.0.0.1"]
```

## 云平台部署

### 云平台部署的独特挑战

在云平台部署 OpenClaw 时，需要特别关注以下挑战：

**数据持久化风险**：云平台的磁盘通常是网络存储，数据安全性依赖于云服务商的基础设施。对于存储敏感凭证的场景，需要评估数据加密、访问控制和备份策略。

**网络连通性**：云平台的网络环境可能与本地网络有显著差异。某些即时通讯渠道可能需要特定的出站规则或代理配置才能正常工作。

**成本控制**：云平台的按需计费模式可能导致成本超出预期。需要建立监控机制，及时发现异常的资源使用。

### Railway 部署

Railway 是一个现代化的云平台，提供简洁的部署体验：

**部署步骤**：

1. 创建 Railway 账户并连接 GitHub
2. 创建新项目，选择 "Empty Project" 或 "Deploy from GitHub"
3. 配置环境变量
4. 部署服务

**railway.json 配置**：

```json
{
  "$schema": "https://railway.app/schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "restartPolicy": {
      "type": "always"
    },
    "healthcheck": {
      "command": "openclaw health",
      "interval": 30,
      "timeout": 10,
      "retries": 3
    }
  }
}
```

**环境变量配置**：

| 变量名 | 必需 | 说明 |
|-------|------|------|
| `ANTHROPIC_API_KEY` | 是 | Anthropic API 密钥 |
| `CLAWDBOT_GATEWAY_TOKEN` | 建议 | 网关认证令牌 |
| `OPENAI_API_KEY` | 可选 | OpenAI API 密钥（备用） |

### Render 部署

Render 提供托管的 Docker 服务：

**render.yaml 配置**：

```yaml
services:
  - type: web
    name: openclaw
    env: docker
    dockerImage: ghcr.io/openclaw/openclaw:latest
    dockerCommand: gateway --port $PORT --bind 0.0.0.0
    healthCheckPath: /health
    envVars:
      - key: ANTHROPIC_API_KEY
        fromDatabase:
          name: openclaw-db
          property: connectionString
      - key: CLAWDBOT_GATEWAY_TOKEN
        generateValue: true
    disk:
      name: openclaw-data
      mountPath: /root/.clawdbot
      sizeGB: 10
```

### Fly.io 部署

Fly.io 提供边缘部署能力，适合需要低延迟的场景：

**部署步骤**：

```bash
# 1. 安装 Fly CLI
curl -L https://fly.io/install.sh | sh

# 2. 登录
fly auth login

# 3. 初始化应用
fly launch --name openclaw --org personal --no-deploy

# 4. 创建持久化存储
fly volumes create openclaw_data --size 10 --region your-region

# 5. 更新配置文件
cat > fly.toml << 'EOF'
app = "openclaw"

[build]
  image = "ghcr.io/openclaw/openclaw:latest"

[http_service]
  internal_port = 18789
  force_https = true

[[services]]
  protocol = "tcp"
  internal_port = 18789

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

  [[services.http_checks]]
    path = "/health"
    interval = 30000
    timeout = 10000

[mounts]
  source = "openclaw_data"
  destination = "/root/.clawdbot"

[processes]
  app = ["gateway", "--port", "18789", "--bind", "0.0.0.0"]
EOF

# 6. 部署
fly deploy
```

## VPS 部署

### Ubuntu/Debian 系统部署

**完整部署脚本**：

```bash
#!/bin/bash
# deploy-openclaw.sh - OpenClaw VPS 部署脚本

set -euo pipefail

echo "=========================================="
echo "OpenClaw VPS 部署脚本"
echo "=========================================="

# 检查是否为 root 用户
if [ "$EUID" -ne 0 ]; then
    echo "错误: 请使用 root 用户运行或 sudo"
    exit 1
fi

# 设置变量
OPENCLAW_USER="openclaw"
OPENCLAW_VERSION="latest"
NODE_VERSION="22"

echo "[1/7] 准备系统环境..."

# 创建专用用户
if ! id "$OPENCLAW_USER" &>/dev/null; then
    useradd -m -s /bin/bash "$OPENCLAW_USER"
fi

# 更新系统
apt update && apt upgrade -y

# 安装依赖
apt install -y curl git build-essential libnss3 libatk-bridge2.0-0 \
    libdrm2 libxkbcommon0 libgtk-3-0 libgbm1 libasound2

echo "[2/7] 安装 Node.js ${NODE_VERSION}..."

# 安装 NodeSource 仓库
curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash -
apt-get install -y nodejs

# 验证版本
node --version

echo "[3/7] 安装 OpenClaw..."

# 安装 OpenClaw
npm install -g openclaw@${OPENCLAW_VERSION}

echo "[4/7] 创建数据目录..."

# 创建数据目录
mkdir -p /var/lib/openclaw
chown -R "$OPENCLAW_USER:$OPENCLAW_USER" /var/lib/openclaw

# 创建日志目录
mkdir -p /var/log/openclaw
chown -R "$OPENCLAW_USER:$OPENCLAW_USER" /var/log/openclaw

echo "[5/7] 配置系统服务..."

# 创建 systemd 服务
cat > /etc/systemd/system/openclaw.service << EOF
[Unit]
Description=OpenClaw Gateway Service
After=network.target

[Service]
Type=simple
User=${OPENCLAW_USER}
WorkingDirectory=/var/lib/openclaw
ExecStart=/usr/bin/openclaw gateway run --bind loopback --port 18789
Restart=always
RestartSec=10

# 环境变量（建议使用 env 文件管理）
EnvironmentFile=-/etc/openclaw/env

# 日志
StandardOutput=append:/var/log/openclaw/openclaw.log
StandardError=append:/var/log/openclaw/openclaw-error.log

# 资源限制
MemoryMax=1G
CPUQuota=80%

[Install]
WantedBy=multi-user.target
EOF

# 创建环境变量文件
cat > /etc/openclaw/env << EOF
CLAWDBOT_GATEWAY_TOKEN=$(openssl rand -base64 32)
ANTHROPIC_API_KEY=
EOF
chmod 600 /etc/openclaw/env
chown root:$OPENCLAW_USER /etc/openclaw/env

# 重新加载 systemd
systemctl daemon-reload

echo "[6/7] 配置防火墙..."

# 配置 UFW 防火墙
ufw allow OpenSSH
ufw allow 18789/tcp
ufw --force enable

echo "[7/7] 启动服务..."

# 启用并启动服务
systemctl enable openclaw
systemctl start openclaw

echo "=========================================="
echo "部署完成！"
echo "=========================================="
echo ""
echo "服务状态: systemctl status openclaw"
echo "查看日志: journalctl -u openclaw -f"
echo "健康检查: curl http://127.0.0.1:18789/health"
echo "=========================================="
```

### 防火墙配置

**UFW 配置**：

```bash
# 查看当前规则
ufw status verbose

# 允许 SSH（防止被锁）
ufw allow OpenSSH

# 仅允许本地网络访问 Gateway
ufw allow from 192.168.0.0/16 to any port 18789

# 或者允许特定 IP
ufw allow from 10.0.0.5 to any port 18789

# 启用防火墙
ufw enable

# 查看状态
ufw status
```

## 远程访问

### 远程访问架构设计

远程访问是生产环境中的常见需求。在设计远程访问方案时，需要平衡便利性和安全性。OpenClaw 的安全模型要求 Gateway 服务始终绑定到本地接口（loopback），远程访问通过额外的安全层实现。

**远程访问方案对比**：

| 方案 | 安全性 | 便利性 | 复杂度 | 推荐场景 |
|------|--------|--------|--------|----------|
| SSH 隧道 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 低 | 临时访问、技术用户 |
| Tailscale | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 中 | 长期使用、跨网络 |
| 反向代理 + HTTPS | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中 | 正式部署、需要分享 |
| VPN | ⭐⭐⭐⭐⭐ | ⭐⭐ | 高 | 企业环境、高安全要求 |

### SSH 隧道

**本地端口转发**：

```bash
# 基础用法：转发本地端口到服务器
ssh -L 18789:127.0.0.1:18789 user@server

# 保持连接活跃（防止超时）
ssh -o ServerAliveInterval=60 -o ServerAliveCountMax=3 \
    -L 18789:127.0.0.1:18789 user@server

# 后台运行
ssh -f -N -L 18789:127.0.0.1:18789 user@server

# 停止隧道
pkill -f "ssh.*-L 18789"
```

**创建 SSH 别名简化连接**：

```bash
# ~/.ssh/config
Host openclaw
    HostName your-server.com
    User username
    Port 22
    LocalForward 127.0.0.1:18789 127.0.0.1:18789
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ForwardAgent no
```

### Tailscale 部署

**Tailscale 集成配置**：

```json5
{
  "gateway": {
    "bind": "loopback",  // 始终保持本地绑定
    "tailscale": {
      "mode": "serve",  // serve: tailnet 内访问 | funnel: 公网访问
      "serveConfig": {
        "web": {
          "port": 18793
        }
      }
    }
  }
}
```

**Tailscale 部署步骤**：

1. 在服务器上安装 Tailscale：`curl -fsSL https://tailscale.com/install.sh | sh`
2. 登录并连接：`sudo tailscale up`
3. 在客户端设备上安装 Tailscale 并登录同一账户
4. 通过 `machine-name.tailnet.ts.net:18789` 访问 Gateway

### 反向代理配置

**Nginx 配置**：

```nginx
# /etc/nginx/sites-available/openclaw
server {
    listen 443 ssl http2;
    server_name openclaw.example.com;

    # SSL 配置
    ssl_certificate /etc/letsencrypt/live/openclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        # 代理到本地 Gateway
        proxy_pass http://127.0.0.1:18789;

        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;

        # 禁用缓冲（适用于流式响应）
        proxy_buffering off;
    }

    # 健康检查端点（无需认证）
    location /health {
        proxy_pass http://127.0.0.1:18789/health;
        proxy_set_header Host $host;
    }
}
```

**Caddy 配置**：

```
# Caddyfile
openclaw.example.com {
    reverse_proxy 127.0.0.1:18789 {
        transport http {
            versions h2c 1.1
        }
    }

    # 启用 WebSocket
    websocket {
        header_upstream X-Real-IP {remote}
    }

    # 健康检查
    route /health {
        respond * "OK" 200
    }
}
```

## 高可用部署

### 高可用的适用场景

高可用部署适用于对服务连续性有严格要求的场景。在考虑高可用方案之前，需要评估以下问题：

**是否真正需要高可用？**

| 问题 | 如果答案为是，考虑高可用 |
|------|------------------------|
| 服务中断是否会造成重大业务损失？ | 是 |
| 用户数量是否超过单机承载能力？ | 是 |
| 是否需要零停机维护？ | 是 |
| 预算和运维能力是否支持？ | 是 |

**注意**：对于个人用户或小型团队，单机部署配合自动重启策略通常已经足够。过早引入高可用架构会增加不必要的复杂度和成本。

### 主备模式架构

```
┌─────────────────────────────────────────────────────────────┐
│                      负载均衡 / VIP                          │
└─────────────────────────────┬───────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
    ┌─────────────────┐             ┌─────────────────┐
    │    主节点       │             │    备节点       │
    │   (Primary)     │  同步        │   (Standby)     │
    │   Active       │ ──────────→  │   Passive       │
    └────────┬────────┘             └────────┬────────┘
             │                               │
    ┌────────┴────────┐             ┌────────┴────────┐
    │  ┌────────────┐ │             │  ┌────────────┐ │
    │  │  Gateway   │ │             │  │  Gateway   │ │
    │  │  (Active)  │ │             │  │  (Idle)    │ │
    │  └────────────┘ │             │  └────────────┘ │
    └─────────────────┘             └─────────────────┘
             │                               │
    ┌────────┴────────────────────────────────┴────────┐
    │              共享存储 (NFS / S3 / EFS)            │
    │  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
    │  │ Config      │  │ Credentials │  │ Sessions  │ │
    │  └─────────────┘  └─────────────┘  └───────────┘ │
    └──────────────────────────────────────────────────┘
```

**主备模式配置要点**：

1. 共享存储：使用 NFS、EFS 或 S3 确保配置和凭证在节点间同步
2. 状态同步：实现会话状态的实时同步
3. 故障转移：通过 VIP 漂移或负载均衡器实现自动故障转移
4. 数据一致性：确保写入操作在主备间的一致性

### 多实例部署

对于不同渠道需要独立运行的场景，可以使用多实例部署：

```bash
#!/bin/bash
# multi-instance-deploy.sh - 多实例部署脚本

# WhatsApp 实例
CLAWDBOT_STATE_DIR=~/.clawdbot-wa \
CLAWDBOT_GATEWAY_TOKEN=wa_token_$(date +%s) \
openclaw gateway --port 19001 \
    --gateway.name "openclaw-whatsapp"

# Telegram 实例
CLAWDBOT_STATE_DIR=~/.clawdbot-tg \
CLAWDBOT_GATEWAY_TOKEN=tg_token_$(date +%s) \
openclaw gateway --port 19002 \
    --gateway.name "openclaw-telegram"

# Discord 实例
CLAWDBOT_STATE_DIR=~/.clawdbot-dc \
CLAWDBOT_GATEWAY_TOKEN=dc_token_$(date +%s) \
openclaw gateway --port 19003 \
    --gateway.name "openclaw-discord"
```

## 安全加固

### 部署安全检查清单

```
部署前安全检查：

1. [ ] 生成强随机网关令牌（32+ 字符）
2. [ ] 配置 API 密钥环境变量
3. [ ] 设置文件权限（配置文件 600，目录 700）
4. [ ] 配置防火墙规则（仅允许必要端口）
5. [ ] 启用 TLS（如果使用反向代理）
6. [ ] 配置日志脱敏
7. [ ] 设置访问控制（allowlist）
```

### 网关令牌配置

```json5
{
  "gateway": {
    "auth": {
      "token": "${CLAWDBOT_GATEWAY_TOKEN}",
      "mode": "token"  // token: 令牌认证 | password: 密码认证
    }
  }
}
```

**生成安全令牌**：

```bash
# 使用 openssl 生成
openssl rand -base64 32

# 使用 uuidgen（macOS）
uuidgen | tr -d '\n'

# 使用 pwgen（如果已安装）
pwgen -s 32 1
```

## 监控配置

### 健康检查集成

```bash
#!/bin/bash
# health-check.sh - 健康检查脚本

# 添加到监控系统（如 Prometheus）
curl -sf http://127.0.0.1:18789/health > /dev/null

if [ $? -eq 0 ]; then
    echo "OK"
    exit 0
else
    echo "CRITICAL"
    exit 1
fi
```

### 日志收集配置

```json5
{
  "logging": {
    "level": "info",
    "format": "json",              // JSON 格式便于日志收集系统解析
    "file": "/var/log/openclaw/openclaw.log",
    "maxSize": "100m",
    "maxFiles": 10,
    "redactSensitive": "all"       // 所有敏感信息脱敏
  }
}
```

## 部署验证

### 部署后验证清单

```bash
#!/bin/bash
# verify-deployment.sh - 部署验证脚本

echo "=========================================="
echo "OpenClaw 部署验证"
echo "=========================================="

# 1. 检查进程状态
echo "[1/6] 检查服务进程..."
if systemctl is-active --quiet openclaw; then
    echo "✓ 服务正在运行"
else
    echo "✗ 服务未运行"
    systemctl status openclaw
fi

# 2. 检查端口监听
echo "[2/6] 检查端口监听..."
if ss -ltnp | grep -q ":18789"; then
    echo "✓ 端口 18789 正在监听"
    ss -ltnp | grep 18789
else
    echo "✗ 端口 18789 未监听"
fi

# 3. 健康检查
echo "[3/6] 执行健康检查..."
if curl -sf http://127.0.0.1:18789/health > /dev/null; then
    echo "✓ 健康检查通过"
    curl -s http://127.0.0.1:18789/health | jq '.'
else
    echo "✗ 健康检查失败"
    curl -s http://127.0.0.1:18789/health
fi

# 4. 检查配置加载
echo "[4/6] 检查配置..."
if openclaw config get > /dev/null; then
    echo "✓ 配置加载成功"
else
    echo "✗ 配置加载失败"
fi

# 5. 检查渠道状态
echo "[5/6] 检查渠道状态..."
openclaw channels status --probe || echo "⚠ 部分渠道可能未配置"

# 6. 检查日志
echo "[6/6] 检查日志..."
if [ -f /var/log/openclaw/openclaw.log ]; then
    echo "✓ 日志文件存在"
    tail -5 /var/log/openclaw/openclaw.log
else
    echo "⚠ 日志文件未找到"
fi

echo "=========================================="
echo "验证完成"
echo "=========================================="
```

## 下一步

完成部署后，建议继续阅读以下文档：

| 文档 | 说明 |
|------|------|
| [监控与日志](/zh-CN/operations/monitoring) | 配置完整的监控和告警体系 |
| [故障排除](/zh-CN/operations/troubleshooting) | 常见问题诊断和解决方法 |
| [运维手册](/zh-CN/operations/index) | 运维最佳实践和日常管理 |
| [安全指南](/zh-CN/gateway/security) | 安全加固和合规要求 |
