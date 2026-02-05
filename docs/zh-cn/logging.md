---
summary: "OpenClaw 日志与诊断完全指南：日志配置、CLI 尾随、诊断系统、OpenTelemetry 集成和可观测性架构"
read_when:
  - 你需要理解 OpenClaw 的日志系统和诊断架构
  - 你想掌握日志配置和定制方法
  - 你需要配置 OpenTelemetry 进行生产监控
  - 你在排查问题并需要可观测性支持
title: "日志记录"
---

# 📋 OpenClaw 日志与诊断完全指南

> **学习目标**：完成本章节学习后，你将能够深入理解 OpenClaw 的日志架构，掌握从基本日志配置到 OpenTelemetry 集成的完整知识体系，能够设计生产级的可观测性方案，并能够高效地诊断和排查各类问题。

---

## 为什么要学习日志与诊断系统

在深入技术细节之前，我们需要先理解**可观测性在分布式系统中的核心地位**。OpenClaw 作为一个连接多渠道、多模型的智能助手系统，其复杂性决定了可观测性不是「锦上添花」，而是「必不可少」。

### 可观测性的三支柱

现代可观测性体系建立在三个支柱之上：

```
┌─────────────────────────────────────────────────────────────────┐
│                     可观测性三支柱                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │    日志       │    │    指标      │    │    追踪      │     │
│   │  (Logs)      │    │ (Metrics)    │    │ (Traces)    │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│         │                  │                  │                │
│         └──────────────────┼──────────────────┘                │
│                            │                                   │
│                            ▼                                   │
│                   ┌─────────────────┐                          │
│                   │   OpenTelemetry │                          │
│                   │    统一标准      │                          │
│                   └─────────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| 支柱 | 作用 | OpenClaw 中的应用 |
|------|------|-------------------|
| **日志** | 记录离散事件 | 调试、审计、错误追踪 |
| **指标** | 量化数值（计数器、直方图） | 性能监控、容量规划 |
| **追踪** | 请求/操作的因果链 | 分布式问题诊断 |

### OpenClaw 日志系统的设计目标

OpenClaw 的日志系统遵循以下设计原则：

1. **双通道架构**：文件日志（持久化）+ 控制台日志（实时调试）
2. **渐进式详细度**：从 `error` 到 `trace`，适应不同场景
3. **开发者友好**：TTY 感知的格式化输出
4. **企业级扩展**：原生支持 OpenTelemetry 导出

---

## 认知负荷管理：学习路径设计

```
日志与诊断技能金字塔：

                    ┌─────────────────────────┐
                    │   专家级：可观测性架构   │  OTel 集成、指标设计、采样策略
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：诊断系统       │  诊断标志、OTel 导出、指标追踪
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：高级配置       │  日志脱敏、格式定制、过滤
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：基础日志       │  位置、读取、CLI 工具
                    └─────────────────────────┘
```

**建议学习路径**：
- 新手：从日志位置和 CLI 尾随开始
- 开发者：掌握日志配置和脱敏
- 运维人员：深入诊断系统和 OTel 集成
- 架构师：设计可观测性架构

---

## 第一部分：基础日志（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 OpenClaw 日志的位置和结构
- [ ] 使用 CLI 工具尾随实时日志
- [ ] 区分文件日志和控制台日志
- [ ] 配置基本的日志级别

### 1.1 日志位置与发现

**文件日志位置**：

OpenClaw 默认在以下位置写入滚动日志文件：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

**日志轮转策略**：

| 策略 | 说明 |
|------|------|
| 日期轮转 | 每天创建新日志文件 |
| 大小限制 | 单文件最大 100MB |
| 保留策略 | 保留最近 7 天的日志 |

**覆盖日志位置**：

```json5
{
  logging: {
    file: "/path/to/custom/openclaw.log"
  }
}
```

**目录结构**：

```
/tmp/openclaw/
├── openclaw-2024-01-15.log      # 历史日志
├── openclaw-2024-01-16.log      # 当前日志
├── openclaw-2024-01-17.log      # 今日日志
└── raw-stream.jsonl              # 原始流日志（如果启用）
```

### 1.2 使用 CLI 尾随日志

**推荐方法：实时尾随**：

```bash
# 实时尾随日志（推荐）
openclaw logs --follow

# 尾随并输出 JSON 格式
openclaw logs --follow --json

# 尾随并强制纯文本输出
openclaw logs --follow --plain
```

**输出模式详解**：

| 模式 | 命令 | 适用场景 |
|------|------|---------|
| TTY 彩色 | `openclaw logs --follow` | 开发调试 |
| 纯文本 | `openclaw logs --follow --plain` | 脚本处理 |
| JSON 格式 | `openclaw logs --follow --json` | 日志分析 |
| 脱敏 | `openclaw logs --follow --no-color` | 分享日志 |

**JSON 模式输出类型**：

```json
// 元数据类型 - 流元数据
{"type": "meta", "file": "/tmp/openclaw/openclaw.log", "cursor": 1024, "size": 2048}

// 日志类型 - 解析的日志条目
{"type": "log", "timestamp": "2024-01-15T10:30:00.000Z", "level": "info", "message": "Gateway started"}

// 通知类型 - 截断/轮换提示
{"type": "notice", "message": "Log file rotated"}

// 原始类型 - 未解析的行
{"type": "raw", "line": "2024-01-15T10:30:00.000Z [INFO]"}
```

### 1.3 渠道特定日志

**获取渠道日志**：

```bash
# 获取 WhatsApp 渠道日志
openclaw channels logs --channel whatsapp

# 获取 Telegram 渠道日志
openclaw channels logs --channel telegram

# 尾随特定渠道日志
openclaw channels logs --channel telegram --follow

# 获取所有渠道日志
openclaw channels logs
```

**渠道日志过滤**：

```bash
# 获取错误级别日志
openclaw channels logs --channel telegram --level error

# 获取最近 100 行
openclaw channels logs --channel telegram --lines 100
```

---

## 第二部分：中级日志配置（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 配置完整的日志系统参数
- [ ] 理解日志级别的详细语义
- [ ] 配置日志脱敏和格式
- [ ] 定制控制台输出样式

### 2.1 日志配置详解

**完整配置结构**：

```json5
{
  logging: {
    // 文件日志级别（持久化日志）
    level: "info",
    
    // 日志文件路径
    file: "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    
    // 控制台日志级别（实时输出）
    consoleLevel: "info",
    
    // 控制台样式
    consoleStyle: "pretty",
    
    // 敏感信息脱敏
    redactSensitive: "tools",
    
    // 自定义脱敏模式
    redactPatterns: ["sk-.*", "eyJ.*"]
  }
}
```

**日志级别语义**：

```
日志级别金字塔（详细 → 简洁）：

                    ┌─────────────────┐
                    │     trace       │  最详细：所有操作、函数调用
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │     debug       │  详细：变量值、请求响应
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │     info       │  正常：重要操作确认
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │     warn        │  警告：潜在问题
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │     error      │  错误：操作失败
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │     fatal       │  严重：系统不可用
                    └─────────────────┘
```

**级别使用指南**：

| 级别 | 何时使用 | 示例 |
|------|---------|------|
| `trace` | 调试底层问题 | 函数入口/出口、变量值 |
| `debug` | 开发调试 | 请求/响应内容 |
| `info` | 生产监控（默认） | 启动、关闭、重要操作 |
| `warn` | 潜在问题 | 超时重试、资源警告 |
| `error` | 已知问题 | 操作失败、异常捕获 |
| `fatal` | 系统级别问题 | 启动失败、配置错误 |

**控制台样式配置**：

```json5
{
  logging: {
    consoleStyle: "pretty"  // 人类友好、彩色、带时间戳
    // consoleStyle: "compact"  // 紧凑输出
    // consoleStyle: "json"    // JSON 格式（机器解析）
  }
}
```

**样式对比**：

```
pretty 模式：
┌────────────────────────────────────────────────────────────┐
│  [10:30:00] INFO  gateway          Gateway started        │
│  [10:30:01] INFO  channels/telegram Bot connected         │
│  [10:30:02] WARN  models/anthropic API rate limit         │
└────────────────────────────────────────────────────────────┘

compact 模式：
┌────────────────────────────────────────────────────────────┐
│ [INFO] Gateway started                                       │
│ [WARN] API rate limit (models/anthropic)                   │
└────────────────────────────────────────────────────────────┘

json 模式：
{"timestamp":"2024-01-15T10:30:00.000Z","level":"info","message":"Gateway started"}
```

### 2.2 敏感信息脱敏

**脱敏配置**：

```json5
{
  logging: {
    // 脱敏模式：off | tools（默认：仅工具输出）
    redactSensitive: "tools",
    
    // 自定义正则表达式模式
    redactPatterns: [
      "sk-[a-zA-Z0-9]{20,}",           // API 密钥
      "eyJ[a-zA-Z0-9_-]*\\.[a-zA-Z0-9_-]*",  // JWT
      "\\b\\d{16}\\b",                   // 信用卡号
      "password[\\s]*[:=][\\s]*\\S+"     // 密码泄露
    ]
  }
}
```

**脱敏工作原理**：

```
┌─────────────────────────────────────────────────────────┐
│               日志脱敏流程                                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. 原始日志条目                                          │
│     { "message": "API call with key sk-ant-12345..." }   │
│           ↓                                              │
│  2. 匹配脱敏模式                                         │
│     匹配到 "sk-[a-zA-Z0-9]{20,}"                         │
│           ↓                                              │
│  3. 替换为掩码                                           │
│     { "message": "API call with key [REDACTED]..." }    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**注意事项**：
- 脱敏仅影响**控制台输出**
- 文件日志保留原始内容
- 自定义模式需谨慎，避免误脱敏

---

## 第三部分：诊断系统（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解诊断系统与日志的区别
- [ ] 配置诊断标志进行定向调试
- [ ] 启用诊断事件收集
- [ ] 理解 OpenTelemetry 集成架构

### 3.1 诊断系统概述

**诊断 vs 日志**：

| 特性 | 日志 | 诊断 |
|------|------|------|
| 目的 | 事件记录 | 性能分析 |
| 格式 | 人类可读 | 机器可读 |
| 粒度 | 粗粒度 | 细粒度 |
| 导出 | 无 | 支持 OTel 导出 |
| 典型应用 | 调试、审计 | 性能监控、链路追踪 |

**诊断事件类型**：

```
┌─────────────────────────────────────────────────────────────────┐
│                    诊断事件分类                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  模型使用诊断                                                     │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ model.usage：token 消耗、成本、延迟、上下文                  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  消息流诊断                                                       │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ webhook.received/processed/error                            │ │
│  │ message.queued/processed                                    │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  队列与会话诊断                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ queue.lane.enforce/dequeue                                  │ │
│  │ session.state/stuck                                         │ │
│  │ run.attempt                                                 │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 诊断标志（定向调试）

**核心概念**：

诊断标志允许你打开**有针对性的调试日志**，而不提高全局日志级别。这对于排查特定组件问题非常有用。

**启用方式**：

```json5
// 配置文件
{
  diagnostics: {
    flags: ["telegram.http", "telegram.payload"]
  }
}
```

```bash
# 环境变量（一次性）
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload pnpm gateway:watch
```

**通配符支持**：

```json5
{
  diagnostics: {
    // 所有 telegram 相关
    flags: ["telegram.*"],
    
    // 所有 webhook 相关
    flags: ["webhook.*"],
    
    // 精确匹配
    flags: ["models.usage"]
  }
}
```

**常用标志列表**：

| 标志 | 作用范围 |
|------|---------|
| `telegram.*` | Telegram 渠道全部 |
| `telegram.http` | Telegram HTTP 请求 |
| `telegram.payload` | Telegram 消息载荷 |
| `whatsapp.*` | WhatsApp 渠道全部 |
| `webhook.*` | Webhook 处理 |
| `models.*` | 模型调用 |
| `models.usage` | 模型使用统计 |
| `gateway.*` | Gateway 核心 |
| `channels.*` | 所有渠道 |

**输出位置**：

- 标志日志进入**标准日志文件**
- 输出受 `logging.redactSensitive` 约束

### 3.3 OpenTelemetry 集成

**架构概述**：

```
┌─────────────────────────────────────────────────────────────────┐
│              OpenTelemetry 集成架构                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   OpenClaw Gateway                                              │
│   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐    │
│   │   日志 (Logs)   │ │   指标 (Metrics)│ │   追踪 (Traces) │    │
│   └────────┬────────┘ └────────┬────────┘ └────────┬────────┘    │
│            │                    │                    │              │
│            └────────────────────┼────────────────────┘              │
│                                 │                                   │
│                                 ▼                                   │
│                    ┌─────────────────────────┐                      │
│                    │     OTLP Exporter      │                      │
│                    │   (HTTP/Protobuf)      │                      │
│                    └───────────┬─────────────┘                      │
│                                │                                    │
│                                ▼                                    │
│                    ┌─────────────────────────┐                     │
│                    │   OpenTelemetry        │                     │
│                    │   Collector            │                     │
│                    └───────────┬─────────────┘                     │
│                                │                                    │
│              ┌─────────────────┼─────────────────┐                 │
│              ▼                 ▼                 ▼                 │
│       ┌───────────┐    ┌───────────┐    ┌───────────┐             │
│       │ Prometheus│    │   Jaeger  │    │  Grafana  │             │
│       │ (指标)    │    │ (追踪)    │    │ (可视化)  │             │
│       └───────────┘    └───────────┘    └───────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**配置示例**：

```json5
{
  // 首先启用诊断
  diagnostics: {
    enabled: true
  },
  
  // 启用 OTel 导出（需要 diagnostics-otel 插件）
  plugins: {
    allow: ["diagnostics-otel"],
    entries: {
      "diagnostics-otel": {
        enabled: true
      }
    }
  },
  
  // OTel 配置
  diagnostics: {
    enabled: true,
    otel: {
      // 是否启用
      enabled: true,
      
      // 收集器端点
      endpoint: "http://otel-collector:4318",
      
      // 协议：目前仅支持 http/protobuf
      protocol: "http/protobuf",
      
      // 服务名称
      serviceName: "openclaw-gateway",
      
      // 导出信号
      traces: true,
      metrics: true,
      logs: true,
      
      // 采样率（0.0-1.0，仅根 span）
      sampleRate: 0.2,
      
      // 导出间隔（毫秒）
      flushIntervalMs: 60000
    }
  }
}
```

**环境变量覆盖**：

```bash
# 端点覆盖
export OTEL_EXPORTER_OTLP_ENDPOINT="http://collector:4318"

# 服务名称覆盖
export OTEL_SERVICE_NAME="openclaw-production"

# 协议覆盖
export OTEL_EXPORTER_OTLP_PROTOCOL="http/protobuf"
```

### 3.4 导出的指标详情

**模型使用指标**：

| 指标名称 | 类型 | 描述 | 标签 |
|----------|------|------|------|
| `openclaw.tokens` | 计数器 | Token 消耗 | channel, provider, model |
| `openclaw.cost.usd` | 计数器 | 成本（美元） | channel, provider, model |
| `openclaw.run.duration_ms` | 直方图 | 运行耗时 | channel, provider, model |
| `openclaw.context.tokens` | 直方图 | 上下文 Token | context, channel, provider, model |

**消息流指标**：

| 指标名称 | 类型 | 描述 | 标签 |
|----------|------|------|------|
| `openclaw.webhook.received` | 计数器 | Webhook 接收数 | channel, webhook |
| `openclaw.webhook.error` | 计数器 | Webhook 错误数 | channel, webhook |
| `openclaw.webhook.duration_ms` | 直方图 | Webhook 处理耗时 | channel, webhook |
| `openclaw.message.queued` | 计数器 | 排队消息数 | channel, source |
| `openclaw.message.processed` | 计数器 | 处理消息数 | channel, outcome |
| `openclaw.message.duration_ms` | 直方图 | 消息处理耗时 | channel, outcome |

**队列与会话指标**：

| 指标名称 | 类型 | 描述 | 标签 |
|----------|------|------|------|
| `openclaw.queue.lane.enqueue` | 计数器 | 入队操作 | lane |
| `openclaw.queue.lane.dequeue` | 计数器 | 出队操作 | lane |
| `openclaw.queue.depth` | 直方图 | 队列深度 | lane, channel=heartbeat |
| `openclaw.queue.wait_ms` | 直方图 | 等待时间 | lane |
| `openclaw.session.state` | 计数器 | 会话状态变化 | state, reason |
| `openclaw.session.stuck` | 计数器 | 会话卡住 | state |

### 3.5 导出的追踪 Span

**核心 Span**：

| Span 名称 | 关键属性 |
|-----------|---------|
| `openclaw.model.usage` | channel, provider, model, sessionKey, sessionId, tokens.* |
| `openclaw.webhook.processed` | channel, webhook, chatId |
| `openclaw.webhook.error` | channel, webhook, chatId, error |
| `openclaw.message.processed` | channel, outcome, chatId, messageId, sessionKey, sessionId, reason |
| `openclaw.session.stuck` | state, ageMs, queueDepth, sessionKey, sessionId |

---

## 第四部分：专家级可观测性架构（⭐⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 设计生产级的可观测性方案
- [ ] 优化采样策略平衡开销与精度
- [ ] 构建告警规则和仪表盘
- [ ] 实施分布式追踪的最佳实践

### 4.1 采样策略设计

**采样配置**：

```json5
{
  diagnostics: {
    otel: {
      // 追踪采样率（0.0 - 1.0）
      sampleRate: 0.2,
      
      // 指标导出间隔（毫秒）
      flushIntervalMs: 60000
    }
  }
}
```

**采样策略矩阵**：

| 环境 | 采样率 | 理由 |
|------|--------|------|
| 开发 | 1.0 (100%) | 需要完整追踪调试 |
| 测试 | 0.5 | 平衡开销与覆盖率 |
| Staging | 0.2 | 接近生产配置 |
| Production | 0.05 - 0.1 | 低开销，关注关键路径 |

**自适应采样**：

```typescript
// 自适应采样策略示例（伪代码）
function shouldSample(span: Span): boolean {
  // 始终采样错误请求
  if (span.hasError()) return true;
  
  // 高价值用户请求提高采样率
  if (span.isPremiumUser()) return true;
  
  // 随机采样
  return Math.random() < 0.1;
}
```

### 4.2 告警规则设计

**关键告警指标**：

```yaml
# alerting rules 示例
groups:
  - name: openclaw_alerts
    rules:
      # 错误率告警
      - alert: HighErrorRate
        expr: |
          rate(openclaw_message_processed{outcome="error"}[5m]) /
          rate(openclaw_message_processed[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "错误率超过 5%"
          
      # 会话卡住告警
      - alert: SessionStuck
        expr: increase(openclaw_session_stuck[10m]) > 0
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "检测到卡住的会话"
          
      # 队列深度告警
      - alert: QueueDepthHigh
        expr: openclaw_queue_depth{lane="default"} > 100
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "队列深度超过 100"
```

### 4.3 仪表盘设计

**推荐仪表盘组件**：

```
┌─────────────────────────────────────────────────────────────────┐
│                     OpenClaw 运维仪表盘                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────┐ ┌─────────────────────┐                │
│  │   消息吞吐量         │ │   错误率趋势         │                │
│  │   [图表]            │ │   [图表]            │                │
│  └─────────────────────┘ └─────────────────────┘                │
│                                                                  │
│  ┌─────────────────────┐ ┌─────────────────────┐                │
│  │   Token 消耗        │ │   模型延迟分布       │                │
│  │   [图表]            │ │   [直方图]          │                │
│  └─────────────────────┘ └─────────────────────┘                │
│                                                                  │
│  ┌─────────────────────┐ ┌─────────────────────┐                │
│  │   渠道状态          │ │   队列深度          │                │
│  │   [状态面板]        │ │   [图表]            │                │
│  └─────────────────────┘ └─────────────────────┘                │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │   最近错误日志（实时）                                      │     │
│  │   [滚动日志面板]                                          │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 第五部分：故障排查速查

### 快速诊断命令

```bash
# Gateway 不可达
openclaw doctor

# 日志为空 - 检查日志路径
openclaw config get logging.file
ls -la $(openclaw config get logging.file)

# 需要更多细节 - 提高日志级别
openclaw config set logging.level debug
openclaw gateway restart

# 诊断特定渠道
openclaw channels logs --channel telegram --level debug

# 启用诊断标志
OPENCLAW_DIAGNOSTICS=telegram.http pnpm gateway:watch
```

### 常见问题与解决方案

**问题一：日志文件为空**

**排查步骤**：

```bash
# 1. 检查 Gateway 是否运行
ps aux | grep openclaw

# 2. 检查日志路径权限
ls -la /tmp/openclaw/

# 3. 检查配置中的日志级别
openclaw config get logging.level

# 4. 手动测试日志写入
touch /tmp/openclaw/test.log
chmod 666 /tmp/openclaw/test.log
```

**问题二：控制台无输出**

**排查步骤**：

```bash
# 1. 检查控制台日志级别
openclaw config get logging.consoleLevel

# 2. 强制控制台输出
openclaw logs --follow --plain --no-color

# 3. 检查终端类型
echo $TERM
```

**问题三：OTel 导出失败**

**排查步骤**：

```bash
# 1. 检查端点可访问性
curl http://otel-collector:4318/v1/traces

# 2. 检查认证头（如果需要）
curl -H "Authorization: Bearer $TOKEN" http://otel-collector:4318/v1/traces

# 3. 验证 OTel 配置
openclaw config get diagnostics.otel
```

---

## 练习与自检

### 基础技能自检

- [ ] 能够找到并读取日志文件
- [ ] 能够使用 CLI 尾随实时日志
- [ ] 能够配置日志级别
- [ ] 理解日志级别的含义

### 进阶技能自检

- [ ] 能够配置日志脱敏
- [ ] 掌握渠道日志过滤
- [ ] 能够启用诊断标志
- [ ] 理解诊断与日志的区别

### 专家技能自检

- [ ] 能够配置 OpenTelemetry 集成
- [ ] 理解采样策略的设计
- [ ] 能够设计告警规则
- [ ] 掌握分布式追踪的最佳实践

---

## 参考资料

### 核心文档

- [调试指南](/zh-CN/debugging) - 调试工具详解
- [诊断标志](/zh-CN/diagnostics/flags) - 定向调试
- [OpenTelemetry 官方文档](https://opentelemetry.io/docs/)

### 相关命令速查

```bash
# 日志命令
openclaw logs --follow        # 尾随日志
openclaw logs --json          # JSON 输出
openclaw logs --lines 100     # 最近 100 行

# 渠道日志
openclaw channels logs --channel <name>

# 诊断命令
openclaw doctor               # 完整诊断

# 配置命令
openclaw config get logging   # 查看日志配置
openclaw config set logging.level=debug  # 调试级别
```

### 扩展阅读

- [OpenTelemetry 概念](https://opentelemetry.io/docs/concepts/)
- [PromQL 查询语言](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana 仪表盘](https://grafana.com/docs/grafana/latest/dashboards/)
