---
summary: "5 分钟快速安装和配置 OpenClaw，涵盖安装向导、连接消息渠道、启动网关、发送测试消息和常见问题解答"
read_when:
  - 首次安装需要快速上手时
  - 需要发送第一条消息时
  - 快速验证安装是否成功时
title: "快速入门"
---

# 🚀 快速入门

**目标**：在 5 分钟内完成安装、配置，并发送第一条 AI 消息。

本指南专为**追求效率**的用户设计，采用最小化步骤，帮助您快速体验 OpenClaw 的核心功能。如果您希望深入理解每个步骤背后的原理，请参考[新手上路](/zh-CN/start/getting-started)完整指南。

---

## 🎯 学习目标

完成本指南学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的核心架构和工作流程
- [ ] 完成 OpenClaw CLI 的安装和验证
- [ ] 运行配置向导并理解每个配置项的作用
- [ ] 连接至少一个消息渠道（WhatsApp、Telegram 或 Discord）
- [ ] 成功发送第一条测试消息

### 进阶目标（建议掌握）

- [ ] 理解网关（Gateway）的角色和运行机制
- [ ] 掌握后台服务的启动和管理
- [ ] 能够诊断常见的安装和配置问题

---

## 💡 首先理解：什么是 OpenClaw？

### 一句话解释

OpenClaw 是**你的个人 AI 助手网关**，它连接你常用的聊天应用（WhatsApp、Telegram、Discord 等），对接强大的 AI 模型（Claude、GPT 等），让 AI 通过你熟悉的界面为你服务。

### 核心工作流程

```
┌─────────────────────────────────────────────────────────────┐
│                      OpenClaw 工作流程                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  你（通过 WhatsApp/Telegram/Discord 发送消息）              │
│                              ↓                               │
│                    OpenClaw 网关（控制平面）                 │
│                    - 接收消息                                │
│                    - 管理会话                                │
│                    - 调用 AI 模型                            │
│                              ↓                               │
│                    AI 大脑（Claude/GPT）                     │
│                              ↓                               │
│                    OpenClaw 网关                             │
│                              ↓                               │
│                  返回回复到你熟悉的聊天界面                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 为什么要用 OpenClaw？

| 优势 | 说明 |
|------|------|
| **多渠道统一** | 通过一个界面管理所有聊天应用 |
| **隐私优先** | 所有数据存储在本地，不经过第三方服务器 |
| **高度可定制** | 支持自定义提示词、技能和工作流程 |
| **完全开源** | 代码透明，可自由修改和扩展 |

---

## 🛠️ 前置要求

### 必须满足的系统要求

在开始安装之前，请确认您的系统满足以下要求：

| 要求 | 说明 | 检查方式 |
|------|------|----------|
| **Node.js >= 22** | 运行时的 JavaScript 引擎 | `node --version` |
| **操作系统** | macOS、Linux 或 Windows（需 WSL2） | `uname -a` 或 `systeminfo` |
| **网络连接** | 需要访问互联网下载依赖 | `ping -c 3 openclaw.ai` |

### 检查 Node.js 版本

打开终端，执行以下命令检查 Node.js 版本：

```bash
# 检查 Node.js 版本
node --version

# 期望输出：v22.x.x 或更高版本
# 如果显示 v20.x.x 或更低，需要先升级 Node.js
```

**如果版本低于 22**，请访问 [nodejs.org](https://nodejs.org) 下载并安装最新 LTS 版本。

> **💡 专家提示**：建议使用 nvm（Node Version Manager）管理多个 Node.js 版本，这可以避免权限问题并简化版本切换。安装命令：`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash`

### 操作系统特定要求

| 平台 | 支持情况 | 备注 |
|------|----------|------|
| **macOS** | ✅ 完全支持 | 推荐作为开发环境 |
| **Linux** | ✅ 完全支持 | 服务器部署首选 |
| **Windows** | ⚠️ 仅支持 WSL2 | 原生 Windows 未经充分测试 |

> **⚠️ Windows 用户注意**：请务必安装 WSL2（Windows Subsystem for Linux 2），然后在 Ubuntu 子系统中运行所有命令。原生 Windows 可能存在兼容性问题。

---

## 📦 第一步：安装 OpenClaw CLI

CLI（命令行工具）是您与 OpenClaw 交互的主要方式。它负责安装、管理网关、发送消息等所有操作。

### 方式一：使用安装脚本（推荐）

安装脚本会自动完成所有配置，包括创建目录、设置权限、添加 PATH 等。

**macOS / Linux：**

```bash
# 下载并执行安装脚本
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell)：**

```powershell
# 以管理员身份运行 PowerShell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 方式二：使用 npm/pnpm 安装

如果您的环境中已经安装了 Node.js，可以使用包管理器直接安装：

```bash
# 使用 npm 安装
npm install -g openclaw@latest

# 或使用 pnpm（推荐，运行速度更快）
pnpm add -g openclaw@latest
```

### 方式三：从源码安装（开发者）

如果您需要修改 OpenClaw 源码或贡献代码，可以从源码安装：

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖
pnpm install

# 构建项目
pnpm build

# 全局链接
pnpm link --global
```

### 验证安装成功

安装完成后，验证 CLI 是否正常工作：

```bash
# 检查版本
openclaw --version

# 期望输出：2026.x.x 或类似版本号
```

> **📚 知识延伸**：OpenClaw 采用 monorepo 结构，使用 pnpm workspace 管理多个子包。CLI 是整个系统的入口点，实际的 AI 处理逻辑在独立的 `gateway` 包中。

---

## 🧙 第二步：运行配置向导

配置向导是一个**交互式安装助手**，它会逐步引导您完成所有必要的配置。您不需要手动编辑复杂的配置文件。

### 启动向导

```bash
# 运行向导并安装后台服务
openclaw onboard --install-daemon
```

**参数说明：**

| 参数 | 作用 |
|------|------|
| `--install-daemon` | 安装后台服务，让网关随系统启动 |
| `--skip-channels` | 跳过渠道配置（仅配置核心功能） |
| `--skip-daemon` | 不安装后台服务（手动管理） |

### 向导会问哪些问题？

向导将引导您完成以下配置步骤：

```
步骤 1：网关模式选择
├── 本地网关（推荐）→ 网关运行在当前机器
└── 远程网关 → 网关运行在远程服务器

步骤 2：认证方式配置
├── Anthropic API Key → 使用 Claude 模型
├── Anthropic OAuth → 使用 Claude Code CLI 的认证
├── OpenAI API Key → 使用 GPT 模型
└── OpenAI OAuth → 使用 Codex CLI 的认证

步骤 3：消息渠道设置
├── WhatsApp → 手机扫码登录
├── Telegram → Bot Token 认证
├── Discord → Bot Token 认证
└── Skip → 跳过，稍后手动配置

步骤 4：后台服务安装
├── macOS → 安装 LaunchAgent
├── Linux → 安装 systemd 用户单元
└── Skip → 手动管理网关进程

步骤 5：网关令牌设置
└── 自动生成安全令牌或使用自定义令牌
```

### 认证信息存储位置

| 认证类型 | 存储位置 |
|----------|----------|
| **OAuth 认证** | `~/.openclaw/credentials/oauth.json` |
| **API Keys** | `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` |

> **🔐 安全提醒**：这些文件包含敏感凭证信息，请确保只有您本人可以访问。建议定期备份，但不要分享给他人。

### 无头/服务器部署场景

如果您想在没有浏览器的服务器上部署：

1. **先在普通机器完成 OAuth 认证**：运行向导完成认证配置
2. **复制凭证文件**：`scp ~/.openclaw/credentials/oauth.json user@server:~/.openclaw/credentials/`
3. **在服务器上运行向导**：跳过认证步骤，使用已复制的凭证

---

## 📱 第三步：连接消息渠道

消息渠道是 AI 接收消息和发送回复的通道。OpenClaw 支持多种主流消息平台。

### 连接 WhatsApp（扫码登录）

WhatsApp 使用手机扫码的方式进行认证：

```bash
# 启动登录流程
openclaw channels login
```

终端会显示一个二维码，然后用手机操作：

1. 打开 WhatsApp 手机应用
2. 点击右上角 **⋮** → **已关联设备**
3. 点击 **关联设备**
4. 扫描终端中显示的二维码

> **📱 扫码失败？** 确保手机和电脑连接在同一网络，且 WhatsApp 版本是最新的。

### 连接 Telegram（Bot Token）

Telegram 使用机器人（Bot）进行认证，需要先创建一个机器人：

**创建机器人的步骤：**

1. 在 Telegram 中搜索 **[@BotFather](https://t.me/BotFather)**
2. 发送 `/newbot` 命令
3. 按照提示输入机器人名称和用户名
4. 保存 Bot Token（格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`）

**在 OpenClaw 中配置：**

```bash
# 方式一：在向导中输入 Token
openclaw onboard

# 方式二：手动配置
openclaw config set channels.telegram.botToken "YOUR_BOT_TOKEN"
```

### 连接 Discord（Bot Token）

Discord 需要创建应用并添加机器人：

**创建 Discord 应用的步骤：**

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 **New Application**，输入应用名称
3. 在 **Bot** 页面，点击 **Add Bot**
4. 获取 **Bot Token**

**邀请机器人到服务器：**

1. 在 Discord Developer Portal 中，进入 **OAuth2** → **URL Generator**
2. 选择 `bot` 权限范围
3. 复制生成的 URL，在浏览器中打开并授权

**在 OpenClaw 中配置：**

```bash
# 配置 Bot Token
openclaw config set channels.discord.token "YOUR_BOT_TOKEN"
```

### 其他渠道

| 渠道 | 认证方式 | 配置复杂度 |
|------|----------|------------|
| Slack | OAuth 2.0 | ⭐⭐ |
| Signal | signal-cli | ⭐⭐⭐ |
| iMessage | macOS 专用 | ⭐⭐⭐ |
| Google Chat | OAuth | ⭐⭐ |

详细配置请参考[渠道配置完整指南](/zh-CN/channels/index)。

---

## 🚀 第四步：启动网关

网关是 OpenClaw 的**核心控制平面**，它负责：

- 维护与各消息渠道的连接
- 管理 AI 代理的生命周期
- 处理消息路由和会话状态
- 提供 WebSocket 控制接口

### 检查网关状态

如果向导已安装后台服务，网关应该已经自动运行：

```bash
# 检查网关运行状态
openclaw gateway status

# 输出示例：
# Status: running
# PID: 12345
# Port: 18789
# Mode: local
```

### 手动启动网关

如果需要手动启动或调试问题，可以在前台运行：

```bash
# 前台启动（查看实时日志）
openclaw gateway --port 18789 --verbose

# 或指定配置文件
openclaw gateway --config /path/to/config.json
```

**常用启动参数：**

| 参数 | 说明 |
|------|------|
| `--port <端口>` | 指定网关监听端口（默认 18789） |
| `--verbose` | 启用详细日志输出 |
| `--config <文件>` | 指定配置文件路径 |
| `--headless` | 无头模式（不启动 Web 界面） |

### 后台服务管理

| 操作 | 命令 |
|------|------|
| **启动服务** | `launchctl start com.openclaw.gateway`（macOS） |
| **停止服务** | `launchctl stop com.openclaw.gateway`（macOS） |
| **查看日志** | `journalctl --user -u openclaw-gateway -f`（Linux） |

> **⚠️ 重要警告**：如果您使用 WhatsApp 或 Telegram 渠道，**必须使用 Node.js 运行网关**，不要使用 Bun！Bun 对这些渠道存在已知的兼容性问题。

---

## ✅ 第五步：发送测试消息

现在您可以验证整个系统是否正常工作。

### 方式一：使用 Web 控制台

最快的测试方式是打开 Web 控制台：

```bash
# 启动控制台（自动打开浏览器）
openclaw dashboard
```

或直接访问：http://127.0.0.1:18789/

在控制台中，您可以：
- 直接与 AI 代理对话
- 查看会话历史
- 管理渠道连接
- 配置系统设置

### 方式二：使用命令行发送

通过命令行发送消息到已连接的渠道：

**发送到 WhatsApp：**

```bash
openclaw message send --target "+15555550123" --message "你好，来自 OpenClaw！"
```

**发送到 Telegram：**

```bash
openclaw message send --target "@username" --message "你好，来自 OpenClaw！"
```

**发送到 Discord：**

```bash
openclaw message send --target "#channel-name" --message "你好，来自 OpenClaw！"
```

### 方式三：使用 Agent 命令

通过 CLI 直接调用 AI 代理：

```bash
# 向 AI 代理发送消息并获取回复
openclaw agent --message "你好，请介绍一下你自己"

# 开启深度思考模式
openclaw agent --message "分析这个问题的解决方案" --thinking high
```

---

## 🧪 第六步：验证安装成功

运行以下命令确认系统各组件正常：

### 整体状态检查

```bash
# 查看完整的系统状态
openclaw status

# 输出示例：
# Gateway: running (PID: 12345)
# Channels: connected (1/3)
# Auth: configured (Anthropic)
# Version: 2026.1.27
```

### 健康检查

```bash
# 检查各组件健康状态
openclaw health

# 检查示例输出：
# ✅ Gateway: healthy
# ✅ Channel (whatsapp): connected
# ❌ Auth: no auth configured
```

### 安全审计

```bash
# 基础安全检查
openclaw security audit

# 深度安全审计（推荐）
openclaw security audit --deep
```

> **🔒 安全提醒**：安全审计会检查配置中的潜在风险，包括开放的 DM 权限、未加密的凭证存储等。请根据警告信息调整配置。

---

## 🔧 常见问题与解决方案

### 问题一：收不到 AI 回复

**症状**：发送消息后，没有收到任何回复。

**可能原因与解决方案：**

| 检查步骤 | 操作 |
|----------|------|
| 1. 检查配对请求 | `openclaw pairing list whatsapp` |
| 2. 批准配对 | `openclaw pairing approve whatsapp <配对码>` |
| 3. 确认认证配置 | `openclaw health` |
| 4. 查看网关日志 | `openclaw gateway --verbose` |

### 问题二：网关无法启动

**症状**：运行 `openclaw gateway` 时报错或无响应。

**排查步骤：**

```bash
# 1. 检查端口占用
lsof -i :18789

# 2. 检查配置文件语法
openclaw doctor

# 3. 尝试自动修复
openclaw doctor --fix
```

### 问题三：WhatsApp 扫码失败

**症状**：扫码后显示超时或验证失败。

**解决方案：**

1. 确保手机和电脑连接在同一网络
2. 检查手机 WhatsApp 版本是否为最新
3. 尝试重新运行 `openclaw channels login`
4. 确保没有其他设备关联了同一个 WhatsApp Web 会话

### 问题四：认证失败

**症状**：`openclaw health` 显示 "no auth configured" 或认证错误。

**解决方案：**

```bash
# 重新运行向导配置认证
openclaw onboard

# 或手动配置 API Key
openclaw config set agents.defaults.model "anthropic/claude-opus-4-5"
openclaw config set "agents.defaults.authProfiles.anthropic.apiKey" "sk-ant-api03-..."
```

---

## 📊 专家思维模型：系统化问题排查

当遇到问题时，不要盲目尝试解决方案，而是采用**假设-验证-迭代**的思维模型：

```
┌─────────────────────────────────────────────────────────────────┐
│                    问题排查决策流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 收集信息                                                    │
│     └── 错误信息是什么？什么时候开始的？最近有什么变化？         │
│                              ↓                                   │
│  2. 形成假设                                                    │
│     └── 最可能的原因是什么？（基于经验和模式识别）               │
│                              ↓                                   │
│  3. 设计验证                                                    │
│     └── 怎么以最小成本验证这个假设？                            │
│                              ↓                                   │
│  4. 执行验证                                                    │
│     └── 检查日志、监控指标、配置文件                            │
│                              ↓                                   │
│  5. 得出结论                                                    │
│     └── 假设成立 → 修复；假设不成立 → 新假设                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 知识点回顾

完成本指南后，您应该掌握以下核心概念：

| 概念 | 理解程度 |
|------|----------|
| **OpenClaw 网关** | ⭐⭐⭐ 完全理解 |
| **消息渠道** | ⭐⭐⭐ 能够配置 |
| **认证方式** | ⭐⭐⭐ 知道如何配置 |
| **网关启动与管理** | ⭐⭐⭐ 能够独立操作 |
| **问题排查** | ⭐⭐ 了解基本流程 |

---

## 🎯 下一步学习

您已经完成了快速入门，建议继续深入学习：

| 学习路径 | 文档链接 | 预计时间 |
|----------|----------|----------|
| **深入理解架构** | [系统架构](/zh-CN/concepts/architecture) | 30 分钟 |
| **完整安装指南** | [安装指南](/zh-CN/install) | 45 分钟 |
| **向导模式详解** | [向导模式](/zh-CN/start/wizard) | 20 分钟 |
| **核心概念** | [概念索引](/zh-CN/concepts/index) | 1 小时 |
| **CLI 命令参考** | [CLI 文档](/zh-CN/cli/index) | 30 分钟 |

---

## 📖 附录

### 附录 A：常用命令速查

| 功能 | 命令 |
|------|------|
| 查看版本 | `openclaw --version` |
| 运行向导 | `openclaw onboard` |
| 启动网关 | `openclaw gateway` |
| 查看状态 | `openclaw status` |
| 健康检查 | `openclaw health` |
| 发送消息 | `openclaw message send --target <目标> --message <内容>` |
| 诊断问题 | `openclaw doctor` |

### 附录 B：术语表

| 英文术语 | 中文术语 | 说明 |
|---------|---------|------|
| Gateway | 网关 | OpenClaw 的核心控制平面 |
| Channel | 渠道 | 消息来源通道（WhatsApp、Telegram 等） |
| Agent | 代理 / 智能体 | 处理消息的 AI 程序 |
| Session | 会话 | 一次连续的对话上下文 |
| OAuth | OAuth | 开放授权协议 |
| CLI | CLI / 命令行工具 | Command Line Interface |

### 附录 C：相关资源

- **官方文档**：[docs.openclaw.ai](https://docs.openclaw.ai)
- **GitHub 仓库**：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- **社区 Discord**：[discord.gg/clawd](https://discord.gg/clawd)
- **问题反馈**：[GitHub Issues](https://github.com/openclaw/openclaw/issues)

---

> **💡 提示**：文档持续更新中！如果发现错误或有改进建议，欢迎通过 GitHub Issues 反馈。
