---
summary: "`openclaw agent` 命令参考（通过网关运行一轮代理）"
read_when:
  - 想从脚本运行一轮代理（可选择投递回复）
title: "agent"
---

# `openclaw agent`

通过网关运行一轮代理对话（使用 `--local` 可切换为嵌入式模式）。使用 `--agent <id>` 可直接指定已配置的代理。

## 为什么需要这个命令

在日常使用中，你可能需要：

- **脚本自动化**：在 CI/CD 或定时任务中调用代理
- **快速测试**：不打开聊天界面，直接通过命令行与代理交互
- **跨渠道投递**：让代理生成回复后自动发送到 Slack、Discord 等渠道
- **调整思考深度**：控制代理的推理级别

## 相关链接

- 代理发送工具：[Agent send](/zh-cn/tools/agent-send)
- 会话概念：[会话](/zh-cn/concepts/session)

## 示例

```bash
# 向指定号码发送消息并投递回复
openclaw agent --to +15555550123 --message "状态更新" --deliver

# 使用已配置的代理
openclaw agent --agent ops --message "总结日志"

# 指定会话 ID 和思考级别
openclaw agent --session-id 1234 --message "总结收件箱" --thinking medium

# 生成报告并投递到 Slack 频道
openclaw agent --agent ops --message "生成报告" --deliver --reply-channel slack --reply-to "#reports"
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `--to <dest>` | 目标地址（电话号码或用户 ID） |
| `--message <msg>` | 发送给代理的消息 |
| `--agent <id>` | 使用指定的已配置代理 |
| `--session-id <id>` | 使用指定的会话 |
| `--deliver` | 将回复投递到渠道 |
| `--reply-channel <ch>` | 回复投递渠道（whatsapp/telegram/slack/discord 等） |
| `--reply-to <dest>` | 回复投递目标 |
| `--thinking <level>` | 思考级别：off/minimal/low/medium/high/xhigh |
| `--local` | 使用嵌入式模式而非网关 |

## 使用场景

### 定时任务

```bash
# 每天早上生成日报
0 9 * * * openclaw agent --agent daily --message "生成今日待办摘要" --deliver --reply-channel telegram --reply-to "@me"
```

### CI/CD 集成

```bash
# 部署后通知
openclaw agent --message "部署完成：${COMMIT_SHA}" --deliver --reply-channel slack --reply-to "#deployments"
```

### 快速问答

```bash
# 命令行快速提问
openclaw agent --message "Node.js 如何读取环境变量？"
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 连接失败 | 网关未运行 | 运行 `openclaw gateway run` |
| 代理未找到 | 代理 ID 错误 | 运行 `openclaw agents list` 查看可用代理 |
| 投递失败 | 渠道未配置 | 检查渠道配置，运行 `openclaw channels status` |
