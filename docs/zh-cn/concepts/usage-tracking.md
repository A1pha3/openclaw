---
summary: "使用量跟踪表面和凭证要求"
read_when:
  - 你正在连接提供程序使用量/配额表面
  - 你需要解释使用量跟踪行为或认证要求
title: "使用量跟踪"
---

# 使用量跟踪

## 它是什么

- 直接从提供程序的使用量/配额端点拉取。
- 没有估计成本；只有提供程序报告的窗口。

## 它显示在哪里

- `/status` 在聊天中：带有会话令牌的丰富表情符号状态卡 + 估计成本（仅限 API 密钥）。当可用时，提供程序使用量显示**当前模型提供程序**。
- `/usage off|tokens|full` 在聊天中：每个回复的使用量页脚（OAuth 仅显示令牌）。
- `/usage cost` 在聊天中：从 OpenClaw 会话日志聚合的本地成本摘要。
- CLI：`openclaw status --usage` 打印完整的每个提供程序分解。
- CLI：`openclaw channels list` 打印与提供程序配置并列的相同使用量快照（使用 `--no-usage` 跳过）。
- macOS 菜单栏：上下文下的"使用量"部分（仅当可用时）。

## 提供程序 + 凭证

- **Anthropic (Claude)**：认证配置文件中的 OAuth 令牌。
- **GitHub Copilot**：认证配置文件中的 OAuth 令牌。
- **Gemini CLI**：认证配置文件中的 OAuth 令牌。
- **Antigravity**：认证配置文件中的 OAuth 令牌。
- **OpenAI Codex**：认证配置文件中的 OAuth 令牌（存在时使用 accountId）。
- **MiniMax**：API 密钥（编码计划密钥；`MINIMAX_CODE_PLAN_KEY` 或 `MINIMAX_API_KEY`）；使用 5 小时编码计划窗口。
- **z.ai**：通过 env/config/认证存储的 API 密钥。

如果没有匹配的 OAuth/API 凭证，则隐藏使用量。
