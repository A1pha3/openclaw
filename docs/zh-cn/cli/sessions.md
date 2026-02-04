---
summary: "CLI 参考 - `openclaw sessions` (列出存储的会话和使用情况)"
read_when:
  - 查看存储的会话和最近活动
title: "sessions"
---

# `openclaw sessions`

列出存储的对话会话。

## 为什么需要这个命令

会话是 OpenClaw 的记忆核心：

- **查看活跃会话**：了解哪些对话正在进行
- **追踪使用情况**：监控 token 消耗
- **调试对话**：查看会话状态和历史

## 示例

```bash
# 列出所有会话
openclaw sessions

# 只显示最近 120 分钟内活跃的会话
openclaw sessions --active 120

# JSON 格式输出
openclaw sessions --json
```

## 选项

| 选项 | 说明 |
|------|------|
| `--active <minutes>` | 只显示指定分钟内活跃的会话 |
| `--json` | JSON 格式输出 |

## 会话标识符

会话使用复合键标识，格式为：

```
agent:<agentId>:<channel>:<type>:<identifier>
```

例如：

- `agent:main:whatsapp:dm:+15555550123` - WhatsApp 私聊
- `agent:main:telegram:group:-1001234567890` - Telegram 群组
- `agent:main:discord:channel:123456789` - Discord 频道

## 使用场景

### 查看活跃对话

```bash
# 查看最近 2 小时的活跃会话
openclaw sessions --active 120
```

### 监控使用量

```bash
# JSON 输出便于脚本处理
openclaw sessions --json | jq '.[] | {key, tokens}'
```

### 调试会话问题

```bash
# 查看所有会话状态
openclaw sessions

# 配合日志诊断
openclaw sessions && openclaw logs -f
```

## 相关命令

| 命令 | 用途 |
|------|------|
| `openclaw sessions` | 列出会话 |
| `openclaw status` | 查看整体状态（包含会话摘要） |
| `openclaw memory` | 管理持久记忆 |

## 会话管理

会话可以通过聊天命令管理：

| 命令 | 效果 |
|------|------|
| `/new` 或 `/reset` | 重置当前会话 |
| `/compact` | 压缩会话上下文 |
| `/status` | 查看会话状态 |
