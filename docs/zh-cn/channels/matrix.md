# Matrix 配置

Matrix 是一个开放的、去中心化的消息协议。OpenClaw 通过插件支持 Matrix 渠道，可以作为 Matrix 用户连接到任何 homeserver。

## 概述

- **集成方式**: 通过 @vector-im/matrix-bot-sdk 插件
- **连接模式**: 作为 Matrix 用户登录
- **支持功能**: 私信、房间、线程、媒体、表情回复、投票、位置、端到端加密（E2EE）
- **路由模式**: 确定性路由，回复始终返回 Matrix

## 插件安装

Matrix 渠道作为插件提供，不包含在核心安装中。

### 从 npm 安装

```bash
openclaw plugins install @openclaw/matrix
```

### 从本地安装

```bash
# Git 仓库本地开发
openclaw plugins install ./extensions/matrix
```

如果在配置/引导过程中选择 Matrix 并检测到 Git 仓库，OpenClaw 会自动提供本地安装路径选项。

## 快速开始

### 1. 安装插件

```bash
openclaw plugins install @openclaw/matrix
```

### 2. 创建 Matrix 账号

在任意 homeserver 上创建账号：
- 浏览托管选项：https://matrix.org/ecosystem/hosting/
- 或自行托管

### 3. 获取访问令牌

使用 Matrix 登录 API 获取访问令牌：

```bash
curl --request POST \
  --url https://matrix.example.org/_matrix/client/v3/login \
  --header 'Content-Type: application/json' \
  --data '{
  "type": "m.login.password",
  "identifier": {
    "type": "m.id.user",
    "user": "your-user-name"
  },
  "password": "your-password"
}'
```

或者配置用户名和密码，让 OpenClaw 自动获取并存储令牌。

### 4. 配置凭证

**环境变量方式**：

```bash
export MATRIX_HOMESERVER="https://matrix.example.org"
export MATRIX_ACCESS_TOKEN="syt_***"
# 或使用密码登录
export MATRIX_USER_ID="@bot:example.org"
export MATRIX_PASSWORD="your-password"
```

**配置文件方式**：

```json5
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.example.org",
      "accessToken": "syt_***",
      "dm": { "policy": "pairing" }
    }
  }
}
```

如果同时设置了环境变量和配置文件，配置文件优先。

### 5. 启动网关

```bash
openclaw gateway run
```

### 6. 开始使用

从任意 Matrix 客户端（Element、Beeper 等）与机器人开始私信或邀请它加入房间。

## 端到端加密（E2EE）

Matrix 支持端到端加密，通过 Rust crypto SDK 实现。

### 启用 E2EE

```json5
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.example.org",
      "accessToken": "syt_***",
      "encryption": true,
      "dm": { "policy": "pairing" }
    }
  }
}
```

### E2EE 功能说明

- 加密房间的消息自动解密
- 发送到加密房间的媒体自动加密
- 首次连接时，OpenClaw 请求其他会话的设备验证
- 需要在另一个 Matrix 客户端（如 Element）中验证设备以启用密钥共享

### 设备验证

启用 E2EE 后，机器人会在启动时请求验证。在 Element 或其他客户端中批准验证请求以建立信任。

### 加密状态存储

加密状态存储在：
`~/.clawdbot/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`

如果访问令牌（设备）更改，会创建新存储，需要重新验证。

### 常见问题

如果看到缺少 crypto 模块的错误：

```bash
# 允许构建脚本并重建
pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs

# 或手动下载二进制文件
node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js
```

## 访问控制

### 私信策略

```json5
{
  "channels": {
    "matrix": {
      "dm": {
        "policy": "pairing",           // pairing | allowlist | open | disabled
        "allowFrom": ["@user:example.org"]
      }
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `pairing` | 默认。未知发送者收到配对码 |
| `allowlist` | 仅处理白名单用户的消息 |
| `open` | 处理所有消息（需要 `allowFrom: ["*"]`） |
| `disabled` | 禁用所有私信 |

### 配对流程

```bash
# 查看待审批的配对请求
openclaw pairing list matrix

# 审批配对
openclaw pairing approve matrix <CODE>
```

### 房间（群组）策略

```json5
{
  "channels": {
    "matrix": {
      "groupPolicy": "allowlist",      // allowlist | open | disabled
      "groups": {
        "!roomId:example.org": { "allow": true },
        "#alias:example.org": { "allow": true }
      },
      "groupAllowFrom": ["@owner:example.org"]
    }
  }
}
```

### 房间配置选项

```json5
{
  "channels": {
    "matrix": {
      "groups": {
        "!roomId:example.org": {
          "allow": true,
          "requireMention": false,     // 禁用提及要求，启用自动回复
          "users": ["@allowed:example.org"]  // 房间内发送者白名单
        },
        "*": {
          "requireMention": true       // 所有房间的默认设置
        }
      }
    }
  }
}
```

### 自动加入房间

```json5
{
  "channels": {
    "matrix": {
      "autoJoin": "allowlist",         // always | allowlist | off
      "autoJoinAllowlist": [
        "!roomId:example.org",
        "#alias:example.org"
      ]
    }
  }
}
```

## 线程支持

Matrix 支持回复线程：

```json5
{
  "channels": {
    "matrix": {
      "threadReplies": "inbound",      // off | inbound | always
      "replyToMode": "off"             // off | first | all
    }
  }
}
```

| threadReplies | 说明 |
|---------------|------|
| `off` | 不使用线程 |
| `inbound` | 默认。回复保持在入站消息的线程中 |
| `always` | 总是使用线程回复 |

| replyToMode | 说明 |
|-------------|------|
| `off` | 默认。不添加 reply-to 元数据 |
| `first` | 回复第一条消息 |
| `all` | 回复所有消息 |

## 功能支持

| 功能 | 状态 |
|------|------|
| 私信 | 支持 |
| 房间 | 支持 |
| 线程 | 支持 |
| 媒体 | 支持 |
| E2EE | 支持（需要 crypto 模块） |
| 表情回复 | 支持（通过工具发送/读取） |
| 投票 | 支持发送；入站投票开始转为文本 |
| 位置 | 支持（geo URI；忽略海拔） |
| 原生命令 | 支持 |

## 多账号配置

```json5
{
  "channels": {
    "matrix": {
      "accounts": {
        "main": {
          "name": "主账号",
          "enabled": true,
          "homeserver": "https://matrix.example.org",
          "accessToken": "syt_***",
          "encryption": true,
          "dm": { "policy": "pairing" }
        },
        "work": {
          "name": "工作账号",
          "enabled": true,
          "homeserver": "https://matrix.work.org",
          "accessToken": "syt_***",
          "dm": { "policy": "allowlist", "allowFrom": ["@boss:work.org"] }
        }
      }
    }
  }
}
```

## 完整配置参考

### 提供者选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enabled` | boolean | false | 启用/禁用渠道 |
| `homeserver` | string | - | Homeserver URL |
| `userId` | string | - | Matrix 用户 ID（使用令牌时可选） |
| `accessToken` | string | - | 访问令牌 |
| `password` | string | - | 登录密码（令牌会被存储） |
| `deviceName` | string | - | 设备显示名称 |
| `encryption` | boolean | false | 启用 E2EE |
| `initialSyncLimit` | number | - | 初始同步限制 |

### 消息选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `textChunkLimit` | number | - | 文本分块大小 |
| `chunkMode` | string | "length" | length \| newline |
| `threadReplies` | string | "inbound" | off \| inbound \| always |
| `replyToMode` | string | "off" | off \| first \| all |
| `mediaMaxMb` | number | - | 媒体大小限制（MB） |

### 访问控制选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `dm.policy` | string | "pairing" | pairing \| allowlist \| open \| disabled |
| `dm.allowFrom` | string[] | [] | 私信白名单（用户 ID 或显示名称） |
| `groupPolicy` | string | "allowlist" | allowlist \| open \| disabled |
| `groupAllowFrom` | string[] | [] | 群组发送者白名单 |
| `groups` | object | - | 群组配置和白名单 |
| `autoJoin` | string | "always" | always \| allowlist \| off |
| `autoJoinAllowlist` | string[] | [] | 自动加入的房间白名单 |

### 工具控制

```json5
{
  "channels": {
    "matrix": {
      "actions": {
        "reactions": true,
        "messages": true,
        "pins": true,
        "memberInfo": true,
        "channelInfo": true
      }
    }
  }
}
```

## Beeper 客户端

Beeper 是一个流行的 Matrix 客户端，但它要求启用 E2EE：

```json5
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.beeper.com",
      "accessToken": "syt_***",
      "encryption": true,
      "dm": { "policy": "pairing" }
    }
  }
}
```

配置完成后，在 Beeper 中验证设备以启用消息收发。

## 故障排除

### 无法连接到 homeserver

1. 验证 homeserver URL 是否正确
2. 检查网络连接
3. 确认访问令牌有效

### E2EE 消息无法解密

1. 确认 `encryption: true` 已设置
2. 在其他客户端中验证设备
3. 检查 crypto 模块是否正确安装

### 房间消息收不到

1. 确认机器人已加入房间
2. 检查 `groupPolicy` 和 `groups` 配置
3. 验证 `groupAllowFrom` 设置

### 私信收不到

1. 检查 `dm.policy` 和 `dm.allowFrom` 配置
2. 验证配对状态：`openclaw pairing list matrix`

## 相关文档

- [渠道概述](/zh-CN/channels)
- [配置参考](/zh-CN/config/reference)
- [插件系统](/zh-CN/developer/plugin-development)
- [故障排除](/zh-CN/operations/troubleshooting)
