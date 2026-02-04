---
summary: "监控模型提供商的 OAuth 过期"
read_when:
  - "设置身份验证过期监控或警报"
  - "自动化 Claude Code / Codex OAuth 刷新检查"
title: "身份验证监控"
---

# 身份验证监控

OpenClaw 通过 `openclaw models status` 暴露 OAuth 过期健康状态。将其用于自动化和警报；脚本是电话工作流程的可选附加项。

## 首选：CLI 检查（可移植）

```bash
openclaw models status --check
```

退出代码：

- `0`：正常
- `1`：凭据已过期或缺失
- `2`：即将过期（24 小时内）

这在 cron/systemd 中工作，不需要额外的脚本。

## 可选脚本（运维 / 电话工作流程）

这些位于 `scripts/` 下，是**可选的**。它们假设对网关主机有 SSH 访问权限，并为 systemd + Termux 进行了调整。

- `scripts/claude-auth-status.sh` 现在使用 `openclaw models status --json` 作为真实来源（如果 CLI 不可用，则回退到直接文件读取），因此请保持 `openclaw` 在 `PATH` 中以用于计时器。
- `scripts/auth-monitor.sh`：cron/systemd 计时器目标；发送警报（ntfy 或电话）。
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`：systemd 用户计时器。
- `scripts/claude-auth-status.sh`：Claude Code + OpenClaw 身份验证检查器（完整/json/简单）。
- `scripts/mobile-reauth.sh`：通过 SSH 引导的重新身份验证流程。
- `scripts/termux-quick-auth.sh`：一键小部件状态 + 打开身份验证 URL。
- `scripts/termux-auth-widget.sh`：完整的引导小部件流程。
- `scripts/termux-sync-widget.sh`：同步 Claude Code 凭据 → OpenClaw。

如果你不需要电话自动化或 systemd 计时器，请跳过这些脚本。
