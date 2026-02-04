---
summary: "`openclaw webhooks` 命令参考（webhook 辅助 + Gmail Pub/Sub）"
read_when:
  - 想要将 Gmail Pub/Sub 事件连接到 OpenClaw
  - 想要使用 webhook 辅助命令
title: "webhooks"
---

# `openclaw webhooks`

Webhook 辅助工具和集成（Gmail Pub/Sub、webhook 辅助）。

## 为什么需要 Webhooks

Webhooks 让外部服务能够触发你的代理：

- **邮件通知**：收到新邮件时自动处理
- **CI/CD 集成**：构建完成时通知代理
- **自定义触发器**：任何支持 webhook 的服务都可以触发代理
- **自动化工作流**：构建复杂的事件驱动自动化

## 相关链接

- Webhook 概念：[Webhook](/zh-cn/automation/webhook)
- Gmail Pub/Sub：[Gmail Pub/Sub](/zh-cn/automation/gmail-pubsub)

## Gmail Pub/Sub

Gmail Pub/Sub 让你的代理能够实时响应邮件事件。

### 设置

```bash
openclaw webhooks gmail setup --account you@example.com
```

这将引导你完成：

1. 创建 Google Cloud 项目
2. 启用 Gmail API 和 Pub/Sub API
3. 配置 OAuth 凭据
4. 设置 Pub/Sub 主题和订阅
5. 授权 OpenClaw 访问你的 Gmail

### 运行

```bash
openclaw webhooks gmail run
```

启动 Gmail 事件监听器。收到新邮件时，代理会收到通知。

### 配置示例

```json5
{
  automation: {
    webhooks: {
      gmail: {
        enabled: true,
        account: "you@example.com",
        // 过滤规则
        filter: {
          // 只处理来自特定发送者的邮件
          from: ["important@example.com"],
          // 只处理特定标签的邮件
          labels: ["INBOX", "IMPORTANT"],
        },
      },
    },
  },
}
```

### Gmail 事件处理

当收到新邮件时，代理会收到类似这样的消息：

```
📧 New email from sender@example.com
Subject: Meeting Tomorrow
Preview: Hi, just wanted to confirm our meeting...
```

你可以配置代理自动：

- 总结邮件内容
- 提取关键信息
- 创建日历事件
- 转发到其他渠道

## 通用 Webhook

除了 Gmail，OpenClaw 还支持通用 webhook 端点。

### 配置

```json5
{
  automation: {
    webhooks: {
      endpoints: [
        {
          path: "/hooks/github",
          secret: "your-webhook-secret",
          handler: "github-events",
        },
        {
          path: "/hooks/stripe",
          secret: "whsec_xxx",
          handler: "payment-events",
        },
      ],
    },
  },
}
```

### 访问 Webhook

Webhook 端点在网关运行时可用：

```
http://localhost:18789/hooks/github
http://localhost:18789/hooks/stripe
```

如果使用 Tailscale Funnel 暴露网关：

```
https://your-machine.tailnet.ts.net/hooks/github
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| Gmail 设置失败 | OAuth 配置错误 | 检查 Google Cloud Console 配置 |
| 无法接收邮件 | Pub/Sub 订阅问题 | 检查 Google Cloud Pub/Sub 控制台 |
| Webhook 无响应 | 网关未运行或端口未暴露 | 检查网关状态和网络配置 |
| 认证失败 | secret 不匹配 | 检查 webhook secret 配置 |

## 安全注意事项

1. **始终使用 secret**：验证 webhook 请求来源
2. **限制 IP**：如果可能，限制 webhook 来源 IP
3. **HTTPS**：生产环境使用 HTTPS
4. **最小权限**：Gmail 只请求必要的权限
