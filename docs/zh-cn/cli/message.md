---
summary: "`openclaw message` 命令参考（发送 + 渠道操作）"
read_when:
  - 添加或修改消息 CLI 操作
  - 更改出站渠道行为
title: "message"
---

# `openclaw message`

统一的出站命令，用于发送消息和执行渠道操作。支持 Discord、Google Chat、Slack、Mattermost（插件）、Telegram、WhatsApp、Signal、iMessage、MS Teams。

## 为什么需要这个命令

这是与外部聊天渠道交互的核心命令：

- **统一接口**：一个命令支持所有渠道
- **丰富功能**：发送、投票、反应、编辑、删除等
- **自动化友好**：支持 JSON 输出和脚本调用
- **批量操作**：支持广播到多个目标

## 基本用法

```bash
openclaw message <子命令> [选项]
```

## 渠道选择

- `--channel`：当配置了多个渠道时必需
- 如果只配置了一个渠道，则自动使用
- 可选值：`whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`

## 目标格式

| 渠道 | 格式示例 |
|------|----------|
| WhatsApp | E.164 格式或群组 JID |
| Telegram | 聊天 ID 或 `@username` |
| Discord | `channel:<id>` 或 `user:<id>` |
| Google Chat | `spaces/<spaceId>` 或 `users/<userId>` |
| Slack | `channel:<id>` 或 `user:<id>` |
| Signal | `+E.164`、`group:<id>` 或 `username:<name>` |
| iMessage | handle、`chat_id:<id>` 或 `chat_guid:<guid>` |
| MS Teams | `conversation:<id>` 或 `user:<aad-object-id>` |

**名称查找**：对于 Discord/Slack 等支持的渠道，频道名称如 `Help` 或 `#help` 会通过目录缓存解析。

## 通用选项

| 选项 | 说明 |
|------|------|
| `--channel <name>` | 目标渠道 |
| `--account <id>` | 账户 ID |
| `--target <dest>` | 目标频道或用户 |
| `--targets <name>` | 多目标（可重复；仅广播） |
| `--json` | JSON 输出 |
| `--dry-run` | 预演模式 |
| `--verbose` | 详细输出 |

## 核心操作

### send（发送）

**支持渠道**：WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost/Signal/iMessage/MS Teams

**必需**：`--target`，加上 `--message` 或 `--media`

**可选**：`--media`、`--reply-to`、`--thread-id`、`--gif-playback`

```bash
# 发送 Discord 回复
openclaw message send --channel discord \
  --target channel:123 --message "你好" --reply-to 456

# 发送 Teams 消息
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "你好"

# Telegram 内联按钮
openclaw message send --channel telegram --target @mychat --message "选择：" \
  --buttons '[ [{"text":"是","callback_data":"cmd:yes"}], [{"text":"否","callback_data":"cmd:no"}] ]'
```

### poll（投票）

**支持渠道**：WhatsApp/Discord/MS Teams

```bash
# Discord 投票
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "零食？" \
  --poll-option 披萨 --poll-option 寿司 \
  --poll-multi --poll-duration-hours 48

# Teams 投票
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "午餐？" \
  --poll-option 披萨 --poll-option 寿司
```

### react（反应）

**支持渠道**：Discord/Google Chat/Slack/Telegram/WhatsApp/Signal

```bash
# Slack 反应
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"

# Signal 群组反应
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

### read（读取）

**支持渠道**：Discord/Slack

```bash
openclaw message read --channel discord --target channel:123 --limit 50
```

### edit（编辑）

**支持渠道**：Discord/Slack

```bash
openclaw message edit --channel discord \
  --target channel:123 --message-id 456 --message "更新的内容"
```

### delete（删除）

**支持渠道**：Discord/Slack/Telegram

```bash
openclaw message delete --channel discord --target channel:123 --message-id 456
```

### pin/unpin（置顶/取消置顶）

**支持渠道**：Discord/Slack

```bash
openclaw message pin --channel discord --target channel:123 --message-id 456
```

## 线程操作

### thread create

```bash
openclaw message thread create --channel discord \
  --target channel:123 --thread-name "讨论话题"
```

### thread list

```bash
openclaw message thread list --channel discord --guild-id 123
```

### thread reply

```bash
openclaw message thread reply --channel discord \
  --target thread:456 --message "回复内容"
```

## 表情和贴纸

### emoji list

```bash
openclaw message emoji list --channel discord --guild-id 123
openclaw message emoji list --channel slack
```

### sticker send

```bash
openclaw message sticker send --channel discord \
  --target channel:123 --sticker-id 456
```

## 成员和角色管理（Discord）

```bash
# 角色信息
openclaw message role info --channel discord --guild-id 123

# 添加/移除角色
openclaw message role add --channel discord \
  --guild-id 123 --user-id 456 --role-id 789

# 成员信息
openclaw message member info --channel discord \
  --guild-id 123 --user-id 456
```

## 审核操作（Discord）

```bash
# 超时
openclaw message timeout --channel discord \
  --guild-id 123 --user-id 456 --duration-min 60

# 踢出
openclaw message kick --channel discord \
  --guild-id 123 --user-id 456 --reason "违规"

# 封禁
openclaw message ban --channel discord \
  --guild-id 123 --user-id 456 --delete-days 7
```

## 广播

```bash
openclaw message broadcast --channel all \
  --targets "#general" --targets "#announcements" \
  --message "重要公告"
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 渠道未找到 | 未配置或未连接 | `openclaw channels status` |
| 目标无效 | 格式错误 | 检查目标格式表 |
| 权限不足 | Bot 权限不够 | 检查渠道 Bot 权限设置 |
| 发送失败 | 网络问题 | 添加 `--verbose` 查看详情 |
