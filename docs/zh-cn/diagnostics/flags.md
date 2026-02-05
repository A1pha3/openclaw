---
summary: OpenClaw 诊断标志系统详解——如何启用定向调试日志而不提升全局日志级别
read_when:
  - 需要在不提高全局日志级别的情况下获取定向调试日志
  - 需要捕获子系统特定的日志以获取支持信息
  - 想要理解诊断标志的工作原理和最佳实践
title: "诊断标志完全指南"
---

# 诊断标志完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解诊断标志的设计理念和核心价值
- 掌握通过配置和环境变量启用诊断标志的方法
- 能够提取和过滤特定子系统的诊断日志

### 进阶目标（建议掌握）

- 理解通配符匹配机制和标志命名规范
- 能够在实时环境中诊断复杂问题
- 设计团队内部的诊断标志使用规范

### 专家目标（挑战）

- 为新子系统设计诊断标志规范
- 构建自动化的诊断日志分析流程
- 优化诊断标志对性能的影响

## 核心概念与设计理念

### 为什么需要诊断标志？

在大型系统中，日志是诊断问题的生命线。然而，全局开启 DEBUG 或 TRACE 级别日志会导致：

- **日志膨胀**：每秒产生数百万条日志，磁盘快速耗尽
- **性能损耗**：日志输出本身成为性能瓶颈
- **信息过载**：真正有用的信息淹没在噪声中
- **排查困难**：难以定位特定子系统的问题

**诊断标志的设计哲学：**

> 「按需获取，精准定位」——只在你关心的子系统上开启调试日志，其他部分保持安静。

### 诊断标志 vs 全局日志级别

| 对比维度 | 全局日志级别 | 诊断标志 |
|----------|--------------|----------|
| 控制粒度 | 整个系统 | 单个子系统 |
| 性能影响 | 全局影响 | 仅影响目标子系统 |
| 配置灵活性 | 二元开关 | 多维度组合 |
| 学习曲线 | 简单 | 需要了解标志命名 |
| 适用场景 | 全局问题排查 | 定向问题定位 |

## 工作原理深度解析

### 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                        OpenClaw 核心                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ 子系统 A    │  │ 子系统 B    │  │ 子系统 C    │            │
│  │              │  │              │  │              │            │
│  │ 检查 flag:  │  │ 检查 flag:  │  │ 检查 flag:  │            │
│  │ "telegram.*" │  │ "gateway.*" │  │ "database.*" │            │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘            │
│         │                │                │                     │
│         └────────────────┼────────────────┘                     │
│                          ▼                                      │
│              ┌─────────────────────┐                           │
│              │   诊断标志过滤器     │                           │
│              │   (DiagnosticFilter) │                           │
│              └──────────┬──────────┘                           │
│                         │                                       │
│                         ▼                                       │
│              ┌─────────────────────┐                           │
│              │  诊断日志文件输出    │                           │
│              │  (JSONL 格式)        │                           │
│              └─────────────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

### 标志匹配机制

**命名规范：**

诊断标志使用点分命名法，格式为 `<子系统>.<模块>[.<子模块>]`，例如：

| 标志示例 | 匹配范围 | 说明 |
|----------|----------|------|
| `telegram` | 仅 `telegram` | 精确匹配单一子系统 |
| `telegram.*` | `telegram` 下所有模块 | 通配符匹配 |
| `telegram.http` | `telegram.http` | 精确到具体模块 |
| `telegram.http.error` | `telegram.http.error` | 精确到子模块 |
| `gateway.*` | `gateway` 下所有模块 | 网关相关全部 |

**匹配算法：**

```
function matchFlag(flag: string, logSource: string): boolean {
  // 1. 精确匹配
  if (flag === logSource) return true;

  // 2. 通配符匹配（仅支持后缀通配符）
  if (flag.endsWith('.*')) {
    const prefix = flag.slice(0, -2); // 移除 ".*"
    return logSource.startsWith(prefix + '.') || logSource === prefix;
  }

  return false;
}
```

### 标志生命周期

```
┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
│   配置读取   │───→│   标志注册   │───→│   日志过滤   │───→│   日志写入   │
└────────────┘    └────────────┘    └────────────┘    └────────────┘
     │                 │                 │                 │
     ▼                 ▼                 ▼                 ▼
  JSON 解析        内存索引建立      按需匹配         JSONL 输出
  环境变量覆盖     全局 Map 存储    条件判断
```

## 配置方法详解

### 方法一：通过配置文件启用

这是最常用的方法，适合长期使用的配置。

**单标志配置：**

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

**多标志配置：**

```json
{
  "diagnostics": {
    "flags": [
      "telegram.http",
      "telegram.payload",
      "gateway.ws",
      "gateway.auth"
    ]
  }
}
```

**通配符配置：**

```json
{
  "diagnostics": {
    "flags": [
      "telegram.*",
      "gateway.*",
      "database.*"
    ]
  }
}
```

**配置生效流程：**

1. 网关启动时读取配置文件
2. 解析 `diagnostics.flags` 数组
3. 建立标志到子系统的映射
4. 后续所有日志调用都会经过过滤器检查

### 方法二：环境变量覆盖（临时使用）

适合一次性诊断场景，无需修改配置文件。

**启用特定标志：**

```bash
# Linux/macOS
export OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
openclaw gateway run
```

**Windows (PowerShell)：**
```powershell
$env:OPENCLAW_DIAGNOSTICS="telegram.http,telegram.payload"
openclaw gateway run
```

**禁用所有标志：**

```bash
# 环境变量设置为 0 或空字符串
OPENCLAW_DIAGNOSTICS=0 openclaw gateway run
```

**优先级规则：**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | 环境变量 | 临时使用，覆盖配置文件 |
| 2 | 配置文件 | 持久化配置 |
| 3 | 默认值 | 无配置时使用 |

### 组合使用场景

**场景：临时诊断特定问题**

```bash
# 诊断 Telegram 连接问题（临时）
OPENCLAW_DIAGNOSTICS=telegram.* openclaw gateway run
```

**场景：完整问题诊断（生产环境）**

```json
{
  "diagnostics": {
    "flags": [
      "telegram.*",
      "gateway.websocket",
      "gateway.auth",
      "channels.ingress"
    ]
  }
}
```

## 日志输出与提取

### 输出位置

诊断日志输出到专用的诊断日志文件，而不是标准日志流。

**默认位置：**

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

**自定义位置：**

如果配置了 `logging.file`，诊断日志会使用该路径：

```json
{
  "logging": {
    "file": "/var/log/openclaw/openclaw.log"
  },
  "diagnostics": {
    "flags": ["telegram.*"]
  }
}
```

**日志格式：**

每条诊断日志都是独立的 JSONL（JSON Lines）格式：

```json
{"timestamp":"2026-02-05T10:30:00Z","level":"debug","flag":"telegram.http","source":"src/channels/telegram/http.ts:45","message":"Outgoing request","method":"POST","url":"https://api.telegram.org/sendMessage","status":200}
{"timestamp":"2026-02-05T10:30:01Z","level":"debug","flag":"telegram.http","source":"src/channels/telegram/http.ts:47","message":"Incoming response","body":{"ok":true,"result":{"message_id":123}}}
```

**字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| timestamp | 字符串 | ISO 8601 时间戳 |
| level | 字符串 | 日志级别（debug/info/warn/error） |
| flag | 字符串 | 触发此日志的诊断标志 |
| source | 字符串 | 代码位置（文件:行号） |
| message | 字符串 | 日志消息 |
| ... | 任意 | 其他上下文字段 |

### 日志提取技巧

**获取最新日志文件：**

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

**过滤特定标志的日志：**

```bash
# 使用 ripgrep 过滤
rg '"flag":"telegram.http"' /tmp/openclaw/openclaw-*.log

# 使用 jq 提取并格式化
jq 'select(.flag == "telegram.http")' /tmp/openclaw/openclaw-*.log
```

**实时查看诊断日志：**

```bash
# 实时查看特定标志
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http"

# 查看所有诊断日志
tail -f /tmp/openclaw/openclaw-$(date +%F).log | jq '.'
```

**远程网关日志获取：**

对于运行在远程服务器上的网关：

```bash
# 使用 OpenClaw CLI 获取日志
openclaw logs --follow --level debug

# 过滤特定标志
openclaw logs --follow | jq 'select(.flag == "telegram.http")'
```

### 敏感信息处理

诊断日志遵循 `logging.redactSensitive` 设置，敏感信息会自动脱敏：

```json
{
  "logging": {
    "redactSensitive": true,
    "patterns": [
      "api_key",
      "password",
      "token"
    ]
  }
}
```

**脱敏示例：**

| 原始值 | 脱敏后 |
|--------|--------|
| `sk-1234567890` | `sk-*****` |
| `mySecretPassword123` | `myS*****` |
| `Bearer eyJhbGci...` | `Bearer *****` |

## 专家思维模型：诊断策略框架

### 问题定位四步法

当遇到问题时，使用以下结构化方法：

```
┌─────────────────────────────────────────────────────────────────┐
│                    问题定位思维流程                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   第 1 步：问题分类                                              │
│   ├── 症状是什么？                                               │
│   ├── 问题发生在哪个子系统？                                      │
│   └── 问题频率如何？（偶发/必现）                                  │
│          │                                                       │
│          ▼                                                       │
│   第 2 步：标志选择                                              │
│   ├── 主要标志：问题子系统                                        │
│   ├── 辅助标志：相关子系统                                        │
│   └── 排除标志：无关子系统                                        │
│          │                                                       │
│          ▼                                                       │
│   第 3 步：日志收集                                              │
│   ├── 开启诊断标志                                               │
│   ├── 复现问题                                                  │
│   └── 收集诊断日志                                               │
│          │                                                       │
│          ▼                                                       │
│   第 4 步：分析定位                                              │
│   ├── 时间线对齐                                                 │
│   ├── 异常模式识别                                               │
│   └── 根因推断                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 常见问题诊断模板

**模板一：性能问题**

```
1. 标志选择：
   - 主要：问题子系统的所有模块
   - 辅助：gateway.performance, database.query

2. 分析重点：
   - 时间戳对齐
   - 耗时操作识别
   - 资源使用模式

3. 关键问题：
   - 慢查询？→ database.*
   - 网络延迟？→ 子系统.http
   - 内存泄漏？→ memory.*
```

**模板二：连接问题**

```
1. 标志选择：
   - 主要：目标通道的所有模块
   - 辅助：gateway.connection, gateway.retry

2. 分析重点：
   - 连接建立流程
   - 认证握手
   - 错误码分析

3. 关键问题：
   - 连接拒绝？→ auth.*
   - 超时？→ timeout.*
   - 重试风暴？→ retry.*
```

**模板三：消息丢失**

```
1. 标志选择：
   - 主要：channels.ingress, channels.egress
   - 辅助：gateway.queue, database.storage

2. 分析重点：
   - 消息生命周期
   - 状态转换
   - 存储确认
```

## 最佳实践

### 团队协作规范

**开发环境：**

```json
{
  "diagnostics": {
    "flags": ["telegram.*", "gateway.*"]
  }
}
```

**测试环境：**

```json
{
  "diagnostics": {
    "flags": ["*.error", "*.warn", "gateway.queue"]
  }
}
```

**生产环境（按需启用）：**

```json
{
  "diagnostics": {
    "flags": []
  }
}
```

### 性能优化建议

1. **按需启用**：只在诊断问题时启用标志
2. **范围控制**：使用最小必要的标志范围
3. **时间限制**：问题定位后及时关闭
4. **磁盘监控**：监控诊断日志磁盘使用量

### 故障排查清单

- [ ] 确认日志文件存在且可读
- [ ] 验证标志名称拼写正确
- [ ] 检查配置语法（JSON 格式）
- [ ] 确认网关已重启（配置变更需要重启）
- [ ] 验证日志级别设置不会抑制诊断输出
- [ ] 检查磁盘空间是否充足

## 常见问题解答

### 问题 1：诊断日志没有输出？

**可能原因：**

1. 标志名称拼写错误
2. 配置未正确加载
3. 全局日志级别过高

**排查步骤：**

```bash
# 1. 检查配置是否生效
openclaw config get diagnostics

# 2. 验证日志文件位置
ls -la /tmp/openclaw/

# 3. 临时使用环境变量测试
OPENCLAW_DIAGNOSTICS=telegram.* openclaw gateway run
```

### 问题 2：诊断日志太多？

**解决方案：**

1. 缩小标志范围
2. 使用更精确的模块名
3. 临时启用后及时关闭

### 问题 3：日志格式混乱？

**说明：**

诊断日志始终为 JSONL 格式。如果需要特定格式，可以使用 `jq` 处理：

```bash
# 格式化输出
jq '.' /tmp/openclaw/openclaw-*.log

# 提取特定字段
jq -c '{time: .timestamp, level: .level, msg: .message}' /tmp/openclaw/openclaw-*.log
```

## 适用场景

### 推荐使用诊断标志的场景

| 场景 | 推荐标志 | 预期效果 |
|------|----------|----------|
| Telegram 连接问题 | `telegram.*` | 获取完整的 HTTP 交互日志 |
| Gateway WebSocket 异常 | `gateway.websocket` | 跟踪 WebSocket 消息流 |
| 认证失败排查 | `gateway.auth`, `channels.*.auth` | 分析认证握手过程 |
| 消息路由问题 | `gateway.routing`, `channels.ingress` | 跟踪消息路由决策 |
| 数据库查询慢 | `database.*` | 分析查询性能和优化点 |

### 不推荐使用的场景

- 常规运行时监控（使用 `logging.level`）
- 安全审计（使用专门的审计日志）
- 性能基准测试（使用性能分析工具）

## 相关资源

### 内部文档

- [日志配置](/logging)
- [故障排查指南](/troubleshooting)
- [CLI 日志命令](/cli/logs)

### 外部参考

- [JSON Lines 格式规范](https://jsonlines.org/)
- [jq 教程](https://stedolan.github.io/jq/tutorial/)
- [ripgrep 使用指南](https://github.com/BurntSushi/ripgexec)
