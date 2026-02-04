---
summary: "`openclaw pairing` 命令参考（批准/列出配对请求）"
read_when:
  - 使用配对模式私信并需要批准发送者
title: "pairing"
---

# `openclaw pairing`

批准或检查私信配对请求（适用于支持配对的渠道）。

## 为什么需要配对

配对是 OpenClaw 的安全机制：

- **防止未授权访问**：陌生人无法直接与你的代理对话
- **控制访问权限**：你决定谁可以使用你的代理
- **简单验证流程**：发送者只需提供一个短配对码

## 工作原理

1. 陌生发送者向你的 Bot 发送消息
2. Bot 回复一个短配对码（例如 `A1B2`）
3. 你使用 `openclaw pairing approve` 批准该发送者
4. 发送者被添加到本地允许列表
5. 之后发送者可以正常与代理对话

## 相关链接

- 配对流程：[Pairing](/zh-cn/start/pairing)
- 安全指南：[Security](/zh-cn/gateway/security)

## 命令

### 列出配对请求

```bash
openclaw pairing list whatsapp
openclaw pairing list telegram
openclaw pairing list discord
```

显示指定渠道的待处理配对请求。

**输出示例**：

```
Pending pairing requests for whatsapp:

Code    Sender          Time
────    ──────          ────
A1B2    +15555550123    2 minutes ago
C3D4    +15555550456    5 minutes ago
```

### 批准配对请求

```bash
openclaw pairing approve whatsapp <code>
openclaw pairing approve whatsapp <code> --notify
```

批准指定渠道的配对请求。

| 选项 | 说明 |
|------|------|
| `<code>` | 配对码（例如 `A1B2`） |
| `--notify` | 批准后通知发送者 |

**示例**：

```bash
# 批准 WhatsApp 配对请求
openclaw pairing approve whatsapp A1B2

# 批准并通知发送者
openclaw pairing approve whatsapp A1B2 --notify
```

**输出**：

```
✓ Approved pairing request A1B2
  Sender: +15555550123
  Channel: whatsapp
```

## 配对策略配置

在配置文件中设置私信策略：

```json5
{
  channels: {
    whatsapp: {
      dm: {
        policy: "pairing", // "pairing" | "open" | "closed"
        allowFrom: [],     // 已批准的发送者
      },
    },
    discord: {
      dm: {
        policy: "pairing",
        allowFrom: [],
      },
    },
  },
}
```

### 策略说明

| 策略 | 行为 |
|------|------|
| `pairing` | 默认。未知发送者收到配对码，需要批准 |
| `open` | 允许所有人（需要在 `allowFrom` 中包含 `"*"`） |
| `closed` | 拒绝所有未在允许列表中的发送者 |

## 支持的渠道

配对机制支持以下渠道：

- WhatsApp
- Telegram
- Discord
- Signal
- iMessage
- MS Teams

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 配对码未出现 | 策略非 pairing | 检查 `dm.policy` 配置 |
| 批准失败 | 配对码过期或错误 | 让发送者重新发送消息 |
| 批准后仍无法对话 | 网关需要重启 | 重启网关 |

## 安全最佳实践

1. **默认使用配对模式**：不要轻易使用 `open` 策略
2. **定期检查允许列表**：移除不再需要访问的用户
3. **使用 --notify**：让发送者知道他们已被批准
4. **运行 doctor**：`openclaw doctor` 会检测不安全的配置
