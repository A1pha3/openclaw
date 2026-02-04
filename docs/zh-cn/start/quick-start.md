---
summary: 5 分钟快速安装和配置 OpenClaw，涵盖安装向导、连接消息渠道、启动网关、发送测试消息和常见问题解答
read_when:
  - 首次安装需要快速上手时
  - 需要发送第一条消息时
  - 快速验证安装是否成功时
title: 快速入门
---

# 快速入门

本指南将帮助您在 5 分钟内完成 OpenClaw 的安装和基础配置，发送您的第一条消息。

## 前置要求

在开始之前，请确保您的系统满足以下要求：

- **Node.js**: 版本 22 或更高
- **操作系统**: macOS、Linux 或 Windows (需使用 WSL2)
- **网络**: 能够访问互联网

### 检查 Node.js 版本

```bash
node --version
# 应显示 v22.x.x 或更高版本
```

如果您还没有安装 Node.js 或版本过低，请访问 [nodejs.org](https://nodejs.org) 下载安装。

## 第一步：安装 OpenClaw

### 方式一：使用安装脚本（推荐）

**macOS / Linux:**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell):**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 方式二：使用 npm 安装

```bash
npm install -g openclaw@latest
```

或使用 pnpm：

```bash
pnpm add -g openclaw@latest
```

### 验证安装

```bash
openclaw --version
```

您应该看到类似 `2026.1.27` 的版本号。

## 第二步：运行配置向导

OpenClaw 提供了交互式配置向导，帮助您完成初始设置：

```bash
openclaw onboard --install-daemon
```

向导将引导您完成以下配置：

1. **网关模式**: 选择本地或远程网关
2. **认证方式**: 配置 OpenAI/Anthropic API 密钥或 OAuth
3. **消息渠道**: 设置 WhatsApp、Telegram、Discord 等
4. **后台服务**: 安装系统服务（launchd/systemd）
5. **网关令牌**: 自动生成安全令牌

> **提示**: 推荐使用 Node.js 运行时（而非 Bun），特别是使用 WhatsApp 或 Telegram 时。

## 第三步：连接消息渠道

### 连接 WhatsApp

WhatsApp 使用二维码扫描方式登录：

```bash
openclaw channels login
```

终端会显示二维码，使用手机 WhatsApp：

1. 打开 WhatsApp → 设置 → 已连接设备
2. 点击"连接设备"
3. 扫描终端中的二维码

### 连接 Telegram

需要先创建 Telegram Bot：

1. 在 Telegram 中搜索 [@BotFather](https://t.me/BotFather)
2. 发送 `/newbot` 创建新机器人
3. 获取 Bot Token
4. 在向导中输入 Token，或手动配置：

```bash
openclaw config set channels.telegram.botToken "YOUR_BOT_TOKEN"
```

### 连接 Discord

需要先创建 Discord 应用：

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 创建新应用，添加 Bot
3. 获取 Bot Token
4. 配置 Token：

```bash
openclaw config set channels.discord.token "YOUR_BOT_TOKEN"
```

## 第四步：启动网关

如果您在向导中选择了安装后台服务，网关应该已经在运行。检查状态：

```bash
openclaw gateway status
```

手动启动网关（前台模式）：

```bash
openclaw gateway --port 18789 --verbose
```

## 第五步：发送测试消息

### 使用控制台界面

最快的测试方式是打开浏览器控制台：

```bash
openclaw dashboard
```

或直接访问：http://127.0.0.1:18789/

在控制台中可以直接与 AI 代理对话。

### 使用命令行发送

发送 WhatsApp 消息：

```bash
openclaw message send --target +15555550123 --message "你好，来自 OpenClaw"
```

## 第六步：验证安装

运行健康检查确认一切正常：

```bash
# 查看整体状态
openclaw status

# 健康检查
openclaw health

# 安全审计
openclaw security audit --deep
```

## 常见问题

### 收不到回复？

如果您的消息没有收到回复，可能需要处理配对请求：

```bash
# 查看待处理的配对请求
openclaw pairing list whatsapp

# 批准配对
openclaw pairing approve whatsapp <code>
```

### 网关无法启动？

检查配置是否有效：

```bash
openclaw doctor
```

如果有问题，尝试自动修复：

```bash
openclaw doctor --fix
```

### 认证失败？

确认您已配置有效的 API 密钥：

```bash
openclaw health
```

如果显示"no auth configured"，需要重新运行向导配置认证。

## 下一步

恭喜！您已经成功完成了 OpenClaw 的基础配置。接下来您可以：

- 📖 阅读 [安装指南](/zh-CN/start/installation) 了解更多安装选项
- 🔧 查看 [配置参考](/zh-CN/config/reference) 进行高级配置
- 🌐 配置更多 [消息渠道](/zh-CN/channels/index)
- 🏗️ 了解 [系统架构](/zh-CN/concepts/architecture) 深入理解工作原理

## 获取帮助

如果遇到问题：

1. 运行 `openclaw doctor` 诊断问题
2. 查看 [故障排除](/zh-CN/operations/troubleshooting) 文档
3. 在 [GitHub Issues](https://github.com/openclaw/openclaw/issues) 提交问题
