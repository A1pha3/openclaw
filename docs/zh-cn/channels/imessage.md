---
summary: 介绍 iMessage 渠道配置，包括概述、快速开始、工作原理、访问控制、白名单格式、群组类线程、独立机器人身份、远程 Mac 配置、媒体和附件
read_when:
  - 配置 iMessage 渠道时
  - 设置访问控制时
  - 配置远程 Mac 时
title: iMessage 配置
---

# iMessage 配置

iMessage 是 Apple 生态系统中的即时通讯服务，OpenClaw 通过 `imsg` 工具在 macOS 上支持 iMessage 渠道。

## 概述

- **集成方式**: 通过 imsg 外部 CLI（JSON-RPC over stdio）
- **平台要求**: 仅 macOS
- **路由模式**: 确定性路由，回复始终返回 iMessage
- **会话模式**: 私信共享代理主会话，群组按 chat_id 隔离

## 快速开始

### 前提条件

1. macOS 系统
2. Messages 应用已登录
3. 安装 imsg 工具
4. 授予必要的系统权限

### 安装 imsg

```bash
# 使用 Homebrew 安装
brew install steipete/tap/imsg
```

### 授予权限

OpenClaw 和 imsg 需要以下 macOS 权限：

1. **完全磁盘访问权限**：访问 Messages 数据库
2. **自动化权限**：发送消息时需要

在"系统设置 → 隐私与安全性"中授予相应权限。

### 最小配置

```json5
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "/usr/local/bin/imsg",
      "dbPath": "/Users/<用户名>/Library/Messages/chat.db"
    }
  }
}
```

### 启动网关

```bash
openclaw gateway run
```

首次启动时，macOS 可能会弹出权限请求对话框，请批准这些请求。

## 工作原理

- `imsg` 以 RPC 模式流式传输消息事件
- 网关将事件规范化为统一信封格式
- 回复始终路由回相同的 chat_id 或 handle

```
Messages App → imsg rpc → stdio → Gateway → Agent
                                     ↓
Messages App ← imsg rpc ← stdio ←────┘
```

## 访问控制

### 私信策略

```json5
{
  "channels": {
    "imessage": {
      "dmPolicy": "pairing",           // pairing | allowlist | open | disabled
      "allowFrom": ["+15557654321"]    // 允许的号码/邮箱
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `pairing` | 默认。未知发送者收到配对码，审批后才处理消息 |
| `allowlist` | 仅处理白名单中的消息 |
| `open` | 处理所有消息（需要 `allowFrom: ["*"]`） |
| `disabled` | 禁用所有私信 |

### 配对流程

```bash
# 查看待审批的配对请求
openclaw pairing list imessage

# 审批配对
openclaw pairing approve imessage <CODE>
```

配对码在 1 小时后过期。

### 群组策略

```json5
{
  "channels": {
    "imessage": {
      "groupPolicy": "allowlist",          // open | allowlist | disabled
      "groupAllowFrom": ["+15557654321"]   // 群组中允许触发的号码
    }
  }
}
```

### 提及触发

iMessage 没有原生提及元数据，使用文本模式匹配：

```json5
{
  "agents": {
    "list": [
      {
        "id": "main",
        "groupChat": {
          "mentionPatterns": ["@clawd", "小助手", "openclaw"]
        }
      }
    ]
  }
}
```

## 白名单格式

iMessage 白名单支持多种标识符格式：

```json5
{
  "channels": {
    "imessage": {
      "allowFrom": [
        "+15557654321",           // E.164 电话号码
        "user@icloud.com",        // 邮箱地址
        "chat_id:42"              // chat_id
      ]
    }
  }
}
```

## 群组类线程

某些 iMessage 线程可能有多个参与者但仍以 `is_group=false` 到达。

如果您在 `channels.imessage.groups` 下显式配置 `chat_id`，OpenClaw 会将该线程视为"群组"：

```json5
{
  "channels": {
    "imessage": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["+15555550123"],
      "groups": {
        "42": { "requireMention": false }
      }
    }
  }
}
```

这对于特定线程使用隔离的会话/模型很有用。

## 独立机器人身份

如果希望机器人使用**独立的 iMessage 身份**（保持个人 Messages 清洁），可以使用专用 Apple ID + 专用 macOS 用户。

### 设置步骤

1. 创建专用 Apple ID（例如：`my-cool-bot@icloud.com`）
2. 创建 macOS 用户（例如：`clawdshome`）
3. 在该用户中登录 Messages
4. 启用远程登录（系统设置 → 通用 → 共享 → 远程登录）
5. 设置 SSH 免密登录

### SSH 包装脚本

创建包装脚本（`chmod +x`）：

```bash
#!/usr/bin/env bash
set -euo pipefail

# 首次运行前执行交互式 SSH 接受主机密钥：
#   ssh <bot-macos-user>@localhost true

exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

### 配置示例

```json5
{
  "channels": {
    "imessage": {
      "enabled": true,
      "accounts": {
        "bot": {
          "name": "机器人",
          "enabled": true,
          "cliPath": "/path/to/imsg-bot",
          "dbPath": "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

## 远程 Mac 配置

如果网关运行在其他机器上，可以通过 SSH 连接到远程 Mac 上的 imsg。

### 基本配置

```json5
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "~/.clawdbot/scripts/imsg-ssh",
      "remoteHost": "user@gateway-host",      // SCP 传输用
      "includeAttachments": true,
      "dbPath": "/Users/user/Library/Messages/chat.db"
    }
  }
}
```

### SSH 包装脚本

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Tailscale 远程 Mac 示例

通过 Tailscale 连接远程 Mac：

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ 网关主机 (Linux/VM)          │──────────────────────────────────▶│ Mac (Messages + imsg)    │
│ - openclaw gateway           │          SCP (附件)               │ - Messages 已登录         │
│ - channels.imessage.cliPath │◀──────────────────────────────────│ - 远程登录已启用          │
└──────────────────────────────┘                                   └──────────────────────────┘
               ▲
               │ Tailscale tailnet
               ▼
         bot@mac-mini.tailnet-1234.ts.net
```

配置示例：

```json5
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "~/.clawdbot/scripts/imsg-ssh",
      "remoteHost": "bot@mac-mini.tailnet-1234.ts.net",
      "includeAttachments": true,
      "dbPath": "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

包装脚本：

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

## 媒体和附件

### 附件配置

```json5
{
  "channels": {
    "imessage": {
      "includeAttachments": true,  // 将附件纳入上下文
      "mediaMaxMb": 16             // 媒体大小限制（MB，默认 16）
    }
  }
}
```

### 远程附件

当 `cliPath` 指向远程主机时，设置 `remoteHost` 以通过 SCP 获取附件：

```json5
{
  "channels": {
    "imessage": {
      "cliPath": "~/imsg-ssh",
      "remoteHost": "user@gateway-host",
      "includeAttachments": true
    }
  }
}
```

## 消息限制

### 文本分块

```json5
{
  "channels": {
    "imessage": {
      "textChunkLimit": 4000,     // 单条消息最大字符数（默认 4000）
      "chunkMode": "length"       // length | newline
    }
  }
}
```

### 历史记录

```json5
{
  "channels": {
    "imessage": {
      "historyLimit": 50,         // 群组历史消息数
      "dmHistoryLimit": 20        // 私信历史限制
    }
  }
}
```

按用户配置：

```json5
{
  "channels": {
    "imessage": {
      "dms": {
        "+15557654321": {
          "historyLimit": 30
        }
      }
    }
  }
}
```

## 投递目标

发送消息时的目标格式：

| 类型 | 格式 | 示例 |
|------|------|------|
| chat_id（推荐） | `chat_id:<id>` | `chat_id:123` |
| chat_guid | `chat_guid:<guid>` | `chat_guid:...` |
| chat_identifier | `chat_identifier:<id>` | `chat_identifier:...` |
| iMessage | `imessage:<handle>` | `imessage:+15551234567` |
| SMS | `sms:<handle>` | `sms:+15551234567` |
| 邮箱 | 直接使用 | `user@example.com` |

### 查看聊天列表

```bash
imsg chats --limit 20
```

### 发送消息示例

```bash
openclaw message send --to "chat_id:123" --message "你好"
openclaw message send --to "imessage:+15551234567" --message "你好"
```

## 多账号配置

```json5
{
  "channels": {
    "imessage": {
      "accounts": {
        "personal": {
          "name": "个人账号",
          "enabled": true,
          "cliPath": "/usr/local/bin/imsg",
          "dbPath": "/Users/me/Library/Messages/chat.db",
          "allowFrom": ["+15557654321"]
        },
        "bot": {
          "name": "机器人账号",
          "enabled": true,
          "cliPath": "/path/to/imsg-bot",
          "dbPath": "/Users/bot/Library/Messages/chat.db",
          "allowFrom": ["*"]
        }
      }
    }
  }
}
```

## 配置写入

默认允许通过 `/config set|unset` 命令更新配置。

禁用配置写入：

```json5
{
  "channels": {
    "imessage": {
      "configWrites": false
    }
  }
}
```

## 完整配置参考

### 提供者选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enabled` | boolean | false | 启用/禁用渠道 |
| `cliPath` | string | - | imsg 路径 |
| `dbPath` | string | - | Messages 数据库路径 |
| `remoteHost` | string | - | SSH 主机（用于 SCP 传输附件） |
| `service` | string | "auto" | imessage \| sms \| auto |
| `region` | string | - | SMS 区域 |

### 访问控制选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `dmPolicy` | string | "pairing" | pairing \| allowlist \| open \| disabled |
| `allowFrom` | string[] | [] | 私信白名单 |
| `groupPolicy` | string | "allowlist" | open \| allowlist \| disabled |
| `groupAllowFrom` | string[] | [] | 群组发送者白名单 |

### 消息选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `textChunkLimit` | number | 4000 | 文本分块大小 |
| `chunkMode` | string | "length" | length \| newline |
| `includeAttachments` | boolean | false | 纳入附件 |
| `mediaMaxMb` | number | 16 | 媒体大小限制（MB） |
| `historyLimit` | number | - | 群组历史消息数 |
| `dmHistoryLimit` | number | - | 私信历史限制 |

### 相关全局配置

| 配置项 | 说明 |
|--------|------|
| `agents.list[].groupChat.mentionPatterns` | 群组提及模式 |
| `messages.groupChat.mentionPatterns` | 全局回退提及模式 |
| `messages.responsePrefix` | 响应前缀 |

## 故障排除

### imsg 无法启动

1. 检查 Messages 是否已登录
2. 验证完全磁盘访问权限
3. 尝试手动运行：`imsg chats --limit 1`

### 权限弹窗

首次运行时可能需要在 GUI 中批准权限。如果使用独立用户：

1. 通过屏幕共享登录到机器人用户
2. 运行 `imsg chats --limit 1`
3. 批准弹出的权限请求

### 消息发送失败

1. 检查自动化权限
2. 验证目标格式是否正确
3. 确认 Messages 应用已登录

### 远程附件获取失败

1. 验证 `remoteHost` 配置
2. 确认 SSH 免密登录正常
3. 检查远程 Mac 上的文件权限

### 收不到消息

1. 检查 allowFrom 配置
2. 确认 DM/群组策略
3. 验证配对状态：`openclaw pairing list imessage`

## 相关文档

- [渠道概述](/zh-CN/channels)
- [配置参考](/zh-CN/config/reference)
- [配对机制](/zh-CN/start/quick-start#配对)
- [故障排除](/zh-CN/operations/troubleshooting)
