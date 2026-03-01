---
summary: "详细介绍 OpenClaw 的各种安装方式，包括安装脚本、npm/pnpm、源码、Docker 以及安装验证、目录结构、多实例和后台服务配置"
read_when:
  - 需要详细了解安装选项时
  - 选择特定安装方式前
  - 遇到安装问题时
  - 配置多实例或后台服务时
title: "安装指南"
---

# 📦 安装指南

**目标**：根据自己的环境选择最合适的安装方式，完成 OpenClaw 的完整安装和基础配置。

本指南涵盖所有安装方式，从最简单的安装脚本到高级的 Docker 部署。无论您是新手还是资深开发者，都能找到适合自己的安装方案。

---

## 🎯 学习目标

完成本指南学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的系统要求和运行环境
- [ ] 掌握至少两种安装方式及其适用场景
- [ ] 完成 OpenClaw CLI 的安装和验证
- [ ] 理解目录结构和配置文件的作用

### 进阶目标（建议掌握）

- [ ] 配置后台服务实现开机自启动
- [ ] 运行多个 OpenClaw 实例
- [ ] 使用 Docker 进行容器化部署
- [ ] 排查常见的安装问题

---

## 💡 为什么理解安装方式很重要？

### 设计决策背景

在选择安装方式之前，我们需要理解**为什么 OpenClaw 提供多种安装选项**：

```
┌─────────────────────────────────────────────────────────────────┐
│                    安装方式设计决策                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  问题：我们如何满足不同用户群体的需求？                            │
│                                                                 │
│  可选方案：                                                      │
│  1. 只提供一种"最佳"方式 → 简单但不够灵活                        │
│  2. 提供源码让用户自己编译 → 灵活但门槛高                        │
│  3. 提供多种安装方式（推荐方案）→ 平衡易用性和灵活性              │
│                                                                 │
│  最终选择：方案 3                                                │
│  理由：                                                          │
│  - 新手：使用安装脚本，一键完成                                   │
│  - 开发者：从源码安装，可自由修改                                 │
│  - 运维：使用 Docker，便于部署和迁移                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 安装方式对比

| 方式 | 适用人群 | 优点 | 缺点 | 推荐度 |
|------|----------|------|------|--------|
| **安装脚本** | 新手、快速体验 | 自动化程度高、简单 | 定制能力有限 | ⭐⭐⭐⭐⭐ |
| **npm/pnpm** | Node.js 开发者 | 熟悉的生态、版本管理 | 需要 Node.js 环境 | ⭐⭐⭐⭐ |
| **源码安装** | 开发者、贡献者 | 可修改源码、最新功能 | 需要构建步骤 | ⭐⭐⭐ |
| **Docker** | 运维、服务器 | 环境隔离、易于部署 | 配置较复杂 | ⭐⭐⭐⭐ |

---

## 🛠️ 前置要求

在开始安装之前，请确认您的系统满足以下要求。

### 基础系统要求

| 组件 | 最低版本 | 推荐版本 | 说明 |
|------|----------|----------|------|
| **Node.js** | 22.12.0 | 最新 LTS | JavaScript 运行时 |
| **操作系统** | macOS 12+ / Ubuntu 20.04+ / Windows 10+ | 最新稳定版 | 平台兼容性 |
| **内存** | 512MB | 2GB+ | 运行 AI 模型需要更多内存 |
| **磁盘空间** | 500MB | 2GB+ | 存储配置和会话数据 |

### 检查 Node.js 版本

```bash
# 检查 Node.js 版本
node --version

# 检查 npm/pnpm 版本
npm --version
# 或
pnpm --version
```

**如果版本不符合要求**，请参考[新手上路指南](/zh-CN/start/getting-started)中的 Node.js 安装说明。

### 平台特定要求

#### macOS

| 要求 | 说明 |
|------|------|
| **构建原生应用** | 需要安装 Xcode 或 Command Line Tools |
| **仅 CLI + Gateway** | 只需 Node.js，Xcode 可选 |

安装 Command Line Tools：

```bash
# 检查是否已安装
xcode-select --version

# 如果未安装，执行安装
xcode-select --install
```

#### Windows

> **⚠️ 强烈推荐**：Windows 用户应使用 **WSL2**（Windows Subsystem for Linux 2）。

**为什么推荐 WSL2？**

| 方面 | 原生 Windows | WSL2 |
|------|--------------|------|
| **兼容性** | 有限，部分功能不支持 | 与 Linux 完全兼容 |
| **性能** | 一般 | 接近原生 Linux |
| **生态** | 需要适配 | 直接使用 Linux 生态 |
| **维护** | 需要单独配置 | 自动更新 |

**WSL2 安装步骤：**

1. 启用 WSL 功能：在 PowerShell（管理员）中运行
   ```powershell
   wsl --install
   ```

2. 重启电脑

3. 安装 Ubuntu（或其他 Linux 发行版）：
   ```powershell
   wsl --install -d Ubuntu
   ```

4. 在 WSL2 中按照 Linux 步骤安装 OpenClaw

#### Linux

| 要求 | 说明 |
|------|------|
| **glibc** | 2.17+（大多数现代发行版默认满足） |
| **systemd** | 用于后台服务管理（Ubuntu 18.04+、CentOS 7+） |
| **curl/git** | 用于下载安装脚本和源码 |

---

## 📦 安装方式详解

### 方式一：安装脚本（推荐新手）

安装脚本是**最简单的方式**，它会自动检测系统环境、安装依赖、配置路径。

#### 为什么设计安装脚本？

```
问题：用户第一次使用 OpenClaw，如何降低安装门槛？

解决方案：提供一个自动化安装脚本
  ✅ 自动检测系统环境
  ✅ 自动安装 CLI 到正确路径
  ✅ 自动配置 PATH 环境变量
  ✅ 自动创建目录结构
  ✅ 处理权限问题
```

#### 执行安装

**macOS / Linux：**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell，以管理员身份运行)：**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

#### 安装脚本做了什么？

```
┌─────────────────────────────────────────────────────────────────┐
│                    安装脚本执行流程                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 环境检测                                                    │
│     ├── 检测操作系统类型                                         │
│     ├── 检测 Node.js 是否安装                                    │
│     └── 检测包管理器（npm/pnpm）                                 │
│                              ↓                                   │
│  2. 下载最新版本                                                 │
│     └── 从 npm registry 获取最新 openclaw 包                     │
│                              ↓                                   │
│  3. 安装 CLI                                                    │
│     ├── 安装到系统 PATH 目录                                     │
│     └── 设置执行权限                                             │
│                              ↓                                   │
│  4. 初始化配置                                                   │
│     ├── 创建默认配置目录                                         │
│     └── 生成初始配置文件                                         │
│                              ↓                                   │
│  5. 验证安装                                                    │
│     ├── 检查版本                                                 │
│     └── 提示下一步操作                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 方式二：npm/pnpm 全局安装

如果您熟悉 Node.js 生态，可以直接使用包管理器。

#### 为什么使用 npm/pnpm？

```
优势：
  ✅ 熟悉的安装流程
  ✅ 版本管理简单
  ✅ 可以轻松更新或回滚
  ✅ 与现有 Node.js 项目集成
```

#### 执行安装

**使用 npm：**

```bash
npm install -g openclaw@latest
```

**使用 pnpm（推荐）：**

```bash
pnpm add -g openclaw@latest
```

> **💡 专家提示**：pnpm 比 npm 更快，磁盘占用更少，且有更好的依赖管理。强烈推荐 Node.js 开发者使用 pnpm。

#### 验证安装

```bash
# 检查版本
openclaw --version

# 期望输出：2026.x.x

# 查看帮助
openclaw --help
```

### 方式三：从源码安装（开发者）

从源码安装适合需要**修改源码**或**贡献代码**的开发者。

#### 源码结构概览

```
openclaw/
├── packages/
│   ├── cli/              # 命令行工具
│   ├── gateway/          # 网关核心
│   ├── pi/              # AI 代理运行时
│   ├── channels/        # 消息渠道实现
│   └── ...
├── scripts/              # 构建和发布脚本
├── docs/                # 文档
└── package.json         # 工作空间配置
```

#### 从源码安装

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

# 5. 全局链接（可选）
pnpm link --global
```

#### 从源码运行

```bash
# 使用 pnpm 运行（推荐）
pnpm openclaw --version
pnpm openclaw gateway

# 或直接运行构建后的产物
node dist/cli.mjs gateway --port 18789
```

### 方式四：Docker 安装（容器化部署）

Docker 安装适合**服务器部署**或需要**环境隔离**的场景。

#### Docker 适用场景

| 场景 | 是否推荐 Docker |
|------|----------------|
| 服务器部署 | ✅ 推荐 |
| 多实例运行 | ✅ 推荐 |
| 本地开发测试 | ✅ 可选 |
| 生产环境 | ✅ 推荐 |
| Windows 原生运行 | ⚠️ 需要 WSL2 |

#### 使用 Docker 运行

**基本命令：**

```bash
# 拉取镜像
docker pull ghcr.io/openclaw/openclaw:latest

# 运行容器
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/.clawdbot:/root/.clawdbot \
  ghcr.io/openclaw/openclaw:latest \
  gateway --port 18789
```

**使用 docker-compose：**

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  openclaw:
    image: ghcr.io/openclaw/openclaw:latest
    container_name: openclaw
    ports:
      - "18789:18789"
    volumes:
      - ~/.clawdbot:/root/.clawdbot
    restart: unless-stopped
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    command: gateway --port 18789
```

**启动服务：**

```bash
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

#### Docker 最佳实践

```
专家建议：
  ✅ 使用 docker-compose 管理多个服务
  ✅ 设置环境变量而非硬编码密钥
  ✅ 使用 volumes 持久化数据
  ✅ 配置 restart 策略实现自启动
  ✅ 定期更新镜像获取安全补丁
```

---

## 🔄 版本与更新管理

### 发布渠道

OpenClaw 维护三个发布渠道：

| 渠道 | 说明 | 安装命令 | 更新频率 |
|------|------|----------|----------|
| **stable** | 正式发布版本，最稳定 | `npm install -g openclaw@latest` | 每月 |
| **beta** | 预发布版本，包含新功能 | `npm install -g openclaw@beta` | 每周 |
| **dev** | 开发版本，最新代码 | 从源码安装 | 每日 |

**切换发布渠道：**

```bash
# 查看当前版本信息
openclaw --version

# 更新到最新稳定版
openclaw update

# 或手动指定版本
npm install -g openclaw@2026.1.27
```

### 版本回滚

如果新版本出现问题，可以回滚到之前的版本：

```bash
# 查看可用版本
npm view openclaw versions --json

# 安装特定版本
npm install -g openclaw@2026.1.15
```

---

## ✅ 安装验证

安装完成后，运行以下命令验证安装是否成功。

### 验证步骤

```bash
# 1. 检查版本
openclaw --version

# 期望输出：2026.x.x

# 2. 查看帮助
openclaw --help

# 3. 运行诊断
openclaw doctor
```

### 诊断输出解读

```
✅ 通过的项目
❌ 需要关注的问题
⚠️  警告（建议修复）
```

如果诊断发现问题，按照提示修复后重新运行 `openclaw doctor`。

---

## 📁 目录结构详解

安装后，OpenClaw 会创建以下目录结构：

```
~/.clawdbot/                           # OpenClaw 主目录
├── openclaw.json                      # 主配置文件
├── credentials/                       # 认证信息目录
│   ├── oauth.json                    # OAuth 凭证（敏感）
│   ├── anthropic.json               # Anthropic API 凭证（敏感）
│   ├── openai.json                 # OpenAI API 凭证（敏感）
│   └── whatsapp/                   # WhatsApp 会话数据
├── agents/                          # AI 代理配置
│   └── <agentId>/                  # 代理唯一标识
│       └── agent/
│           ├── auth-profiles.json   # 认证配置
│           └── sessions/           # 会话数据
├── sessions/                        # Pi 会话日志
├── workspace/                       # 工作区（可配置）
│   ├── skills/                     # 技能目录
│   ├── AGENTS.md                  # 代理提示词
│   ├── SOUL.md                    # 身份配置
│   └── TOOLS.md                   # 工具配置
└── logs/                           # 日志文件
```

### 目录作用说明

| 目录/文件 | 作用 | 敏感度 |
|-----------|------|--------|
| `openclaw.json` | 主配置文件，定义全局行为 | 低 |
| `credentials/` | 存储各平台的认证凭证 | 🔴 高 |
| `agents/` | AI 代理的配置和状态 | 中 |
| `sessions/` | 会话历史和上下文 | 中 |
| `workspace/` | 用户工作区，包含自定义配置 | 低 |

> **🔐 安全提醒**：`credentials/` 目录包含敏感认证信息，请确保目录权限只对您可见：
> ```bash
> chmod -R 700 ~/.clawdbot/credentials
> ```

---

## ⚙️ 环境变量配置

OpenClaw 支持通过环境变量进行配置，适用于需要**自动化部署**或**容器化**的场景。

### 支持的环境变量

| 变量 | 说明 | 默认值 | 敏感度 |
|------|------|--------|--------|
| `CLAWDBOT_CONFIG_PATH` | 配置文件路径 | `~/.clawdbot/openclaw.json` | 低 |
| `CLAWDBOT_STATE_DIR` | 状态目录 | `~/.clawdbot` | 低 |
| `CLAWDBOT_GATEWAY_TOKEN` | 网关认证令牌 | 自动生成 | 🔴 高 |
| `OPENAI_API_KEY` | OpenAI API 密钥 | - | 🔴 高 |
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 | - | 🔴 高 |
| `ANTHROPIC_CLIENT_ID` | Anthropic OAuth Client ID | - | 中 |
| `ANTHROPIC_CLIENT_SECRET` | Anthropic OAuth Secret | - | 🔴 高 |

### 设置环境变量

**方式一：在 shell 配置文件中设置**

```bash
# 编辑 ~/.bashrc 或 ~/.zshrc
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.bashrc
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc

# 重新加载配置
source ~/.bashrc
```

**方式二：使用 .env 文件**

在 `~/.clawdbot/.env` 文件中设置：

```bash
# ~/.clawdbot/.env
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
ANTHROPIC_CLIENT_ID=your-client-id
ANTHROPIC_CLIENT_SECRET=your-client-secret
```

**方式三：在 Docker 中使用**

```bash
docker run -d \
  -e OPENAI_API_KEY=${OPENAI_API_KEY} \
  -e ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY} \
  ghcr.io/openclaw/openclaw:latest
```

---

## 🖥️ 后台服务配置

配置后台服务可以让 OpenClaw **随系统启动**，无需手动运行。

### macOS (launchd)

**安装后台服务：**

```bash
openclaw onboard --install-daemon
```

这会创建 `~/Library/LaunchAgents/openclaw.gateway.plist`。

**手动管理服务：**

```bash
# 查看服务状态
openclaw service status

# 启动服务
openclaw service start

# 停止服务
openclaw service stop

# 重启服务
openclaw service restart
```

**launchd 原理：**

```
┌─────────────────────────────────────────────────────────────────┐
│                    macOS launchd 工作机制                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Login (用户登录)                                          │
│         ↓                                                       │
│  launchd 读取 ~/Library/LaunchAgents/                           │
│         ↓                                                       │
│  加载 openclaw.gateway.plist                                    │
│         ↓                                                       │
│  启动 openclaw gateway (作为用户进程)                           │
│         ↓                                                       │
│  用户注销时，服务自动停止                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Linux (systemd)

**安装后台服务：**

```bash
openclaw onboard --install-daemon
```

这会创建 `~/.config/systemd/user/openclaw-gateway.service`。

**systemd 管理命令：**

```bash
# 查看服务状态
systemctl --user status openclaw-gateway

# 启动服务
systemctl --user start openclaw-gateway

# 停止服务
systemctl --user stop openclaw-gateway

# 重启服务
systemctl --user restart openclaw-gateway

# 查看日志
journalctl --user -u openclaw-gateway -f

# 设置开机自启
loginctl enable-linger $(whoami)
```

> **💡 专家提示**：使用 `loginctl enable-linger` 可以让 systemd 用户服务在用户注销后继续运行。

---

## 🔀 多实例配置

如果需要运行**多个独立的 OpenClaw 实例**（例如为不同用户或不同用途），可以通过环境变量隔离配置。

### 多实例场景

| 场景 | 需求 | 解决方案 |
|------|------|----------|
| **不同用户** | 每个用户独立配置 | 使用不同 STATE_DIR |
| **不同用途** | 工作/个人分开 | 使用不同 CONFIG_PATH |
| **测试环境** | 隔离生产配置 | 完全独立配置 |

### 启动多个实例

```bash
# 实例 A：个人使用
CLAWDBOT_CONFIG_PATH=~/.clawdbot/personal.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-personal \
openclaw gateway --port 18789

# 实例 B：工作使用（另一个终端）
CLAWDBOT_CONFIG_PATH=~/.clawdbot/work.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-work \
openclaw gateway --port 18790
```

### docker-compose 多实例

```yaml
version: '3.8'

services:
  openclaw-personal:
    image: ghcr.io/openclaw/openclaw:latest
    ports:
      - "18789:18789"
    volumes:
      - ~/.clawdbot/personal:/root/.clawdbot
    environment:
      - CLAWDBOT_CONFIG_PATH=/root/.clawdbot/openclaw.json
    restart: unless-stopped

  openclaw-work:
    image: ghcr.io/openclaw/openclaw:latest
    ports:
      - "18790:18789"
    volumes:
      - ~/.clawdbot/work:/root/.clawdbot
    environment:
      - CLAWDBOT_CONFIG_PATH=/root/.clawdbot/openclaw.json
    restart: unless-stopped
```

---

## 🗑️ 卸载指南

### 完全卸载

```bash
# 1. 停止服务
openclaw service stop

# 2. 卸载 CLI
npm uninstall -g openclaw
# 或
pnpm remove -g openclaw

# 3. 删除配置和数据（谨慎！）
rm -rf ~/.clawdbot

# 4. 删除后台服务配置
rm -f ~/Library/LaunchAgents/openclaw.gateway.plist       # macOS
rm -f ~/.config/systemd/user/openclaw-gateway.service      # Linux
```

### 仅卸载 CLI（保留数据）

```bash
npm uninstall -g openclaw
# 或
pnpm remove -g openclaw

# 配置和数据保留在 ~/.clawdbot/
```

---

## 🔧 常见问题与解决方案

### 问题一：安装脚本失败

**症状**：运行安装脚本时出错。

**排查步骤：**

```bash
# 1. 检查网络连接
curl -I https://openclaw.ai

# 2. 尝试使用 npm 直接安装
npm install -g openclaw@latest

# 3. 查看详细错误信息
curl -fsSL https://openclaw.ai/install.sh | bash -x
```

### 问题二：权限被拒绝

**症状**：`EACCES: permission denied` 错误。

**解决方案：**

```bash
# macOS/Linux - 不要使用 sudo 安装全局包！
# 而是修复 npm 权限

# 方案 1：配置 npm 使用用户目录
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH

# 方案 2：使用 nvm 管理 Node.js（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 22
nvm use 22
npm install -g openclaw@latest
```

### 问题三：Node.js 版本不兼容

**症状**：`Error: The engine "node" is incompatible` 错误。

**解决方案：**

```bash
# 使用 nvm 安装正确版本
nvm install 22
nvm use 22

# 验证版本
node --version  # 应显示 v22.x.x

# 重新安装 OpenClaw
npm uninstall -g openclaw
npm install -g openclaw@latest
```

### 问题四：后台服务无法启动

**症状**：服务状态显示 `failed` 或无法连接网关。

**排查步骤：**

```bash
# 1. 检查服务日志
journalctl --user -u openclaw-gateway -e    # Linux
# 或
openclaw service logs                        # CLI 查看

# 2. 手动运行网关排查
openclaw gateway --verbose

# 3. 检查端口占用
lsof -i :18789
```

---

## 📊 专家思维模型：安装决策框架

在选择安装方式时，使用以下决策框架：

```
┌─────────────────────────────────────────────────────────────────┐
│                    安装方式决策树                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  问：您使用 OpenClaw 的目的是什么？                              │
│                              ↓                                   │
│  ┌────────────────┬────────────────┬─────────────────┐         │
│  │ 快速体验       │ 日常使用       │ 开发/贡献       │         │
│  ↓                ↓                ↓                 ↓         │
│  安装脚本         npm/pnpm         源码安装          │         │
│                  或 Docker        或 Docker         │         │
│                              ↓                 ↓         │
│                              └─────────┬─────────┘         │
│                                        ↓                   │
│                         问：是否需要容器化部署？                 │
│                              ↓                ↓             │
│                          是             否                  │
│                           ↓              ↓                  │
│                       Docker        npm/pnpm               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 知识点回顾

完成本指南后，您应该掌握以下技能：

| 技能 | 掌握程度 |
|------|----------|
| 系统要求检查 | ⭐⭐⭐⭐⭐ |
| 选择合适的安装方式 | ⭐⭐⭐⭐ |
| 验证安装成功 | ⭐⭐⭐⭐⭐ |
| 理解目录结构 | ⭐⭐⭐ |
| 配置后台服务 | ⭐⭐⭐ |
| 运行多实例 | ⭐⭐ |

---

## 🎯 下一步学习

安装完成后，建议继续学习：

| 主题 | 文档链接 | 预计时间 |
|------|----------|----------|
| **快速上手** | [快速入门](/zh-CN/start/quick-start) | 5 分钟 |
| **完整指南** | [新手上路](/zh-CN/start/getting-started) | 30 分钟 |
| **向导模式** | [向导详解](/zh-CN/start/wizard) | 20 分钟 |
| **配置参考** | [配置手册](/zh-CN/config/reference) | 45 分钟 |

---

## 📖 附录

### 附录 A：命令速查表

| 操作 | 命令 |
|------|------|
| 检查版本 | `openclaw --version` |
| 查看帮助 | `openclaw --help` |
| 运行诊断 | `openclaw doctor` |
| 安装服务 | `openclaw onboard --install-daemon` |
| 查看服务状态 | `openclaw service status` |
| 启动服务 | `openclaw service start` |
| 停止服务 | `openclaw service stop` |

### 附录 B：术语表

| 英文术语 | 中文术语 | 说明 |
|---------|---------|------|
| CLI | CLI / 命令行工具 | Command Line Interface |
| Gateway | 网关 | OpenClaw 核心控制平面 |
| Daemon | 后台服务 | 长期运行的系统服务 |
| systemd | systemd | Linux 系统服务管理器 |
| launchd | launchd | macOS 系统服务管理器 |
| Docker | Docker | 容器化平台 |

### 附录 C：相关资源

- **官方文档**：[docs.openclaw.ai](https://docs.openclaw.ai)
- **GitHub**：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- **更新日志**：[CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

---

> **💡 提示**：如果本指南未能解决您的问题，请访问[故障排除](/zh-CN/help/troubleshooting)页面或提交 GitHub Issue。
