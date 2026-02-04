---
summary: "Onboarding 流程详解 - 初始配置向导和工作区设置"
read_when:
  - 首次配置 OpenClaw
  - 理解向导流程
  - 配置 AI 认证和工作区
title: "Onboarding 流程"
---

# 🧙 Onboarding 流程

本文档详细介绍 OpenClaw 的初始化配置流程（Onboarding），帮助您从安装到首次使用的完整配置。

## 🎯 什么是 Onboarding？

**Onboarding** 是 OpenClaw 的引导配置流程，通过交互式向导帮助您完成：

```
安装 CLI
    │
    ▼
运行 onboarding
    │
    ├── 网关配置（本地/远程）
    │
    ├── AI 认证（API Key / OAuth）
    │
    ├── 渠道配置（WhatsApp/Telegram等）
    │
    ├── 技能选择（Web搜索等）
    │
    └── 后台服务（可选）
```

> **提示**：Onboarding 是推荐的首选配置方式，自动处理所有复杂设置。

---

## 🚀 启动 Onboarding

### 首次运行

```bash
# 完整引导（推荐）
openclaw onboard --install-daemon
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--install-daemon` | 安装后台服务（网关随系统启动） |
| `--reset` | 重置所有配置后重新引导 |
| `--non-interactive` | 非交互模式（使用默认配置或配置文件） |
| `--workspace <path>` | 指定工作区路径 |
| `--config <path>` | 指定配置文件路径 |

### 跳过 Onboarding

如果只想安装 CLI，稍后手动配置：

```bash
# 安装 CLI（跳过引导）
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard

# 或手动安装后运行
npm install -g openclaw@latest

# 后续手动运行引导
openclaw onboard
```

---

## 📋 Onboarding 步骤详解

### 步骤 1：网关配置

```
问题：您希望如何运行网关？
选项：
  1. 本地模式（推荐）- 仅本地访问
  2. 远程模式 - 通过网络访问
```

**本地模式**（默认）：
```json5
{
  gateway: {
    bind: "loopback",
    port: 18789
  }
}
```

**远程模式**：
```json5
{
  gateway: {
    bind: "lan",  // 或 "tailnet"（Tailscale）
    port: 18789
  }
}
```

> **远程访问**：需要额外配置令牌认证和网络安全策略。

### 步骤 2：AI 认证配置

```
问题：如何配置 AI 模型访问？
选项：
  1. Anthropic API Key
  2. Anthropic OAuth（Claude Code 用户）
  3. OpenAI API Key
  4. OpenAI OAuth（Codex 用户）
  5. 其他提供商
```

**推荐配置**（Anthropic API Key）：

```bash
# 输入 API Key
export ANTHROPIC_API_KEY="sk-ant-api03-..."
```

**OAuth 方式**（更安全，不需要存储 API Key）：

```bash
# 打开浏览器完成 OAuth 授权
# OpenClaw 会自动保存凭证
```

> **凭证存储位置**：
> - OAuth: `~/.openclaw/credentials/oauth.json`
> - API Key: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

### 步骤 3：渠道配置

```
问题：您想配置哪些消息渠道？
选项：
  ✓ WhatsApp（扫码登录）
  ✓ Telegram（Bot Token）
  ✓ Discord（Bot Token）
  ○ Slack
  ○ Signal
  ○ iMessage（仅 macOS）
```

**WhatsApp 登录**：
```bash
# Onboarding 中选择 WhatsApp 后
# 会自动显示二维码
# 用手机 WhatsApp 扫描即可
```

**Telegram 配置**：
```bash
# 输入 Bot Token
# 格式：123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

**Discord 配置**：
```bash
# 输入 Bot Token
# 需要先在 Discord Developer Portal 创建应用
```

### 步骤 4：技能选择

```
问题：您想启用哪些技能？
选项：
  ✓ Web Search（网页搜索）- 推荐
  ✓ GitHub（代码仓库访问）
  ○ 文件读写
  ○ 命令执行
  ○ 浏览器控制
```

### 步骤 5：后台服务配置

```
问题：是否安装后台服务？
选项：
  1. 是（推荐）- 网关随系统启动
  2. 否 - 手动启动网关
```

**后台服务类型**：

| 平台 | 服务类型 |
|------|----------|
| macOS | LaunchAgent |
| Linux | systemd 用户单元 |
| Windows | 服务（通过 WSL2） |

---

## 📁 工作区配置

### 工作区目录

Onboarding 会创建默认工作区：

```
~/.openclaw/workspace/
├── AGENTS.md       # 代理行为指南
├── SOUL.md         # 代理个性定义
├── USER.md         # 用户信息
├── MEMORY.md       # 长期记忆
├── TOOLS.md        # 工具说明
└── skills/         # 技能目录
```

### 自定义工作区

```bash
# 在 Onboarding 中指定
openclaw onboard --workspace ~/my-openclaw-workspace

# 或手动配置
openclaw config set agents.defaults.workspace ~/my-workspace
```

### 工作区文件说明

| 文件 | 用途 | 说明 |
|------|------|------|
| `AGENTS.md` | 代理行为 | 定义 AI 的工作方式和约束 |
| `SOUL.md` | 代理个性 | 定义 AI 的性格和沟通风格 |
| `USER.md` | 用户信息 | 告诉 AI 关于您的信息 |
| `MEMORY.md` | 长期记忆 | AI 会记住的重要信息 |
| `TOOLS.md` | 工具说明 | 自定义工具使用指南 |

---

## 🔧 重新运行 Onboarding

### 何时需要重新运行？

- 想要更改配置
- 遇到配置问题
- 添加新渠道
- 更新 AI 认证

### 重新运行

```bash
# 完整重新引导
openclaw onboard --reset

# 保留现有配置，仅运行向导
openclaw onboard
```

### 备份配置

重新运行前，建议备份：

```bash
# 备份配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 备份工作区
cp -r ~/.openclaw/workspace ~/workspace.backup
```

---

## 🖥️ 非交互模式

适合自动化脚本或 CI/CD：

```bash
# 使用配置文件
openclaw onboard --non-interactive --config ~/openclaw-config.json
```

**配置文件示例**（`openclaw-config.json`）：

```json5
{
  "gateway": {
    "bind": "loopback",
    "port": 18789
  },
  "auth": {
    "provider": "anthropic",
    "apiKey": "${ANTHROPIC_API_KEY}"
  },
  "channels": {
    "whatsapp": {
      "enabled": true
    },
    "telegram": {
      "enabled": true,
      "botToken": "${TELEGRAM_BOT_TOKEN}"
    }
  }
}
```

---

## 📊 Onboarding 状态

### 检查 Onboarding 完成状态

```bash
# 查看配置状态
openclaw status

# 检查认证状态
openclaw health

# 查看渠道状态
openclaw channels status
```

### 常见状态

| 状态 | 说明 |
|------|------|
| `✅ Configured` | 已配置 |
| `⚠️ No auth` | 未配置 AI 认证 |
| `❌ Gateway not running` | 网关未运行 |

---

## 🐛 故障排除

### Onboarding 失败

```bash
# 查看详细错误
openclaw doctor

# 检查日志
openclaw logs --lines 100
```

### 常见问题

**问题：API Key 无效**
```bash
# 重新运行 Onboarding
openclaw onboard --reset
# 重新输入正确的 API Key
```

**问题：WhatsApp 扫码无响应**
```bash
# 检查网络连接
# 尝试重新登录
openclaw channels logout whatsapp
openclaw channels login whatsapp
```

**问题：OAuth 授权失败**
```bash
# 检查凭证文件
cat ~/.openclaw/credentials/oauth.json

# 重新授权
rm ~/.openclaw/credentials/oauth.json
openclaw onboard
```

---

## 📝 最佳实践

### 推荐配置流程

1. **首次使用**：使用完整 Onboarding（`--install-daemon`）
2. **日常使用**：通过 CLI 微调配置
3. **重大变更**：重新运行 Onboarding

### 工作区管理

```
推荐工作区结构：
~/openclaw-workspace/
├── AGENTS.md         # 代理行为
├── SOUL.md          # 个性定义
├── USER.md          # 用户信息
├── MEMORY.md        # 长期记忆
├── projects/        # 项目目录
│   ├── project-a/
│   └── project-b/
└── skills/          # 自定义技能
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw onboard` | 运行引导配置 |
| `openclaw onboard --reset` | 重置后重新引导 |
| `openclaw onboard --install-daemon` | 安装后台服务 |
| `openclaw config` | 配置管理 |
| `openclaw channels` | 渠道管理 |
| `openclaw doctor` | 诊断检查 |

---

## 📚 相关文档

- [新手上路](/zh-CN/start/getting-started) - 从零开始
- [快速入门](/zh-CN/start/quick-start) - 5分钟上手
- [向导模式详解](/zh-CN/start/wizard) - 向导工作原理
- [CLI 参考](/zh-CN/cli) - 命令行工具
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

**Onboarding 让配置变得简单！跟着向导走，几分钟就能开始使用 OpenClaw。** 🦞
