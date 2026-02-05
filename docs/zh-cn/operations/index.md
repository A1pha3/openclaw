---
summary: 面向运维工程师的生产环境部署和维护指南，涵盖部署架构、服务管理、监控、日志管理、备份恢复、版本更新、安全加固、故障排除和性能调优
read_when:
  - 负责生产环境部署时
  - 需要了解运维最佳实践时
  - 管理运行中的 OpenClaw 服务时
title: 运维手册
---

# 运维手册

本手册面向负责 OpenClaw 生产环境部署和维护的运维工程师，提供全面的运维指南。在阅读本文档之前，建议您已经完成 OpenClaw 的基础安装和配置，并具备基本的 Linux/macOS 系统管理经验。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 理解 OpenClaw 的整体架构设计及其运维边界
- 掌握在不同部署模式下的服务管理方法
- 具备独立配置监控、日志和告警体系的能力
- 能够执行备份恢复和版本更新等常规运维操作
- 具备故障诊断和性能调优的基础能力

### 适用场景

本手册适用于以下运维场景：

| 场景 | 推荐阅读章节 | 优先级 |
|------|-------------|--------|
| 首次部署生产环境 | 部署架构、服务管理 | 高 |
| 日常运维巡检 | 监控、日志管理 | 高 |
| 故障应急响应 | 故障排除、服务管理 | 高 |
| 性能优化调优 | 性能调优、监控 | 中 |
| 安全合规审计 | 安全加固、备份恢复 | 中 |

## 运维概述

### 为什么需要可靠的运维策略

OpenClaw 是一个长期运行的服务，其稳定性直接影响到用户体验和业务连续性。与传统的 Web 应用不同，OpenClaw 需要维护多种即时通讯渠道的持久连接、代理会话状态以及敏感的凭证信息。这些特性决定了运维工作需要特别关注以下几点：

首先，渠道连接的稳定性至关重要。WhatsApp、Telegram 等渠道使用长连接机制，一旦连接中断，可能需要重新进行身份验证，这不仅影响用户体验，还可能导致会话丢失。其次，代理运行时需要持续占用系统资源，内存管理和进程监控成为日常巡检的重点内容。最后，凭证数据的安全存储和定期轮换是安全合规的基本要求。

### 核心运维任务体系

有效的运维工作需要建立系统化的任务体系，将日常操作分为不同优先级和执行频率。以下表格详细说明了各类运维任务的性质和要求：

| 任务类别 | 具体内容 | 执行频率 | 优先级 | 自动化程度 |
|---------|---------|---------|--------|------------|
| 持续监控 | 服务状态、渠道连接、资源使用 | 实时/5分钟 | 高 | 全自动 |
| 日志管理 | 日志收集、异常告警、审计追踪 | 每日 | 高 | 半自动 |
| 数据保护 | 配置备份、凭证备份、恢复演练 | 每日/每周 | 高 | 半自动 |
| 安全维护 | 令牌轮换、权限审计、漏洞扫描 | 每周 | 中 | 手动 |
| 版本更新 | 功能升级、安全补丁、兼容性验证 | 按需 | 中 | 半自动 |
| 性能优化 | 资源调优、队列配置、缓存策略 | 按需 | 低 | 手动 |

### 专家思维模型：运维决策框架

在日常运维工作中，遇到问题时需要遵循系统化的决策流程。以下框架可以帮助运维工程师快速定位问题并采取正确的应对措施：

```
问题诊断决策流程：

1. 影响范围评估
   └── 服务完全不可用 → 紧急（P0）
   └── 部分功能受损 → 严重（P1）
   └── 性能下降 → 一般（P2）
   └── 无影响 → 建议（P3）

2. 根因快速定位
   └── 检查服务进程状态
   └── 检查系统资源使用
   └── 检查渠道连接状态
   └── 查看最近日志记录

3. 应对策略选择
   └── 自动恢复：重启服务、清理缓存
   └── 配置调整：修改参数、重载配置
   └── 人工介入：联系支持、社区求助
   └── 根本修复：代码修复、架构调整

4. 复盘与预防
   └── 记录故障时间线
   └── 分析根本原因
   └── 制定预防措施
   └── 更新运维文档
```

## 部署架构

### 架构设计原则

在选择部署架构时，需要根据实际业务需求、资源条件和运维能力进行综合考量。OpenClaw 的架构设计遵循以下几个核心原则，这些原则直接影响部署方案的选择：

**本地优先原则**：OpenClaw 的设计哲学强调本地运行，用户的敏感数据（如凭证、会话历史）都存储在本地设备上。这一原则决定了部署方案应当优先考虑本地运行，只有在明确了解数据外流风险的情况下才考虑云端部署。

**单点简化原则**：对于个人用户场景，单机部署是最简单也是最可靠的方案。避免过度架构化带来的复杂性，维护成本往往超过预期收益。

**边界清晰原则**：无论采用何种部署方式，Gateway 服务始终绑定到本地接口（loopback），远程访问通过 VPN、Tailscale 或反向代理实现。这种设计将安全边界集中在一处，便于管理和审计。

### 单机部署架构（推荐大多数场景）

单机部署是 OpenClaw 的标准部署模式，适用于个人用户、小型团队以及大多数生产环境。这种架构的优势在于简单直观、故障域最小、资源占用可控。

```
┌─────────────────────────────────────────────────────────────────┐
│                          服务器主机                               │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    OpenClaw Gateway 服务                      │ │
│  │  ┌─────────────────────────────────────────────────────────┐ │ │
│  │  │  进程管理器 (systemd/launchd)                            │ │ │
│  │  │  监听端口：18789 (仅本地绑定)                            │ │ │
│  │  │  WebSocket 服务端点：ws://127.0.0.1:18789               │ │ │
│  │  └─────────────────────────────────────────────────────────┘ │ │
│  │                           │                                   │ │
│  │  ┌────────────────────────┴────────────────────────────┐  │ │
│  │  │              状态数据目录 ~/.clawdbot/                  │  │ │
│  │  │  ├── openclaw.json (主配置文件)                        │  │ │
│  │  │  ├── credentials/ (渠道凭证加密存储)                   │  │ │
│  │  │  ├── agents/ (代理运行时数据)                           │  │ │
│  │  │  └── sessions/ (会话历史记录)                           │  │ │
│  │  └─────────────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌───────────────────────────┴───────────────────────────────┐  │
│  │                      操作系统层                            │  │
│  │  - Linux: systemd 用户服务                                 │  │
│  │  - macOS: launchd 守护进程                                │  │
│  │  - Windows: 服务 wrapper (WSL2 推荐)                       │  │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**适用场景分析**：

| 场景类型 | 推荐配置 | 说明 |
|---------|---------|------|
| 个人使用 | 默认配置 | 满足所有需求，无需额外优化 |
| 小团队 (1-5人) | 默认配置 + 基础监控 | 增加健康检查频率 |
| 中等规模 (5-20人) | 优化资源配置 + 远程访问 | 考虑性能调优 |
| 高频使用 (20+人) | 分布式部署评估 | 可能需要多实例方案 |

### Docker 容器化部署

对于需要更高隔离性或统一管理环境的场景，Docker 部署提供了标准化的解决方案。容器化部署的核心价值在于环境一致性、快速迁移和资源隔离。

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: ghcr.io/openclaw/openclaw:latest
    container_name: openclaw-gateway
    ports:
      # 仅映射本地端口，外部访问通过反向代理
      - "127.0.0.1:18789:18789"
    volumes:
      # 持久化数据卷挂载
      - openclaw-data:/root/.clawdbot
    # 重启策略：自动重启，除非手动停止
    restart: unless-stopped
    environment:
      # API 密钥通过环境变量注入
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - CLAWDBOT_GATEWAY_TOKEN=${GATEWAY_TOKEN}
    healthcheck:
      # 健康检查配置
      test: ["CMD", "openclaw", "health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    # 安全增强：非 root 用户运行（如果镜像支持）
    user: "1000:1000"
    # 网络隔离
    networks:
      - openclaw-internal

networks:
  openclaw-internal:
    driver: bridge

volumes:
  openclaw-data:
    driver: local
```

**容器化部署的关键考量**：

```
选择容器化部署时的决策点：

1. 数据持久化策略
   └── 使用具名数据卷而非主机绑定挂载
   └── 凭证目录需要特别保护
   └── 定期备份数据卷

2. 网络访问控制
   └── 默认仅绑定 127.0.0.1
   └── 外部访问通过反向代理
   └── 避免暴露管理端口

3. 资源限制配置
   └── CPU 限制：根据实际负载配置
   └── 内存限制：建议 1-2GB 基础
   └── 存储限制：防止日志膨胀

4. 日志管理
   └── 配置日志轮转
   └── 集中日志收集（如需要）
   └── 敏感信息脱敏
```

### 云平台部署选项

对于不具备本地服务器条件的用户，云平台部署提供了便捷的替代方案。然而，在选择云平台部署时，必须充分认识到数据安全和隐私保护的挑战。

**支持的云平台概览**：

| 平台 | 部署方式 | 数据存储 | 成本等级 | 推荐程度 |
|------|---------|---------|---------|----------|
| Railway | Docker 容器 | 持久化磁盘 | 中等 | ⭐⭐⭐ |
| Render | Docker Web Service | 持久化磁盘 | 中等 | ⭐⭐⭐ |
| Northflank | Kubernetes | 持久化存储 | 较高 | ⭐⭐ |
| Fly.io | 边缘容器 | 分布式存储 | 中等 | ⭐⭐⭐ |
| Hetzner VPS | 手动安装 | 本地磁盘 | 低 | ⭐⭐⭐⭐ |

**云平台部署的核心挑战**：

在云平台部署 OpenClaw 时，需要特别关注以下挑战：

首先，**凭证安全**是最大的挑战。渠道登录凭证（如 WhatsApp 的会话文件）需要持久化存储，而云平台的磁盘可能存在数据泄露风险。建议在部署前评估渠道对持久化的依赖程度，对于敏感凭证考虑本地存储方案。

其次，**网络连通性**问题可能影响渠道稳定性。云平台的网络环境与本地网络存在差异，某些渠道可能需要特殊的网络配置或代理才能正常工作。

最后，**成本控制**需要持续关注。云平台的按需计费模式可能导致意外的费用增长，特别是在高流量场景下。

### 部署架构选择决策树

```
如何选择适合的部署架构：

问题 1：是否有专用的本地服务器或个人电脑？
├── 是 → 问题 2
└── 否 → 考虑云平台部署（Hetzner VPS 推荐）

问题 2：用户规模是多少？
├── 1-5人 → 单机部署
├── 5-20人 → 单机部署 + 性能优化
└── 20+人 → 考虑多实例或分布式架构

问题 3：是否需要频繁迁移？
├── 是 → Docker 部署
└── 否 → 本地安装更简单

问题 4：运维能力如何？
├── 熟悉容器 → Docker 部署
└── 不熟悉 → 本地安装 + systemd/launchd
```

## 服务管理

### 服务生命周期管理

OpenClaw Gateway 作为长期运行的服务，需要通过操作系统的服务管理机制进行生命周期管理。正确理解服务生命周期是运维工作的基础。

**服务状态转换模型**：

```
           ┌─────────────────────────────────────────────────┐
           │                    服务状态                       │
           └─────────────────────────────────────────────────┘
                                │
           ┌────────────────────┼────────────────────┐
           ▼                    ▼                    ▼
    ┌──────────┐         ┌──────────┐         ┌──────────────┐
    │  启动    │ ───成功──→ │  运行中  │ ───信号──→ │  优雅停止   │
    │ (start)  │         │ (active) │         │ (graceful)   │
    └──────────┘         └────┬─────┘         └──────┬───────┘
                              │                      │
                              │         ┌─────────────┘
                              │         │
                              ▼         ▼
                       ┌──────────┐  ┌──────────────┐
                       │  故障    │  │  强制终止    │
                       │(failure) │  │(force kill)  │
                       └────┬─────┘  └──────────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │  自动重启策略    │
                   │  (restart policy)│
                   └─────────────────┘
```

### Linux 系统服务管理 (systemd)

在 Linux 系统上，推荐使用 systemd 用户服务来管理 OpenClaw Gateway。这种方式可以确保服务在系统启动时自动运行，并在故障时自动重启。

**安装服务（推荐方式）**：

```bash
# 使用向导自动安装
openclaw onboard --install-daemon

# 或手动安装 systemd 用户服务
# 1. 创建服务文件
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/openclaw-gateway.service << 'EOF'
[Unit]
Description=OpenClaw Gateway Service
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/openclaw gateway run --bind loopback --port 18789
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

# 环境变量（如需要）
Environment=CLAWDBOT_GATEWAY_TOKEN=your_token_here

# 资源限制
MemoryMax=1G
CPUQuota=80%

[Install]
WantedBy=default.target
EOF

# 2. 重新加载 systemd
systemctl --user daemon-reload

# 3. 启用开机自启动
systemctl --user enable openclaw-gateway

# 4. 启动服务
systemctl --user start openclaw-gateway

# 5. 验证状态
systemctl --user status openclaw-gateway
```

**服务操作命令速查**：

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

# 查看最近的日志条目
journalctl --user -u openclaw-gateway -n 100

# 检查服务是否启用
systemctl --user is-enabled openclaw-gateway

# 查看服务资源使用
systemctl --user status openclaw-gateway -o json
```

### macOS 系统服务管理 (launchd)

在 macOS 系统上，使用 launchd 作为服务管理框架。通过 OpenClaw CLI 可以便捷地管理服务生命周期。

**服务管理命令**：

```bash
# 安装并启动服务（推荐）
openclaw onboard --install-daemon

# 查看服务状态
openclaw service status

# 启动服务
openclaw service start

# 停止服务
openclaw service stop

# 重启服务
openclaw service restart

# 查看实时日志
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# 使用统一日志系统查询
./scripts/clawlog.sh --follow

# 通过 launchctl 直接管理
launchctl print gui/$UID | grep openclaw
launchctl load ~/Library/LaunchAgents/openclaw.gateway.plist
launchctl unload ~/Library/LaunchAgents/openclaw.gateway.plist
```

**launchd 服务文件示例**：

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
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/openclaw/openclaw.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/openclaw/openclaw-error.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin</string>
    </dict>
</dict>
</plist>
```

### 手动运行（调试场景）

对于需要调试或排查问题的场景，可以在前台手动运行 Gateway 服务：

```bash
# 前台运行（用于调试）
openclaw gateway --port 18789 --verbose

# 指定配置文件
openclaw gateway --config /path/to/openclaw.json --verbose

# 查看所有可用选项
openclaw gateway --help

# 后台运行（生产环境不推荐）
nohup openclaw gateway --port 18789 > /tmp/openclaw.log 2>&1 &

# 记录 PID 以便后续管理
echo $! > /var/run/openclaw-gateway.pid
```

### 服务故障处理策略

```
服务异常处理决策流程：

1. 初步诊断
   └── systemctl/launchctl status
   └── journalctl/tail 查看日志
   └── ps aux | grep openclaw

2. 问题分类
   ├── 配置错误 → 修复配置 → 重启
   ├── 资源不足 → 释放资源/调整限制 → 重启
   ├── 依赖服务异常 → 解决依赖 → 重启
   └── 未知原因 → 查看详细日志 → 社区求助

3. 恢复操作
   └── 首选：优雅重启 (systemctl restart)
   └── 次选：强制终止后启动 (pkill && systemctl start)
   └── 最后：清理状态后重新初始化 (rm -rf state && reinit)
```

## 监控

### 监控体系设计原则

建立完善的监控体系是保障服务稳定运行的关键。OpenClaw 的监控设计遵循以下原则：**可观测性优先**，所有组件都暴露健康检查端点和状态信息；**渐进式深入**，从基础状态检查到深度诊断，覆盖不同场景；**自动化告警**，异常情况自动触发告警，减少人工巡检负担。

### 健康检查体系

**基础健康检查**：

```bash
# 快速健康检查
openclaw health

# 带探测的深度检查（检查渠道连接）
openclaw status --deep

# 完整状态报告（适合粘贴分享）
openclaw status --all

# HTTP 健康检查端点
curl http://127.0.0.1:18789/health
```

**健康检查端点响应示例**：

```json
{
  "status": "healthy",
  "timestamp": "2026-02-05T10:30:00Z",
  "version": "2026.1.27",
  "components": {
    "gateway": {
      "status": "running",
      "uptime_seconds": 86400
    },
    "channels": {
      "whatsapp": "connected",
      "telegram": "connected"
    },
    "resources": {
      "memory_mb": 256,
      "cpu_percent": 5.2
    }
  }
}
```

### 关键性能指标

| 指标类别 | 指标名称 | 正常范围 | 告警阈值 | 监控方法 |
|---------|---------|---------|---------|----------|
| 资源使用 | 内存占用 | < 500MB | > 1GB | openclaw status |
| 资源使用 | CPU 使用率 | < 30% | > 80% | 系统监控 |
| 渠道状态 | 连接数 | = 配置数 | < 配置数 | channels status |
| 消息处理 | 队列长度 | < 10 | > 50 | status --deep |
| 消息处理 | 平均延迟 | < 2s | > 10s | 日志分析 |
| 服务可用性 | 最后活动时间 | < 5min | > 30min | 主动探测 |

### 日志监控

**日志查看命令**：

```bash
# 实时查看日志（最后 100 行）
openclaw logs --tail 100

# 查看特定日志级别
openclaw logs --level error

# 查看特定渠道日志
openclaw logs --channel telegram

# 按时间范围过滤
openclaw logs --since "2026-02-05 00:00:00" --until "2026-02-05 12:00:00"

# 搜索特定内容
openclaw logs --grep "error\|failed" --level error

# macOS 统一日志查询
./scripts/clawlog.sh
./scripts/clawlog.sh --follow
./scripts/clawlog.sh --category gateway --level error
```

**日志配置优化**：

```json5
{
  "logging": {
    "level": "info",              // debug | info | warn | error
    "format": "pretty",           // pretty | json | compact
    "file": "~/.clawdbot/logs/openclaw.log",
    "maxSize": "10m",            // 单个文件最大大小
    "maxFiles": 5,               // 保留的日志文件数量
    "consoleLevel": "warn",      // 控制台日志级别（生产环境建议 warn/error）
    "redactSensitive": "tools"    // 敏感信息脱敏：off | tools | all
  }
}
```

### 告警配置

**Prometheus Alertmanager 集成**：

```yaml
# alertmanager 告警规则
groups:
  - name: openclaw
    interval: 30s
    rules:
      - alert: OpenClawDown
        expr: up{job="openclaw"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "OpenClaw Gateway 服务不可用"
          description: "OpenClaw 服务已停止超过 1 分钟"

      - alert: OpenClawHighMemory
        expr: openclaw_memory_bytes > 1073741824
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "OpenClaw 内存使用过高"
          description: "当前内存使用超过 1GB"

      - alert: OpenClawChannelDisconnected
        expr: openclaw_channels_connected < expected_channels
        for: 3m
        labels:
          severity: error
        annotations:
          summary: "渠道连接断开"
          description: "当前连接的渠道数量低于预期"
```

## 日志管理

### 日志位置与分类

| 日志类型 | 路径 | 说明 | 轮转策略 |
|---------|------|------|----------|
| 应用日志 | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | 日常运行日志 | logrotate |
| 系统日志 | journalctl / log stream | 系统级日志 | 系统自动 |
| 渠道日志 | `~/.clawdbot/logs/` | 渠道特定日志 | 手动管理 |
| 会话日志 | `~/.clawdbot/agents/<id>/sessions/` | 代理会话记录 | 定期归档 |

### 日志轮转配置

```bash
# /etc/logrotate.d/openclaw

# 每日轮转
/tmp/openclaw/openclaw-*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root admin
    dateext
    dateformat -%Y-%m-%d
    postrotate
        # 日志轮转后通知服务重新打开日志文件
        systemctl --user restart openclaw-gateway 2>/dev/null || true
    endscript
}
```

## 备份与恢复

### 备份内容与优先级

| 数据类型 | 存储路径 | 重要性 | 备份频率 | 保留周期 |
|---------|---------|--------|---------|----------|
| 主配置文件 | `~/.clawdbot/openclaw.json` | 高 | 每日 | 30天 |
| 渠道凭证 | `~/.clawdbot/credentials/` | 高 | 每日 | 90天 |
| OAuth 令牌 | `~/.clawdbot/credentials/oauth.json` | 高 | 每日 | 90天 |
| 代理数据 | `~/.clawdbot/agents/` | 中 | 每周 | 30天 |
| 会话历史 | `~/.clawdbot/sessions/` | 低 | 每月 | 7天 |

### 备份脚本实现

```bash
#!/bin/bash
# backup-openclaw.sh - OpenClaw 数据备份脚本

set -euo pipefail

# 配置
BACKUP_DIR="/backup/openclaw"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 定义备份文件
BACKUP_FILE="$BACKUP_DIR/clawdbot_$DATE.tar.gz"

# 备份关键数据
echo "[$(date)] 开始备份 OpenClaw 数据..."

tar -czf "$BACKUP_FILE" \
    -C ~ \
    .clawdbot/openclaw.json \
    .clawdbot/credentials/ \
    .clawdbot/agents/ 2>/dev/null || {
    echo "[ERROR] 备份失败"
    exit 1
}

# 验证备份文件
if [ -f "$BACKUP_FILE" ] && [ -s "$BACKUP_FILE" ]; then
    echo "[$(date)] 备份成功: $BACKUP_FILE"
    BACKUP_SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "[INFO] 备份大小: $BACKUP_SIZE"
else
    echo "[ERROR] 备份文件无效"
    rm -f "$BACKUP_FILE"
    exit 1
fi

# 清理旧备份
echo "[$(date)] 清理超过 $RETENTION_DAYS 天的旧备份..."
find "$BACKUP_DIR" -name "clawdbot_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "[$(date)] 备份完成"
```

### 恢复流程

```bash
#!/bin/bash
# restore-openclaw.sh - OpenClaw 数据恢复脚本

set -euo pipefail

# 配置
BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
    echo "用法: $0 <backup_file.tar.gz>"
    exit 1
fi

if [ ! -f "$BACKUP_FILE" ]; then
    echo "错误: 备份文件不存在"
    exit 1
fi

echo "[$(date)] 准备恢复 OpenClaw 数据..."
echo "[WARNING] 这将覆盖现有数据"

# 停止服务
echo "[$(date)] 停止 OpenClaw 服务..."
openclaw service stop 2>/dev/null || systemctl --user stop openclaw-gateway

# 备份当前数据（以防万一）
mkdir -p ~/.clawdbot-backup-$(date +%Y%m%d%H%M%S)
cp -r ~/.clawdbot/* ~/.clawdbot-backup-$(date +%Y%m%d%H%M%S)/ 2>/dev/null || true

# 执行恢复
echo "[$(date)] 解压恢复数据..."
tar -xzf "$BACKUP_FILE" -C ~

# 验证配置文件
if [ -f ~/.clawdbot/openclaw.json ]; then
    echo "[$(date)] 配置文件恢复成功"
else
    echo "[ERROR] 配置文件恢复失败"
    exit 1
fi

# 运行诊断修复
echo "[$(date)] 运行诊断修复..."
openclaw doctor --fix

# 启动服务
echo "[$(date)] 启动 OpenClaw 服务..."
openclaw service start 2>/dev/null || systemctl --user start openclaw-gateway

# 验证健康状态
sleep 5
if openclaw health; then
    echo "[SUCCESS] OpenClaw 恢复完成并正常运行"
else
    echo "[WARNING] 服务可能存在问题，请检查状态"
fi
```

## 版本更新

### 更新前准备

```bash
# 查看当前版本
openclaw --version

# 检查可用更新
npm view openclaw version

# 检查当前运行的进程
ps aux | grep openclaw

# 创建配置备份
cp ~/.clawdbot/openclaw.json ~/.clawdbot/openclaw.json.bak.$(date +%Y%m%d)

# 检查磁盘空间
df -h ~
```

### 执行更新

```bash
# 方式一：使用 CLI 更新命令
openclaw update

# 方式二：手动更新
npm install -g openclaw@latest

# 方式三：指定版本更新
npm install -g openclaw@2026.1.27

# 更新后验证
openclaw doctor
openclaw health
```

### 回滚策略

```bash
# 方案一：使用 npm 回滚到指定版本
npm install -g openclaw@2026.1.26

# 方案二：恢复配置备份
cp ~/.clawdbot/openclaw.json.bak.* ~/.clawdbot/openclaw.json

# 方案三：从备份恢复完整状态
./restore-openclaw.sh /backup/openclaw/clawdbot_20260204.tar.gz
```

## 安全加固

### 安全加固检查清单

```
安全加固检查项目：

1. 认证配置
   ├── [ ] 配置网关认证令牌
   ├── [ ] 启用 API 访问控制
   └── [ ] 定期轮换令牌

2. 网络安全
   ├── [ ] 绑定到本地接口 (loopback)
   ├── [ ] 配置防火墙规则
   └── [ ] 启用 TLS（如使用反向代理）

3. 数据保护
   ├── [ ] 配置文件权限 (600)
   ├── [ ] 凭证目录访问控制
   └── [ ] 启用敏感信息脱敏

4. 访问控制
   ├── [ ] 配置 allowlist
   ├── [ ] 设置 DM 策略
   └── [ ] 审查群组访问规则

5. 审计日志
   ├── [ ] 启用访问日志
   ├── [ ] 配置日志保留
   └── [ ] 定期审查异常访问
```

### 核心安全配置

```json5
{
  // 网关认证
  "gateway": {
    "auth": {
      "token": "${CLAWDBOT_GATEWAY_TOKEN}",
      "mode": "token"  // token | password | none
    },
    // 网络绑定
    "bind": "loopback",  // 始终使用本地绑定
    "port": 18789
  },

  // 日志安全
  "logging": {
    "level": "info",
    "redactSensitive": "all",  // 脱敏所有敏感信息
    "consoleLevel": "error"
  }
}
```

### 定期安全审计

```bash
# 深度安全审计
openclaw security audit --deep

# 检查文件权限
openclaw doctor

# 查看最近认证失败
grep -i "auth\|fail" /tmp/openclaw/openclaw-*.log | tail -50

# 检查配置中的潜在风险
openclaw config get --audit
```

## 故障排除

### 故障诊断流程

```
故障排除思维模型：

1. 症状收集
   └── 记录错误信息和时间戳
   └── 确定影响范围
   └── 收集相关日志

2. 问题分类
   ├── 启动失败 → 检查配置和端口
   ├── 运行崩溃 → 检查资源使用
   ├── 功能异常 → 检查渠道状态
   └── 性能下降 → 检查负载和配置

3. 快速恢复
   ├── 优先尝试重启服务
   ├── 其次尝试清理缓存
   └── 最后考虑数据恢复

4. 根因分析
   ├── 分析日志找规律
   ├── 检查变更历史
   └── 制定预防措施
```

### 常见问题速查

#### Gateway 无法启动

```bash
# 1. 检查配置语法
openclaw doctor

# 2. 检查端口占用
lsof -i :18789
ss -ltnp | grep 18789

# 3. 查看详细日志
openclaw gateway --verbose

# 4. 验证 Node.js 版本
node --version  # 需要 >= 22
```

#### WhatsApp 断开连接

```bash
# 1. 检查连接状态
openclaw channels status whatsapp

# 2. 重新登录渠道
openclaw channels logout whatsapp
openclaw channels login whatsapp

# 3. 检查网络连接
ping web.whatsapp.com

# 4. 查看连接日志
openclaw logs --channel whatsapp --grep "disconnect\|reconnect"
```

#### 内存泄漏

```bash
# 1. 检查内存使用趋势
openclaw status --deep

# 2. 监控系统资源
top -p $(pgrep -f openclaw)

# 3. 清理会话历史
openclaw sessions prune --older-than 7d

# 4. 重启服务释放内存
openclaw service restart
```

### 诊断命令速查表

| 命令 | 用途 | 使用场景 |
|------|------|----------|
| `openclaw doctor` | 综合诊断 | 首选诊断工具 |
| `openclaw health` | 快速健康检查 | 初步状态确认 |
| `openclaw status --all` | 完整状态报告 | 问题报告/分享 |
| `openclaw status --deep` | 深度诊断 | 详细分析 |
| `openclaw logs --tail 100` | 实时日志 | 动态问题追踪 |
| `openclaw channels status` | 渠道状态 | 渠道问题诊断 |

## 性能调优

### 性能优化决策框架

```
性能调优优先级：

1. 配置层面（高投入产出比）
   ├── 消息队列参数调整
   ├── 内存限制配置
   └── 日志级别优化

2. 系统层面（中投入产出比）
   ├── 文件描述符限制
   ├── 内存分配策略
   └── 进程优先级调整

3. 架构层面（低投入产出比，需要重构）
   ├── 多实例部署
   ├── 分布式架构
   └── 缓存层引入
```

### 消息队列优化

```json5
{
  "messages": {
    "queue": {
      "mode": "collect",      // collect | sequential | immediate
      "debounceMs": 1000,     // 消息收集窗口（毫秒）
      "cap": 20              // 最大批量大小
    }
  }
}
```

### 沙箱资源限制

```json5
{
  "agents": {
    "defaults": {
      "sandbox": {
        "docker": {
          "cpuLimit": "2",       // CPU 核心数限制
          "memoryLimit": "2g",   // 内存限制
          "networkDisabled": false  // 是否禁用网络
        }
      }
    }
  }
}
```

### 日志性能优化

```json5
{
  "logging": {
    "level": "warn",           // 生产环境使用 warn 或 error
    "consoleLevel": "error",   // 减少控制台输出
    "consoleStyle": "json"     // JSON 格式便于聚合
  }
}
```

## 运维检查清单

### 每日检查清单

- [ ] 网关进程正在运行
- [ ] 所有配置渠道已连接
- [ ] 无严重错误日志
- [ ] 待处理配对请求已处理
- [ ] 内存使用在正常范围内

### 每周检查清单

- [ ] 资源使用趋势分析
- [ ] 错误日志统计和趋势
- [ ] 备份文件完整性验证
- [ ] 依赖版本安全扫描
- [ ] 配置审计和优化

### 每月检查清单

- [ ] 性能基准对比分析
- [ ] 安全配置审计
- [ ] 长期存档日志清理
- [ ] 恢复演练测试
- [ ] 运维文档更新

## 下一步

完成本运维手册的学习后，建议按以下路径继续深入：

| 路径 | 目标文档 | 说明 |
|------|---------|------|
| 深入部署 | [部署指南](/zh-CN/operations/deployment) | 详细的部署步骤和配置选项 |
| 监控体系 | [监控与日志](/zh-CN/operations/monitoring) | 完整的监控配置和告警策略 |
| 故障处理 | [故障排除](/zh-CN/operations/troubleshooting) | 系统化的故障诊断方法 |
| 架构设计 | [核心概念](/zh-CN/concepts/architecture) | 深入理解系统架构 |
