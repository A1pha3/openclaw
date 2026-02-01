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

本文档专为**完全新手**设计，我们会深入浅出地讲解每个步骤，帮助你真正理解 OpenClaw 的工作原理。

## 🎯 快速导航

| 你想要... | 查看 |
|-----------|------|
| 5分钟快速体验 | [快速入门](/zh-CN/start/quick-start) |
| 完整安装指南 | [安装指南](/zh-CN/install) |
| 了解向导模式 | [向导模式详解](/zh-CN/start/wizard) |
| 故障排除 | [常见问题](/zh-CN/help/faq) |

---

## 💡 首先理解：什么是 OpenClaw？

### 一句话解释

OpenClaw 是**你的个人 AI 助手网关**，它能将 WhatsApp、Telegram、Discord 等你常用的聊天应用，连接到强大的 AI（如 Claude、GPT），让 AI 通过你熟悉的聊天界面为你服务。

### 类比理解

想象 OpenClaw 是一个**智能接线员**：

```
你 (WhatsApp/Telegram/Discord) 
    ↓
OpenClaw 网关（智能接线员）
    ↓
AI 大脑 (Claude/GPT)
```

就像你可以通过电话、邮件、微信联系同一个人，OpenClaw 让你可以通过不同的聊天应用，与同一个 AI 助手对话。

### 核心概念（新手必知）

| 概念 | 通俗解释 | 类比 |
|------|---------|------|
| **Gateway（网关）** | 核心控制中心，管理所有连接 | 公司的总机系统 |
| **Channel（渠道）** | 消息来源（WhatsApp/Telegram等） | 不同的电话号码 |
| **Agent（代理）** | 实际处理消息的 AI 程序 | 接线员 |
| **Session（会话）** | 一次连续的对话上下文 | 通话记录 |
| **Workspace（工作区）** | 配置和数据的存储位置 | 办公桌抽屉 |

---

## 🛠️ 前置要求

### 必须准备

1. **Node.js >= 22**
   ```bash
   # 检查版本
   node -v
   # 如果版本低于 22，访问 https://nodejs.org 下载安装
   ```

2. **pnpm**（可选，推荐用于源码构建）
   ```bash
   npm install -g pnpm
   ```

3. **Brave Search API Key**（强烈推荐，用于网页搜索）
   - 后续配置：`openclaw configure --section web`

### 平台支持

| 平台 | 支持情况 | 备注 |
|------|---------|------|
| macOS | ✅ 完全支持 | 推荐开发环境 |
| Linux | ✅ 完全支持 | 服务器部署首选 |
| Windows | ⚠️ 仅支持 WSL2 | 强烈推荐使用 WSL2，原生 Windows 未经充分测试 |

**Windows 用户注意**：请先安装 WSL2，然后在 Ubuntu 子系统中运行所有命令。

---

## 📦 第一步：安装 CLI（推荐方式）

### 为什么用 CLI？

CLI（命令行工具）是你与 OpenClaw 交互的主要方式。就像微信有手机 App，OpenClaw 有命令行工具。

### 安装命令

**macOS / Linux：**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell)：**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 安装方式对比

| 方式 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **安装脚本（推荐）** | 快速开始 | 自动完成所有设置 | 需要网络 |
| **npm 全局安装** | 已有 Node 环境 | 简单直接 | 需要手动配置 |
| **源码安装** | 开发者/贡献者 | 可修改源码 | 需要构建 |
| **Docker** | 服务器部署 | 环境隔离 | 配置复杂 |

**备用方案** - npm 全局安装：
```bash
npm install -g openclaw@latest
# 或
pnpm add -g openclaw@latest
```

---

## 🧙 第二步：运行引导向导

### 什么是向导？

向导就像**智能安装助手**，会一步步问你问题，自动配置好一切。你不需要手动编辑配置文件。

### 启动向导

```bash
openclaw onboard --install-daemon
```

**参数说明：**
- `--install-daemon`：安装后台服务，让网关随系统启动

### 向导配置内容

向导会帮你配置：

1. **本地 vs 远程网关**（推荐本地开始）
2. **AI 认证**（OAuth 或 API Key）
3. **消息渠道**（WhatsApp、Telegram、Discord 等）
4. **后台服务**（可选）
5. **推荐技能**（可选）

### 认证方式选择

| 方式 | 适用情况 | 推荐度 |
|------|---------|--------|
| **Anthropic API Key** | 有 Claude API | ⭐⭐⭐ 推荐 |
| **Anthropic OAuth** | 有 Claude Code CLI | ⭐⭐⭐ 推荐 |
| **OpenAI API Key** | 有 OpenAI API | ⭐⭐⭐ 推荐 |
| **OpenAI OAuth (Codex)** | 有 Codex CLI | ⭐⭐⭐ 推荐 |

**重要**：认证信息存储位置
- OAuth: `~/.openclaw/credentials/oauth.json`
- API Keys: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

**无头/服务器部署提示**：先在普通机器完成 OAuth，然后复制 `oauth.json` 到服务器。

---

## 🚀 第三步：启动网关

### 什么是网关？

**网关 = OpenClaw 的心脏**

它是唯一长期运行的进程，负责：
- 维护与 WhatsApp/Telegram 的连接
- 管理 WebSocket 控制平面
- 协调所有组件通信

### 检查网关状态

```bash
openclaw gateway status
```

### 手动启动（前台运行）

```bash
openclaw gateway --port 18789 --verbose
```

**参数说明：**
- `--port 18789`：默认端口
- `--verbose`：显示详细日志

### 后台服务（推荐）

如果在向导时选择了安装后台服务：

- **macOS**: LaunchAgent（用户登录时启动）
- **Linux**: systemd 用户单元（支持 linger 保持运行）

### ⚠️ 重要警告：Bun 运行时问题

**如果你使用 WhatsApp 或 Telegram，必须用 Node.js 运行网关，不要用 Bun！**

Bun 对这些渠道有已知兼容性问题。

---

## ✅ 第四步：快速验证（2分钟）

### 基础检查

```bash
# 查看整体状态
openclaw status

# 健康检查
openclaw health

# 深度安全检查
openclaw security audit --deep
```

### 仪表盘访问

浏览器打开：http://127.0.0.1:18789/

如果配置了 token，在设置中粘贴 token。

---

## 💬 第五步：配置第一个聊天渠道

### 方式 A：WhatsApp（扫码登录）

```bash
openclaw channels login
```

然后：
1. 打开手机 WhatsApp
2. 设置 → 已关联设备
3. 扫描二维码

详细文档：[WhatsApp 配置](/zh-CN/channels/whatsapp)

### 方式 B：Telegram（机器人 Token）

1. 找 @BotFather 创建机器人
2. 获取 token（格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`）
3. 向导会自动帮你配置

详细文档：[Telegram 配置](/zh-CN/channels/telegram)

### 方式 C：Discord

1. 在 Discord Developer Portal 创建应用
2. 获取 Bot Token
3. 配置权限并邀请入服务器

详细文档：[Discord 配置](/zh-CN/channels/discord)

---

## 🔐 第六步：配对与安全

### 什么是配对？

默认安全设置：不认识的人给你发消息时，会收到一个**配对码**，你需要手动批准，AI 才会回复。

这就像**好友申请** - 防止陌生人骚扰。

### 批准配对

```bash
# 查看待批准列表
openclaw pairing list whatsapp

# 批准某个配对码
openclaw pairing approve whatsapp <配对码>
```

**提示**：第一次发送消息如果没有回复，多半是配对未批准！

---

## 🧪 第七步：端到端测试

### 发送测试消息

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw!"
```

### 测试仪表盘聊天

1. 运行 `openclaw dashboard` 或访问 http://127.0.0.1:18789/
2. 无需配置渠道即可聊天
3. 这是最快的体验方式

### 如果 `openclaw health` 显示 "no auth configured"

回到向导重新配置认证：`openclaw onboard`

---

## 🎓 从源码运行（开发者）

如果你想修改 OpenClaw 源码或贡献代码：

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖
pnpm install

# 构建 UI（首次自动安装 UI 依赖）
pnpm ui:build

# 构建项目
pnpm build

# 运行引导
openclaw onboard --install-daemon
```

**没有全局安装时**：使用 `pnpm openclaw ...` 运行命令

---

## 🐛 故障排除快速指南

| 问题 | 解决方案 |
|------|---------|
| 命令找不到 | 检查 PATH：`export PATH="$(npm prefix -g)/bin:$PATH"` |
| WhatsApp 登录失败 | 确保使用 Node.js（非 Bun），检查网络 |
| 没有回复 | 检查配对是否批准：`openclaw pairing list` |
| 认证失败 | 运行 `openclaw onboard` 重新配置 |
| 网关不启动 | 检查端口是否被占用：`lsof -i :18789` |

更多帮助：[故障排除](/zh-CN/help/troubleshooting)

---

## 🎯 下一步

完成以上步骤后，你已经拥有了一个工作的 OpenClaw 系统！建议继续学习：

1. **[核心概念](/zh-CN/concepts/architecture)** - 深入理解系统架构
2. **[渠道配置](/zh-CN/channels)** - 连接更多聊天应用
3. **[技能系统](/zh-CN/tools/skills)** - 扩展 AI 能力
4. **[CLI 命令](/zh-CN/cli)** - 掌握所有管理命令

---

## 📚 深入理解：沙箱配置（进阶）

默认沙箱设置使用 `session.mainKey`（默认 `"main"`），这意味着群组/频道会话会被沙箱隔离。

如果你想让主代理始终在主机上运行（不沙箱化），添加显式配置：

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

**什么是沙箱？**

沙箱是**安全隔离环境**，类似于浏览器的无痕模式。代理在沙箱中运行时：
- ✅ 可以安全执行代码
- ✅ 不会影响主机系统
- ✅ 隔离的文件系统

沙箱模式选项：
- `"off"`：无沙箱（完全访问主机）
- `"non-main"`：仅非主会话使用沙箱（默认）
- `"all"`：所有会话都使用沙箱

---

**恭喜！你已经完成了新手上路！** 🎉

遇到问题？查看 [FAQ](/zh-CN/help/faq) 或在 GitHub 提交 Issue。
