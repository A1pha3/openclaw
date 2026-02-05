---
summary: "从零开始：30分钟内完成安装并发送第一条消息"
read_when:
  - 第一次安装 OpenClaw
  - 想要最快从安装到发送第一条消息
  - 完全不了解这个系统的新手
title: "新手上路"
---

# 🚀 新手上路

**目标**：从零开始 → 第一次成功聊天（使用合理默认配置）

本文档专为**完全新手**设计，我们会深入浅出地讲解每个步骤，帮助你真正理解 OpenClaw 的工作原理。即使你从未使用过命令行或 AI 助手，也能通过本指南顺利完成配置。

---

## 🎯 学习目标

完成本指南学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的核心概念和架构
- [ ] 完成 CLI 的安装和验证
- [ ] 运行向导完成基础配置
- [ ] 连接至少一个消息渠道（WhatsApp、Telegram 或 Discord）
- [ ] 成功发送第一条测试消息

### 进阶目标（建议掌握）

- [ ] 理解网关（Gateway）的工作机制
- [ ] 理解配对（Pairing）安全机制
- [ ] 掌握基本的故障排查方法
- [ ] 理解沙箱（Sandbox）安全概念

---

## 💡 首先理解：什么是 OpenClaw？

### 一句话解释

OpenClaw 是**你的个人 AI 助手网关**，它能将 WhatsApp、Telegram、Discord 等你常用的聊天应用，连接到强大的 AI（如 Claude、GPT），让 AI 通过你熟悉的聊天界面为你服务。

### 类比理解

为了更好地理解 OpenClaw，让我们用日常生活中的类比来说明：

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw 类比：智能接线员                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  传统方式：                                                     │
│    你 ──📱WhatsApp──→ AI（在线服务）                         │
│    你 ──📱Telegram──→ AI（在线服务）                         │
│    你 ──💬Discord───→ AI（在线服务）                         │
│                                                                 │
│  问题：每个应用都需要单独配置、维护，AI 是远程的              │
│                                                                 │
│  OpenClaw 方式：                                                │
│    ┌─────────────────────────────────────────────────────┐   │
│    │                   OpenClaw                          │   │
│    │   （智能接线员，本地运行）                            │   │
│    │                                                     │   │
│    │   ┌─────────┐  ┌─────────┐  ┌─────────┐           │   │
│    │   │ WhatsApp │  │ Telegram│  │ Discord │  ← 渠道 │   │
│    │   └────┬────┘  └────┬────┘  └────┬────┘           │   │
│    │        │             │             │                  │   │
│    │        └────────────┼─────────────┘                  │   │
│    │                     ↓                                │   │
│    │              ┌─────────────┐                         │   │
│    │              │ AI 大脑    │  ← Claude/GPT          │   │
│    │              │ (本地调用)  │                         │   │
│    │              └─────────────┘                         │   │
│    └─────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│                   返回回复到你的聊天应用                      │
│                                                                 │
│  优势：                                                         │
│    ✅ 统一管理多个聊天应用                                      │
│    ✅ AI 在本地调用，数据不经过第三方                          │
│    ✅ 可定制化程度高                                           │
│    ✅ 完全开源透明                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 核心概念（新手必知）

在开始安装之前，让我们先理解几个核心概念：

| 概念 | 通俗解释 | 生活类比 | 重要性 |
|------|----------|----------|--------|
| **Gateway（网关）** | 核心控制中心，管理所有连接 | 公司的总机系统 | ⭐⭐⭐⭐⭐ |
| **Channel（渠道）** | 消息来源（WhatsApp/Telegram等） | 不同的电话号码 | ⭐⭐⭐⭐⭐ |
| **Agent（代理）** | 实际处理消息的 AI 程序 | 接线员 | ⭐⭐⭐⭐ |
| **Session（会话）** | 一次连续的对话上下文 | 通话记录 | ⭐⭐⭐⭐ |
| **Workspace（工作区）** | 配置和数据的存储位置 | 办公桌抽屉 | ⭐⭐⭐ |

### OpenClaw vs 其他 AI 助手

| 对比维度 | OpenClaw | ChatGPT 网页版 | Claude 网页版 |
|----------|----------|----------------|---------------|
| **部署方式** | 本地部署 | 云端 | 云端 |
| **数据隐私** | ⭐⭐⭐⭐⭐ 完全本地 | ⭐⭐⭐ 发送到云端 | ⭐⭐⭐ 发送到云端 |
| **消息渠道** | 多渠道统一 | 仅网页 | 仅网页 |
| **自定义程度** | ⭐⭐⭐⭐⭐ 完全可定制 | ⭐⭐ 受限 | ⭐⭐ 受限 |
| **成本** | 仅 API 费用 | 订阅制 | 订阅制 |
| **上手难度** | ⭐⭐⭐ 需要配置 | ⭐ 立即可用 | ⭐ 立即可用 |

---

## 🛠️ 前置要求

在开始安装之前，请确保您的系统满足以下要求。

### 必须准备

| 要求 | 说明 | 检查方式 |
|------|------|----------|
| **Node.js >= 22** | JavaScript 运行时环境 | `node -v` |
| **操作系统** | macOS、Linux 或 Windows（需 WSL2） | `uname -a` |
| **网络连接** | 需要访问互联网下载和运行 | `ping openclaw.ai` |

#### 检查 Node.js 版本

```bash
# 检查 Node.js 版本
node --version

# 期望输出：v22.x.x 或更高版本
# 如果显示 v20.x.x 或更低版本，需要升级
```

**如果版本低于 22**，请访问 [nodejs.org](https://nodejs.org) 下载并安装最新 LTS 版本。

#### 安装 pnpm（可选，推荐）

```bash
# 使用 npm 安装 pnpm
npm install -g pnpm
```

> **💡 专家提示**：pnpm 是一个比 npm 更快的包管理器，在大型项目中可以显著节省安装时间和磁盘空间。

#### Brave Search API Key（强烈推荐）

虽然不是必需，但配置 Brave Search API Key 可以让 AI 进行网页搜索，获取最新信息：

```bash
# 后续配置命令
openclaw configure --section web
```

### 平台支持

| 平台 | 支持情况 | 推荐度 | 备注 |
|------|----------|--------|------|
| **macOS** | ✅ 完全支持 | ⭐⭐⭐⭐⭐ | 推荐作为开发环境 |
| **Linux** | ✅ 完全支持 | ⭐⭐⭐⭐⭐ | 服务器部署首选 |
| **Windows WSL2** | ✅ 完全支持 | ⭐⭐⭐⭐ | 需要先安装 WSL2 |
| **原生 Windows** | ⚠️ 有限支持 | ⭐⭐ | 不推荐，可能遇到兼容性问题 |

#### Windows 用户特别说明

> **⚠️ 强烈建议**：Windows 用户请使用 **WSL2**（Windows Subsystem for Linux 2），不要在原生 Windows 上安装。

**为什么推荐 WSL2？**

| 对比 | 原生 Windows | WSL2 |
|------|--------------|------|
| **兼容性** | 部分功能不支持 | 与 Linux 完全兼容 |
| **性能** | 一般 | 接近原生 Linux |
| **工具链** | 需要单独配置 | 直接使用 Linux 工具 |
| **维护** | 复杂 | 简单 |

**WSL2 安装步骤：**

```powershell
# 在 PowerShell（管理员）中运行
wsl --install

# 重启电脑后，安装 Ubuntu
wsl --install -d Ubuntu

# 在 Ubuntu 中按照 Linux 步骤安装 OpenClaw
```

---

## 📦 第一步：安装 CLI

CLI（命令行工具）是你与 OpenClaw 交互的主要方式。就像微信有手机 App，OpenClaw 有命令行工具。

### 为什么用 CLI？

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLI vs GUI 对比                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLI（命令行）的优势：                                           │
│    ✅ 资源占用少                                                │
│    ✅ 自动化友好                                                │
│    ✅ 可脚本化                                                   │
│    ✅ 启动快速                                                   │
│                                                                 │
│  CLI 的劣势：                                                   │
│    ❌ 需要学习基本命令                                           │
│    ❌ 没有图形界面直观                                           │
│                                                                 │
│  OpenClaw 的设计：                                              │
│    CLI 是主要入口，Web 界面是补充                                │
│    推荐使用 CLI 完成配置和管理                                    │
│    Web 界面用于日常聊天                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 安装命令

**macOS / Linux：**

```bash
# 官方安装脚本（推荐）
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell，以管理员身份运行)：**

```powershell
# PowerShell 安装脚本
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 安装方式对比

| 方式 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| **安装脚本（推荐）** | 快速开始 | 自动化程度高、一键完成 | 需要网络 |
| **npm 全局安装** | 已有 Node 环境 | 版本管理简单 | 需要手动配置 |
| **pnpm 全局安装** | 开发者 | 速度快、磁盘占用小 | 需要安装 pnpm |
| **源码安装** | 开发者/贡献者 | 可修改源码 | 需要构建步骤 |
| **Docker** | 服务器部署 | 环境隔离 | 配置较复杂 |

### 备用方案：npm 安装

如果安装脚本失败，可以使用 npm：

```bash
# 使用 npm 安装
npm install -g openclaw@latest

# 或使用 pnpm
pnpm add -g openclaw@latest
```

### 验证安装

```bash
# 检查版本
openclaw --version

# 期望输出：2026.x.x 或类似版本号

# 查看帮助
openclaw --help

# 应该显示可用命令列表
```

---

## 🧙 第二步：运行引导向导

### 什么是向导？

向导就像**智能安装助手**，会一步步问你问题，自动配置好一切。你不需要手动编辑复杂的配置文件。

```
┌─────────────────────────────────────────────────────────────────┐
│                    向导工作流程                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  运行向导前：                                                    │
│    ❌ 不了解配置结构                                             │
│    ❌ 不知道该设置什么选项                                       │
│    ❌ 担心配置错误导致问题                                       │
│                                                                 │
│  向导帮你解决：                                                 │
│    ✅ 问答式交互，只需回答问题                                   │
│    ✅ 提供合理默认值，大多数直接回车即可                         │
│    ✅ 自动验证配置，发现问题及时提示                             │
│    ✅ 完成后可直接使用                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 启动向导

```bash
# 运行向导并安装后台服务（推荐）
openclaw onboard --install-daemon
```

**参数说明：**

| 参数 | 作用 |
|------|------|
| `--install-daemon` | 安装后台服务，让网关随系统启动 |
| `--reset` | 重置所有配置后重新引导 |
| `--skip-channels` | 跳过渠道配置（仅配置核心功能） |

### 向导配置内容

向导会依次帮你配置：

```
步骤 1：网关模式
├── 本地网关（推荐）→ 仅本地访问
└── 远程网关 → 通过网络访问

步骤 2：认证方式
├── Anthropic API Key
├── Anthropic OAuth
├── OpenAI API Key
└── OpenAI OAuth

步骤 3：消息渠道（可选）
├── WhatsApp（二维码登录）
├── Telegram（Bot Token）
└── Discord（Bot Token）

步骤 4：后台服务
├── 安装服务（推荐）→ 开机自启动
└── 手动管理

步骤 5：技能选择（可选）
└── 推荐技能安装
```

### 认证方式选择

| 方式 | 适用情况 | 推荐度 | 说明 |
|------|----------|--------|------|
| **Anthropic API Key** | 有 Claude API Key | ⭐⭐⭐⭐⭐ | 最推荐的配置方式 |
| **Anthropic OAuth** | 有 Claude Code CLI | ⭐⭐⭐⭐⭐ | 更安全，无需存储 Key |
| **OpenAI API Key** | 有 OpenAI API Key | ⭐⭐⭐⭐ | 通用配置方式 |
| **OpenAI OAuth (Codex)** | 有 Codex CLI | ⭐⭐⭐⭐ | 开发者友好 |

### 认证信息存储位置

| 认证类型 | 存储位置 | 敏感度 |
|----------|----------|--------|
| **OAuth** | `~/.clawdbot/credentials/oauth.json` | 🔴 高 |
| **API Keys** | `~/.clawdbot/agents/<agentId>/agent/auth-profiles.json` | 🔴 高 |

> **🔐 安全提醒**：这些文件包含敏感凭证信息，请确保只有您本人可以访问。

### 无头/服务器部署提示

如果您想在服务器上部署，没有浏览器可以完成 OAuth 认证：

1. **先在普通机器完成 OAuth**：运行向导完成认证配置
2. **复制凭证文件**：
   ```bash
   scp ~/.clawdbot/credentials/oauth.json user@server:~/.clawdbot/credentials/
   ```
3. **在服务器上运行向导**：跳过认证步骤，使用已复制的凭证

---

## 🚀 第三步：启动网关

### 什么是网关？

**网关 = OpenClaw 的心脏**

它是唯一长期运行的进程，负责：

```
┌─────────────────────────────────────────────────────────────────┐
│                    网关核心功能                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 连接管理                                                    │
│     ├── 维护与 WhatsApp/Telegram 的连接                          │
│     ├── 管理 Discord Bot 连接                                    │
│     └── 处理消息收发                                             │
│                                                                 │
│  2. 会话管理                                                    │
│     ├── 创建和销毁会话                                           │
│     ├── 管理对话上下文                                           │
│     └── 处理历史消息                                             │
│                                                                 │
│  3. AI 集成                                                    │
│     ├── 调用 AI 模型 API                                         │
│     ├── 处理流式响应                                             │
│     └── 管理模型认证                                             │
│                                                                 │
│  4. 工具执行                                                    │
│     ├── 执行 AI 选择的工具（搜索、文件等）                        │
│     └── 返回结果给 AI                                           │
│                                                                 │
│  5. WebSocket 服务                                              │
│     ├── 提供 WebSocket 控制平面                                  │
│     └── 支持 CLI 和 Web 界面连接                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 检查网关状态

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

如果需要调试或在特定场景下运行：

```bash
# 前台启动（查看实时日志）
openclaw gateway --port 18789 --verbose
```

**常用参数：**

| 参数 | 说明 |
|------|------|
| `--port <端口>` | 指定网关监听端口（默认 18789） |
| `--verbose` | 启用详细日志输出 |
| `--config <文件>` | 指定配置文件路径 |
| `--headless` | 无头模式（不启动 Web 界面） |

### 后台服务（推荐）

如果在向导中选择了安装后台服务，网关会随系统自动启动：

| 平台 | 服务类型 | 启动时机 |
|------|----------|----------|
| **macOS** | LaunchAgent | 用户登录时 |
| **Linux** | systemd 用户单元 | 用户登录后（支持 linger） |

**管理命令：**

```bash
# macOS
launchctl start com.openclaw.gateway    # 启动
launchctl stop com.openclaw.gateway     # 停止

# Linux
systemctl --user start openclaw-gateway
systemctl --user stop openclaw-gateway
```

### ⚠️ 重要警告：运行时选择

**如果您使用 WhatsApp 或 Telegram 渠道，必须用 Node.js 运行网关，不要用 Bun！**

```
问题：Bun 对 WhatsApp/Telegram 有兼容性问题

症状：
  - 消息发送失败
  - 连接频繁断开
  - 某些功能无法使用

解决方案：
  - 使用 Node.js 运行网关（推荐）
  - 检查运行时：node --version
  - 确保是 Node.js，不是 Bun
```

---

## ✅ 第四步：快速验证

安装完成后，让我们快速验证系统是否正常工作。

### 基础检查

```bash
# 查看整体状态
openclaw status

# 健康检查
openclaw health

# 安全审计
openclaw security audit --deep
```

**健康检查输出解读：**

```
✅ 通过的项目
⚠️  需要关注的问题
❌  需要立即修复的问题
```

### 仪表盘访问

最快的验证方式是打开 Web 仪表盘：

```bash
# 启动仪表盘（自动打开浏览器）
openclaw dashboard
```

或直接访问：http://127.0.0.1:18789/

在仪表盘中：
- 无需配置渠道即可与 AI 对话
- 可以测试基本的聊天功能
- 可以查看系统状态

### 如果显示 "no auth configured"

如果健康检查显示没有配置认证：

```bash
# 重新运行向导
openclaw onboard
```

---

## 💬 第五步：配置第一个聊天渠道

让我们配置第一个消息渠道，开始真正的 AI 聊天体验。

### 方式 A：WhatsApp（扫码登录）

WhatsApp 使用手机扫码的方式进行认证：

```bash
# 启动登录流程
openclaw channels login
```

**扫码步骤：**

1. 打开手机 WhatsApp 应用
2. 点击右上角 **⋮** → **已关联设备**
3. 点击 **关联设备**
4. 扫描终端中显示的二维码

> **📱 常见问题**：
> - 确保手机和电脑连接在同一网络
> - 确保 WhatsApp 版本是最新的
> - 扫码后等待几秒钟完成认证

### 方式 B：Telegram（机器人 Token）

Telegram 需要先创建一个机器人：

**创建机器人的步骤：**

1. 在 Telegram 中搜索 **[@BotFather](https://t.me/BotFather)**
2. 发送 `/newbot` 命令
3. 按照提示输入机器人名称和用户名
4. **保存 Bot Token**（格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`）

**在 OpenClaw 中配置：**

```bash
# 方式一：在向导中配置
openclaw onboard

# 方式二：手动配置
openclaw config set channels.telegram.botToken "YOUR_BOT_TOKEN"
```

### 方式 C：Discord

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

---

## 🔐 第六步：配对与安全

### 什么是配对？

这是 OpenClaw 的**默认安全机制**，保护您免受陌生人骚扰：

```
┌─────────────────────────────────────────────────────────────────┐
│                    配对安全机制                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  默认行为：                                                      │
│    1. 陌生人发送 DM 给 OpenClaw                                 │
│    2. OpenClaw 检测到来未知发送者                              │
│    3. 不回复，发送配对码给发送者                                 │
│    4. 你（管理员）收到配对请求                                   │
│    5. 你批准配对 → 发送者被添加到允许列表                       │
│    6. 此后 AI 正常回复该发送者                                   │
│                                                                 │
│  类比：                                                          │
│    就像微信的"好友申请"机制                                      │
│    不是所有消息都能收到回复，需要你同意                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 批准配对请求

```bash
# 查看所有渠道的待批准列表
openclaw pairing list

# 查看特定渠道的待批准列表
openclaw pairing list whatsapp
openclaw pairing list telegram
openclaw pairing list discord

# 批准配对
openclaw pairing approve whatsapp <配对码>
```

**操作示例：**

```bash
# 查看 WhatsApp 配对请求
$ openclaw pairing list whatsapp

Pending Pairing Requests:
┌─────────────┬──────────────────┬─────────────────────────┐
│ Channel     │ Sender            │ Code                     │
├─────────────┼──────────────────┼─────────────────────────┤
│ whatsapp    │ +1234567890      │ ABC-123-XYZ             │
└─────────────┴──────────────────┴─────────────────────────┘

# 批准配对
$ openclaw pairing approve whatsapp ABC-123-XYZ

✅ Pairing approved for +1234567890
```

> **💡 提示**：第一次发送消息如果没有收到回复，90% 的情况是因为配对未批准！

### 替代方案：允许列表

如果您想让某些人或群组直接可以对话，可以添加到允许列表：

```bash
# 允许所有私聊（不推荐，安全风险）
openclaw config set channels.whatsapp.allowFrom ["*"]

# 允许特定号码
openclaw config set channels.whatsapp.allowFrom ["+1234567890", "+0987654321"]

# 查看当前允许列表
openclaw config get channels.whatsapp.allowFrom
```

---

## 🧪 第七步：端到端测试

现在让我们测试整个系统是否正常工作。

### 发送测试消息

**使用命令行发送：**

```bash
# 发送 WhatsApp 消息
openclaw message send --target "+15555550123" --message "你好，这是来自 OpenClaw 的测试消息！"

# 发送 Telegram 消息
openclaw message send --target "@username" --message "你好，这是来自 OpenClaw 的测试消息！"

# 发送 Discord 消息
openclaw message send --target "#general" --message "你好，这是来自 OpenClaw 的测试消息！"
```

### 测试仪表盘聊天

这是最直观的方式：

```bash
# 启动仪表盘
openclaw dashboard
```

然后在浏览器中：
1. 输入你想测试的内容
2. 发送消息
3. 观察 AI 的回复

### 测试 AI 能力

可以尝试以下测试：

```markdown
测试 1：基础对话
  输入："你好，请介绍一下你自己"
  期望：AI 正常回复，介绍自己是 OpenClaw

测试 2：工具使用
  输入："今天天气怎么样？"（需要配置 Web Search）
  期望：AI 调用搜索，返回天气信息

测试 3：代码能力
  输入："用 Python 写一个快速排序算法"
  期望：AI 返回正确代码
```

---

## 🎓 从源码运行（开发者）

如果您是开发者，想修改 OpenClaw 源码：

### 源码安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 2. 安装依赖
pnpm install

# 3. 构建 UI（首次会自动安装 UI 依赖）
pnpm ui:build

# 4. 构建项目
pnpm build

# 5. 运行引导
pnpm openclaw onboard --install-daemon
```

### 开发模式运行

```bash
# 监听模式（自动重新加载）
pnpm gateway:watch

# 或使用 pnpm 运行 CLI
pnpm openclaw --version
```

---

## 🐛 故障排除快速指南

遇到问题时，不要慌张，按照以下步骤排查：

### 快速排查检查表

| 问题 | 排查步骤 | 解决方案 |
|------|----------|----------|
| **命令找不到** | `which openclaw` | 检查 PATH：`export PATH="$(npm prefix -g)/bin:$PATH"` |
| **WhatsApp 登录失败** | `node --version` | 确保使用 Node.js（非 Bun），检查网络连接 |
| **没有收到回复** | `openclaw pairing list` | 检查配对是否批准 |
| **认证失败** | `openclaw health` | 运行 `openclaw onboard` 重新配置 |
| **网关不启动** | `lsof -i :18789` | 检查端口是否被占用 |
| **Telegram 无响应** | `openclaw config get channels.telegram.botToken` | 检查 Bot Token 是否正确 |

### 诊断命令

```bash
# 综合诊断
openclaw doctor

# 详细诊断
openclaw doctor --fix

# 查看日志
openclaw logs --lines 100

# 健康检查
openclaw health
```

### 常见错误及解决方案

#### 错误一：端口被占用

```bash
# 检查端口占用
lsof -i :18789

# 或
netstat -tlnp | grep 18789

# 解决方案：使用其他端口
openclaw gateway --port 18790
```

#### 错误二：Node.js 版本不兼容

```bash
# 检查版本
node --version

# 使用 nvm 切换版本
nvm install 22
nvm use 22

# 重新安装 OpenClaw
npm uninstall -g openclaw
npm install -g openclaw@latest
```

#### 错误三：认证失败

```bash
# 检查认证配置
openclaw health

# 查看详细错误
openclaw doctor

# 重新认证
openclaw onboard
```

---

## 📊 专家思维模型：系统化问题排查

当遇到问题时，采用以下系统化方法：

```
┌─────────────────────────────────────────────────────────────────┐
│                    问题排查决策流程                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  步骤 1：收集信息                                                │
│     └── 问题是什么？什么时候开始？最近有什么变化？               │
│                              ↓                                   │
│  步骤 2：缩小范围                                                │
│     └── 是所有渠道都有问题，还是特定渠道？                      │
│     └── 是发送问题，还是接收问题？                              │
│     └── 是所有功能都不行，还是特定功能？                        │
│                              ↓                                   │
│  步骤 3：检查常见原因                                            │
│     └── 网关是否运行？                                           │
│     └── 认证是否有效？                                           │
│     └── 配对是否批准？                                           │
│     └── 网络是否正常？                                           │
│                              ↓                                   │
│  步骤 4：查看日志                                                │
│     └── openclaw logs --lines 100                               │
│                              ↓                                   │
│  步骤 5：运行诊断                                                │
│     └── openclaw doctor                                         │
│                              ↓                                   │
│  步骤 6：寻求帮助                                                │
│     └── 查看文档                                                │
│     └── 搜索 GitHub Issues                                       │
│     └── 提交新 Issue                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 下一步学习

完成以上步骤后，您已经拥有了一个工作的 OpenClaw 系统！建议继续学习：

### 推荐学习路径

| 顺序 | 主题 | 文档链接 | 预计时间 | 优先级 |
|------|------|----------|----------|--------|
| 1 | **核心概念** | [系统架构](/zh-CN/concepts/architecture) | 30 分钟 | ⭐⭐⭐⭐⭐ |
| 2 | **CLI 命令** | [CLI 文档](/zh-CN/cli/index) | 30 分钟 | ⭐⭐⭐⭐⭐ |
| 3 | **渠道配置** | [Channels](/zh-CN/channels/index) | 45 分钟 | ⭐⭐⭐⭐ |
| 4 | **技能系统** | [Skills](/zh-CN/tools/skills) | 30 分钟 | ⭐⭐⭐⭐ |
| 5 | **进阶配置** | [配置参考](/zh-CN/config/reference) | 60 分钟 | ⭐⭐⭐ |

### 进阶主题

| 主题 | 说明 |
|------|------|
| **沙箱配置** | 理解安全隔离机制 |
| **多代理系统** | 运行多个独立的 AI 代理 |
| **自定义技能** | 开发自己的工具和技能 |
| **远程网关** | 在服务器上部署 |

---

## 📚 深入理解：沙箱配置（进阶）

> **此部分为进阶内容，初学者可跳过。**

### 什么是沙箱？

沙箱是**安全隔离环境**，类似于浏览器的无痕模式：

```
┌─────────────────────────────────────────────────────────────────┐
│                    沙箱安全机制                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  无沙箱模式（sandbox: "off"）：                                  │
│    ├── AI 有完全访问权限                                        │
│    ├── 可以执行任何命令                                         │
│    ├── 可以读写任何文件                                         │
│    └── 风险：如果 AI 被诱导执行危险命令...                       │
│                                                                 │
│  沙箱模式（sandbox: "non-main"）：                              │
│    ├── AI 在隔离环境中运行                                      │
│    ├── 文件系统隔离                                             │
│    ├── 网络访问受限                                             │
│    └── 即使 AI 被攻击，不会影响主机系统                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 配置沙箱

**默认配置**（群组/频道会话使用沙箱）：

```json5
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.clawdbot/workspace",
        "sandbox": { "mode": "non-main" }
      }
    }
  }
}
```

**沙箱模式选项：**

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| `"off"` | 无沙箱 | 完全信任的环境、开发调试 |
| `"non-main"` | 仅非主会话使用沙箱（默认） | 生产环境推荐 |
| `"all"` | 所有会话都使用沙箱 | 高安全要求环境 |

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
| 诊断问题 | `openclaw doctor` |
| 发送消息 | `openclaw message send --target <目标> --message <内容>` |
| 配对列表 | `openclaw pairing list` |
| 批准配对 | `openclaw pairing approve <渠道> <配对码>` |
| 渠道登录 | `openclaw channels login` |

### 附录 B：术语表

| 英文术语 | 中文术语 | 说明 |
|---------|---------|------|
| Gateway | 网关 | OpenClaw 的核心控制平面 |
| Channel | 渠道 | 消息来源通道（WhatsApp、Telegram 等） |
| Agent | 代理 / 智能体 | 处理消息的 AI 程序 |
| Session | 会话 | 一次连续的对话上下文 |
| Workspace | 工作区 | 配置和运行时目录 |
| OAuth | OAuth | 开放授权协议 |
| CLI | CLI / 命令行工具 | Command Line Interface |
| Sandbox | 沙箱 | 安全隔离环境 |

### 附录 C：相关资源

- **官方文档**：[docs.openclaw.ai](https://docs.openclaw.ai)
- **GitHub 仓库**：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- **社区 Discord**：[discord.gg/clawd](https://discord.gg/clawd)
- **问题反馈**：[GitHub Issues](https://github.com/openclaw/openclaw/issues)

---

## 🎉 恭喜！

**您已经完成了新手上路！**

您现在应该能够：
- ✅ 安装 OpenClaw CLI
- ✅ 运行向导完成配置
- ✅ 连接至少一个消息渠道
- ✅ 发送和接收 AI 消息
- ✅ 理解基本的安全机制

如果您在实践中遇到问题，请参考[常见问题](/zh-CN/help/faq)或[故障排除](/zh-CN/help/troubleshooting)文档。

> *"最好的学习方式就是动手做。安装它，使用它，你会理解的。"*
>
> *— OpenClaw 社区*
