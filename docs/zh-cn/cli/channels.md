---
summary: "`openclaw channels` 命令参考（账户、状态、登录/登出、日志）"
read_when:
  - 想要添加/删除渠道账户（WhatsApp/Telegram/Discord/Google Chat/Slack/Signal/iMessage）
  - 想要检查渠道状态或查看渠道日志
title: "channels"
---

# `openclaw channels`

管理聊天渠道账户及其在网关上的运行状态。

## 为什么需要这个命令

渠道是 OpenClaw 与外部世界连接的桥梁：

- **统一管理**：一个命令管理所有消息渠道
- **状态监控**：实时查看各渠道连接状态
- **快速调试**：查看日志、检测能力、解析名称
- **灵活配置**：支持多账户和动态添加/删除

## 相关链接

- 渠道指南：[Channels](/zh-cn/channels/index)
- 网关配置：[Configuration](/zh-cn/gateway/configuration)

## 常用命令

```bash
# 列出所有渠道
openclaw channels list

# 查看渠道状态
openclaw channels status

# 检查渠道能力
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123

# 解析名称到 ID
openclaw channels resolve --channel slack "#general" "@jane"

# 查看渠道日志
openclaw channels logs --channel all
```

## 添加/删除账户

```bash
# 添加 Telegram 账户
openclaw channels add --channel telegram --token <bot-token>

# 删除渠道（包括凭据）
openclaw channels remove --channel telegram --delete
```

**提示**：运行 `openclaw channels add --help` 查看各渠道的特定选项（token、app token、signal-cli 路径等）。

## 登录/登出（交互式）

```bash
# WhatsApp 扫码登录
openclaw channels login --channel whatsapp

# 登出
openclaw channels logout --channel whatsapp
```

## 能力探测

获取渠道能力提示（可用的 intents/scopes）和静态功能支持：

```bash
# 所有渠道
openclaw channels capabilities

# 指定渠道和目标
openclaw channels capabilities --channel discord --target channel:123
```

**说明**：

- `--channel` 可选；省略时列出所有渠道（包括扩展）
- `--target` 接受 `channel:<id>` 或原始数字 ID，仅适用于 Discord
- 探测因渠道而异：
  - Discord：intents + 可选频道权限
  - Slack：bot + user scopes
  - Telegram：bot 标志 + webhook
  - Signal：daemon 版本
  - MS Teams：app token + Graph roles/scopes

## 解析名称到 ID

使用渠道目录将名称解析为 ID：

```bash
# Slack
openclaw channels resolve --channel slack "#general" "@jane"

# Discord
openclaw channels resolve --channel discord "My Server/#support" "@someone"

# Matrix
openclaw channels resolve --channel matrix "Project Room"
```

**选项**：

- `--kind user|group|auto`：强制指定目标类型
- 当有多个同名条目时，优先匹配活跃的

## 渠道状态表

| 状态 | 含义 |
|------|------|
| `connected` | 正常连接 |
| `disconnected` | 已断开 |
| `connecting` | 正在连接 |
| `error` | 连接错误 |
| `paused` | 已暂停 |

## 故障排查

```bash
# 运行深度探测
openclaw status --deep

# 引导式修复
openclaw doctor

# 查看实时日志
openclaw channels logs --channel whatsapp --follow
```

### 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| Claude: HTTP 403 | 缺少 `user:profile` scope | 使用 `--no-usage`，或提供 session key |
| WhatsApp 断开 | 会话过期 | 重新登录 `openclaw channels login --channel whatsapp` |
| Telegram 无响应 | Token 错误 | 检查 bot token 配置 |
| Discord 权限不足 | Bot 缺少 intents | 在 Discord Developer Portal 启用所需 intents |

## 渠道特定配置

### WhatsApp

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: ["*"], // 允许所有群组
    },
  },
}
```

### Telegram

```json5
{
  channels: {
    telegram: {
      botToken: "123456:ABCDEF",
      groups: {
        "*": { requireMention: true },
      },
    },
  },
}
```

### Discord

```json5
{
  channels: {
    discord: {
      token: "1234abcd",
      dm: {
        policy: "pairing",
        allowFrom: [],
      },
    },
  },
}
```

### Signal

需要安装并配置 `signal-cli`，详见 [Signal 渠道指南](/zh-cn/channels/signal)。

### iMessage

仅 macOS，需要登录 Messages 应用。详见 [iMessage 渠道指南](/zh-cn/channels/imessage)。
