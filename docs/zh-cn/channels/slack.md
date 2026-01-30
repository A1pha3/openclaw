# Slack

## Socket 模式（默认）

### 快速设置（新手）

1. 创建 Slack 应用并启用 **Socket Mode**。
2. 创建 **App Token**（`xapp-...`）和 **Bot Token**（`xoxb-...`）。
3. 为 Moltbot 设置 Token 并启动网关。

最小配置：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

### 完整设置

1. 在 https://api.slack.com/apps 创建 Slack 应用（From scratch）。
2. **Socket Mode** → 开启。然后进入 **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes**，添加 scope `connections:write`。复制 **App Token**（`xapp-...`）。
3. **OAuth & Permissions** → 添加 Bot Token Scopes（使用下面的 manifest）。点击 **Install to Workspace**。复制 **Bot User OAuth Token**（`xoxb-...`）。
4. 可选：**OAuth & Permissions** → 添加 **User Token Scopes**（见下面的只读列表）。重新安装应用并复制 **User OAuth Token**（`xoxp-...`）。
5. **Event Subscriptions** → 启用事件并订阅：
   - `message.*`（包括编辑/删除/线程广播）
   - `app_mention`
   - `reaction_added`、`reaction_removed`
   - `member_joined_channel`、`member_left_channel`
   - `channel_rename`
   - `pin_added`、`pin_removed`
6. 邀请机器人到你想让它读取的频道。
7. 如果使用 `channels.slack.slashCommand`，在 Slash Commands 中创建 `/clawd`。
8. **App Home** → 启用 **Messages Tab** 让用户可以私信机器人。

多账号支持：使用 `channels.slack.accounts` 配置每个账号的 Token 和可选的 `name`。详见 [网关配置](/zh-cn/config/reference)。

### Moltbot 配置（最小）

通过环境变量设置 Token（推荐）：

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

或通过配置：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-..."
    }
  }
}
```

### 用户 Token（可选）

Moltbot 可以使用 Slack 用户 Token（`xoxp-...`）进行读取操作（历史、置顶、反应、表情、成员信息）。默认保持只读：读取优先使用用户 Token（如果存在），写入仍使用 Bot Token，除非你显式选择。

用户 Token 在配置文件中配置（不支持环境变量）。对于多账号，设置 `channels.slack.accounts.<id>.userToken`。

示例（Bot + App + User Token）：

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-..."
    }
  }
}
```

### 历史上下文

- `channels.slack.historyLimit`（或 `channels.slack.accounts.*.historyLimit`）控制包装到提示中的最近频道/群组消息数量。
- 回退到 `messages.groupChat.historyLimit`。设置 `0` 禁用（默认 50）。

## HTTP 模式（Events API）

当你的网关可以通过 HTTPS 被 Slack 访问时使用 HTTP webhook 模式（典型的服务器部署）。

### 设置

1. 创建 Slack 应用并**禁用 Socket Mode**。
2. **Basic Information** → 复制 **Signing Secret**。
3. **OAuth & Permissions** → 安装应用并复制 **Bot User OAuth Token**。
4. **Event Subscriptions** → 启用事件并设置 **Request URL** 为你的网关 webhook 路径（默认 `/slack/events`）。
5. **Interactivity & Shortcuts** → 启用并设置相同的 **Request URL**。
6. **Slash Commands** → 为你的命令设置相同的 **Request URL**。

### Moltbot 配置（HTTP 模式）

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events"
    }
  }
}
```

## 应用 Manifest

使用此 Slack 应用 manifest 快速创建应用：

```json
{
  "display_information": {
    "name": "Moltbot",
    "description": "Moltbot 的 Slack 连接器"
  },
  "features": {
    "bot_user": {
      "display_name": "Moltbot",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/clawd",
        "description": "发送消息给 Moltbot",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

## Scopes（权限范围）

### Bot Token Scopes（必需）

- `chat:write` - 发送/更新/删除消息
- `im:write` - 打开私信
- `channels:history`、`groups:history`、`im:history`、`mpim:history` - 读取历史
- `channels:read`、`groups:read`、`im:read`、`mpim:read` - 读取频道信息
- `users:read` - 用户查询
- `reactions:read`、`reactions:write` - 反应
- `pins:read`、`pins:write` - 置顶
- `emoji:read` - 表情列表
- `files:write` - 文件上传

### User Token Scopes（可选，默认只读）

- `channels:history`、`groups:history`、`im:history`、`mpim:history`
- `channels:read`、`groups:read`、`im:read`、`mpim:read`
- `users:read`
- `reactions:read`
- `pins:read`
- `emoji:read`
- `search:read`

## 配置参考

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      groupPolicy: "allowlist",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["U123", "U456", "*"],
        groupEnabled: false,
        groupChannels: ["G123"],
        replyToMode: "all"
      },
      channels: {
        "C123": { allow: true, requireMention: true },
        "#general": {
          allow: true,
          requireMention: true,
          users: ["U123"],
          skills: ["search", "docs"],
          systemPrompt: "保持回答简短。"
        }
      },
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off",
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "clawd",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      mediaMaxMb: 20
    }
  }
}
```

## 限制

- 出站文本分块到 `channels.slack.textChunkLimit`（默认 4000）。
- 可选换行分块：设置 `channels.slack.chunkMode="newline"` 在长度分块前按空行分割。
- 媒体上传限制 `channels.slack.mediaMaxMb`（默认 20）。

## 回复线程

默认情况下，Moltbot 在主频道回复。使用 `channels.slack.replyToMode` 控制自动线程：

| 模式 | 行为 |
|------|------|
| `off` | **默认。** 在主频道回复。仅当触发消息已在线程中时才线程化。 |
| `first` | 第一条回复进入线程（在触发消息下），后续回复进入主频道。 |
| `all` | 所有回复进入线程。保持对话集中但可能降低可见性。 |

### 按聊天类型的线程

可以通过 `channels.slack.replyToModeByChatType` 为每种聊天类型配置不同的线程行为：

```json5
{
  channels: {
    slack: {
      replyToMode: "off",        // 频道默认
      replyToModeByChatType: {
        direct: "all",           // 私信始终线程化
        group: "first"           // 群组私信/MPIM 第一条回复线程化
      }
    }
  }
}
```

支持的聊天类型：
- `direct`：1:1 私信（Slack `im`）
- `group`：群组私信 / MPIM（Slack `mpim`）
- `channel`：标准频道（公开/私有）

### 手动线程标签

使用这些标签进行细粒度控制：
- `[[reply_to_current]]` — 回复触发消息（开始/继续线程）。
- `[[reply_to:<id>]]` — 回复特定消息 ID。

## 会话和路由

- 私信共享 `main` 会话（像 WhatsApp/Telegram）。
- 频道映射到 `agent:<agentId>:slack:channel:<channelId>` 会话。
- Slash 命令使用 `agent:<agentId>:slack:slash:<userId>` 会话。

## 私信安全（配对）

- 默认：`channels.slack.dm.policy="pairing"` — 未知私信发送者收到配对码（1小时后过期）。
- 审批：`moltbot pairing approve slack <code>`。
- 允许任何人：设置 `channels.slack.dm.policy="open"` 和 `channels.slack.dm.allowFrom=["*"]`。
- `channels.slack.dm.allowFrom` 接受用户 ID、@handle 或电子邮件。

## 群组策略

- `channels.slack.groupPolicy` 控制频道处理（`open|disabled|allowlist`）。
- `allowlist` 要求频道在 `channels.slack.channels` 中列出。
- 配置向导接受 `#channel` 名称并在可能时解析为 ID。

频道选项：
- `allow`：当 `groupPolicy="allowlist"` 时允许/拒绝频道
- `requireMention`：频道的提及门控
- `tools`：可选的每频道工具策略覆盖
- `toolsBySender`：可选的每发送者工具策略覆盖
- `allowBots`：允许此频道中的机器人消息（默认：false）
- `users`：可选的每频道用户白名单
- `skills`：技能过滤器
- `systemPrompt`：频道的额外系统提示
- `enabled`：设置 `false` 禁用频道

## 投递目标

用于 cron/CLI 发送：
- `user:<id>` 用于私信
- `channel:<id>` 用于频道

## 工具操作

Slack 工具操作可以通过 `channels.slack.actions.*` 门控：

| 操作组 | 默认 | 说明 |
|--------|------|------|
| reactions | 启用 | 反应 + 列出反应 |
| messages | 启用 | 读取/发送/编辑/删除 |
| pins | 启用 | 置顶/取消置顶/列出 |
| memberInfo | 启用 | 成员信息 |
| emojiList | 启用 | 自定义表情列表 |

## 安全说明

- 写入默认使用 Bot Token，因此状态更改操作保持在应用的机器人权限和身份范围内。
- 设置 `userTokenReadOnly: false` 允许在 Bot Token 不可用时使用用户 Token 进行写入操作。将用户 Token 视为高特权并保持操作门控和白名单严格。

## 相关文档

- [网关配置](/zh-cn/config/reference)
- [消息路由](/zh-cn/concepts/routing)
- [故障排除](/zh-cn/operations/troubleshooting)
