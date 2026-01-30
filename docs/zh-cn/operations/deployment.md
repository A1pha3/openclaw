# 部署指南

本文档介绍如何在各种环境中部署 Moltbot。

## 部署模式

### 个人/开发部署

适用于个人使用或开发测试：

- 单机运行
- 本地网络访问
- 手动管理

### 生产部署

适用于长期稳定运行：

- 系统服务托管
- 远程访问配置
- 监控和备份

## 本地部署

### 系统服务安装

**推荐方式**：使用向导安装服务

```bash
moltbot onboard --install-daemon
```

这会根据您的系统自动安装：
- **macOS**: launchd 服务
- **Linux**: systemd 用户服务

### 手动运行

前台运行（用于调试）：

```bash
moltbot gateway --port 18789 --verbose
```

后台运行：

```bash
nohup moltbot gateway --port 18789 > /tmp/moltbot.log 2>&1 &
```

## Docker 部署

### 快速启动

```bash
docker run -d \
  --name moltbot \
  -p 18789:18789 \
  -v ~/.clawdbot:/root/.clawdbot \
  ghcr.io/moltbot/moltbot:latest \
  gateway --port 18789
```

### docker-compose

```yaml
version: '3.8'

services:
  moltbot:
    image: ghcr.io/moltbot/moltbot:latest
    container_name: moltbot
    ports:
      - "18789:18789"
      - "18793:18793"
    volumes:
      - moltbot-data:/root/.clawdbot
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - CLAWDBOT_GATEWAY_TOKEN=${GATEWAY_TOKEN}
    command: gateway --port 18789 --bind 0.0.0.0
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "moltbot", "health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  moltbot-data:
```

启动：

```bash
docker-compose up -d
```

### 构建自定义镜像

```dockerfile
FROM ghcr.io/moltbot/moltbot:latest

# 添加自定义配置
COPY moltbot.json /root/.clawdbot/moltbot.json

# 安装插件
RUN moltbot plugins install @moltbot/mattermost
```

## 云平台部署

### Railway

1. 创建新项目
2. 连接 GitHub 仓库或使用模板
3. 设置环境变量
4. 部署

```yaml
# railway.yaml
services:
  moltbot:
    image: ghcr.io/moltbot/moltbot:latest
    command: gateway --port $PORT --bind 0.0.0.0
    healthcheck: moltbot health
```

### Render

1. 创建 Web Service
2. 选择 Docker 部署
3. 配置环境变量
4. 设置持久化存储

```yaml
# render.yaml
services:
  - type: web
    name: moltbot
    env: docker
    dockerImage: ghcr.io/moltbot/moltbot:latest
    dockerCommand: gateway --port $PORT --bind 0.0.0.0
    healthCheckPath: /health
    disk:
      name: moltbot-data
      mountPath: /root/.clawdbot
      sizeGB: 10
```

### Fly.io

```bash
# 初始化
fly launch

# 创建持久化存储
fly volumes create moltbot_data --size 10

# 部署
fly deploy
```

`fly.toml`:

```toml
app = "moltbot"

[build]
  image = "ghcr.io/moltbot/moltbot:latest"

[[services]]
  internal_port = 18789
  protocol = "tcp"
  
  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]
  
  [[services.http_checks]]
    path = "/health"
    interval = 30000
    timeout = 10000

[mounts]
  source = "moltbot_data"
  destination = "/root/.clawdbot"
```

## VPS 部署

### Ubuntu/Debian

```bash
# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# 安装 Moltbot
npm install -g moltbot@latest

# 配置
moltbot onboard --install-daemon

# 启动服务
systemctl --user enable --now moltbot-gateway
```

### 配置防火墙

```bash
# UFW
sudo ufw allow 18789/tcp

# 或仅允许特定 IP
sudo ufw allow from 192.168.1.0/24 to any port 18789
```

## 远程访问

### SSH 隧道

最简单的远程访问方式：

```bash
# 本地机器
ssh -L 18789:127.0.0.1:18789 user@server
```

然后访问 `http://127.0.0.1:18789/`

### Tailscale

使用 Tailscale 创建安全的私有网络：

1. 在服务器和客户端安装 Tailscale
2. 配置网关绑定到 Tailnet

```json5
{
  gateway: {
    bind: "tailnet"
  }
}
```

### 反向代理

使用 nginx：

```nginx
server {
    listen 443 ssl http2;
    server_name moltbot.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 3600s;
    }
}
```

使用 Caddy：

```
moltbot.example.com {
    reverse_proxy 127.0.0.1:18789
}
```

## 高可用部署

### 主备模式

```
┌─────────────────────────────────────┐
│           负载均衡/VIP              │
└─────────────────┬───────────────────┘
         ┌────────┴────────┐
         ↓                 ↓
   ┌──────────┐      ┌──────────┐
   │ Primary  │      │ Standby  │
   │ Active   │      │ Passive  │
   └──────────┘      └──────────┘
         │                 │
   ┌─────┴─────────────────┴─────┐
   │       共享存储 (NFS/S3)      │
   └─────────────────────────────┘
```

### 多实例

不同渠道/用途使用独立实例：

```bash
# WhatsApp 实例
CLAWDBOT_STATE_DIR=~/.clawdbot-wa \
moltbot gateway --port 19001

# Telegram 实例
CLAWDBOT_STATE_DIR=~/.clawdbot-tg \
moltbot gateway --port 19002
```

## 安全建议

### 网关令牌

始终配置网关令牌：

```json5
{
  gateway: {
    auth: {
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

### 网络隔离

生产环境推荐：

1. 使用私有网络
2. 配置防火墙
3. 使用 VPN 或 Tailscale
4. 启用 HTTPS

### 定期更新

```bash
# 定期检查更新
moltbot update

# 自动更新（需要自行配置 cron）
0 2 * * * moltbot update && moltbot service restart
```

## 监控配置

### 健康检查

```bash
# 添加到监控系统
curl -f http://127.0.0.1:18789/health || alert "Moltbot is down"
```

### 日志收集

配置日志输出：

```json5
{
  logging: {
    level: "info",
    file: "/var/log/moltbot/moltbot.log",
    consoleStyle: "json"  // 便于日志收集
  }
}
```

## 下一步

- [监控与日志](/zh-cn/operations/monitoring)
- [故障排除](/zh-cn/operations/troubleshooting)
- [备份恢复](/zh-cn/operations/backup)
