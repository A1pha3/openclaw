---
summary: "直接 `openclaw agent` CLI 运行（带可选传递）"
read_when:
  - 添加或修改代理 CLI 入口点
title: "代理发送"
---

# `openclaw agent`（直接代理运行）

`openclaw agent` 运行单个代理轮次，无需入站聊天消息。默认它通过**网关**；添加 `--local` 以强制在当前机器上运行嵌入式运行时。

## 行为

- 必需：`--message <text>`
- 会话选择：
  - `--to <dest>` 派生会话密钥（组/频道目标保持隔离；直接聊天折叠到 `main`），**或者**
  - `--session-id <id>` 按 ID 重用现有会话，**或者**
  - `--agent <id>` 直接定位配置的代理（使用该代理的 `main` 会话密钥）
- 运行与正常入站回复相同的嵌入式代理运行时。
- Thinking/verbose 标志持久化到会话存储。
- 输出：
  - 默认：打印回复文本（加上 `MEDIA:<url>` 行）
  - `--json`：打印结构化 payload + 元数据
- 可选传递回带有 `--deliver` + `--channel` 的频道（目标格式匹配 `openclaw message --target`）。
- 使用 `--reply-channel`/`--reply-to`/`--reply-account` 覆盖传递而不更改会话。

如果网关不可达，CLI **回退**到嵌入式本地运行。

## 示例

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## 标志

- `--local`：本地运行（需要在你的 shell 中提供模型提供程序 API 密钥）
- `--deliver`：将回复发送到选定的频道
- `--channel`：传递频道（`whatsapp|telegram|discord|googlechat|slack|signal|imessage`，默认：`whatsapp`）
- `--reply-to`：传递目标覆盖
- `--reply-channel`：传递频道覆盖
- `--reply-account`：传递账户 ID 覆盖
- `--thinking <off|minimal|low|medium|high|xhigh>`：持久化思考级别（仅 GPT-5.2 + Codex 模型）
- `--verbose <on|full|off>`：持久化详细级别
- `--timeout <seconds>`：覆盖代理超时
- `--json`：输出结构化 JSON
