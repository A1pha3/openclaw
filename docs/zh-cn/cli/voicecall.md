---
summary: "`openclaw voicecall` 命令参考 - 语音通话插件命令"
read_when:
  - 使用语音通话插件
  - 想了解 voicecall call/continue/status/tail/expose 等命令
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` 是一个**插件命令**。只有安装并启用语音通话插件后才会出现。

## 为什么需要这个

语音通话插件让 OpenClaw 可以通过电话与用户对话。适用于：

- **通知场景**: 发送语音通知给用户
- **交互场景**: 进行语音对话，收集用户反馈
- **自动化场景**: 在工作流中加入语音环节

## 前置条件

1. 安装语音通话插件
2. 配置 Twilio 或其他 VoIP 提供商凭据
3. 确保有可用的电话号码

详细配置请参考: [语音通话插件](/zh-cn/plugins/voice-call)

## 常用命令

### 发起通话

```bash
# 发起通知型通话
openclaw voicecall call --to "+15555550123" --message "您好，这是一条通知" --mode notify

# 发起交互型通话
openclaw voicecall call --to "+15555550123" --message "您好，请问有什么可以帮您？" --mode interactive
```

### 管理通话

```bash
# 查看通话状态
openclaw voicecall status --call-id <id>

# 继续通话（发送新消息）
openclaw voicecall continue --call-id <id> --message "还有其他问题吗？"

# 结束通话
openclaw voicecall end --call-id <id>
```

### 暴露 Webhook（Tailscale）

语音通话需要接收来自 VoIP 提供商的回调。可以通过 Tailscale 安全暴露：

```bash
# 使用 Tailscale Serve（仅 tailnet 内可访问，推荐）
openclaw voicecall expose --mode serve

# 使用 Tailscale Funnel（公网可访问）
openclaw voicecall expose --mode funnel

# 关闭暴露
openclaw voicecall unexpose
```

## 安全提示

| 模式 | 可访问性 | 推荐场景 |
|------|---------|---------|
| `serve` | 仅 tailnet 内部 | 个人使用、测试环境 |
| `funnel` | 公网可访问 | 需要外部 VoIP 回调 |

> **重要**: 只在信任的网络中暴露 webhook 端点。尽可能使用 `serve` 模式而非 `funnel`。

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 命令不存在 | 插件未安装 | 运行 `openclaw plugins install voice-call` |
| 无法发起通话 | VoIP 凭据配置错误 | 检查 Twilio 配置 |
| 回调失败 | webhook 未暴露 | 运行 `openclaw voicecall expose` |

## 相关资源

- 语音通话插件详细配置: [Voice Call](/zh-cn/plugins/voice-call)
- Tailscale 配置: [Tailscale 指南](/zh-cn/gateway/tailscale)
- 插件管理: [plugins 命令](/zh-cn/cli/plugins)
