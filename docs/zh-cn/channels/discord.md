---
summary: 介绍 Discord Bot 配置，包括快速设置、工作原理、白名单、速率限制、配置参考、回复标签和安全运维
read_when:
  - 配置 Discord Bot 时
  - 设置服务器频道时
  - 调试机器人行为时
title: Discord 配置
---

# Discord 配置

状态：通过官方 Discord Bot Gateway 支持私信和服务器文字频道。

## 快速设置（新手）

1. 创建 Discord 机器人并复制 Bot Token。
2. 在 Discord 应用设置中，启用 **Message Content Intent**（如果需要使用白名单或名称查询，还需启用 **Server Members Intent**）。
3. 为 OpenClaw 设置 Token：
   - 环境变量：`DISCORD_BOT_TOKEN=...`
   - 或配置：`channels.discord.token: "..."`
   - 如果两者都设置，配置文件优先（环境变量仅作为默认账号的后备）。
4. 邀请机器人到你的服务器并授予消息权限（如果只想用私信，可以创建一个私人服务器）。
5. 启动网关。
6. 私信默认使用配对模式；首次联系时需要审批配对码。

最小配置：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

## 目标

- 通过 Discord 私信或服务器频道与 OpenClaw 交流。
- 私聊合并到代理的主会话（默认 `agent:main:main`）；服务器频道隔离为 `agent:<agentId>:discord:channel:<channelId>`（显示名称使用 `discord:<guildSlug>#<channelSlug>`）。
- 群组私信默认忽略；通过 `channels.discord.dm.groupEnabled` 启用，可选通过 `channels.discord.dm.groupChannels` 限制。
- 保持路由确定性：回复始终返回到消息来源的频道。

## 工作原理

1. 创建 Discord 应用 → Bot，启用所需的 Intent（私信 + 服务器消息 + 消息内容），获取 Bot Token。
2. 邀请机器人到服务器，授予读取/发送消息的权限。
3. 配置 OpenClaw 的 `channels.discord.token`（或使用 `DISCORD_BOT_TOKEN` 作为后备）。
4. 运行网关；当 Token 可用（配置优先，环境变量后备）且 `channels.discord.enabled` 不为 `false` 时，自动启动 Discord 频道。
5. 私聊：使用 `user:<id>`（或 `<@id>` 提及）进行投递；所有对话落入共享的 `main` 会话。
6. 服务器频道：使用 `channel:<channelId>` 进行投递。默认需要提及，可按服务器或频道设置。
7. 私聊安全：默认通过 `channels.discord.dm.policy`（默认：`"pairing"`）。未知发送者收到配对码（1小时后过期）；通过 `openclaw pairing approve discord <code>` 审批。

## 配置写入

默认情况下，Discord 允许通过 `/config set|unset` 触发配置更新（需要 `commands.config: true`）。

禁用方式：

```json5
{
  channels: { discord: { configWrites: false } }
}
```

## 如何创建自己的机器人

这是在服务器频道（如 `#help`）运行 OpenClaw 的 Discord 开发者门户设置。

### 1) 创建 Discord 应用 + 机器人用户

1. Discord 开发者门户 → **Applications** → **New Application**
2. 在应用中：
   - **Bot** → **Add Bot**
   - 复制 **Bot Token**（这就是 `DISCORD_BOT_TOKEN` 的值）

### 2) 启用 OpenClaw 需要的 Gateway Intent

Discord 默认阻止"特权 Intent"，需要显式启用。

在 **Bot** → **Privileged Gateway Intents** 中，启用：
- **Message Content Intent**（大多数服务器中读取消息文本必需）
- **Server Members Intent**（推荐；某些成员/用户查询和白名单匹配需要）

通常**不需要** **Presence Intent**。

### 3) 生成邀请链接（OAuth2 URL Generator）

在应用中：**OAuth2** → **URL Generator**

**Scopes**
- `bot`
- `applications.commands`（原生命令必需）

**Bot Permissions**（最小基线）
- View Channels
- Send Messages
- Read Message History
- Embed Links
- Attach Files
- Add Reactions（可选但推荐）
- Use External Emojis / Stickers（可选）

除非调试且完全信任机器人，否则避免 **Administrator**。

复制生成的 URL，打开它，选择服务器并安装机器人。

### 4) 获取 ID（服务器/用户/频道）

Discord 到处使用数字 ID；OpenClaw 配置优先使用 ID。

1. Discord（桌面/网页）→ **用户设置** → **高级** → 启用 **开发者模式**
2. 右键点击：
   - 服务器名称 → **复制服务器 ID**
   - 频道（如 `#help`）→ **复制频道 ID**
   - 你的用户 → **复制用户 ID**

### 5) 配置 OpenClaw

#### Token

通过环境变量设置 Bot Token（服务器推荐）：

```bash
DISCORD_BOT_TOKEN=...
```

或通过配置：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

多账号支持：使用 `channels.discord.accounts` 配置每个账号的 Token 和可选的 `name`。详见 [网关配置](/zh-CN/config/reference)。

#### 白名单 + 频道路由

示例"单服务器，只允许我，只允许 #help"：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

注意：
- `requireMention: true` 表示机器人只在被提及时回复（共享频道推荐）。
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）也算作服务器消息的提及。
- 如果存在 `channels`，未列出的频道默认被拒绝。
- 使用 `"*"` 频道条目应用默认值；显式频道条目覆盖通配符。
- 线程继承父频道配置，除非显式添加线程频道 ID。
- 机器人消息默认忽略；设置 `channels.discord.allowBots=true` 允许它们。

### 6) 验证是否工作

1. 启动网关。
2. 在服务器频道发送：`@YourBot hello`。
3. 如果没反应：检查下面的故障排除。

## 故障排除

- 首先：运行 `openclaw doctor` 和 `openclaw channels status --probe`。
- **"Used disallowed intents"**：在开发者门户启用 **Message Content Intent**（可能还需要 **Server Members Intent**），然后重启网关。
- **机器人连接但在服务器频道不回复**：
  - 缺少 **Message Content Intent**，或
  - 机器人缺少频道权限（View/Send/Read History），或
  - 配置要求提及但你没提及，或
  - 服务器/频道白名单拒绝了频道/用户。
- **`requireMention: false` 但仍不回复**：
  - `channels.discord.groupPolicy` 默认 **allowlist**；设置为 `"open"` 或在 `channels.discord.guilds` 下添加服务器条目。
- **私信不工作**：`channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`，或尚未被批准。

## 功能和限制

- 私信和服务器文字频道（线程作为独立频道处理；不支持语音）。
- 尽力发送输入指示器；消息分块使用 `channels.discord.textChunkLimit`（默认 2000），按行数分割长回复（`channels.discord.maxLinesPerMessage`，默认 17）。
- 可选换行分块：设置 `channels.discord.chunkMode="newline"` 在长度分块前按空行（段落边界）分割。
- 文件上传支持，最大 `channels.discord.mediaMaxMb`（默认 8 MB）。
- 默认服务器回复需要提及以避免嘈杂的机器人。
- 消息引用另一条消息时注入回复上下文（引用内容 + ID）。
- 原生回复线程默认**关闭**；通过 `channels.discord.replyToMode` 和回复标签启用。

## 重试策略

出站 Discord API 调用在速率限制（429）时重试，使用 Discord 的 `retry_after`（可用时），带指数退避和抖动。通过 `channels.discord.retry` 配置。详见 [重试策略](/zh-CN/concepts/routing)。

## 配置参考

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "username"],
        groupEnabled: false,
        groupChannels: ["dm-channel-slug"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "my-server",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "保持回答简短。"
            }
          }
        }
      }
    }
  }
}
```

### 配置选项说明

| 选项 | 说明 |
|------|------|
| `dm.enabled` | 设置 `false` 忽略所有私信（默认 `true`） |
| `dm.policy` | 私信访问控制（推荐 `pairing`） |
| `dm.allowFrom` | 私信白名单（用户 ID 或名称） |
| `dm.groupEnabled` | 启用群组私信（默认 `false`） |
| `dm.groupChannels` | 群组私信频道 ID 或 slug 的可选白名单 |
| `groupPolicy` | 控制服务器频道处理（`open|disabled|allowlist`） |
| `guilds` | 按服务器 ID（首选）或 slug 的每服务器规则 |
| `guilds."*"` | 无显式条目时应用的默认每服务器设置 |
| `textChunkLimit` | 出站文本块大小（字符）。默认：2000 |
| `chunkMode` | `length`（默认）或 `newline` |
| `maxLinesPerMessage` | 每条消息的软最大行数。默认：17 |
| `mediaMaxMb` | 入站媒体大小限制 |
| `historyLimit` | 回复提及时包含的最近服务器消息数 |
| `retry` | 出站 Discord API 调用的重试策略 |
| `actions` | 每操作工具开关 |
| `replyToMode` | `off`（默认）、`first` 或 `all` |

### 工具操作默认值

| 操作组 | 默认 | 说明 |
|--------|------|------|
| reactions | 启用 | 反应 + 列出反应 + emojiList |
| stickers | 启用 | 发送贴纸 |
| emojiUploads | 启用 | 上传表情 |
| stickerUploads | 启用 | 上传贴纸 |
| polls | 启用 | 创建投票 |
| permissions | 启用 | 频道权限快照 |
| messages | 启用 | 读取/发送/编辑/删除 |
| threads | 启用 | 创建/列出/回复 |
| pins | 启用 | 置顶/取消置顶/列出 |
| search | 启用 | 消息搜索（预览功能） |
| memberInfo | 启用 | 成员信息 |
| roleInfo | 启用 | 角色列表 |
| channelInfo | 启用 | 频道信息 + 列表 |
| channels | 启用 | 频道/分类管理 |
| voiceStatus | 启用 | 语音状态查询 |
| events | 启用 | 列出/创建计划活动 |
| roles | 禁用 | 角色添加/移除 |
| moderation | 禁用 | 超时/踢出/封禁 |

## 回复标签

要请求线程回复，模型可以在输出中包含一个标签：
- `[[reply_to_current]]` — 回复触发的 Discord 消息。
- `[[reply_to:<id>]]` — 回复上下文/历史中的特定消息 ID。

当前消息 ID 作为 `[message_id: ...]` 附加到提示中；历史条目已包含 ID。

行为由 `channels.discord.replyToMode` 控制：
- `off`：忽略标签。
- `first`：只有第一个出站块/附件是回复。
- `all`：每个出站块/附件都是回复。

## 安全和运维

- 将 Bot Token 视为密码；在受监督的主机上优先使用 `DISCORD_BOT_TOKEN` 环境变量，或锁定配置文件权限。
- 只授予机器人所需的权限（通常是读取/发送消息）。
- 如果机器人卡住或被速率限制，确认没有其他进程占用 Discord 会话后重启网关（`openclaw gateway --force`）。

## 相关文档

- [网关配置](/zh-CN/config/reference)
- [消息路由](/zh-CN/concepts/routing)
- [故障排除](/zh-CN/operations/troubleshooting)
