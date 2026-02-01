# 监控指南

本文档介绍如何监控 OpenClaw 网关和相关服务的运行状态。

## 健康检查

### 基本状态检查

```bash
# 查看网关状态
openclaw gateway status

# 查看所有渠道状态
openclaw channels status

# 带探测的深度检查
openclaw channels status --probe
```

### 诊断工具

```bash
# 运行完整诊断
openclaw doctor

# 检查特定问题
openclaw doctor --check config
openclaw doctor --check channels
openclaw doctor --check permissions
```

## 日志

### 查看日志

```bash
# 实时查看日志
openclaw logs --tail 100

# 查看特定级别
openclaw logs --level error

# 查看特定渠道
openclaw logs --channel telegram
```

### macOS 统一日志

使用项目提供的日志查询脚本：

```bash
# 查看 OpenClaw 子系统日志
./scripts/clawlog.sh

# 跟踪模式
./scripts/clawlog.sh --follow

# 按类别过滤
./scripts/clawlog.sh --category gateway
```

### 日志配置

```json5
{
  "logging": {
    "level": "info",           // debug | info | warn | error
    "format": "pretty",        // pretty | json
    "file": "~/.clawdbot/logs/openclaw.log",
    "maxSize": "10m",          // 单个文件最大大小
    "maxFiles": 5              // 保留的日志文件数量
  }
}
```

### 日志级别说明

| 级别 | 用途 |
|------|------|
| `debug` | 详细调试信息（开发用） |
| `info` | 正常操作信息 |
| `warn` | 警告（可能的问题） |
| `error` | 错误（需要关注） |

## 进程管理

### 检查进程状态

```bash
# 检查网关进程
ps aux | grep openclaw

# 检查端口占用
ss -ltnp | grep 18789
lsof -i :18789

# macOS launchd 状态
launchctl print gui/$UID | grep openclaw
```

### 重启服务

```bash
# 重启网关（推荐通过 macOS 应用）
./scripts/restart-mac.sh

# 强制重启（Linux/远程）
pkill -9 -f openclaw-gateway || true
nohup openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &
```

## 渠道监控

### 渠道状态

```bash
# 所有渠道概览
openclaw channels status

# 特定渠道详情
openclaw channels status whatsapp
openclaw channels status telegram --verbose
```

### 渠道健康指标

| 指标 | 说明 | 正常值 |
|------|------|--------|
| 连接状态 | 渠道是否连接 | connected |
| 最后活动 | 上次消息时间 | < 5 分钟 |
| 错误计数 | 累计错误数 | 0 或低 |
| 重连次数 | 重连尝试次数 | 0 或低 |

### 配对请求监控

```bash
# 查看待处理的配对请求
openclaw pairing list

# 按渠道查看
openclaw pairing list telegram
openclaw pairing list whatsapp
```

## 性能监控

### 资源使用

```bash
# 内存和 CPU 使用
top -p $(pgrep -f openclaw)

# 更详细的资源信息
htop -p $(pgrep -f openclaw)
```

### 会话统计

```bash
# 查看活跃会话
openclaw sessions list

# 会话详情
openclaw sessions info <session-id>
```

## 告警设置

### 使用 Cron 作业监控

配置定时检查并发送告警：

```json5
{
  "cron": {
    "jobs": [
      {
        "id": "health-check",
        "schedule": "*/5 * * * *",        // 每 5 分钟
        "agent": "main",
        "message": "检查网关状态，如果有问题请告诉我",
        "deliverTo": "telegram:admin_id"
      }
    ]
  }
}
```

### 外部监控集成

#### Uptime Kuma

配置 HTTP 探针检查网关健康端点：

```
URL: http://localhost:18789/health
间隔: 60 秒
超时: 10 秒
```

#### Prometheus (自定义)

可以编写自定义导出器收集指标：

```typescript
// 示例：自定义指标收集
const metrics = {
  gateway_uptime_seconds: process.uptime(),
  channels_connected: getConnectedChannelsCount(),
  messages_processed_total: getMessageCount(),
  active_sessions: getActiveSessionCount()
}
```

## 远程监控

### SSH 检查

```bash
# 远程检查网关状态
ssh gateway-host "openclaw channels status --probe"

# 远程查看日志
ssh gateway-host "tail -n 100 /tmp/openclaw-gateway.log"
```

### Tailscale 访问

通过 Tailscale 远程访问控制 UI：

```json5
{
  "gateway": {
    "tailscale": {
      "mode": "serve"      // tailnet 内访问
    }
  }
}
```

然后通过 `https://your-machine.tailnet.ts.net:18789` 访问控制 UI。

## 常见问题排查

### 网关无法启动

1. 检查端口占用：`lsof -i :18789`
2. 检查配置文件语法
3. 查看启动日志
4. 运行 `openclaw doctor`

### 渠道断开连接

1. 检查网络连接
2. 验证凭证/令牌
3. 查看渠道特定日志
4. 尝试重新认证

### 消息延迟

1. 检查系统资源使用
2. 查看队列状态
3. 检查外部 API 响应时间
4. 考虑扩展资源

### 内存使用过高

1. 检查活跃会话数
2. 查看历史记录配置
3. 考虑调整 `historyLimit`
4. 重启服务释放内存

## 生产环境建议

### 日志管理

```json5
{
  "logging": {
    "level": "info",
    "format": "json",              // 便于日志聚合
    "file": "/var/log/openclaw/openclaw.log",
    "maxSize": "100m",
    "maxFiles": 10
  }
}
```

### 进程管理

使用 systemd 管理服务（Linux）：

```ini
# /etc/systemd/user/openclaw.service
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/openclaw gateway run --bind loopback --port 18789
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

```bash
# 启用服务
systemctl --user enable openclaw
systemctl --user start openclaw

# 查看状态
systemctl --user status openclaw
```

### 备份

定期备份配置和数据：

```bash
# 备份配置
cp -r ~/.clawdbot/openclaw.json ~/backups/

# 备份凭证
cp -r ~/.clawdbot/credentials/ ~/backups/

# 备份会话数据
cp -r ~/.clawdbot/sessions/ ~/backups/
```

## 监控清单

### 每日检查

- [ ] 网关进程运行
- [ ] 所有渠道已连接
- [ ] 无严重错误日志
- [ ] 待处理配对请求

### 每周检查

- [ ] 资源使用趋势
- [ ] 错误日志统计
- [ ] 备份完整性
- [ ] 依赖更新

### 每月检查

- [ ] 性能基准对比
- [ ] 配置审计
- [ ] 安全更新
- [ ] 日志存档清理

## 相关文档

- [网关配置](/zh-CN/concepts/gateway)
- [故障排除](/zh-CN/operations/troubleshooting)
- [部署指南](/zh-CN/operations/deployment)
- [安全指南](/zh-CN/concepts/architecture#安全模型)
