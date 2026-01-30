# 运维手册

本手册面向负责 Moltbot 生产环境部署和维护的运维工程师，提供全面的运维指南。

## 运维概述

Moltbot 是一个长期运行的服务，需要可靠的运维策略来确保高可用性和稳定性。

### 核心运维任务

| 任务 | 频率 | 优先级 |
|------|------|--------|
| 健康检查 | 持续 | 高 |
| 日志监控 | 每日 | 高 |
| 版本更新 | 按需 | 中 |
| 备份 | 每日 | 中 |
| 安全审计 | 每周 | 中 |
| 性能调优 | 按需 | 低 |

## 部署架构

### 单机部署（推荐大多数场景）

```
┌─────────────────────────────────────┐
│              服务器                  │
│  ┌─────────────────────────────┐   │
│  │     Moltbot Gateway         │   │
│  │     (systemd/launchd)       │   │
│  │     Port: 18789             │   │
│  └─────────────────────────────┘   │
│               │                     │
│  ┌────────────┴────────────┐       │
│  │      状态目录            │       │
│  │   ~/.clawdbot/          │       │
│  └─────────────────────────┘       │
└─────────────────────────────────────┘
```

### Docker 部署

```yaml
# docker-compose.yml
version: '3.8'
services:
  moltbot:
    image: ghcr.io/moltbot/moltbot:latest
    ports:
      - "18789:18789"
    volumes:
      - moltbot-data:/root/.clawdbot
    restart: unless-stopped
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - CLAWDBOT_GATEWAY_TOKEN=${GATEWAY_TOKEN}
    healthcheck:
      test: ["CMD", "moltbot", "health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  moltbot-data:
```

### 云平台部署

支持的平台：
- **Railway** - 一键部署
- **Render** - 容器服务
- **Northflank** - Kubernetes 托管
- **Fly.io** - 边缘部署
- **Hetzner** - VPS 部署

## 服务管理

### systemd (Linux)

```bash
# 查看状态
systemctl --user status moltbot-gateway

# 启动服务
systemctl --user start moltbot-gateway

# 停止服务
systemctl --user stop moltbot-gateway

# 重启服务
systemctl --user restart moltbot-gateway

# 查看日志
journalctl --user -u moltbot-gateway -f

# 开机自启
loginctl enable-linger $USER
```

### launchd (macOS)

```bash
# 查看状态
moltbot service status

# 启动服务
moltbot service start

# 停止服务
moltbot service stop

# 重启服务
moltbot service restart

# 查看日志
tail -f /tmp/moltbot/moltbot-$(date +%Y-%m-%d).log
```

## 监控

### 健康检查

```bash
# 基础健康检查
moltbot health

# 深度健康检查
moltbot status --deep

# 只读状态报告（适合粘贴分享）
moltbot status --all
```

### 健康检查端点

Gateway 提供 HTTP 健康检查：

```bash
curl http://127.0.0.1:18789/health
```

### 关键指标

| 指标 | 正常范围 | 告警阈值 |
|------|----------|----------|
| 内存使用 | < 500MB | > 1GB |
| CPU 使用 | < 30% | > 80% |
| 连接数 | 根据渠道 | 无响应 |
| 消息延迟 | < 5s | > 30s |

### 日志监控

```bash
# 实时日志
moltbot logs --tail 100

# 查看特定日期日志
cat /tmp/moltbot/moltbot-2026-01-30.log

# 搜索错误
grep -i error /tmp/moltbot/moltbot-*.log
```

### 告警配置

结合外部监控工具（如 Prometheus + Alertmanager）：

```yaml
# alertmanager 示例规则
groups:
  - name: moltbot
    rules:
      - alert: MoltbotDown
        expr: up{job="moltbot"} == 0
        for: 5m
        labels:
          severity: critical
```

## 日志管理

### 日志位置

| 类型 | 路径 |
|------|------|
| 应用日志 | `/tmp/moltbot/moltbot-YYYY-MM-DD.log` |
| 系统日志 | `journalctl` / `log stream` |
| 会话日志 | `~/.clawdbot/agents/<id>/sessions/` |

### 日志配置

```json5
{
  logging: {
    level: "info",           // debug, info, warn, error
    file: "/var/log/moltbot/moltbot.log",
    consoleLevel: "warn",
    consoleStyle: "compact", // pretty, compact, json
    redactSensitive: "tools" // off, tools
  }
}
```

### 日志轮转

使用 logrotate (Linux)：

```
# /etc/logrotate.d/moltbot
/tmp/moltbot/moltbot-*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
```

## 备份与恢复

### 备份内容

| 数据 | 路径 | 重要性 |
|------|------|--------|
| 配置文件 | `~/.clawdbot/moltbot.json` | 高 |
| WhatsApp 会话 | `~/.clawdbot/credentials/whatsapp/` | 高 |
| OAuth 凭证 | `~/.clawdbot/credentials/oauth.json` | 高 |
| 代理数据 | `~/.clawdbot/agents/` | 中 |
| 会话历史 | `~/.clawdbot/sessions/` | 低 |

### 备份脚本

```bash
#!/bin/bash
# backup-moltbot.sh

BACKUP_DIR="/backup/moltbot/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# 备份关键数据
tar -czf "$BACKUP_DIR/clawdbot.tar.gz" \
  ~/.clawdbot/moltbot.json \
  ~/.clawdbot/credentials/ \
  ~/.clawdbot/agents/

# 保留最近 7 天
find /backup/moltbot -maxdepth 1 -mtime +7 -type d -exec rm -rf {} \;
```

### 恢复流程

1. 停止 Gateway
2. 恢复备份文件
3. 运行 `moltbot doctor --fix`
4. 启动 Gateway
5. 验证状态

```bash
moltbot service stop
tar -xzf /backup/moltbot/20260130/clawdbot.tar.gz -C ~/
moltbot doctor --fix
moltbot service start
moltbot health
```

## 版本更新

### 更新前检查

```bash
# 查看当前版本
moltbot --version

# 检查可用更新
npm view moltbot version

# 备份配置
cp ~/.clawdbot/moltbot.json ~/.clawdbot/moltbot.json.bak
```

### 执行更新

```bash
# 更新到最新版
moltbot update

# 或手动更新
npm install -g moltbot@latest

# 更新后验证
moltbot doctor
moltbot health
```

### 回滚

```bash
# 回滚到指定版本
npm install -g moltbot@2026.1.15

# 恢复配置（如需要）
cp ~/.clawdbot/moltbot.json.bak ~/.clawdbot/moltbot.json
```

## 安全加固

### 网关令牌

```json5
{
  gateway: {
    auth: {
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

### 网络限制

```json5
{
  gateway: {
    bind: "loopback",  // 仅本地访问
    port: 18789
  }
}
```

### 安全审计

```bash
# 深度安全审计
moltbot security audit --deep

# 检查文件权限
moltbot doctor
```

### 定期检查清单

- [ ] 更新到最新安全版本
- [ ] 检查 API 密钥有效性
- [ ] 审查 allowlist 配置
- [ ] 检查文件权限
- [ ] 审查访问日志

## 故障排除

### 常见问题

#### Gateway 无法启动

```bash
# 检查配置
moltbot doctor

# 检查端口占用
lsof -i :18789

# 查看详细日志
moltbot gateway --verbose
```

#### WhatsApp 断开连接

```bash
# 检查状态
moltbot channels status whatsapp

# 重新登录
moltbot channels login
```

#### 内存泄漏

```bash
# 检查内存使用
moltbot status

# 重启服务
moltbot service restart
```

### 诊断命令

```bash
# 综合诊断
moltbot doctor

# 健康检查
moltbot health

# 状态报告
moltbot status --all

# 查看日志
moltbot logs --tail 200
```

## 性能调优

### 消息队列

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20
    }
  }
}
```

### 沙箱资源

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          cpuLimit: "2",
          memoryLimit: "2g"
        }
      }
    }
  }
}
```

### 日志优化

```json5
{
  logging: {
    level: "warn",        // 生产环境使用 warn 或 error
    consoleLevel: "error"
  }
}
```

## 下一步

- [部署指南](/zh-cn/operations/deployment) - 详细部署步骤
- [监控与日志](/zh-cn/operations/monitoring) - 监控配置
- [故障排除](/zh-cn/operations/troubleshooting) - 问题诊断
- [备份恢复](/zh-cn/operations/backup) - 数据保护
- [安全加固](/zh-cn/operations/security-hardening) - 安全最佳实践
