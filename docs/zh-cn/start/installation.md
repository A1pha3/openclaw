# 安装指南

本文档详细介绍 Moltbot 的各种安装方式，帮助您根据自己的环境选择最合适的安装方法。

## 系统要求

### 基础要求

| 组件 | 最低版本 | 推荐版本 |
|------|----------|----------|
| Node.js | 22.12.0 | 最新 LTS |
| 操作系统 | macOS 12+ / Ubuntu 20.04+ / Windows 10+ | 最新稳定版 |
| 内存 | 512MB | 2GB+ |
| 磁盘空间 | 500MB | 2GB+ |

### 平台特定要求

**macOS:**
- 如需构建原生应用，需安装 Xcode 或 Command Line Tools
- 仅使用 CLI + Gateway 时，只需 Node.js

**Windows:**
- 强烈推荐使用 **WSL2**（Windows Subsystem for Linux）
- 原生 Windows 支持有限，可能遇到兼容性问题
- WSL2 安装后，按 Linux 步骤操作

**Linux:**
- 需要 glibc 2.17+（大多数现代发行版都满足）
- 某些功能需要 systemd（用于后台服务）

## 安装方式

### 方式一：安装脚本（推荐新手）

这是最简单的安装方式，脚本会自动处理依赖和配置。

**macOS / Linux:**

```bash
curl -fsSL https://molt.bot/install.sh | bash
```

**Windows (PowerShell，以管理员身份运行):**

```powershell
iwr -useb https://molt.bot/install.ps1 | iex
```

安装脚本会：
- 检测您的系统环境
- 安装 Moltbot CLI
- 配置 PATH 环境变量
- 创建必要的目录结构

### 方式二：npm/pnpm 全局安装

如果您熟悉 Node.js 生态，可以直接使用包管理器：

```bash
# 使用 npm
npm install -g moltbot@latest

# 或使用 pnpm（推荐）
pnpm add -g moltbot@latest
```

### 方式三：从源码安装

适合开发者或需要最新功能的用户：

```bash
# 克隆仓库
git clone https://github.com/moltbot/moltbot.git
cd moltbot

# 安装依赖
pnpm install

# 构建 UI
pnpm ui:build

# 构建项目
pnpm build

# 运行配置向导
pnpm moltbot onboard --install-daemon
```

从源码运行网关：

```bash
node moltbot.mjs gateway --port 18789 --verbose
```

### 方式四：Docker 安装

适合容器化部署：

```bash
# 拉取镜像
docker pull ghcr.io/moltbot/moltbot:latest

# 运行容器
docker run -d \
  --name moltbot \
  -p 18789:18789 \
  -v ~/.clawdbot:/root/.clawdbot \
  ghcr.io/moltbot/moltbot:latest \
  gateway --port 18789
```

使用 docker-compose：

```yaml
version: '3.8'
services:
  moltbot:
    image: ghcr.io/moltbot/moltbot:latest
    ports:
      - "18789:18789"
    volumes:
      - ~/.clawdbot:/root/.clawdbot
    command: gateway --port 18789
    restart: unless-stopped
```

## 版本与更新渠道

Moltbot 提供三个发布渠道：

| 渠道 | 说明 | 安装命令 |
|------|------|----------|
| **stable** | 正式发布版本，最稳定 | `npm install -g moltbot@latest` |
| **beta** | 预发布版本，包含新功能 | `npm install -g moltbot@beta` |
| **dev** | 开发版本，最新代码 | 从源码安装 |

### 更新 Moltbot

```bash
# 更新到最新稳定版
moltbot update

# 或手动更新
npm update -g moltbot
```

### 版本回滚

如果新版本有问题，可以回滚到之前的版本：

```bash
npm install -g moltbot@2026.1.15
```

## 安装验证

安装完成后，运行以下命令验证：

```bash
# 检查版本
moltbot --version

# 查看帮助
moltbot --help

# 运行诊断
moltbot doctor
```

## 目录结构

安装后，Moltbot 会创建以下目录结构：

```
~/.clawdbot/
├── moltbot.json          # 主配置文件
├── credentials/          # 认证信息
│   ├── oauth.json        # OAuth 凭证
│   └── whatsapp/         # WhatsApp 会话
├── agents/               # AI 代理数据
│   └── <agentId>/
│       └── agent/
│           ├── auth-profiles.json
│           └── sessions/
├── sessions/             # Pi 会话日志
└── .env                  # 环境变量（可选）
```

## 环境变量

Moltbot 支持通过环境变量进行配置：

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `CLAWDBOT_CONFIG_PATH` | 配置文件路径 | `~/.clawdbot/moltbot.json` |
| `CLAWDBOT_STATE_DIR` | 状态目录 | `~/.clawdbot` |
| `CLAWDBOT_GATEWAY_TOKEN` | 网关认证令牌 | - |
| `OPENAI_API_KEY` | OpenAI API 密钥 | - |
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 | - |

您可以在 `~/.clawdbot/.env` 中设置这些变量：

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
```

## 多实例安装

如果需要运行多个 Moltbot 实例：

```bash
# 实例 A
CLAWDBOT_CONFIG_PATH=~/.clawdbot/a.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-a \
moltbot gateway --port 19001

# 实例 B
CLAWDBOT_CONFIG_PATH=~/.clawdbot/b.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-b \
moltbot gateway --port 19002
```

## 后台服务安装

### macOS (launchd)

```bash
moltbot onboard --install-daemon
```

这会创建 `~/Library/LaunchAgents/bot.molt.gateway.plist`。

手动管理服务：

```bash
# 查看状态
moltbot service status

# 启动
moltbot service start

# 停止
moltbot service stop

# 重启
moltbot service restart
```

### Linux (systemd)

```bash
moltbot onboard --install-daemon
```

这会创建 `~/.config/systemd/user/moltbot-gateway.service`。

手动管理：

```bash
systemctl --user status moltbot-gateway
systemctl --user start moltbot-gateway
systemctl --user stop moltbot-gateway
systemctl --user restart moltbot-gateway
```

## 卸载

### 完全卸载

```bash
# 停止服务
moltbot service stop

# 卸载 CLI
npm uninstall -g moltbot

# 删除配置和数据（谨慎！）
rm -rf ~/.clawdbot
```

### 仅卸载 CLI（保留数据）

```bash
npm uninstall -g moltbot
```

## 故障排除

### 安装脚本失败

如果安装脚本失败，尝试：

1. 检查网络连接
2. 使用 npm 直接安装
3. 查看详细错误信息

### 权限问题

如果遇到权限问题：

```bash
# macOS/Linux - 不要使用 sudo 安装全局包
# 而是修复 npm 权限
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH
```

### Node.js 版本问题

如果 Node.js 版本不符合要求：

```bash
# 使用 nvm 安装正确版本
nvm install 22
nvm use 22
```

## 下一步

安装完成后，继续阅读：

- [初次配置](/zh-cn/start/first-setup) - 完成基础配置
- [快速入门](/zh-cn/start/quick-start) - 发送第一条消息
- [配置参考](/zh-cn/config/reference) - 详细配置选项
