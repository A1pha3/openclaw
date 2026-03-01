---
summary: 介绍如何监控 OpenClaw 网关和相关服务的运行状态，涵盖健康检查、日志、进程管理、渠道监控、性能监控、告警设置和远程监控
read_when:
  - 需要监控服务状态时
  - 配置监控和告警时
  - 排查性能问题时
title: 监控指南
---

# 监控指南

本指南详细介绍如何建立完善的 OpenClaw 监控体系。有效的监控体系是保障服务稳定运行的关键基础设施，能够帮助运维人员及时发现异常、快速定位问题、优化系统性能。在开始配置监控之前，建议您先理解 OpenClaw 的整体架构和组件依赖关系，这将有助于设计合理的监控策略。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 理解 OpenClaw 监控体系的设计原则和最佳实践
- 配置多层次的健康检查和状态监控
- 建立有效的告警机制，确保异常情况及时通知
- 实施日志收集和分析策略
- 进行性能监控和容量规划

### 适用场景

| 场景 | 监控重点 | 推荐策略 |
|------|---------|----------|
| 个人使用 | 基础状态监控 | CLI 命令 + 简单告警 |
| 小团队 | 渠道稳定性监控 | 定时检查 + 邮件告警 |
| 企业部署 | 全方位监控 | Prometheus + Grafana + 多渠道告警 |
| 高频服务 | 性能趋势监控 | 持续指标收集 + 容量规划 |

## 监控体系设计原则

### 为什么需要系统化的监控

OpenClaw 作为一个长期运行的多渠道即时通讯代理服务，其监控需求具有独特的复杂性。与传统的 Web 应用不同，OpenClaw 需要同时维护多个渠道的长连接，任何一个渠道的异常都可能影响用户体验。此外，代理运行时涉及 AI 模型调用，其响应延迟和资源消耗也是需要重点关注的指标。

系统化的监控体系能够帮助运维团队实现以下目标：**可观测性**，通过多维度的指标数据了解系统运行状态；**可预期性**，通过趋势分析预测容量需求和问题苗头；**可恢复性**，通过快速告警和预案减少故障影响时间；**可优化性**，通过性能数据指导系统调优方向。

### 监控层次模型

```
监控层次金字塔：

                    ┌─────────────────────────┐
                    │      业务层监控          │  ← 消息吞吐量、用户体验
                    │  (Business Metrics)     │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ▼                   ▼                   ▼
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │    应用层      │   │    中间件层    │   │    基础设施层  │
    │ (Application)  │   │(Middleware)   │   │(Infrastructure)│
    └───────┬───────┘   └───────┬───────┘   └───────┬───────┘
            │                   │                   │
            ├─ Gateway 状态      ├─ Redis 缓存      ├─ CPU/内存/磁盘
            ├─ 渠道连接         ├─ 数据库连接      ├─ 网络 I/O
            ├─ 代理运行时       └─ 消息队列        └─ 容器状态
            └─ 会话管理
```

**各层次监控重点**：

| 层次 | 监控对象 | 关键指标 | 采集方法 |
|------|---------|---------|----------|
| 基础设施 | CPU、内存、磁盘、网络 | 使用率、IOPS、延迟 | 系统工具 |
| 中间件 | 连接池、缓存、队列 | 命中率、队列深度 | 状态查询 |
| 应用 | Gateway、渠道、代理 | 连接数、消息量、延迟 | 健康检查 + 指标 |
| 业务 | 用户体验、消息质量 | 送达率、满意度 | 用户反馈 |

## 健康检查体系

### 健康检查的设计哲学

健康检查是监控体系的第一道防线。一个设计良好的健康检查体系应该能够：**快速响应**，在秒级时间内完成检查；**层次分明**，从基础服务到深度功能逐层检查；**信息丰富**，检查结果应包含足够的诊断信息；**可操作**，检查失败后应能直接指向问题所在。

### 基本状态检查命令

**CLI 健康检查命令**：

```bash
# 快速健康检查（首选）
openclaw health

# 完整状态报告（适合诊断和分享）
openclaw status --all

# 深度检查（探测渠道连接）
openclaw status --deep

# 查看渠道状态
openclaw channels status

# 带探测的渠道检查
openclaw channels status --probe
```

**健康检查响应解读**：

```json
// 成功响应示例
{
  "status": "healthy",
  "timestamp": "2026-02-05T10:30:00Z",
  "version": "2026.1.27",
  "components": {
    "gateway": {
      "status": "running",
      "uptime_seconds": 86400,
      "memory_bytes": 268435456,
      "cpu_percent": 5.2
    },
    "channels": {
      "whatsapp": {
        "status": "connected",
        "last_activity": "2026-02-05T10:29:55Z",
        "error_count": 0
      },
      "telegram": {
        "status": "connected",
        "last_activity": "2026-02-05T10:29:50Z",
        "error_count": 0
      }
    }
  }
}

// 警告响应示例
{
  "status": "degraded",
  "warnings": [
    {
      "code": "HIGH_MEMORY",
      "message": "内存使用超过 80%",
      "current_value_mb": 850,
      "threshold_mb": 800
    }
  ]
}

// 失败响应示例
{
  "status": "unhealthy",
  "errors": [
    {
      "code": "CHANNEL_DISCONNECTED",
      "channel": "whatsapp",
      "message": "WhatsApp 渠道已断开连接",
      "last_error": "Connection timeout"
    }
  ]
}
```

### HTTP 健康检查端点

**端点说明**：

| 端点 | 方法 | 用途 | 认证 |
|------|------|------|------|
| `/health` | GET | 基础健康检查 | 无 |
| `/status` | GET | 完整状态信息 | 令牌 |
| `/metrics` | GET | Prometheus 指标 | 令牌 |

**健康检查端点使用示例**：

```bash
# 基础健康检查（无认证）
curl http://127.0.0.1:18789/health

# 带认证的完整状态
curl -H "Authorization: Bearer $CLAWDBOT_GATEWAY_TOKEN" \
    http://127.0.0.1:18789/status

# Prometheus 指标端点
curl -H "Authorization: Bearer $CLAWDBOT_GATEWAY_TOKEN" \
    http://127.0.0.1:18789/metrics
```

### 诊断工具

**openclaw doctor 命令**：

`openclaw doctor` 是最重要的诊断工具，能够自动检测和修复常见问题：

```bash
# 综合诊断（推荐）
openclaw doctor

# 自动修复发现的问题
openclaw doctor --fix

# 自动确认所有修复（无人值守场景）
openclaw doctor --yes

# 检查特定类别
openclaw doctor --check config          # 配置检查
openclaw doctor --check channels        # 渠道检查
openclaw doctor --check permissions    # 权限检查
openclaw doctor --check security        # 安全检查
openclaw doctor --check storage        # 存储检查
```

**诊断输出示例**：

```
========================================
OpenClaw 诊断报告
========================================
检查时间: 2026-02-05 10:30:00
版本: 2026.1.27

[✓] Node.js 版本检查
    版本: v22.12.0 (满足要求 >= 22.0.0)

[✓] 配置文件检查
    路径: /home/user/.clawdbot/openclaw.json
    语法: 有效

[✓] Gateway 服务检查
    状态: 运行中 (PID: 12345)
    端口: 18789
    绑定: 127.0.0.1

[⚠] 内存使用检查
    当前: 512MB
    建议: < 500MB
    操作: 建议重启服务或优化配置

[✓] 渠道连接检查
    WhatsApp: 已连接
    Telegram: 已连接
    Discord: 已连接

[✓] 磁盘空间检查
    可用: 45.2GB
    状态: 正常

========================================
诊断结果: 发现 1 个警告
建议: 执行 'openclaw doctor --fix' 修复
========================================
```

## 日志管理

### 日志系统架构

OpenClaw 的日志系统采用分层架构，支持多级别输出和灵活的配置：

```
日志输出层次：

┌─────────────────────────────────────────────────────────────┐
│                      日志源层                               │
│  ┌──────────────┐ ┌──────────────┐ ┌────────────────────┐ │
│  │ Gateway 日志 │ │ 渠道日志     │ │ 代理运行时日志     │ │
│  └──────┬───────┘ └──────┬───────┘ └─────────┬──────────┘ │
└─────────┼────────────────┼───────────────────┼─────────────┘
          │                │                   │
          ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                      日志处理层                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              日志级别过滤                            │   │
│  │  debug → info → warn → error → fatal              │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                │
│          ┌───────────────┼───────────────┐                │
│          ▼               ▼               ▼                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │
│  │  文件输出   │ │ 控制台输出  │ │  JSON 格式输出      │ │
│  └─────────────┘ └─────────────┘ └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 日志查看命令

**实时日志监控**：

```bash
# 实时查看日志（最后 100 行）
openclaw logs --tail 100

# 实时追踪日志
openclaw logs --follow

# 查看特定日志级别
openclaw logs --level error    # 只看错误
openclaw logs --level warn     # 警告及以上
openclaw logs --level debug    # 调试及以上（详细）

# 查看特定渠道日志
openclaw logs --channel telegram
openclaw logs --channel whatsapp
openclaw logs --channel discord

# 组合过滤
openclaw logs --level error --channel telegram

# 按时间范围过滤
openclaw logs --since "2026-02-05 00:00:00"
openclaw logs --until "2026-02-05 12:00:00"
openclaw logs --since "2026-02-05 00:00:00" --until "2026-02-05 12:00:00"

# 搜索特定内容
openclaw logs --grep "error\|failed\|timeout"
openclaw logs --grep "whatsapp" --level info

# 输出到文件供分析
openclaw logs --level info --since "1 hour ago" > /tmp/openclaw_analysis.log
```

**macOS 统一日志查询**：

```bash
# 使用项目提供的日志脚本
./scripts/clawlog.sh

# 查看 OpenClaw 子系统日志（需要 sudo）
sudo ./scripts/clawlog.sh --follow

# 按类别过滤
./scripts/clawlog.sh --category gateway
./scripts/clawlog.sh --category channel
./scripts/clawlog.sh --category agent

# 按级别过滤
./scripts/clawlog.sh --level error
./scripts/clawlog.sh --level warning

# 组合过滤
./scripts/clawlog.sh --category agent --level error --since "1 hour ago"
```

### 日志配置

**日志配置选项**：

```json5
{
  "logging": {
    // 日志级别：debug | info | warn | error | fatal
    "level": "info",

    // 日志格式：pretty | json | compact
    "format": "pretty",

    // 日志文件路径
    "file": "~/.clawdbot/logs/openclaw.log",

    // 单个日志文件最大大小
    "maxSize": "10m",

    // 保留的日志文件数量
    "maxFiles": 5,

    // 控制台日志级别
    "consoleLevel": "warn",

    // 控制台输出格式
    "consoleStyle": "pretty",  // pretty | json | compact

    // 敏感信息脱敏：off | tools | all
    "redactSensitive": "tools",

    // 时间戳格式
    "timestampFormat": "ISO"  // ISO | Local | Unix
  }
}
```

**日志级别说明**：

| 级别 | 使用场景 | 输出内容 |
|------|---------|----------|
| `debug` | 开发调试 | 详细执行步骤、变量值、调用栈 |
| `info` | 正常操作 | 主要操作完成、状态变更 |
| `warn` | 潜在问题 | 异常但可恢复的情况 |
| `error` | 错误 | 操作失败但不影响整体运行 |
| `fatal` | 致命错误 | 导致服务不可用的问题 |

### 日志轮转配置

**Linux logrotate 配置**：

```bash
# /etc/logrotate.d/openclaw

# 日志文件路径模式
/tmp/openclaw/openclaw-*.log
~/.clawdbot/logs/openclaw/*.log {
    # 每日轮转
    daily

    # 保留 14 天的日志
    rotate 14

    # 压缩旧日志
    compress
    delaycompress

    # 如果日志文件为空，不轮转
    notifempty

    # 创建新日志文件权限
    create 0640 root admin

    # 日期后缀格式
    dateext
    dateformat -%Y-%m-%d

    # 轮转后执行脚本
    postrotate
        # 通知服务重新打开日志文件
        systemctl --user restart openclaw-gateway 2>/dev/null || true
        /bin/kill -HUP $(cat /var/run/openclaw-gateway.pid 2>/dev/null) 2>/dev/null || true
    endscript

    # 忽略错误
    missingok
}
```

### 生产环境日志策略

```
生产环境日志策略设计：

1. 存储策略
   ├── 本地存储：最近 7 天的详细日志
   ├── 归档存储：7-30 天的压缩日志
   └── 长期存档：30 天以上的聚合日志

2. 级别策略
   ├── 文件日志：info 级别（问题追溯）
   ├── 控制台：warn 级别（实时监控）
   └── 告警触发：error 级别自动告警

3. 安全策略
   ├── 敏感信息自动脱敏
   ├── 凭证数据不写入日志
   └── 日志访问权限控制
```

## 进程管理

### 进程状态检查

**基础进程监控**：

```bash
# 检查进程是否运行
ps aux | grep openclaw
ps aux | grep "openclaw.*gateway"

# 检查进程 PID
pgrep -f openclaw-gateway

# 查看进程详细信息
ps -p $(pgrep -f openclaw) -o pid,ppid,cmd,%mem,%cpu,etime

# 进程树（如果存在子进程）
pstree -p $(pgrep -f openclaw-gateway)
```

**端口和连接状态**：

```bash
# 检查端口监听状态
ss -ltnp | grep 18789
lsof -i :18789

# 查看网络连接
ss -tnp | grep 18789
netstat -tnp | grep 18789

# 检查连接数统计
ss -s

# macOS 特定命令
lsof -i :18789 -a
```

**systemd 服务状态（Linux）**：

```bash
# 检查服务状态
systemctl --user status openclaw-gateway

# 检查是否启用开机启动
systemctl --user is-enabled openclaw-gateway

# 查看服务单元详情
systemctl --user status openclaw-gateway -l

# 查看最近的日志
journalctl --user -u openclaw-gateway -n 50
journalctl --user -u openclaw-gateway -f

# 查看实时资源使用
systemctl --user status openclaw-gateway -o json
```

**launchd 服务状态（macOS）**：

```bash
# 检查 launchd 服务
launchctl print gui/$UID | grep openclaw

# 查看服务详细信息
launchctl list | grep openclaw

# 查看 plist 配置
cat ~/Library/LaunchAgents/openclaw.gateway.plist
```

### 服务重启策略

**安全重启流程**：

```bash
# 推荐的优雅重启流程

# 1. 记录当前状态
echo "=== 重启前状态 ==="
openclaw status --all
openclaw channels status

# 2. 通知用户（如有必要）
# curl -X POST https://hooks.slack.com/... -d "{'text':'OpenClaw 即将重启'}"

# 3. 停止服务
echo "=== 停止服务 ==="
systemctl --user stop openclaw-gateway

# 4. 等待完全停止
sleep 5

# 5. 清理（如有必要）
# rm -rf ~/.clawdbot/cache/*

# 6. 启动服务
echo "=== 启动服务 ==="
systemctl --user start openclaw-gateway

# 7. 等待服务就绪
sleep 10

# 8. 验证状态
echo "=== 验证 ==="
openclaw health
openclaw channels status --probe
```

**macOS 重启脚本**：

```bash
#!/bin/bash
# restart-mac.sh - macOS 重启脚本

echo "OpenClaw Gateway 重启中..."

# 停止服务
openclaw service stop

# 等待进程完全退出
sleep 3

# 检查是否还在运行
if pgrep -f "openclaw.*gateway" > /dev/null; then
    echo "强制终止进程..."
    pkill -9 -f "openclaw.*gateway"
    sleep 2
fi

# 启动服务
openclaw service start

# 等待就绪
sleep 5

# 验证
if openclaw health; then
    echo "✓ 重启成功"
else
    echo "✗ 重启失败，请检查日志"
    openclaw logs --tail 50
fi
```

## 渠道监控

### 渠道状态检查

**渠道状态命令**：

```bash
# 查看所有渠道概览
openclaw channels status

# 查看特定渠道详情
openclaw channels status whatsapp
openclaw channels status telegram --verbose
openclaw channels status discord --probe

# 带探测的深度检查（验证连接）
openclaw channels status --probe

# 查看渠道配置
openclaw config get channels
```

**渠道健康指标**：

| 指标 | 说明 | 正常值 | 异常表现 |
|------|------|--------|----------|
| `connection_status` | 连接状态 | `connected` | `disconnected`, `connecting` |
| `last_activity` | 最后活动时间 | < 5 分钟 | > 30 分钟 |
| `error_count` | 累计错误数 | 0 或低 | 持续增长 |
| `reconnect_attempts` | 重连尝试次数 | 0 或低 | 持续重连 |
| `message_queue` | 待处理消息数 | < 10 | > 50 |

### 配对请求监控

**配对管理命令**：

```bash
# 查看所有待处理配对请求
openclaw pairing list

# 按渠道查看
openclaw pairing list telegram
openclaw pairing list whatsapp

# 查看配对详情
openclaw pairing show <pairing-id>

# 批准配对请求
openclaw pairing approve <channel> <pairing-code>

# 拒绝配对请求
openclaw pairing reject <channel> <pairing-id>

# 清除过期配对
openclaw pairing clean --older-than 24h
```

### 渠道连接故障排查

```
渠道连接问题排查流程：

1. 检查渠道整体状态
   openclaw channels status --probe

2. 如果某渠道断开，查看具体错误
   openclaw logs --channel <channel> --level error --tail 50

3. 常见原因分析
   ├── 凭证过期 → 重新登录
   ├── 网络问题 → 检查网络连接
   ├── API 限制 → 查看渠道文档
   └── 服务端异常 → 查看官方状态

4. 恢复操作
   openclaw channels logout <channel>
   openclaw channels login <channel>
```

## 性能监控

### 资源使用监控

**系统资源检查**：

```bash
# 内存和 CPU 使用（持续监控）
top -p $(pgrep -f openclaw)

# 更详细的资源监控
htop -p $(pgrep -f openclaw)

# 内存详情
ps -p $(pgrep -f openclaw) -o pid,vsz,rss,%mem,%cpu,etime

# I/O 使用
iotop -p $(pgrep -f openclaw)

# 网络使用
nethogs -p $(pgrep -f openclaw)
```

**OpenClaw 状态指标**：

```bash
# 查看资源使用状态
openclaw status

# 深度分析
openclaw status --deep

# 性能报告
openclaw status --all > /tmp/openclaw-status.txt
cat /tmp/openclaw-status.txt
```

### 会话统计

**会话管理命令**：

```bash
# 查看活跃会话
openclaw sessions list

# 会话详情
openclaw sessions info <session-id>

# 查看会话历史
openclaw sessions history <session-id>

# 清理过期会话
openclaw sessions prune --older-than 7d
openclaw sessions prune --older-than 30d --force

# 会话统计
openclaw sessions stats
```

### 性能调优参数

```
性能调优决策框架：

问题诊断流程：

1. 资源瓶颈识别
   ├── CPU 使用率高？
   │   └── 减少并发、优化算法、升级硬件
   ├── 内存使用率高？
   │   └── 清理会话、限制缓存、重启服务
   ├── 磁盘 I/O 高？
   │   └── 日志轮转、优化存储、升级磁盘
   └── 网络延迟高？
       └── 优化连接、检查网络、CDN

2. 配置优化
   ├── 消息队列参数调整
   └── 代理运行时配置

3. 架构优化
   ├── 增加实例
   └── 分布式部署
```

## 告警设置

### 告警策略设计

**告警级别定义**：

| 级别 | 名称 | 说明 | 响应时间 | 通知方式 |
|------|------|------|---------|----------|
| P1 | 紧急 | 服务完全不可用 | < 15 分钟 | 电话 + 多渠道 |
| P2 | 严重 | 主要功能受损 | < 1 小时 | 多渠道 |
| P3 | 警告 | 潜在问题 | < 4 小时 | 单渠道 |
| P4 | 信息 | 正常变更 | < 24 小时 | 可选 |

### Cron 作业监控

**定时健康检查**：

```json5
{
  "cron": {
    "jobs": [
      {
        "id": "health-check-5min",
        "schedule": "*/5 * * * *",  // 每 5 分钟
        "agent": "main",
        "message": "检查网关状态，如果有问题请告诉我",
        "deliverTo": "telegram:admin_id",
        "condition": {
          "type": "health",
          "status": "degraded"
        }
      },
      {
        "id": "daily-status-report",
        "schedule": "0 9 * * *",  // 每天 9:00
        "agent": "main",
        "message": "生成每日状态报告",
        "deliverTo": "telegram:admin_id"
      }
    ]
  }
}
```

### 外部监控集成

**Uptime Kuma 配置**：

```
监控配置：

名称: OpenClaw Gateway
类型: HTTP(s)
URL: http://localhost:18789/health
间隔: 60 秒
超时: 10 秒
重试次数: 3

告警通知:
├── 离线: Telegram, Email
└── 延迟 > 5s: Telegram
```

**Prometheus 指标收集**：

```typescript
// custom-metrics-exporter.ts - 自定义指标收集器

import { Client, Gateway } from './gateway'

interface Metrics {
  // 服务指标
  gateway_uptime_seconds: number
  gateway_started_at: number

  // 资源指标
  memory_bytes: number
  cpu_percent: number

  // 渠道指标
  channels_total: number
  channels_connected: number
  channels_disconnected: number

  // 消息指标
  messages_sent_total: number
  messages_received_total: number
  message_errors_total: number

  // 会话指标
  active_sessions: number
  session_errors_total: number
}

class MetricsExporter {
  private gateway: Gateway
  private interval: NodeJS.Timer

  constructor(gateway: Gateway) {
    this.gateway = gateway
  }

  async collect(): Promise<Metrics> {
    const status = await this.gateway.getStatus()
    const channels = await this.gateway.getChannels()

    return {
      gateway_uptime_seconds: status.uptime,
      gateway_started_at: status.startedAt,

      memory_bytes: status.memory,
      cpu_percent: status.cpu,

      channels_total: channels.length,
      channels_connected: channels.filter(c => c.status === 'connected').length,
      channels_disconnected: channels.filter(c => c.status !== 'connected').length,

      messages_sent_total: status.messages.sent,
      messages_received_total: status.messages.received,
      message_errors_total: status.messages.errors,

      active_sessions: status.sessions.active,
      session_errors_total: status.sessions.errors
    }
  }

  // Prometheus 格式输出
  async export(): Promise<string> {
    const metrics = await this.collect()

    return `
# HELP openclaw_gateway_uptime_seconds Gateway uptime in seconds
# TYPE openclaw_gateway_uptime_seconds counter
openclaw_gateway_uptime_seconds ${metrics.gateway_uptime_seconds}

# HELP openclaw_memory_bytes Memory usage in bytes
# TYPE openclaw_memory_bytes gauge
openclaw_memory_bytes ${metrics.memory_bytes}

# HELP openclaw_cpu_percent CPU usage percentage
# TYPE openclaw_cpu_percent gauge
openclaw_cpu_percent ${metrics.cpu_percent}

# HELP openclaw_channels_connected Number of connected channels
# TYPE openclaw_channels_connected gauge
openclaw_channels_connected ${metrics.channels_connected}

# HELP openclaw_active_sessions Number of active sessions
# TYPE openclaw_active_sessions gauge
openclaw_active_sessions ${metrics.active_sessions}
    `.trim()
  }
}
```

## 远程监控

### SSH 远程检查

**远程诊断命令**：

```bash
# 远程检查网关状态
ssh gateway-host "openclaw channels status --probe"

# 远程查看日志
ssh gateway-host "tail -n 100 /tmp/openclaw-gateway.log"

# 远程健康检查
ssh gateway-host "curl -s http://127.0.0.1:18789/health"

# 远程执行诊断
ssh gateway-host "openclaw doctor"

# 组合命令：检查状态并获取诊断
ssh gateway-host "openclaw status --all > /tmp/status.txt" && \
    scp gateway-host:/tmp/status.txt ./remote-status.txt
```

**自动化远程监控脚本**：

```bash
#!/bin/bash
# remote-monitor.sh - 远程监控脚本

HOSTS=("gateway-1" "gateway-2" "gateway-3")
DATE=$(date +%Y%m%d_%H%M%S)
OUTPUT_DIR="/var/monitoring/openclaw"

mkdir -p "$OUTPUT_DIR"

for HOST in "${HOSTS[@]}"; do
    echo "=== 检查 $HOST ==="

    # 健康检查
    ssh "$HOST" "curl -sf http://127.0.0.1:18789/health" > "$OUTPUT_DIR/${HOST}_health_${DATE}.json"

    # 状态报告
    ssh "$HOST" "openclaw status --all" > "$OUTPUT_DIR/${HOST}_status_${DATE}.txt"

    # 错误日志
    ssh "$HOST" "openclaw logs --level error --since '24 hours'" > "$OUTPUT_DIR/${HOST}_errors_${DATE}.log"

    echo "--- $HOST 完成 ---"
done

# 生成汇总报告
echo "=== 监控汇总 $(date) ===" > "$OUTPUT_DIR/summary_${DATE}.txt"
for HOST in "${HOSTS[@]}"; do
    echo "--- $HOST ---" >> "$OUTPUT_DIR/summary_${DATE}.txt"
    cat "$OUTPUT_DIR/${HOST}_health_${DATE}.json" >> "$OUTPUT_DIR/summary_${DATE}.txt"
done
```

### Tailscale 远程访问

**Tailscale Serve 配置**：

```json5
{
  "gateway": {
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

**通过 Tailscale 访问**：

```
访问地址: https://your-machine.tailnet.ts.net:18789
控制台:   https://your-machine.tailnet.ts.net:18793
健康检查: https://your-machine.tailnet.ts.net:18789/health
```

## 常见问题排查

### 网关无法启动

```
问题：Gateway 服务无法启动

排查步骤：

1. 检查端口占用
   lsof -i :18789
   ss -ltnp | grep 18789

2. 检查配置文件语法
   openclaw doctor --check config

3. 查看启动日志
   journalctl --user -u openclaw-gateway -n 100
   tail -100 /tmp/openclaw-gateway.log

4. 手动测试启动
   openclaw gateway --verbose

5. 常见原因
   ├── 端口被占用 → 使用其他端口或终止占用进程
   ├── 配置错误 → 运行 doctor 修复
   ├── 权限不足 → 检查文件权限
   └── Node.js 版本不兼容 → 升级 Node.js >= 22
```

### 渠道断开连接

```
问题：渠道连接频繁断开

排查步骤：

1. 检查网络连通性
   ping api.whatsapp.com  // WhatsApp
   ping api.telegram.org  // Telegram

2. 验证凭证状态
   ls -la ~/.clawdbot/credentials/

3. 查看渠道日志
   openclaw logs --channel <channel> --level error --tail 100

4. 检查是否有 API 限制
   ├── WhatsApp: 检查手机网络
   ├── Telegram: 检查 bot 余额
   └── Discord: 检查 bot 权限

5. 解决方案
   ├── 重新登录渠道
   ├── 清理凭证后重新配置
   └── 联系平台支持
```

### 消息延迟

```
问题：消息响应延迟过高

排查步骤：

1. 检查系统资源
   openclaw status --deep
   top -p $(pgrep -f openclaw)

2. 查看队列状态
   openclaw status | grep queue

3. 检查外部 API 响应
   curl -w "\ntime: %{time_total}s" https://api.anthropic.com

4. 优化建议
   ├── 减少历史会话长度
   ├── 调整消息队列参数
   ├── 增加系统资源
   └── 考虑升级网络
```

## 生产环境监控清单

### 每日检查清单

- [ ] 网关进程正在运行
- [ ] 所有配置渠道已连接
- [ ] 无严重错误日志（过去 24 小时）
- [ ] 待处理配对请求已处理
- [ ] 内存使用在正常范围内（< 80%）
- [ ] 磁盘空间充足（> 20%）

### 每周检查清单

- [ ] 资源使用趋势分析
- [ ] 错误日志统计和模式分析
- [ ] 备份文件完整性验证
- [ ] 依赖版本安全扫描
- [ ] 监控告警规则优化

### 每月检查清单

- [ ] 性能基准对比分析
- [ ] 配置审计和优化
- [ ] 长期存档日志清理
- [ ] 监控策略评审和调整
- [ ] 恢复演练测试

## 相关文档

| 文档 | 说明 |
|------|------|
| [运维手册](/zh-CN/operations/index) | 运维最佳实践和整体架构 |
| [部署指南](/zh-CN/operations/deployment) | 详细部署步骤和配置 |
| [故障排除](/zh-CN/operations/troubleshooting) | 问题诊断和解决方法 |
| [网关配置](/zh-CN/gateway/configuration) | 配置选项参考 |
| [安全指南](/zh-CN/gateway/security) | 安全加固和合规要求 |
