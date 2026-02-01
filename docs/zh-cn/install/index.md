---
summary: "OpenClaw 安装指南：推荐安装脚本、全局安装或源码安装"
read_when:
  - 安装 OpenClaw
  - 想要从 GitHub 安装
  - 选择安装方式
title: "安装指南"
---

# 📦 安装指南

本文档介绍多种安装 OpenClaw 的方法，从最简单到最灵活。

## 🎯 快速选择

| 你的情况 | 推荐方式 | 命令 |
|---------|---------|------|
| **只想快速开始** | 安装脚本（推荐） | `curl -fsSL https://openclaw.ai/install.sh \| bash` |
| **已有 Node 环境** | npm 全局安装 | `npm install -g openclaw@latest` |
| **开发者/贡献者** | 源码安装 | `git clone ...` |
| **服务器部署** | Docker | 见 [Docker 安装](/zh-CN/install/docker) |
| **Nix 用户** | Nix 包管理器 | 见 [Nix 安装](/zh-CN/install/nix) |

---

## 🚀 推荐方式：安装脚本

### 为什么推荐？

安装脚本会自动完成：
- ✅ 安装 CLI 工具
- ✅ 运行引导向导
- ✅ 配置环境
- ✅ 可选：安装后台服务

### 安装命令

**macOS / Linux：**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell)：**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 安装后步骤

如果跳过了引导向导，手动运行：
```bash
openclaw onboard --install-daemon
```

### 安装脚本选项

查看所有选项：
```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
```

常用选项：
```bash
# 跳过引导（仅安装）
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard

# 从源码安装（GitHub）
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git

# 非交互模式（适合自动化）
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-prompt

# 干运行（查看会做什么，但不执行）
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --dry-run
```

**环境变量方式**（适合 CI/自动化）：
```bash
export OPENCLAW_INSTALL_METHOD=git        # npm 或 git
export OPENCLAW_GIT_DIR=~/openclaw        # 源码目录
export OPENCLAW_NO_PROMPT=1               # 禁用提示
export OPENCLAW_DRY_RUN=1                 # 干运行
export OPENCLAW_NO_ONBOARD=1              # 跳过引导

curl -fsSL https://openclaw.ai/install.sh | bash
```

---

## 📋 系统要求

### 必需

| 组件 | 最低版本 | 说明 |
|------|---------|------|
| **Node.js** | >= 22 | 运行时环境 |
| **操作系统** | - | macOS、Linux、Windows WSL2 |
| **包管理器** | - | npm（内置）或 pnpm（源码构建） |

### 检查系统

```bash
# 检查 Node 版本
node -v

# 检查 npm
npm -v

# 推荐：安装 pnpm
npm install -g pnpm
```

### 平台特别说明

**macOS**：
- 仅需 CLI + 网关：Node.js 即可
- 构建 App：需要 Xcode / Command Line Tools

**Windows**：
- ⚠️ **必须使用 WSL2**（Ubuntu 推荐）
- 原生 Windows 未经测试，兼容性差

---

## 🛠️ 方式一：npm 全局安装

适合已有 Node.js 环境的用户。

### 安装命令

```bash
npm install -g openclaw@latest
```

或 pnpm：
```bash
pnpm add -g openclaw@latest
```

### 常见问题：sharp 安装失败

如果遇到 sharp（图像处理库）安装错误：

```bash
# 强制使用预编译二进制文件（跳过本地编译）
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

如果提示 `node-gyp` 错误：

**macOS**：
```bash
# 安装 Xcode Command Line Tools
xcode-select --install

# 安装 node-gyp
npm install -g node-gyp
```

或者使用上面的 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 跳过编译。

### 安装后配置

```bash
# 运行引导向导
openclaw onboard --install-daemon
```

---

## 💻 方式二：源码安装

适合想要修改代码或为项目贡献的开发者。

### 克隆仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 安装依赖

```bash
pnpm install
```

### 构建 UI

```bash
pnpm ui:build
```

首次运行会自动安装 UI 依赖。

### 构建项目

```bash
pnpm build
```

此命令同时会打包 A2UI 资源。

### 仅打包 A2UI（如果需要）

```bash
pnpm canvas:a2ui:bundle
```

### 运行引导

```bash
openclaw onboard --install-daemon
```

**没有全局安装时**：使用 `pnpm openclaw ...` 运行命令

### 从仓库运行网关

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

---

## 🐳 其他安装选项

### Docker

适合服务器部署或需要环境隔离的场景。

详细指南：[Docker 安装](/zh-CN/install/docker)

### Nix

适合 NixOS 用户或想要可复现构建的用户。

详细指南：[Nix 安装](/zh-CN/install/nix)

### Ansible

适合批量部署到多台服务器。

详细指南：[Ansible 安装](/zh-CN/install/ansible)

### Bun（仅 CLI）

⚠️ **不推荐用于网关运行**

Bun 对某些渠道（WhatsApp、Telegram）有兼容性问题。仅用于 CLI 命令。

详细指南：[Bun 安装](/zh-CN/install/bun)

---

## 🔧 安装方式对比

| 特性 | 安装脚本 | npm 全局 | 源码 | Docker | Nix |
|------|---------|---------|------|--------|-----|
| **复杂度** | 低 | 低 | 中 | 中 | 中 |
| **自动配置** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **可修改源码** | ❌ | ❌ | ✅ | ❌ | ❌ |
| **环境隔离** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **版本锁定** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **适合生产** | ✅ | ✅ | ⚠️ | ✅ | ✅ |

---

## 🩺 安装后检查清单

```bash
# 1. 检查安装
openclaw doctor

# 2. 查看状态
openclaw status

# 3. 健康检查
openclaw health

# 4. 打开仪表盘
openclaw dashboard
```

---

## 🐛 故障排除：命令找不到

### 诊断步骤

```bash
# 检查 Node 和 npm
node -v
npm -v

# 查看 npm 全局安装路径
npm prefix -g

# 检查 PATH
echo "$PATH"
```

### 问题：路径不在 PATH 中

如果 `$(npm prefix -g)/bin`（macOS/Linux）不在 `PATH` 中，添加它：

**zsh（macOS 默认）：**
```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**bash：**
```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Windows**：
将 `npm prefix -g` 的输出添加到系统 PATH 环境变量。

然后重新打开终端（或运行 `rehash` / `hash -r`）。

---

## 🔄 安装方法：npm vs git

安装脚本支持两种安装方法：

### npm 方法（默认）

```bash
npm install -g openclaw@latest
```

- 安装已发布的 npm 包
- 稳定版本
- 适合大多数用户

### git 方法

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

- 从 GitHub 克隆源码
- 最新开发版本
- 可以修改代码
- 需要手动 `git pull` 更新

### 切换安装方式

**从 npm 切换到 git：**
```bash
# 卸载 npm 版本
npm uninstall -g openclaw

# 重新用 git 方式安装
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

**从 git 切换到 npm：**
```bash
# 删除源码目录
rm -rf ~/openclaw

# 重新用 npm 方式安装
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method npm
```

---

## 📝 下一步

安装完成后：

1. **[新手上路](/zh-CN/start/getting-started)** - 30分钟快速入门
2. **[配置向导](/zh-CN/start/wizard)** - 详细理解引导流程
3. **[更新指南](/zh-CN/install/updating)** - 保持系统最新
4. **[迁移指南](/zh-CN/install/migrating)** - 换新机器时的数据迁移

---

## 🗑️ 卸载

需要卸载？查看 [卸载指南](/zh-CN/install/uninstall)。

---

**遇到问题？** 查看 [常见问题](/zh-CN/help/faq) 或访问 [GitHub Issues](https://github.com/openclaw/openclaw/issues)。
