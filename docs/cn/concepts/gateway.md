---
read_when:
  - 配置网关时
  - 启动网关时
  - 使用 RPC API 时
summary: "Gateway 网关完整指南：架构、启动、网络配置、认证、服务管理、RPC API、故障排查和最佳实践"
title: "Gateway 网关"
---

# Gateway 网关

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 Gateway 在 OpenClaw 架构中的核心作用
- [ ] 掌握 Gateway 的启动、配置和管理
- [ ] 了解网络配置和认证机制
- [ ] 能够诊断和解决 Gateway 问题

### 进阶目标（建议掌握）

- [ ] 掌握 RPC API 的使用方法
- [ ] 配置远程访问和多实例部署
- [ ] 优化 Gateway 性能和日志管理
- [ ] 实现高可用部署

---

## 💡 为什么需要 Gateway？

### 类比：Gateway 是 AI 代理的"中央指挥部"

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Gateway 中央指挥部                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  没有 Gateway 的混乱场景：                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  WhatsApp ─┐                                                      │   │
│  │  Telegram ─┼──► 每个渠道独立运行，无法协调                       │   │
│  │  Discord ──┘                                                      │   │
│  │                                                                 │   │
│  │  问题：                                                          │   │
│  │  • 上下文无法共享（同用户多渠道重复）                            │   │
│  │  • 重复管理多个进程                                              │   │
│  │  • 资源浪费（每个都加载模型）                                    │   │
│  │  • 配置分散（难以统一管理）                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  有 Gateway 的有序场景：                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        ┌───────────────┐                          │   │
│  │  WhatsApp ──────────────►               │                          │   │
│  │  Telegram ──────────────►   Gateway     ├──► 统一的 AI 代理       │   │
│  │  Discord ───────────────►   中央指挥部   │     (上下文共享)        │   │
│  │  iMessage ───────────────►               │                          │   │
│  │                        └───────────────┘                          │   │
│  │                                                                 │   │
│  │  优势：                                                          │   │
│  │  ✅ 统一控制平面 — 一个进程管理所有渠道                          │   │
│  │  ✅ 上下文连续性 — 用户在同会话中跨渠道                          │   │
│  │  ✅ 高效资源利用 — 共享模型和工具执行                            │   │
│  │  ✅ 简化管理 — 单一配置点和 API                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 核心价值

| 价值 | 说明 |
|------|------|
| **统一消息处理** | 所有渠道消息进入单一管道 |
| **会话连续性** | 用户跨渠道保持同一会话 |
| **资源共享** | 工具执行和模型调用统一管理 |
| **API 访问** | WebSocket 和 HTTP 接口统一 |

---

## 🏗️ Gateway 架构

### 架构概览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Gateway 架构详解                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         渠道层（Channels）                          │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │   │
│  │  │ WhatsApp │ │ Telegram │ │ Discord  │ │ iMessage │ ...          │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘              │   │
│  └───────┼────────────┼────────────┼────────────┼──────────────────────┘   │
│          │            │            │            │                          │
│          ▼            ▼            ▼            ▼                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      消息路由层（Routing）                          │   │
│  │  • 会话识别    • 用户匹配    • 权限检查    • 命令解析              │   │
│  └────────────────────────────────────┬────────────────────────────────┘   │
│                                       │                                    │
│                                       ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      智能体运行时（Agent Runtime）                    │   │
│  │  ┌──────────────────────────────────────────────────────────────┐  │   │
│  │  │  • 会话管理    • 上下文组装    • 模型调用    • 工具执行     │  │   │
│  │  └──────────────────────────────────────────────────────────────┘  │   │
│  └────────────────────────────────────┬────────────────────────────────┘   │
│                                       │                                    │
│                                       ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      API 层（WebSocket + HTTP）                      │   │
│  │  • 控制面板     • RPC 调用    • 事件流     • Canvas Host           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 职责 | 技术实现 |
|------|------|----------|
| **WebSocket 服务器** | 实时双向通信 | ws 协议，端口 18789 |
| **HTTP API** | RPC 调用接口 | RESTful 风格 |
| **会话管理器** | 上下文和历史 | JSONL 持久化 |
| **工具执行器** | 文件、命令等操作 | 沙箱隔离 |
| **渠道连接器** | 第三方平台集成 | 插件化架构 |

---

## 🚀 概述

Gateway 是一个长期运行的进程，作为 OpenClaw 的核心服务，负责：

- **WebSocket 服务器**（实时通信）
- **HTTP API**（RPC 接口）
- **消息渠道管理**（所有入站/出站消息）
- **会话管理**（上下文和历史）
- **工具执行环境**（文件操作、浏览器、命令等）

---

## ▶️ 启动网关

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

### 启动流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Gateway 启动流程                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 加载配置                                                             │
│     │                                                                   │
│     ▼                                                                   │
│  2. 初始化渠道连接器                                                     │
│     │                                                                   │
│     ▼                                                                   │
│  3. 启动 WebSocket 服务器（端口 18789）                                 │
│     │                                                                   │
│     ▼                                                                   │
│  4. 启动 HTTP API（同端口）                                             │
│     │                                                                   │
│     ▼                                                                   │
│  5. 启动 Canvas Host（端口 18793）                                      │
│     │                                                                   │
│     ▼                                                                   │
│  6. 加载会话历史                                                         │
│     │                                                                   │
│     ▼                                                                   │
│  7. 准备就绪 ✓                                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🌐 网络配置

### 绑定模式

```json5
{
  gateway: {
    bind: "loopback",  // loopback | tailnet | lan | <ip>
    port: 18789
  }
}
```

| 模式 | 绑定地址 | 使用场景 |
|------|----------|----------|
| `loopback` | 127.0.0.1 | 仅本地访问，最安全 |
| `tailnet` | Tailscale IP | 通过 Tailscale 远程访问 |
| `lan` | 0.0.0.0 | 局域网内设备访问 |
| `<ip>` | 指定 IP | 绑定到特定网络接口 |

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

### 端口占用检查

```bash
# 检查端口是否被占用
lsof -i :18789

# 查找占用进程
netstat -tulnp | grep 18789

# 强制释放端口
pkill -9 -f openclaw-gateway
```

---

## 🔐 认证

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

### 生成令牌

```bash
# 方法 1：使用 OpenSSL
openssl rand -hex 32

# 方法 2：使用 Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# 方法 3：使用 Python
python3 -c "import secrets; print(secrets.token_hex(32))"
```

### 使用令牌

**CLI 连接时**：

```bash
export CLAWDBOT_GATEWAY_TOKEN="your-token"
openclaw status
```

**Control UI 连接时**：在设置中输入令牌。

### 令牌存储位置

| 方式 | 位置 | 安全性 |
|------|------|--------|
| 环境变量 | `.env` 或 shell 配置 | ⭐⭐⭐ |
| 配置文件 | `openclaw.json` | ⭐⭐ |
| 密钥管理 | 1Password / 系统密钥链 | ⭐⭐⭐⭐⭐ |

---

## 🛠️ 服务管理

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
# 前台运行（调试用）
openclaw gateway --verbose

# 后台运行
nohup openclaw gateway > /tmp/openclaw.log 2>&1 &

# 使用 tmux/screen
tmux new -s openclaw
openclaw gateway --verbose
# Ctrl+B, D 分离
```

### macOS 启动

```bash
# 通过菜单栏应用
# OpenClaw → Start Gateway

# 或通过命令
open -a OpenClaw
```

---

## 📊 状态检查

### 健康检查

```bash
openclaw health
```

输出示例：
```
✓ Gateway is running (PID: 12345)
✓ WebSocket listening on 18789
✓ 3 channels connected
✓ 12 active sessions
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
# 基本健康检查
curl http://127.0.0.1:18789/health

# 返回示例
{"status":"healthy","uptime":1234567}
```

### 状态报告解读

| 字段 | 说明 | 正常值 |
|------|------|--------|
| `status` | 整体状态 | `healthy` |
| `uptime` | 运行时长（秒） | > 0 |
| `channels` | 已连接渠道数 | ≥ 0 |
| `sessions` | 活跃会话数 | ≥ 0 |

---

## ⚙️ 配置管理

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

### 配置验证

```bash
# 验证当前配置
openclaw config validate

# 显示当前配置
openclaw config get
```

---

## 📡 RPC API

### 调用 RPC 方法

```bash
openclaw gateway call <method> --params '<json>'
```

### 常用方法

| 方法 | 说明 | 参数 |
|------|------|------|
| `config.get` | 获取配置 | `{}` |
| `config.apply` | 应用配置 | `{config: {...}}` |
| `config.patch` | 部分更新配置 | `{path: "...", value: ...}` |
| `channels.status` | 渠道状态 | `{channel: "whatsapp"}` |
| `sessions.list` | 会话列表 | `{active: 60}` |
| `gateway.restart` | 重启网关 | `{}` |

### 使用示例

```bash
# 获取完整配置
openclaw gateway call config.get --params '{}'

# 获取特定渠道状态
openclaw gateway call channels.status --params '{"channel": "whatsapp"}'

# 列出活跃会话
openclaw gateway call sessions.list --params '{"active": 60}'

# 更新单个配置项
openclaw gateway call config.patch --params '{"path": "gateway.verbose", "value": true}'
```

### RPC 返回格式

```json
{
  "success": true,
  "data": {
    // 返回数据
  },
  "error": null
}
```

---

## 🖼️ Canvas Host

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

### Canvas 目录结构

```
<workspace>/canvas/
├── index.html       # 主页面
├── styles.css       # 样式文件
└── script.js        # 脚本文件
```

---

## 📝 日志

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
# 实时跟踪日志
openclaw logs --tail 100

# 持续跟踪（类似 tail -f）
openclaw logs --follow

# 指定日期
openclaw logs --date 2026-03-05
```

### 日志级别

| 级别 | 用途 | 示例内容 |
|------|------|----------|
| `debug` | 详细调试信息 | 函数调用、变量值 |
| `info` | 一般信息 | 启动、连接、断开 |
| `warn` | 警告信息 | 重试、降级 |
| `error` | 错误信息 | 失败、异常 |

---

## 🌍 远程访问

### SSH 隧道

```bash
# 本地端口转发
ssh -L 18789:127.0.0.1:18789 user@gateway-host

# 后台运行
ssh -f -N -L 18789:127.0.0.1:18789 user@gateway-host
```

### Tailscale

```json5
{
  gateway: {
    bind: "tailnet"
  }
}
```

### 反向代理（nginx）

```nginx
server {
    listen 443 ssl;
    server_name openclaw.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

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
}
```

### Cloudflare 隧道

```bash
# 安装 cloudflared
brew install cloudflared

# 创建隧道
cloudflared tunnel --url http://127.0.0.1:18789
```

---

## 🔁 多实例

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

### 多实例场景

| 场景 | 配置 | 注意事项 |
|------|------|----------|
| 测试环境 | 独立配置目录 | 端口不冲突 |
| 多用户 | 每用户一个实例 | 资源隔离 |
| 负载均衡 | 前置代理层 | 会话亲和性 |

---

## 🔧 故障排除

### 无法启动

```bash
# 1. 诊断
openclaw doctor

# 2. 检查端口占用
lsof -i :18789

# 3. 检查配置
openclaw config validate

# 4. 详细日志
openclaw gateway --verbose
```

### 常见启动问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `EADDRINUSE` | 端口被占用 | `pkill -9 -f openclaw-gateway` 或更换端口 |
| `EACCES` | 权限不足 | 检查文件/目录权限 |
| `Config error` | 配置无效 | 运行 `openclaw config validate` |
| `Channel connection failed` | 渠道凭证问题 | 重新登录渠道 |

### 连接问题

```bash
# 测试 WebSocket 连接
wscat -c ws://127.0.0.1:18789

# 测试 HTTP 端点
curl http://127.0.0.1:18789/health

# 检查防火墙
sudo ufw status  # Linux
# macOS 系统防火墙在"系统偏好设置"中
```

### 服务崩溃

```bash
# 查看日志
openclaw logs --tail 200

# 检查系统日志
journalctl --user -u openclaw-gateway

# macOS 统一日志
./scripts/clawlog.sh --tail 100
```

### 性能问题

```bash
# 检查资源使用
top | grep openclaw

# 查看会话数量
openclaw sessions list --active 60 | wc -l

# 检查消息队列（如适用）
```

### 常见错误代码

| 错误 | 含义 | 解决方案 |
|------|------|----------|
| `AUTH_FAILED` | 认证失败 | 检查令牌配置 |
| `CHANNEL_NOT_CONNECTED` | 渠道未连接 | 检查渠道状态 |
| `SESSION_NOT_FOUND` | 会话不存在 | 验证会话 ID |
| `RATE_LIMITED` | 速率限制 | 减慢请求频率 |

---

## 🎯 最佳实践

### 部署建议

| 实践 | 说明 | 优先级 |
|------|------|--------|
| **使用系统服务** | 比手动运行更可靠，自动重启 | ⭐⭐⭐⭐⭐ |
| **启用令牌认证** | 即使是本地访问也要保护 | ⭐⭐⭐⭐⭐ |
| **定期健康检查** | 监控运行状态 | ⭐⭐⭐⭐ |
| **日志轮转** | 避免磁盘空间问题 | ⭐⭐⭐⭐ |
| **备份配置** | 定期备份 `~/.clawdbot/` | ⭐⭐⭐ |

### 安全建议

```bash
# 1. 使用强令牌（32 字节随机）
openssl rand -hex 32

# 2. 限制绑定地址
openclaw config set gateway.bind loopback

# 3. 启用 HTTPS（远程访问时）
# 使用反向代理 + SSL 证书

# 4. 定期更新
npm i -g openclaw@latest
```

### 性能优化

| 优化项 | 配置 | 效果 |
|--------|------|------|
| 压缩日志 | 设置合理的日志级别 | 减少 I/O |
| 会话清理 | 配置自动归档 | 控制内存 |
| 渠道限流 | 配置消息速率 | 防止过载 |

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [系统架构](/concepts/architecture) | 整体架构 |
| [配置参考](/config/reference) | 完整配置 |
| [远程访问](/operations/deployment) | 部署指南 |
| [故障排查](/operations/troubleshooting) | 详细诊断 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 启动和管理 Gateway | ⭐⭐⭐⭐⭐ |
| 配置网络和认证 | ⭐⭐⭐⭐ |
| 使用 RPC API | ⭐⭐⭐ |
| 故障排查 | ⭐⭐⭐ |

---

> **💡 专家提示**：首次配置时建议在本地测试（`bind: loopback`），确认一切正常后再考虑远程访问配置！
