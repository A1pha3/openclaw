---
summary: "OpenClaw 调试完全指南：监控模式、原始流日志、推理泄漏追踪和开发工作流程"
read_when:
  - 你需要掌握 OpenClaw 的调试工具和工作流程
  - 你想学习如何追踪推理泄漏和原始模型输出
  - 你在调试复杂的网关和代理行为问题
title: "调试指南"
---

# 🐛 OpenClaw 调试完全指南

> **学习目标**：完成本章节学习后，你将能够熟练使用 OpenClaw 提供的各种调试工具，掌握从简单到复杂的调试工作流程，能够独立诊断和解决网关、代理、模型调用等各类问题。

---

## 为什么要学习 OpenClaw 的调试系统

在开始调试之前，我们需要理解 OpenClaw 调试系统的**设计哲学**。与传统的单体应用不同，OpenClaw 是一个分布式系统，涉及多个组件的协作：

```
OpenClaw 调试系统架构：

┌─────────────────────────────────────────────────────────────────┐
│                        调试层                                   │
├─────────────────────────────────────────────────────────────────┤
│  /debug        │  运行时配置覆盖                                  │
│  gateway:watch │  文件监控 + 热重载                              │
│  --dev         │  开发配置隔离                                   │
│  --raw-stream  │  原始流日志                                    │
│  --trace       │  详细追踪日志                                   │
└─────────────────────────────────────────────────────────────────┘
           │                    │                    │
           ▼                    ▼                    ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   Gateway       │   │   Agent/Runtime │   │   Providers     │
│   (控制平面)     │   │   (代理运行时)   │   │   (模型提供商)   │
└─────────────────┘   └─────────────────┘   └─────────────────┘
```

**调试系统的核心目标**：

1. **可观测性**：提供足够的运行时信息来理解系统行为
2. **可重复性**：确保调试过程可重复，便于问题复现
3. **安全性**：敏感信息脱敏，防止调试数据泄露
4. **渐进式**：从简单日志到详细追踪，适应不同调试需求

---

## 认知负荷管理：调试技能层次

```
调试技能金字塔：

                    ┌─────────────────────────┐
                    │   专家级：分布式追踪     │  跨组件问题诊断、性能优化
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：原始流分析      │  推理泄漏追踪、格式问题
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：监控与日志      │  gateway:watch、详细日志
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：基础调试       │  /debug 命令、基本日志
                    └─────────────────────────┘
```

**建议学习路径**：
- 新手：从 `/debug` 命令开始，学习基本运行时调试
- 中级：掌握 `gateway:watch` 和日志配置
- 高级：学习原始流日志和推理泄漏追踪
- 专家：整合所有工具进行分布式问题诊断

---

## 第一部分：基础调试（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 使用 `/debug` 命令进行运行时配置覆盖
- [ ] 理解调试覆盖的作用域和生命周期
- [ ] 掌握基本的调试信息获取方法
- [ ] 识别常见的简单调试场景

### 1.1 `/debug` 命令：运行时配置覆盖

**核心概念**：

`/debug` 是 OpenClaw 提供的**内存中**配置覆盖机制，它允许你在不修改配置文件的情况下临时调整运行时行为。

**为什么需要运行时覆盖**：

- 快速迭代：无需重启网关即可测试不同配置
- 隔离调试：不影响其他用户或会话
- 临时性：重启后自动恢复，不会污染持久化配置

**命令语法**：

```bash
/debug show              # 显示当前调试配置
/debug set <key>=<value> # 设置调试配置
/debug unset <key>       # 取消设置
/debug reset             # 重置所有调试配置
```

**使用示例**：

```bash
# 查看当前调试配置
/debug show

# 设置响应前缀（仅本次会话有效）
/debug set messages.responsePrefix="[调试模式]"

/debug set logging.level=debug

/debug unset messages.responsePrefix

# 清除所有调试覆盖
/debug reset
```

**调试覆盖的作用域**：

```
┌─────────────────────────────────────────┐
│           /debug 覆盖机制                 │
├─────────────────────────────────────────┤
│  作用范围：当前会话（Session）             │
│  生命周期：直到网关重启                   │
│  持久化：不写入磁盘                       │
│  优先级：高于配置文件，低于 CLI 参数        │
└─────────────────────────────────────────┘
```

**适用场景**：

| 场景 | 操作 | 示例 |
|------|------|------|
| 临时提高日志级别 | `/debug set logging.level=trace` | 排查详细问题 |
| 调整响应格式 | `/debug set messages.responsePrefix="[DEBUG]"` | 区分调试输出 |
| 禁用特定功能 | `/debug set features.sandbox=false` | 测试无沙箱行为 |
| 模拟错误场景 | `/debug set errors.simulate=true` | 测试错误处理 |

### 1.2 启用调试功能

**前提条件**：

要使用 `/debug` 命令，需要先在配置中启用：

```json5
{
  commands: {
    debug: true  // 启用 /debug 命令
  }
}
```

**默认行为**：

- 如果未启用 `commands.debug`，`/debug` 命令会被忽略
- 建议在开发环境中全局启用，生产环境中按需启用

---

## 第二部分：中级调试技巧（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 使用 `gateway:watch` 实现文件监控和热重载
- [ ] 配置开发环境隔离
- [ ] 理解 `--dev` 标志的作用和用法
- [ ] 掌握快速迭代的调试工作流程

### 2.1 Gateway 监控模式：`gateway:watch`

**核心概念**：

`gateway:watch` 是 OpenClaw 提供的**文件监控 + 自动重载**机制，当你修改源代码时，Gateway 会自动重启并应用更改。

**工作原理**：

```
┌─────────────────────────────────────────────────────┐
│              gateway:watch 工作流程                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. 启动文件监控器（监听 src/ 目录变化）              │
│           ↓                                          │
│  2. 检测到文件变更（保存时触发）                      │
│           ↓                                          │
│  3. 自动终止当前 Gateway 进程                        │
│           ↓                                          │
│  4. 重新编译 TypeScript                             │
│           ↓                                          │
│  5. 启动新的 Gateway 进程                            │
│           ↓                                          │
│  6. 恢复所有活动的会话和连接                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**使用方法**：

```bash
# 标准用法（推荐）
pnpm gateway:watch --force

# 等效命令
tsx watch src/entry.ts gateway --force
```

**参数传递**：

所有传递给 `gateway:watch` 的参数都会传递给 Gateway 进程：

```bash
# 启用详细日志
pnpm gateway:watch --force --verbose

# 指定配置文件
pnpm gateway:watch --force --config /path/to/config.json

# 组合多个参数
pnpm gateway:watch --force --verbose --raw-stream
```

**热重载行为**：

| 变更类型 | 重载行为 | 影响的配置 |
|----------|----------|------------|
| TypeScript 源码 | 完全重启 | 所有配置重新加载 |
| 配置文件修改 | 平滑重载 | 仅相关配置 |
| 代理提示词文件 | 动态加载 | AGENTS.md 等 |
| 新增工具文件 | 动态注册 | TOOLS.md 等 |

### 2.2 开发配置隔离：`--dev` 模式

**核心概念**：

OpenClaw 提供两种 `--dev` 标志，分别用于**配置文件隔离**和**开发引导**：

**两种 `--dev` 标志的区别**：

| 标志 | 作用域 | 作用 |
|------|--------|------|
| **全局 `--dev`**（配置文件） | 全局 | 隔离状态目录、派生端口 |
| **`gateway --dev`** | Gateway 实例 | 开发引导、自动创建默认配置 |

**推荐工作流程**：

```bash
# 启动开发 Gateway
pnpm gateway:dev

# 在另一个终端中打开 TUI
OPENCLAW_PROFILE=dev openclaw tui
```

** `--dev` 的具体行为**：

**1. 配置文件隔离（全局 `--dev`）**：

```bash
# 设置环境变量
OPENCLAW_PROFILE=dev

# 等效效果：
OPENCLAW_STATE_DIR=~/.openclaw-dev        # 状态目录隔离
OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json  # 配置路径
OPENCLAW_GATEWAY_PORT=19001               # Gateway 端口派生
```

**2. 开发引导（`gateway --dev`）**：

```
┌─────────────────────────────────────────────────┐
│         gateway --dev 自动执行的操作              │
├─────────────────────────────────────────────────┤
│                                                 │
│  ✅ 如果缺少配置，写入最小配置                    │
│     - gateway.mode = "local"                    │
│     - bind = "loopback"                         │
│                                                 │
│  ✅ 设置开发工作区                               │
│     - agent.workspace = "~/dev-workspace"      │
│                                                 │
│  ✅ 跳过 BOOTSTRAP.md                            │
│     - agent.skipBootstrap = true                │
│                                                 │
│  ✅ 植入开发工作区文件                           │
│     - AGENTS.md（协议机器人 C3-PO）              │
│     - SOUL.md、TOOLS.md、IDENTITY.md            │
│     - USER.md、HEARTBEAT.md                     │
│                                                 │
│  ✅ 跳过渠道提供商初始化                          │
│     - OPENCLAW_SKIP_CHANNELS = 1                │
│                                                 │
└─────────────────────────────────────────────────┘
```

**重置开发环境**：

```bash
# 完全重置开发环境
pnpm gateway:dev:reset

# 这会：
# 1. 删除 ~/.openclaw-dev 目录
# 2. 重新创建默认开发配置
# 3. 重置所有开发会话
```

**手动指定开发模式**：

```bash
# 如果全局 --dev 有冲突
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

---

## 第三部分：高级调试技术（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 捕获和分析原始模型流
- [ ] 追踪推理泄漏问题
- [ ] 理解 OpenAI 兼容块的原始日志
- [ ] 掌握安全调试的最佳实践

### 3.1 原始流日志（OpenClaw）

**核心概念**：

原始流日志记录了**解析/格式化之前**的模型输出流，这是诊断推理泄漏和格式问题的关键工具。

**为什么要捕获原始流**：

- **推理泄漏检测**：某些模型会将内部推理作为纯文本输出
- **格式问题诊断**：区分模型输出格式问题与应用层解析问题
- **时序分析**：理解 token 到达的精确时序

**启用方法**：

```bash
# 方法一：通过命令行参数
pnpm gateway:watch --force --raw-stream

# 方法二：指定日志路径
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl

# 方法三：环境变量
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

**日志格式**：

```jsonl
{"timestamp":"2024-01-15T10:30:00.000Z","chunk":"Hello","isFirst":true,"provider":"openai","model":"gpt-4"}
{"timestamp":"2024-01-15T10:30:00.050Z","chunk":" world","isFirst":false,"provider":"openai","model":"gpt-4"}
{"timestamp":"2024-01-15T10:30:00.100Z","chunk":"!","isFirst":false,"provider":"openai","model":"gpt-4","isLast":true}
```

**字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| `timestamp` | ISO 8601 | 块到达时间（高精度） |
| `chunk` | 字符串 | 原始文本块 |
| `isFirst` | 布尔 | 是否是第一个块 |
| `isLast` | 布尔 | 是否是最后一个块 |
| `provider` | 字符串 | 模型提供商 |
| `model` | 字符串 | 模型名称 |

### 3.2 原始块日志（pi-mono）

**核心概念**：

pi-mono 的原始块日志记录了 **OpenAI 兼容块** 在解析之前的状态，这是理解提供商特定格式的窗口。

**启用方法**：

```bash
# 环境变量
PI_RAW_STREAM=1

# 指定日志路径
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

**注意**：此日志仅由使用 `openai-compat` 提供商的进程发出。

**日志格式**：

```jsonl
{"raw":"{\"id\":\"chatcmpl-abc123\",\"object\":\"chat.completion.chunk\",\"created\":1699000000,\"model\":\"gpt-4\",\"choices\":[{\"index\":0,\"delta\":{\"content\":\"Hello\"},\"finish_reason\":null}]}"}
{"raw":"{\"id\":\"chatcmpl-abc123\",\"object\":\"chat.completion.chunk\",\"created\":1699000000,\"model\":\"gpt-4\",\"choices\":[{\"index\":0,\"delta\":{\"content\":\" world\"},\"finish_reason\":null}]}"}
```

**应用场景**：

| 场景 | 使用日志 | 分析重点 |
|------|----------|----------|
| 推理泄漏 | 原始流日志 | 检查推理标记是否被发送到输出 |
| 格式错误 | 原始块日志 | 对比提供商响应与期望格式 |
| 流中断 | 两者结合 | 分析最后一个块的 finish_reason |
| Token 计数 | 原始流日志 | 累加 chunk 长度 |

### 3.3 安全调试最佳实践

**⚠️ 安全警告**：

原始日志可能包含：
- 完整提示词（包括系统提示词）
- 工具输出
- 用户数据
- API 密钥片段

**安全处理原则**：

```markdown
## 调试安全 Checklist

### 调试前
- [ ] 确认调试环境的网络隔离级别
- [ ] 了解将要捕获的数据类型
- [ ] 准备脱敏工具

### 调试中
- [ ] 仅捕获必要的数据
- [ ] 标记日志文件的敏感级别
- [ ] 避免在公共环境保存日志

### 调试后
- [ ] 审查日志内容，标记敏感信息
- [ ] 脱敏后再分享
- [ ] 及时删除临时日志文件
```

**日志脱敏示例**：

```bash
# 分享前脱敏
cat raw-stream.jsonl | \
  sed 's/sk-[a-zA-Z0-9]*/[REDACTED_API_KEY]/g' | \
  sed 's/"user": "[^"]*"/"user": "[REDACTED]"/g' \
  > sanitized-log.jsonl
```

---

## 第四部分：专家级调试方法论（⭐⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 设计系统化的调试工作流程
- [ ] 运用科学方法论诊断复杂问题
- [ ] 构建团队级别的调试最佳实践
- [ ] 优化调试效率，减少问题解决时间

### 4.1 系统化调试方法论

**专家思维模型：分层诊断框架**：

```markdown
## 分层诊断框架

遇到复杂问题时，按层次逐步排查：

### 第 1 层：配置层
┌─────────────────────────────────────────┐
│ 检查点：                                   │
│ - 配置文件是否正确加载                      │
│ - 环境变量是否设置正确                     │
│ - CLI 参数是否传递正确                     │
│ 工具：openclaw doctor                    │
└─────────────────────────────────────────┘
           ↓ 不通过则修复配置
### 第 2 层：网络层
┌─────────────────────────────────────────┐
│ 检查点：                                   │
│ - Gateway 是否正常运行                    │
│ - WebSocket 连接是否建立                  │
│ - 渠道连接是否正常                        │
│ 工具：openclaw status、logs --follow     │
└─────────────────────────────────────────┘
           ↓ 不通过则检查网络
### 第 3 层：代理层
┌─────────────────────────────────────────┐
│ 检查点：                                   │
│ - Agent 是否响应                          │
│ - 会话状态是否正常                         │
│ - 工具调用是否成功                         │
│ 工具：/debug、日志分析                    │
└─────────────────────────────────────────┘
           ↓ 不通过则检查代理
### 第 4 层：模型层
┌─────────────────────────────────────────┐
│ 检查点：                                   │
│ - API 调用是否成功                        │
│ - 响应格式是否正确                        │
│ - 推理是否正常完成                         │
│ 工具：--raw-stream、--trace              │
└─────────────────────────────────────────┘
```

### 4.2 常见问题诊断流程

**问题诊断模板**：

```markdown
## 问题诊断报告模板

### 问题描述
**症状**：简洁描述问题的外在表现
**影响范围**：哪些功能/用户受影响
**复现步骤**：复现问题的步骤

### 环境信息
- OpenClaw 版本：vX.X.X
- 操作系统：macOS/Linux/Windows
- Node.js 版本：v22.x
- 模型提供商：anthropic/openai/...

### 排查过程

#### 步骤 1：收集信息
```bash
# 保存诊断信息
openclaw doctor > diagnosis.txt 2>&1

# 捕获日志
openclaw logs --follow > debug.log 2>&1
```

#### 步骤 2：分析日志
```bash
# 查看错误
grep -i error debug.log

# 查看特定时间范围
awk '/2024-01-15T10:/' debug.log

# 过滤渠道日志
openclaw channels logs --channel telegram
```

#### 步骤 3：验证假设
[列出每个假设和验证方法]

### 解决方案

**根本原因**：
[深入分析后的问题根源]

**修复方法**：
[具体的修复步骤]

**预防措施**：
[如何防止类似问题再次发生]
```

### 4.3 调试效率优化

**预定义的调试命令别名**：

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc

# 快速诊断
alias ocdiag='openclaw doctor && echo "---" && openclaw status'

# 详细日志
alias oclog='openclaw logs --follow --json | jq "select(.level == \"error\" or .level == \"warn\")"'

# 原始流捕获
alias occapture='pnpm gateway:watch --force --raw-stream --raw-stream-path ~/raw-capture.jsonl'

# 开发环境重置
alias ocdev-reset='pnpm gateway:dev:reset && echo "开发环境已重置"'
```

**自动化调试脚本**：

```bash
#!/bin/bash
# debug-session.sh - 自动化调试会话

echo "=== OpenClaw 调试会话 ==="
echo "时间：$(date)"
echo ""

echo "1. 运行诊断..."
openclaw doctor
echo ""

echo "2. 收集日志（最近 100 行）..."
openclaw logs --lines 100
echo ""

echo "3. 检查进程状态..."
ps aux | grep openclaw | grep -v grep
echo ""

echo "4. 检查端口占用..."
ss -ltnp | grep 18789
```

---

## 第五部分：故障排查速查

### 快速索引

| 问题 | 症状 | 快速解决方案 |
|------|------|---------------|
| Gateway 不可达 | 连接超时 | `openclaw doctor` 检查 |
| 日志为空 | 无日志输出 | 检查日志文件路径和权限 |
| 配置不生效 | 修改无效 | 重启 Gateway |
| 原始流无输出 | 无原始日志 | 检查 `OPENCLAW_RAW_STREAM=1` |
| 推理泄漏 | 推理内容在输出中 | 使用 `--raw-stream` 捕获分析 |

### Gateway 不可达

**排查命令序列**：

```bash
# 1. 检查进程
ps aux | grep openclaw

# 2. 检查端口
ss -ltnp | grep 18789

# 3. 检查日志
openclaw logs --follow

# 4. 完整诊断
openclaw doctor --verbose
```

**常见原因**：
- Gateway 未运行 → `openclaw gateway start`
- 端口冲突 → 修改 `gateway.port` 配置
- 配置文件错误 → `openclaw doctor` 检查

### 日志为空

**排查命令序列**：

```bash
# 1. 检查日志级别
openclaw config get logging.level

# 2. 检查日志路径
openclaw config get logging.file

# 3. 手动创建日志（测试权限）
touch $(openclaw config get logging.file)

# 4. 设置详细日志级别
openclaw config set logging.level debug
```

### 配置不生效

**排查命令序列**：

```bash
# 1. 确认配置已加载
openclaw config get agents.defaults.model

# 2. 检查是否需要重启
openclaw gateway restart

# 3. 检查是否有调试覆盖
/debug show

# 4. 重置调试覆盖
/debug reset
```

---

## 练习与自检

### 基础技能自检

- [ ] 能够使用 `/debug` 进行运行时配置调整
- [ ] 能够启动 `gateway:watch` 进行开发
- [ ] 能够解释 `--dev` 模式的两种用法
- [ ] 能够获取基本的诊断信息

### 进阶技能自检

- [ ] 能够配置原始流日志捕获
- [ ] 能够分析日志诊断复杂问题
- [ ] 理解推理泄漏的检测方法
- [ ] 能够设计团队调试工作流程

### 专家技能自检

- [ ] 能够构建自动化调试工具链
- [ ] 掌握分层诊断方法论
- [ ] 能够优化大规模调试效率
- [ ] 能够培训团队成员调试技能

---

## 参考资料

### 核心文档

- [日志记录](/zh-CN/logging) - 日志系统详解
- [诊断标志](/zh-CN/diagnostics/flags) - 定向调试日志
- [开发模式](/zh-CN/developer) - 开发配置

### 相关命令速查

```bash
# 基础调试
/debug show              # 显示调试配置
/debug set key=value    # 设置调试配置
pnpm gateway:watch      # 监控模式

# 开发模式
pnpm gateway:dev       # 开发 Gateway
pnpm gateway:dev:reset  # 重置开发环境

# 日志与诊断
openclaw logs --follow  # 尾随日志
openclaw doctor         # 诊断检查

# 原始流捕获
OPENCLAW_RAW_STREAM=1    # 启用原始流日志
PI_RAW_STREAM=1          # 启用 pi-mono 原始块日志
```
