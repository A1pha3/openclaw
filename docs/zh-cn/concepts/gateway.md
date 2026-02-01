# Gateway 网关

Gateway 是 OpenClaw 的核心服务，负责管理所有消息渠道和 AI 代理的通信。

## 概述

Gateway 是一个长期运行的进程，提供：

- WebSocket 服务器（实时通信）
- HTTP API（RPC 接口）
- 消息渠道管理
- 会话管理
- 工具执行环境

## 启动网关

### 基本启动

```bash
openclaw gateway
```

### 带参数启动

```bash
openclaw gateway --port 18789 --verbose
```

### 常用选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--port` | 监听端口 | 18789 |
| `--bind` | 绑定地址 | loopback |
| `--verbose` | 详细日志 | false |
| `--force` | 强制启动 | false |
| `--dev` | 开发模式 | false |

## 网络配置

### 绑定模式

```json5
{
  gateway: {
    bind: "loopback",  // loopback | tailnet | lan | <ip>
    port: 18789
  }
}
```

| 模式 | 说明 |
|------|------|
| `loopback` | 仅本地访问（127.0.0.1） |
| `tailnet` | Tailscale 网络 |
| `lan` | 局域网（0.0.0.0） |
| `<ip>` | 指定 IP 地址 |

### 端口配置

```json5
{
  gateway: {
    port: 18789,         // WebSocket 端口
    canvasHost: {
      port: 18793        // Canvas HTTP 端口
    }
  }
}
```

## 认证

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

生成令牌：

```bash
openssl rand -hex 32
```

### 使用令牌

CLI 连接时：

```bash
export CLAWDBOT_GATEWAY_TOKEN="your-token"
openclaw status
```

Control UI 连接时，在设置中输入令牌。

## 服务管理

### 后台服务

使用系统服务运行（推荐）：

```bash
# 安装服务
openclaw onboard --install-daemon

# 查看状态
openclaw service status

# 启动/停止/重启
openclaw service start
openclaw service stop
openclaw service restart
```

### 手动运行

```bash
# 前台运行
openclaw gateway --verbose

# 后台运行
nohup openclaw gateway > /tmp/openclaw.log 2>&1 &
```

## 状态检查

### 健康检查

```bash
openclaw health
```

### 状态查询

```bash
# 基本状态
openclaw status

# 深度检查（带探测）
openclaw status --deep

# 完整报告
openclaw status --all
```

### HTTP 健康端点

```bash
curl http://127.0.0.1:18789/health
```

## 配置管理

### 热重载

配置变更后自动重启：

```bash
openclaw config set channels.telegram.enabled true
# Gateway 自动重启
```

### 手动重启

```bash
openclaw gateway restart
```

### 通过 RPC 重启

```bash
openclaw gateway call gateway.restart --params '{}'
```

## RPC API

### 调用 RPC 方法

```bash
openclaw gateway call <method> --params '<json>'
```

### 常用方法

| 方法 | 说明 |
|------|------|
| `config.get` | 获取配置 |
| `config.apply` | 应用配置 |
| `config.patch` | 部分更新配置 |
| `channels.status` | 渠道状态 |
| `sessions.list` | 会话列表 |
| `gateway.restart` | 重启网关 |

### 示例

```bash
# 获取配置
openclaw gateway call config.get --params '{}'

# 获取渠道状态
openclaw gateway call channels.status --params '{"channel": "whatsapp"}'
```

## Canvas Host

Canvas Host 是 HTTP 文件服务器，用于节点 WebView：

```
http://<gateway>:18793/__openclaw__/canvas/
```

### 配置

```json5
{
  gateway: {
    canvasHost: {
      enabled: true,
      port: 18793
    }
  }
}
```

## 日志

### 日志位置

默认：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

### 配置日志

```json5
{
  logging: {
    level: "info",              // debug | info | warn | error
    file: "/var/log/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty"      // pretty | compact | json
  }
}
```

### 查看日志

```bash
openclaw logs --tail 100
```

## 远程访问

### SSH 隧道

```bash
ssh -L 18789:127.0.0.1:18789 user@gateway-host
```

### Tailscale

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
    listen 443 ssl;
    server_name openclaw.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## 多实例

### 运行多个网关

```bash
# 实例 A
CLAWDBOT_CONFIG_PATH=~/.clawdbot/a.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-a \
openclaw gateway --port 19001

# 实例 B
CLAWDBOT_CONFIG_PATH=~/.clawdbot/b.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-b \
openclaw gateway --port 19002
```

## 故障排除

### 无法启动

```bash
# 诊断
openclaw doctor

# 检查端口
lsof -i :18789

# 详细日志
openclaw gateway --verbose
```

### 连接问题

```bash
# 测试连接
curl http://127.0.0.1:18789/health

# 检查防火墙
```

### 服务崩溃

```bash
# 查看日志
openclaw logs --tail 200

# 检查系统日志
journalctl --user -u openclaw-gateway
```

## 最佳实践

1. **使用系统服务**: 比手动运行更可靠
2. **启用令牌认证**: 即使是本地访问
3. **定期健康检查**: 监控运行状态
4. **日志轮转**: 避免磁盘空间问题
5. **备份配置**: 定期备份 `~/.clawdbot/`

## 下一步

- [系统架构](/zh-cn/concepts/architecture) - 整体架构
- [配置参考](/zh-cn/config/reference) - 完整配置
- [远程访问](/zh-cn/operations/deployment) - 部署指南
