---
summary: "OpenClaw 日志系统架构：滚动诊断文件日志 + macOS 统一日志隐私配置指南"
read_when:
  - 捕获 macOS 日志或调查私有数据日志问题
  - 调试语音唤醒/会话生命周期问题
  - 配置详细日志输出
  - 理解 macOS 统一日志隐私机制
title: "macOS 日志系统"
---

# macOS 日志系统配置指南

本章节深入解析 OpenClaw 在 macOS 平台上的日志系统架构，涵盖 swift-log 日志路由、滚动诊断文件日志配置、macOS 统一日志隐私机制以及日志调试最佳实践。通过学习本章节，你将掌握 macOS 应用日志的完整技术栈，能够高效捕获诊断信息并保护敏感数据隐私。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 **macOS 日志系统**的架构组成（统一日志 + 文件日志）
- [ ] 配置滚动诊断文件日志并解读日志格式
- [ ] 掌握日志隐私标志的作用与配置方法
- [ ] 使用 clawlog.sh 脚本高效查询日志

### 进阶目标（建议掌握）

- [ ] 理解 **swift-log** 日志框架的路由机制与日志级别
- [ ] 配置日志轮转策略以管理磁盘空间
- [ ] 设计日志过滤规则聚焦特定模块
- [ ] 分析日志模式识别系统问题

### 专家目标（挑战）

- [ ] 自定义日志格式与输出目标
- [ ] 构建日志聚合分析流水线
- [ ] 实现实时日志监控与告警

---

## 第一部分：日志系统架构

### macOS 日志生态概览

macOS 采用 **统一日志系统（Unified Logging）** 作为系统级日志基础设施，替代了传统的 syslog 与 ASL（Apple System Log）。OpenClaw 充分利用这一系统，同时提供文件日志作为补充。

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw 日志架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Swift 应用层                            │   │
│   │              (swift-log 框架)                        │   │
│   │   log.info("User message received")                 │   │
│   │   log.error("Connection failed")                    │   │
│   └────────────────────────┬────────────────────────────┘   │
│                            │                                  │
│                 ┌──────────┴──────────┐                    │
│                 ▼                      ▼                     │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │   统一日志输出      │    │   滚动文件日志      │       │
│   │  logd 系统守护进程  │    │  diagnostics.jsonl │       │
│   │                     │    │                     │       │
│   │  • 系统级日志聚合   │    │  • JSONL 格式      │       │
│   │  • log 命令查询    │    │  • 本地持久化      │       │
│   │  • 隐私自动处理    │    │  • 轮转管理        │       │
│   └─────────────────────┘    └─────────────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 日志级别语义

OpenClaw 使用标准日志级别，语义如下：

| 级别 | 数值 | 语义 | 使用场景 |
|------|------|------|---------|
| **TRACE** | 0 | 最详细 | 调试时记录所有执行路径 |
| **DEBUG** | 1 | 详细 | 开发时记录变量值与流程 |
| **INFO** | 2 | 信息 | 正常运行事件（如连接建立） |
| **WARN** | 3 | 警告 | 潜在问题（重试、降级） |
| **ERROR** | 4 | 错误 | 操作失败但应用可继续 |
| **CRITICAL** | 5 | 严重 | 致命错误，应用可能崩溃 |

**swift-log 默认级别**：INFO

### 日志组件

| 组件 | 功能 | 输出目标 |
|------|------|----------|
| **Swift Logger** | 应用内日志 API | 统一日志 + 文件日志 |
| **logd** | macOS 系统日志守护进程 | 统一日志系统 |
| **diagnostics.jsonl** | 滚动诊断文件 | 本地文件系统 |

---

## 第二部分：诊断文件日志配置

### 滚动诊断日志概述

OpenClaw 支持将日志写入滚动诊断文件，适合需要持久化捕获或离线分析的场境：

**启用方式**：

```
调试面板 → Logs → App logging → "Write rolling diagnostics log (JSONL)"
```

**日志位置**：`~/Library/Logs/OpenClaw/diagnostics.jsonl`

**日志格式**：

```jsonl
{"level":"INFO","timestamp":"2026-02-05T14:30:00.123Z","message":"Gateway started","subsystem":"bot.molt","category":"gateway"}
{"level":"DEBUG","timestamp":"2026-02-05T14:30:01.456Z","message":"WebSocket connected","subsystem":"bot.molt","category":"websocket"}
{"level":"ERROR","timestamp":"2026-02-05T14:30:05.789Z","message":"Connection failed","subsystem":"bot.molt","category":"network","error":"ECONNREFUSED"}
```

**日志轮转规则**：

| 轮转条件 | 行为 |
|---------|------|
| 文件大小 > 10MB | 重命名 `.1`，创建新文件 |
| 最多保留 5 个历史文件 | 超出后删除最旧文件 |
| 应用重启 | 无特殊处理 |

**轮转后文件**：`diagnostics.jsonl.1`、`diagnostics.jsonl.2` ...

### 日志详细程度配置

通过调试面板配置日志详细程度：

```
调试面板 → Logs → App logging → Verbosity
```

**详细程度选项**：

| 级别 | 输出内容 |
|------|---------|
| **Default** | INFO 及以上级别 |
| **Debug** | DEBUG 及以上级别 |
| **Trace** | 所有级别（包括 TRACE） |

**对应 swift-log 级别**：

```
Verbose  ──────────────────────────────────────→ Minimal

TRACE  DEBUG  INFO  WARN  ERROR  CRITICAL

┌─────────────────────────────────────────────────────┐
│  • Default: INFO, WARN, ERROR, CRITICAL             │
│  • Debug:   DEBUG, INFO, WARN, ERROR, CRITICAL      │
│  • Trace:   TRACE, DEBUG, INFO, WARN, ERROR, CRITICAL│
└─────────────────────────────────────────────────────┘
```

### 日志文件管理

**查看日志**：

```bash
# 实时跟踪日志
tail -f ~/Library/Logs/OpenClaw/diagnostics.jsonl

# 查看最近 100 行
tail -n 100 ~/Library/Logs/OpenClaw/diagnostics.jsonl

# 查看历史日志
cat ~/Library/Logs/OpenClaw/diagnostics.jsonl.1
```

**日志分析**：

```bash
# 统计错误数量
grep '"level":"ERROR"' ~/Library/Logs/OpenClaw/diagnostics.jsonl | wc -l

# 筛选特定模块日志
grep '"category":"gateway"' ~/Library/Logs/OpenClaw/diagnostics.jsonl

# 提取时间戳
grep -oP '"timestamp":"\K[^"]+' ~/Library/Logs/OpenClaw/diagnostics.jsonl
```

**日志清除**：

```
调试面板 → Logs → App logging → "Clear"
```

或命令行：

```bash
# 清空当前日志
> ~/Library/Logs/OpenClaw/diagnostics.jsonl

# 删除历史日志
rm ~/Library/Logs/OpenClaw/diagnostics.jsonl.*
```

---

## 第三部分：macOS 统一日志隐私机制

### 隐私保护原理

macOS 统一日志系统内置**隐私保护机制**，默认隐藏敏感数据：

```
┌─────────────────────────────────────────────────────────────┐
│              统一日志隐私处理示例                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  原始日志内容：                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Received message from +15551234567: "Hello, world!"  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  日志输出（默认隐私模式）：                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Received message from <private>: "<private>"         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  日志输出（隐私关闭后）：                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Received message from +15551234567: "Hello, world!"  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**受保护数据类型**：

| 数据类型 | 示例 | 默认显示 |
|---------|------|---------|
| 电话号码 | `+15551234567` | `<private>` |
| 邮箱地址 | `user@example.com` | `<private>` |
| 消息正文 | `"Hello"` | `<private>` |
| 文件路径 | `/Users/username/...` | `<private>` |
| IP 地址 | `192.168.1.1` | `<private>` |

### 为 OpenClaw 启用私有数据

当需要调试敏感数据相关问题时，需配置子系统级隐私标志：

**配置步骤**：

```bash
# 1. 创建 plist 配置
cat > /tmp/bot.molt.plist <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF

# 2. 以 root 身份安装配置
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist \
    /Library/Preferences/Logging/Subsystems/bot.molt.plist

# 3. 验证安装
cat /Library/Preferences/Logging/Subsystems/bot.molt.plist

# 4. 重新触发要调试的事件
#（只有新日志行会包含私有数据）
```

**配置说明**：

| 步骤 | 说明 |
|------|------|
| 创建 plist | 定义子系统级别的日志选项 |
| root 安装 | 需要管理员权限修改系统配置 |
| 事件触发 | 已有日志行不会改变，只有新事件会应用新配置 |

### 调试后禁用隐私配置

完成调试后，应及时禁用私有数据以保护隐私：

```bash
# 移除覆盖配置
sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist

# 可选：强制 logd 立即刷新配置
sudo log config --reload

# 验证已移除
log config status | grep bot.molt
```

**安全提醒**：

> ⚠️ 启用私有数据后，日志可能包含电话号码、消息内容等敏感信息。仅在调试需要时启用，调试完成后立即禁用。分享日志前务必审查内容。

---

## 第四部分：clawlog.sh 脚本使用指南

### 脚本功能概述

OpenClaw 提供 `clawlog.sh` 脚本简化日志查询：

**脚本位置**：`scripts/clawlog.sh`

**基本用法**：

```bash
# 查看最近日志
./scripts/clawlog.sh --last 5m

# 实时跟踪日志
./scripts/clawlog.sh --follow

# 过滤特定类别
./scripts/clawlog.sh --category WebChat
./scripts/clawlog.sh --category Gateway
./scripts/clawlog.sh --category VoiceWake
```

### 命令选项详解

| 选项 | 说明 | 示例 |
|------|------|------|
| `--last <duration>` | 显示最近时段的日志 | `--last 10m`、`--last 1h` |
| `--follow` | 实时跟踪日志（Ctrl+C 退出） | `--follow --last 5m` |
| `--category <name>` | 过滤日志类别 | `--category WebChat` |
| `--predicate <query>` | 自定义日志过滤 | `--predicate 'eventMessage CONTAINS "error"'` |
| `--level <level>` | 按级别过滤 | `--level error` |

**类别过滤选项**：

| 类别 | 说明 | 典型用途 |
|------|------|----------|
| **WebChat** | WebChat 界面日志 | 调试 WebSocket 连接问题 |
| **Gateway** | 网关服务日志 | 诊断连接与消息路由问题 |
| **VoiceWake** | 语音唤醒日志 | 调试语音识别与触发问题 |
| **Canvas** | Canvas 面板日志 | 排查界面渲染问题 |
| **TCC** | 权限相关日志 | 调试权限拒绝问题 |

### 日志查询示例

**示例一：查询 WebChat 问题**

```bash
# 查看最近 WebChat 相关日志
./scripts/clawlog.sh --category WebChat --last 10m

# 实时跟踪 WebChat 日志
./scripts/clawlog.sh --category WebChat --follow
```

**示例二：查询错误日志**

```bash
# 查看所有错误日志（最近 1 小时）
./scripts/clawlog.sh --last 1h | grep -i error

# 实时跟踪错误日志
./scripts/clawlog.sh --follow --last 5m | grep -i error
```

**示例三：查询特定时间段**

```bash
# 查看今天的所有日志
./scripts/clawlog.sh --last 24h

# 查看指定时间范围
log show --start "2026-02-05 10:00:00" --end "2026-02-05 12:00:00"
```

---

## 第五部分：调试场景指南

### 场景一：调试语音唤醒问题

**症状**：唤醒词无法识别、识别延迟或误触发

**日志配置**：

1. 启用详细日志（Verbose → Trace）
2. 启用滚动诊断文件日志
3. 为 bot.molt 子系统启用私有数据

**关键日志标记**：

```bash
# 语音唤醒相关日志
./scripts/clawlog.sh --category VoiceWake --last 5m

# 语音识别结果
log show --predicate 'eventMessage CONTAINS "transcript"' --last 5m

# 唤醒触发事件
log show --predicate 'eventMessage CONTAINS "trigger"' --last 5m
```

**分析要点**：

```
语音识别流程日志：

┌─────────────────────────────────────────────────────────────┐
│  音频输入 → VAD → ASR → NLU → 意图匹配 → 动作执行          │
│                                                              │
│  关键日志点：                                                │
│  ├── AUDIO_CAPTURED：捕获音频数据                           │
│  ├── TRANSCRIPT：语音识别文本结果                           │
│  ├── WAKEWORD_DETECTED：唤醒词匹配                          │
│  ├── INTENT_PARSED：解析用户意图                            │
│  └── ACTION_EXECUTED：执行动作                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 场景二：追踪消息流

**症状**：消息丢失、延迟或重复

**日志配置**：

1. 确保日志级别至少为 DEBUG
2. 启用私有数据查看消息内容

**关键日志标记**：

```bash
# 消息路由日志
./scripts/clawlog.sh --category Gateway --last 5m | grep -i "message\|route"

# WebSocket 消息日志
log show --predicate 'eventMessage CONTAINS "ws" OR eventMessage CONTAINS "websocket"' --last 5m

# 渠道连接日志
log show --predicate 'eventMessage CONTAINS "channel" OR eventMessage CONTAINS "connect"' --last 5m
```

**消息流追踪点**：

```
消息处理流程日志：

┌─────────────────────────────────────────────────────────────┐
│  消息接收 → 预处理 → AI 处理 → 响应生成 → 消息发送           │
│                                                              │
│  关键日志点：                                                │
│  ├── MESSAGE_RECEIVED：原始消息入站                          │
│  ├── MESSAGE_NORMALIZED：消息标准化                          │
│  ├── AI_REQUEST_SENT：发送给 AI 处理                         │
│  ├── AI_RESPONSE_RECEIVED：AI 响应返回                       │
│  └── MESSAGE_SENT：消息发送成功                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 场景三：WebChat 问题诊断

**症状**：WebChat 无法连接、界面空白或消息无法发送

**日志配置**：

1. 启用详细日志
2. 过滤 WebChat 类别

**关键日志标记**：

```bash
# WebSocket 连接日志
./scripts/clawlog.sh --category WebChat --last 5m | grep -i "websocket\|connect"

# UI 渲染日志
./scripts/clawlog.sh --category WebChat --last 5m | grep -i "render\|ui"

# 消息发送日志
./scripts/clawlog.sh --category WebChat --last 5m | grep -i "send\|message"
```

**诊断流程**：

```
WebChat 问题排查清单：

1. 检查网关状态
   └── openclaw gateway status

2. 检查 WebSocket 端口
   └── lsof -nP -iTCP:18789 -sTCP:LISTEN

3. 检查 WebChat 日志
   └── ./scripts/clawlog.sh --category WebChat --last 5m

4. 浏览器控制台（F12）
   └── 查看 WebSocket 连接状态
```

### 场景四：会话生命周期问题

**症状**：会话创建失败、会话丢失或状态不一致

**日志配置**：

1. 启用详细日志
2. 包含会话 ID 过滤

**关键日志标记**：

```bash
# 会话管理日志
./scripts/clawlog.sh --category Gateway --last 5m | grep -i "session"

# 会话状态日志
log show --predicate 'eventMessage CONTAINS "session" OR eventMessage CONTAINS "session"' --last 5m
```

---

## 第六部分：故障排查指南

### 日志相关问题

**问题一：日志仍显示 `<private>`**

**原因**：plib 配置未正确安装，或仅修改了配置但未触发新日志

**解决方案**：

```bash
# 1. 验证 plist 文件存在
cat /Library/Preferences/Logging/Subsystems/bot.molt.plist

# 2. 如果不存在，重新安装
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist \
    /Library/Preferences/Logging/Subsystems/bot.molt.plist

# 3. 触发新日志事件
#（运行要调试的操作）

# 4. 重新查询日志
./scripts/clawlog.sh --last 5m
```

**问题二：日志文件增长过快**

**原因**：日志级别设置过高，或存在大量错误/调试输出

**解决方案**：

```bash
# 1. 降低日志详细程度
# 调试面板 → Logs → App logging → Verbosity → Default

# 2. 禁用滚动日志（如果不需要持久化）
# 调试面板 → Logs → App logging → 取消勾选 "Write rolling diagnostics log"

# 3. 清理已有日志
./scripts/clawlog.sh --last 5m > /tmp/keep.log
> ~/Library/Logs/OpenClaw/diagnostics.jsonl
cat /tmp/keep.log >> ~/Library/Logs/OpenClaw/diagnostics.jsonl
```

**问题三：clawlog.sh 找不到**

**原因**：脚本不在 PATH 中，或工作目录不对

**解决方案**：

```bash
# 1. 使用完整路径
/Volumes/mini_matrix/github/a1pha3/openclaw/scripts/clawlog.sh --last 5m

# 2. 或添加到 PATH
export PATH="/Volumes/mini_matrix/github/a1pha3/openclaw/scripts:$PATH"
clawlog.sh --last 5m

# 3. 检查脚本是否存在
ls -la scripts/clawlog.sh
```

**问题四：权限被拒绝**

**原因**：访问系统日志需要适当权限

**解决方案**：

```bash
# 1. 使用 sudo（仅限系统日志）
sudo log show --predicate 'subsystem == "bot.molt"' --last 5m

# 2. 确保文件权限正确
chmod 644 ~/Library/Logs/OpenClaw/diagnostics.jsonl

# 3. 检查用户权限
id
```

---

## 适用场景与最佳实践

### 日志级别使用建议

| 场景 | 推荐日志级别 | 说明 |
|------|-------------|------|
| **生产环境** | INFO | 记录正常操作事件 |
| **问题诊断** | DEBUG | 捕获详细执行信息 |
| **性能分析** | DEBUG + 性能标记 | 识别瓶颈 |
| **安全审计** | INFO（不记录敏感） | 合规与审计 |
| **持续集成** | WARN + ERROR | CI 通过/失败判断 |

### 日志管理最佳实践

**日志保留策略**：

| 类型 | 保留期限 | 说明 |
|------|---------|------|
| **详细日志** | 7 天 | 用于问题诊断 |
| **错误日志** | 30 天 | 用于趋势分析 |
| **审计日志** | 1 年 | 用于合规要求 |

**日志安全**：

- [ ] 定期审查日志内容，确保无敏感数据泄露
- [ ] 共享日志前脱敏处理（电话号码、令牌等）
- [ ] 限制日志文件访问权限
- [ ] 自动化日志清理任务

---

## 章节总结

本章节全面介绍了 OpenClaw macOS 日志系统的架构与配置。通过学习，你应该已经掌握：

1. **日志架构**：理解统一日志与文件日志的双层架构
2. **诊断配置**：配置滚动日志与详细程度
3. **隐私机制**：理解并配置 macOS 统一日志隐私保护
4. **调试技巧**：使用 clawlog.sh 高效查询日志
5. **场景应用**：针对常见问题的日志分析方法

后续建议结合 [权限配置](/zh-cn/platforms/mac/permissions) 与 [故障排查](/zh-cn/platforms/mac/health) 章节，完善系统运维知识体系。

相关文档：[日志配置参考](/zh-cn/logging)
