---
summary: "Tlon/Urbit 支持状态、功能和配置"
read_when:
  - 开发 Tlon/Urbit 渠道功能
title: "Tlon"
---

# Tlon（插件）

Tlon 是构建在 Urbit 之上的去中心化信使。OpenClaw 连接到你的 Urbit ship，可以回复私信和群聊消息。群组回复默认需要 @ 提及，并且可以通过允许列表进一步限制。

状态：通过插件支持。私信、群组提及、线程回复和仅文本媒体回退（附加到标题的 URL）。不支持反应、投票和本机媒体上传。

## 需要插件

Tlon 作为插件提供，不与核心安装捆绑。

通过 CLI 安装（npm 仓库）：

```bash
openclaw plugins install @openclaw/tlon
```

本地检出（当从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/tlon
```

详情请参阅：[插件](/plugin)

## 设置

1. 安装 Tlon 插件。
2. 收集你的 ship URL 和登录码。
3. 配置 `channels.tlon`。
4. 重启 Gateway。
5. 向机器人发送私信或在群组频道中提及它。

最小配置（单个账户）：

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

## 群组频道

默认启用自动发现。你也可以手动固定频道：

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

禁用自动发现：

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## 访问控制

私信允许列表（空 = 允许所有）：

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

群组授权（默认受限）：

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## 投递目标（CLI/cron）

将这些与 `openclaw message send` 或 cron 投递一起使用：

- 私信：`~sampel-palnet` 或 `dm/~sampel-palnet`
- 群组：`chat/~host-ship/channel` 或 `group:~host-ship/channel`

## 说明

- 群组回复需要提及（例如 `~your-bot-ship`）才能回复。
- 线程回复：如果入站消息在线程中，OpenClaw 在线程中回复。
- 媒体：`sendMedia` 回退到文本 + URL（本机不上传）。
