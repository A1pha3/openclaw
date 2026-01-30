# WhatsApp 配置

WhatsApp 是 Moltbot 最常用的消息渠道之一，通过 Baileys 库实现 WhatsApp Web 协议。

## 工作原理

Moltbot 使用 WhatsApp Web 协议连接，就像在浏览器中使用 WhatsApp Web 一样：

```
手机 WhatsApp ←→ WhatsApp 服务器 ←→ Moltbot (Baileys)
```

> **注意**: 这是基于 WhatsApp Web 的实现，需要手机保持在线。

## 快速开始

### 1. 扫码登录

```bash
moltbot channels login
```

终端会显示二维码，使用手机扫描：

1. 打开手机 WhatsApp
2. 进入 **设置** → **已连接设备**
3. 点击 **连接设备**
4. 扫描终端中的二维码

### 2. 验证连接

```bash
moltbot channels status whatsapp
```

应该显示 `connected` 状态。

### 3. 发送测试消息

```bash
moltbot message send --channel whatsapp --target +15555550123 --message "测试消息"
```

## 基础配置

### 最小配置

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"]  // 您的手机号
    }
  }
}
```

### 完整配置示例

```json5
{
  channels: {
    whatsapp: {
      // 基础设置
      enabled: true,
      
      // DM 策略
      dmPolicy: "pairing",  // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      
      // 群组设置
      groupPolicy: "allowlist",
      groups: {
        "*": { requireMention: true },
        "120363403215116621@g.us": {
          allowFrom: ["+15555550123"],
          systemPrompt: "保持回复简短。"
        }
      },
      
      // 消息设置
      textChunkLimit: 4000,
      chunkMode: "length",  // length | newline
      mediaMaxMb: 50,
      
      // 已读回执
      sendReadReceipts: true
    }
  }
}
```

## DM 策略

控制如何处理私信：

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| `pairing` | 未知用户需要配对审批 | 个人使用（推荐） |
| `allowlist` | 仅允许白名单用户 | 严格控制访问 |
| `open` | 允许所有用户 | 公开服务 |
| `disabled` | 禁用所有私信 | 仅群组使用 |

### 配对流程

使用 `pairing` 策略时：

1. 未知用户发送消息
2. 系统返回配对码
3. 管理员审批配对

```bash
# 查看待审批的配对
moltbot pairing list whatsapp

# 审批配对
moltbot pairing approve whatsapp ABC123

# 拒绝配对
moltbot pairing reject whatsapp ABC123
```

## 群组配置

### 群组策略

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",  // allowlist | open | disabled
      groups: {
        // 默认规则（所有群组）
        "*": { 
          requireMention: true  // 需要 @提及 才响应
        },
        
        // 特定群组规则
        "120363403215116621@g.us": {
          allowFrom: ["+15555550123"],  // 发言人白名单
          requireMention: false,         // 所有消息都响应
          systemPrompt: "这是工作群，保持专业。"
        }
      }
    }
  }
}
```

### 提及触发

配置触发词：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@clawd", "小助手", "机器人"]
        }
      }
    ]
  }
}
```

### 自聊模式

当 `allowFrom` 包含自己的号码时，启用自聊模式：

- 忽略原生 @提及（防止自己触发自己）
- 仅响应 `mentionPatterns` 中的文本触发词

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],  // 您自己的号码
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["reisponde", "@clawd"]  // 仅这些词触发
        }
      }
    ]
  }
}
```

## 多账号配置

运行多个 WhatsApp 账号：

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        business: {
          // 可选：自定义认证目录
          // authDir: "~/.clawdbot/credentials/whatsapp/business"
        }
      }
    }
  }
}
```

每个账号需要单独扫码登录：

```bash
# 登录默认账号
moltbot channels login --account default

# 登录其他账号
moltbot channels login --account business
```

## 消息设置

### 消息分块

长消息会自动分块发送：

```json5
{
  channels: {
    whatsapp: {
      textChunkLimit: 4000,  // 最大字符数
      chunkMode: "length"    // length | newline
    }
  }
}
```

### 已读回执

```json5
{
  channels: {
    whatsapp: {
      sendReadReceipts: true  // 发送蓝勾
    }
  }
}
```

### 媒体限制

```json5
{
  channels: {
    whatsapp: {
      mediaMaxMb: 50  // 最大媒体大小 (MB)
    }
  }
}
```

## 常用命令

### 状态检查

```bash
# 查看连接状态
moltbot channels status whatsapp

# 带探测的深度检查
moltbot channels status whatsapp --probe
```

### 重新登录

```bash
# 注销当前会话
moltbot channels logout whatsapp

# 重新登录
moltbot channels login
```

### 发送消息

```bash
# 发送文本
moltbot message send --channel whatsapp --target +15555550123 --message "你好"

# 发送到群组
moltbot message send --channel whatsapp --target "120363403215116621@g.us" --message "群消息"
```

## 故障排除

### 二维码无法扫描

- 确保终端支持显示 Unicode 字符
- 尝试调大终端窗口
- 使用 `--qr-format ascii` 选项

### 连接频繁断开

- 检查网络稳定性
- 确保手机 WhatsApp 在线
- 查看日志：`moltbot logs --tail 100`

### 收不到消息

1. 检查 `allowFrom` 配置
2. 检查 `dmPolicy` 设置
3. 处理待定的配对请求
4. 确认群组 `requireMention` 设置

### 会话过期

WhatsApp 会话可能过期，需要重新登录：

```bash
moltbot channels logout whatsapp
moltbot channels login
```

## 最佳实践

### 个人使用

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

### 团队使用

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123", "+447700900123"],
      groupPolicy: "allowlist",
      groups: {
        "TEAM_GROUP_ID@g.us": {
          allowFrom: ["*"],
          requireMention: true
        }
      }
    }
  }
}
```

### 安全建议

1. 不要在公开服务器使用 `dmPolicy: "open"`
2. 定期检查配对请求
3. 使用 `groupPolicy: "allowlist"` 限制群组访问
4. 备份 `~/.clawdbot/credentials/whatsapp/` 目录

## 下一步

- [Telegram 配置](/zh-cn/channels/telegram)
- [Discord 配置](/zh-cn/channels/discord)
- [渠道概述](/zh-cn/channels/index)
