---
summary: "CLI 参考 - `openclaw doctor` (健康检查 + 引导修复)"
read_when:
  - 遇到连接或认证问题需要引导修复
  - 更新后想进行完整性检查
title: "doctor"
---

# `openclaw doctor`

健康检查 + 快速修复网关和渠道问题。

## 为什么需要这个命令

`doctor` 是你的私人诊断助手：

- **自动检测**：扫描配置、连接、认证等常见问题
- **引导修复**：不只是告诉你哪里出错，还会指导你怎么修
- **安全备份**：修复前自动备份配置文件

## 相关文档

- 故障排除：[故障排除](/zh-cn/gateway/troubleshooting)
- 安全审计：[安全](/zh-cn/gateway/security)

## 示例

```bash
# 基础诊断
openclaw doctor

# 诊断并修复
openclaw doctor --repair

# 深度检查（包括网络探测）
openclaw doctor --deep
```

## 选项

| 选项 | 说明 |
|------|------|
| `--repair` / `--fix` | 尝试自动修复发现的问题 |
| `--deep` | 深度检查（包括网络连接测试） |
| `--non-interactive` | 非交互模式（跳过所有提示） |

## 运行模式说明

- **交互式修复**（如 keychain/OAuth 修复）只在终端环境且未设置 `--non-interactive` 时运行
- **无头运行**（cron、Telegram、无终端）会跳过交互提示
- **修复备份**：`--fix` 会先备份到 `~/.openclaw/openclaw.json.bak`，然后移除未知配置项并列出每个删除项

## macOS：`launchctl` 环境变量覆盖

如果你之前运行过 `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`（或 `...PASSWORD`），该值会覆盖配置文件设置，可能导致持续的"未授权"错误。

检查并清除这些覆盖：

```bash
# 检查是否存在覆盖
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

# 清除覆盖
launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

## 常见诊断场景

### 更新后检查

```bash
# 更新后运行完整诊断
openclaw update && openclaw doctor
```

### 渠道连接问题

```bash
# 深度诊断渠道连接
openclaw doctor --deep
```

### 配置清理

```bash
# 修复并清理无效配置项
openclaw doctor --fix
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| "unauthorized" 错误持续出现 | 检查并清除 `launchctl` 环境变量覆盖 |
| 交互修复没有运行 | 确保在终端中运行且未使用 `--non-interactive` |
| 配置被意外清理 | 从 `~/.openclaw/openclaw.json.bak` 恢复 |
