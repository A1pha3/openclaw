---
summary: "`openclaw directory` 命令参考（自身、联系人、群组）"
read_when:
  - 想要查找渠道的联系人/群组/自身 ID
  - 正在开发渠道目录适配器
title: "directory"
---

# `openclaw directory`

支持目录功能的渠道的查找命令（联系人、群组和"我"）。

## 为什么需要目录查找

目录查找帮助你：

- **获取正确的 ID**：找到可用于其他命令的用户/群组 ID
- **调试消息发送**：确认目标 ID 格式正确
- **探索渠道结构**：了解渠道中的用户和群组

## 通用选项

| 选项 | 说明 |
|------|------|
| `--channel <name>` | 渠道 ID/别名（配置了多个渠道时必需） |
| `--account <id>` | 账户 ID（默认：渠道默认值） |
| `--json` | 输出 JSON |

## 说明

- `directory` 用于帮助你找到可以粘贴到其他命令中的 ID（特别是 `openclaw message send --target ...`）
- 对于许多渠道，结果基于配置（允许列表/已配置群组）而非实时提供商目录
- 默认输出是 `id`（有时包含 `name`），用制表符分隔；使用 `--json` 便于脚本处理

## 与 message send 配合使用

```bash
# 先查找用户 ID
openclaw directory peers list --channel slack --query "U0"

# 然后发送消息
openclaw message send --channel slack --target user:U012ABCDEF --message "你好"
```

## ID 格式（按渠道）

| 渠道 | DM 格式 | 群组格式 |
|------|---------|----------|
| WhatsApp | `+15551234567` | `1234567890-1234567890@g.us` |
| Telegram | `@username` 或数字 chat id | 数字 id |
| Slack | `user:U…` | `channel:C…` |
| Discord | `user:<id>` | `channel:<id>` |
| Matrix | `user:@user:server` | `room:!roomId:server` 或 `#alias:server` |
| MS Teams | `user:<id>` | `conversation:<id>` |
| Zalo | 用户 id（Bot API） | - |
| Zalo Personal | 线程 id | 线程 id |

## 查询自身 ("me")

```bash
openclaw directory self --channel zalouser
openclaw directory self --channel whatsapp
openclaw directory self --channel telegram
```

输出你的用户 ID，用于调试或配置。

## 查询联系人/用户

```bash
# 列出所有联系人
openclaw directory peers list --channel zalouser

# 搜索联系人
openclaw directory peers list --channel zalouser --query "name"

# 限制结果数量
openclaw directory peers list --channel zalouser --limit 50
```

### 选项

| 选项 | 说明 |
|------|------|
| `--query <text>` | 搜索关键词 |
| `--limit <n>` | 限制返回数量 |

## 查询群组

```bash
# 列出所有群组
openclaw directory groups list --channel zalouser

# 搜索群组
openclaw directory groups list --channel zalouser --query "work"

# 查看群组成员
openclaw directory groups members --channel zalouser --group-id <id>
```

### 选项

| 选项 | 说明 |
|------|------|
| `--query <text>` | 搜索关键词 |
| `--group-id <id>` | 群组 ID（用于 members 子命令） |

## 示例输出

```bash
$ openclaw directory peers list --channel slack --json
[
  {"id": "U012ABCDEF", "name": "张三"},
  {"id": "U034GHIJKL", "name": "李四"}
]

$ openclaw directory groups list --channel discord
channel:123456789012345678    general
channel:987654321098765432    announcements
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 结果为空 | 允许列表为空 | 配置渠道允许列表 |
| 渠道不支持 | 渠道未实现目录 | 检查渠道文档 |
| 查询无结果 | 关键词不匹配 | 尝试更宽泛的搜索词 |
