---
summary: "Zalo 机器人渠道配置 - 使用官方 Bot API"
read_when:
  - 配置 Zalo 机器人功能或 webhook
title: "Zalo"
---

# Zalo（Bot API）

**状态:** 实验性。仅支持私信；群聊功能按 Zalo 文档所述"即将推出"。

## 为什么需要这个

Zalo 是越南最流行的即时通讯应用。通过 Zalo Bot API 渠道，你可以：

- **客服支持**: 为越南用户提供自动化客服
- **通知推送**: 向用户发送系统通知
- **业务自动化**: 在 Zalo 上构建智能助手

## 安装插件

Zalo 作为插件提供，不包含在核心安装中。

```bash
# 从 npm 安装
openclaw plugins install @openclaw/zalo

# 或从本地源码安装
openclaw plugins install ./extensions/zalo
```

也可以在引导向导中选择 **Zalo** 并确认安装提示。

## 快速设置

### 1. 创建机器人令牌

1. 访问 **https://bot.zaloplatforms.com** 并登录
2. 创建新机器人并配置设置
3. 复制机器人令牌（格式：`12345689:abc-xyz`）

### 2. 配置令牌

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

或使用环境变量：`ZALO_BOT_TOKEN=...`

### 3. 重启网关

### 4. 完成配对

首次联系时，批准配对码即可开始使用。

## 工作原理

- 入站消息会被规范化为统一的渠道消息格式
- 回复始终路由回同一个 Zalo 聊天
- 默认使用长轮询；可配置 webhook 模式

## 限制

| 限制类型 | 说明 |
|---------|------|
| 文本长度 | 出站文本分块为 2000 字符（Zalo API 限制） |
| 媒体大小 | 由 `channels.zalo.mediaMaxMb` 控制（默认 5MB） |
| 流式传输 | 默认禁用（2000 字符限制使流式传输意义不大） |

## 访问控制

### 私信策略

| 策略 | 说明 |
|------|------|
| `pairing`（默认） | 未知发送者收到配对码，消息在批准前被忽略（1 小时过期） |
| `allowlist` | 仅 `allowFrom` 中的用户 ID 可发消息 |
| `open` | 开放公共私信（需要 `allowFrom: ["*"]`） |
| `disabled` | 忽略所有私信 |

### 批准配对

```bash
openclaw pairing list zalo
openclaw pairing approve zalo <CODE>
```

注意：`allowFrom` 接受数字用户 ID（无用户名查询功能）。

## 长轮询 vs Webhook

### 长轮询（默认）

无需公网 URL，开箱即用。

### Webhook 模式

设置以下配置启用：

```json5
{
  channels: {
    zalo: {
      webhookUrl: "https://your-domain.com/webhook/zalo",
      webhookSecret: "your-secret-8-256-chars",
    },
  },
}
```

要求：
- Webhook URL 必须使用 HTTPS
- Secret 必须是 8-256 个字符
- Zalo 使用 `X-Bot-Api-Secret-Token` 头进行验证

**注意:** 根据 Zalo API 文档，轮询和 webhook 互斥，不能同时使用。

## 支持的消息类型

| 类型 | 状态 |
|------|------|
| 文本消息 | ✅ 完全支持（2000 字符分块） |
| 图片消息 | ✅ 支持下载和发送 |
| 贴纸 | ⚠️ 记录但不处理 |
| 其他类型 | ⚠️ 仅记录 |

## 功能矩阵

| 功能 | 状态 |
|------|------|
| 私信 | ✅ 支持 |
| 群组 | ❌ 即将推出（Zalo 文档） |
| 媒体（图片） | ✅ 支持 |
| 表情回应 | ❌ 不支持 |
| 话题回复 | ❌ 不支持 |
| 投票 | ❌ 不支持 |
| 原生命令 | ❌ 不支持 |
| 流式传输 | ⚠️ 已禁用（2000 字符限制） |

## CLI 发送消息

```bash
openclaw message send --channel zalo --target 123456789 --message "你好"
```

## 配置参考

### 基础配置

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enabled` | boolean | - | 启用/禁用渠道 |
| `botToken` | string | - | Zalo Bot Platform 的机器人令牌 |
| `tokenFile` | string | - | 从文件读取令牌 |
| `dmPolicy` | string | `pairing` | 私信策略 |
| `allowFrom` | string[] | - | 允许的用户 ID 列表 |
| `mediaMaxMb` | number | 5 | 媒体大小限制（MB） |

### Webhook 配置

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `webhookUrl` | string | Webhook URL（需要 HTTPS） |
| `webhookSecret` | string | Webhook 密钥（8-256 字符） |
| `webhookPath` | string | 网关 HTTP 服务器上的 webhook 路径 |
| `proxy` | string | API 请求的代理 URL |

### 多账号配置

```json5
{
  channels: {
    zalo: {
      accounts: {
        work: {
          botToken: "token-for-work",
          name: "工作账号",
          dmPolicy: "pairing",
        },
        personal: {
          botToken: "token-for-personal",
          name: "个人账号",
        },
      },
    },
  },
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 机器人无响应 | 令牌无效 | 运行 `openclaw channels status --probe` |
| 机器人无响应 | 发送者未批准 | 检查配对或 allowFrom |
| Webhook 收不到事件 | URL 不是 HTTPS | 确保使用 HTTPS |
| Webhook 收不到事件 | Secret 格式错误 | 确保 8-256 字符 |
| Webhook 收不到事件 | 同时使用轮询 | 轮询和 webhook 互斥 |

## 相关资源

- 完整配置参考: [配置](/zh-cn/gateway/configuration)
- 配对流程: [配对](/zh-cn/start/pairing)
- 插件管理: [插件](/zh-cn/plugin)
