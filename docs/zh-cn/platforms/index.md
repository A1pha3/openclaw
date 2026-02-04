---
summary: "部署平台指南 - macOS、Linux、Windows、iOS、Android 等平台配置"
read_when:
  - 在不同操作系统上部署
  - 配置移动节点
  - 选择部署方式
title: "部署平台"
---

# 💻 部署平台

本文档介绍 OpenClaw 在各种平台上的安装和配置方法。

## 🎯 平台支持概览

### 桌面平台

| 平台 | 支持情况 | 网关 | CLI | 节点 |
|------|----------|------|-----|------|
| [macOS](/zh-CN/platforms/macos) | ✅ 完全支持 | ✅ | ✅ | ✅ |
| [Linux](/zh-CN/platforms/linux) | ✅ 完全支持 | ✅ | ✅ | ❌ |
| [Windows (WSL2)](zh-CN/platforms/windows) | ⚠️ 推荐 WSL2 | ✅ | ✅ | ❌ |

### 移动平台

| 平台 | 支持情况 | 说明 |
|------|----------|------|
| [iOS](/zh-CN/platforms/ios) | ✅ 完全支持 | 节点模式 |
| [Android](/zh-CN/platforms/android) | ✅ 完全支持 | 节点模式 |

### 云平台

| 平台 | 说明 |
|------|------|
| [Docker](/zh-CN/install/docker) | 容器化部署 |
| [Fly.io](/zh-CN/platforms/fly) | 边缘部署 |

### 单板电脑

| 平台 | 说明 |
|------|------|
| [树莓派](/zh-CN/platforms/raspberry-pi) | ARM 架构 |

---

## 🍎 macOS

### 系统要求

| 要求 | 最低 | 推荐 |
|------|------|------|
| macOS 版本 | 13.0 (Ventura) | 14.0+ (Sonoma) |
| 内存 | 8 GB | 16 GB |
| 存储 | 10 GB | 50 GB+ |

### 安装方式

**方式 1：安装脚本（推荐）**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**方式 2：Homebrew**
```bash
brew install openclaw
```

**方式 3：手动安装**
```bash
npm install -g openclaw@latest
```

### macOS 特有功能

| 功能 | 说明 |
|------|------|
| 菜单栏应用 | 网关控制、健康状态 |
| Voice Wake | 语音唤醒 |
| Talk Mode | 语音对话 |
| iMessage | 集成 macOS 消息 |
| Canvas | 实时画布 |

### 权限配置

macOS 需要以下权限：

| 权限 | 用途 | 配置位置 |
|------|------|----------|
| 辅助功能 | 控制其他应用 | 系统设置 → 隐私与安全性 |
| 完全磁盘访问 | 访问文件 | 系统设置 → 隐私与安全性 |
| 屏幕录制 | 屏幕截图/录制 | 系统设置 → 隐私与安全性 |
| 麦克风 | 语音输入 | 系统设置 → 隐私与安全性 |

---

## 🐧 Linux

### 支持的发行版

| 发行版 | 支持情况 | 说明 |
|--------|----------|------|
| Ubuntu | ✅ 完全支持 | 推荐 |
| Debian | ✅ 完全支持 | |
| Fedora | ✅ 完全支持 | |
| Arch Linux | ✅ 完全支持 | |
| NixOS | ✅ 完全支持 | [专用指南](/zh-CN/install/nix) |

### 安装方式

**方式 1：安装脚本（推荐）**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**方式 2：npm**
```bash
npm install -g openclaw@latest
```

**方式 3：源码安装**
```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

### 系统服务

使用 systemd 用户服务：

```bash
# 安装服务
openclaw onboard --install-daemon

# 查看状态
systemctl --user status openclaw-gateway

# 管理服务
systemctl --user start openclaw-gateway
systemctl --user stop openclaw-gateway
systemctl --user restart openclaw-gateway

# 开机启动
loginctl enable-linger
```

---

## 🪟 Windows (WSL2)

### ⚠️ 重要说明

**强烈推荐使用 WSL2**，而非原生 Windows。原生 Windows 未经充分测试，可能存在兼容性问题。

### WSL2 安装

```powershell
# 安装 WSL2（以 Ubuntu 为例）
wsl --install -d Ubuntu

# 重启后打开 Ubuntu 终端
```

### WSL2 中安装 OpenClaw

```bash
# 进入 WSL2 Ubuntu
wsl -d Ubuntu

# 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 运行引导
openclaw onboard --install-daemon
```

### WSL2 访问 Windows 应用

```bash
# 从 WSL2 打开 Windows 应用
explorer.exe .

# 运行 Windows 命令
cmd.exe /c "dir"
```

### 故障排除

**问题：WSL2 网络配置**
```powershell
# 重置 WSL2 网络
netsh winsock reset
netsh int ip reset all
ipconfig /release
ipconfig /renew
```

---

## 📱 iOS

### 系统要求

| 要求 | 最低 | 推荐 |
|------|------|------|
| iOS 版本 | 16.0 | 17.0+ |
| 设备 | iPhone | iPhone 15+ |

### 安装方式

从 App Store 下载 **OpenClaw** 应用。

### 功能

| 功能 | 说明 |
|------|------|
| Canvas | 实时画布交互 |
| Voice Wake | 语音唤醒 |
| Talk Mode | 语音对话 |
| 相机 | 图像理解 |
| 屏幕录制 | 屏幕捕获 |

### 配对流程

1. 在 iOS 设备上打开 OpenClaw 应用
2. 点击「配对设备」
3. 使用 CLI 配对：

```bash
# 查看配对码
openclaw pairing list

# 批准配对
openclaw pairing approve ios <配对码>
```

---

## 🤖 Android

### 系统要求

| 要求 | 最低 | 推荐 |
|------|------|------|
| Android 版本 | 13.0 | 14.0+ |
| 内存 | 6 GB | 8 GB+ |

### 安装方式

从 Google Play Store 或 F-Droid 下载 **OpenClaw** 应用。

### 功能

| 功能 | 说明 |
|------|------|
| Canvas | 实时画布交互 |
| Talk Mode | 语音对话 |
| 相机 | 图像理解 |
| 屏幕录制 | 屏幕捕获 |
| SMS | 短信功能（可选） |

### 配对流程

1. 在 Android 设备上打开 OpenClaw 应用
2. 点击「配对设备」
3. 使用 CLI 配对：

```bash
openclaw pairing approve android <配对码>
```

---

## 🐳 Docker

### 使用官方镜像

```bash
# 拉取镜像
docker pull openclaw/openclaw:latest

# 运行容器
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -p 18793:18793 \
  -v ~/.openclaw:/root/.openclaw \
  openclaw/openclaw:latest
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    ports:
      - "18789:18789"
      - "18793:18793"
    volumes:
      - ~/.openclaw:/root/.openclaw
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    restart: unless-stopped
```

详细指南：[Docker 安装](/zh-CN/install/docker)

---

## ☁️ 云平台

### Fly.io

```bash
# 安装 flyctl
curl -L https://fly.io/install.sh | sh

# 部署
fly launch
fly deploy
```

详细指南：[Fly.io 部署](/zh-CN/platforms/fly)

### 本地模型 (Ollama)

```bash
# 安装 Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 拉取模型
ollama pull llama3

# 配置 OpenClaw 使用本地模型
openclaw config set models.providers.ollama.baseUrl "http://localhost:11434"
openclaw config set agents.defaults.model "ollama/llama3"
```

---

## 🛠️ 平台特定配置

### 无头服务器

无图形界面的服务器配置：

```json5
{
  gateway: {
    bind: "0.0.0.0",
    port: 18789
  },
  "web": {
    "enabled": true
  }
}
```

### 资源受限环境

低内存设备配置：

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-haiku-3-5-20241022",
      sandbox: {
        mode: "all",
        docker: {
          memoryLimit: "1g"
        }
      }
    }
  }
}
```

---

## 📊 性能要求

| 平台 | CPU | 内存 | 存储 |
|------|-----|------|------|
| macOS | 任意现代 CPU | 8 GB+ | 10 GB+ |
| Linux | 任意现代 CPU | 4 GB+ | 10 GB+ |
| iOS | A14+ | 6 GB+ | - |
| Android | Snapdragon 8+ | 8 GB+ | - |

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw status` | 查看平台状态 |
| `openclaw nodes list` | 查看连接的节点 |
| `openclaw gateway` | 管理网关 |
| `openclaw doctor` | 诊断平台问题 |

---

## 📚 相关文档

- [macOS 详细指南](/zh-CN/platforms/macos)
- [Linux 详细指南](/zh-CN/platforms/linux)
- [Windows (WSL2) 指南](/zh-CN/platforms/windows)
- [iOS 详细指南](/zh-CN/platforms/ios)
- [Android 详细指南](/zh-CN/platforms/android)
- [Docker 安装](/zh-CN/install/docker)
- [节点系统](/zh-CN/nodes) - 移动节点配置

---

**选择最适合您的平台，开始使用 OpenClaw！** 🦞
