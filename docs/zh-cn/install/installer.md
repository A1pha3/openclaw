---
summary: "安装脚本内部原理：install.sh 和 install.ps1 的工作方式、参数选项和自动化配置"
read_when:
  - 想了解安装脚本的工作原理
  - 想要自动化安装（CI/无人值守环境）
  - 想要从 GitHub 源码安装
title: "安装器内部原理"
---

# 🔧 安装器内部原理

OpenClaw 提供多个安装脚本，了解它们的工作原理可以帮助你更好地自定义安装过程。

## 📋 安装脚本一览

| 脚本 | 平台 | 用途 |
|------|------|------|
| `install.sh` | macOS / Linux | 推荐安装脚本（默认 npm，可选 git） |
| `install-cli.sh` | macOS / Linux | 非 root 友好的安装器（自带独立 Node） |
| `install.ps1` | Windows | PowerShell 安装脚本 |

---

## 🐚 install.sh（推荐）

### 查看帮助

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
```

### 工作流程

安装脚本会自动完成以下步骤：

```
1. 检测操作系统（macOS / Linux / WSL）
       ↓
2. 确保 Node.js >= 22
   - macOS：通过 Homebrew 安装
   - Linux：通过 NodeSource 安装
       ↓
3. 选择安装方式
   - npm（默认）：npm install -g openclaw@latest
   - git：克隆源码并构建
       ↓
4. Linux 特殊处理：避免全局 npm 权限问题
   - 自动切换 npm prefix 到 ~/.npm-global
       ↓
5. 升级场景：运行 openclaw doctor --non-interactive
       ↓
6. 处理 sharp 依赖（图像处理库）
   - 默认设置 SHARP_IGNORE_GLOBAL_LIBVIPS=1
```

### 为什么需要这些步骤？

#### 1. Node.js 版本检测

OpenClaw 需要 Node.js 22+ 才能正常运行。脚本会检查现有版本，如果不满足要求会尝试自动安装。

#### 2. sharp 依赖处理

`sharp` 是一个高性能图像处理库，但它需要本地编译。常见问题：

- **问题**：系统已有 libvips（如通过 Homebrew 安装），导致编译冲突
- **解决**：脚本默认设置 `SHARP_IGNORE_GLOBAL_LIBVIPS=1`，强制使用预编译二进制

如果你**需要**链接系统 libvips（调试场景）：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.ai/install.sh | bash
```

#### 3. Linux npm 权限问题

在某些 Linux 系统上，npm 全局安装目录归 root 所有，导致 `EACCES` 权限错误。

**脚本的解决方案**：
- 自动创建 `~/.npm-global` 目录
- 配置 npm 使用该目录
- 自动添加到 `~/.bashrc` 或 `~/.zshrc` 的 PATH

### 命令行选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `--install-method npm\|git` | 选择安装方式 | `--install-method git` |
| `--git-dir <path>` | 源码目录（git 方式） | `--git-dir ~/my-openclaw` |
| `--no-git-update` | 跳过 git pull（使用现有代码） | |
| `--no-prompt` | 禁用交互提示（CI 必需） | |
| `--dry-run` | 只显示会做什么，不执行 | |
| `--no-onboard` | 跳过引导向导 | |

### 环境变量

环境变量方式更适合自动化脚本：

```bash
# 安装方式
export OPENCLAW_INSTALL_METHOD=git    # 或 npm

# Git 安装目录
export OPENCLAW_GIT_DIR=~/openclaw

# 是否更新现有 Git 仓库
export OPENCLAW_GIT_UPDATE=0          # 0=不更新，1=更新

# 禁用交互提示
export OPENCLAW_NO_PROMPT=1

# 干运行模式
export OPENCLAW_DRY_RUN=1

# 跳过引导
export OPENCLAW_NO_ONBOARD=1

# sharp 编译选项
export SHARP_IGNORE_GLOBAL_LIBVIPS=1  # 默认值

curl -fsSL https://openclaw.ai/install.sh | bash
```

### 智能检测：已有源码目录

如果你在 OpenClaw 源码目录内运行安装脚本（检测到 `package.json` + `pnpm-workspace.yaml`），脚本会提示：

- 使用当前目录更新并安装（git 方式）
- 或迁移到全局 npm 安装

在非交互环境（无 TTY / `--no-prompt`）中，必须明确指定 `--install-method`，否则脚本会退出。

---

## 🔌 install-cli.sh（非 root 安装器）

适合无法修改系统 Node/npm 的环境。

### 工作原理

这个脚本会：
1. 在指定目录（默认 `~/.openclaw`）安装独立的 Node 运行时
2. 在该目录安装 OpenClaw CLI
3. 不影响系统的 Node 或 npm

### 查看帮助

```bash
curl -fsSL https://openclaw.ai/install-cli.sh | bash -s -- --help
```

### 使用场景

- 共享服务器，没有 root 权限
- 不想修改系统 Node 版本
- 需要完全隔离的安装环境

---

## 🪟 install.ps1（Windows PowerShell）

### 查看帮助

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

### 工作流程

1. 确保 Node.js >= 22（通过 winget/Chocolatey/Scoop 或手动安装）
2. 选择安装方式（npm 或 git）
3. 升级时运行 `openclaw doctor --non-interactive`

### 使用示例

```powershell
# 默认安装（npm）
iwr -useb https://openclaw.ai/install.ps1 | iex

# Git 方式安装
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git

# 指定 Git 目录
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\openclaw"
```

### 环境变量

```powershell
$env:OPENCLAW_INSTALL_METHOD = "git"
$env:OPENCLAW_GIT_DIR = "C:\openclaw"
```

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| `spawn git ENOENT` | 安装 [Git for Windows](https://git-scm.com/download/win)，重启 PowerShell |
| `openclaw 不是命令` | 将 `npm prefix -g` 输出路径添加到系统 PATH |

---

## 🔄 为什么需要 Git？

### git 安装方式

必需。用于克隆和更新源码。

### npm 安装方式

通常不需要，但某些情况下依赖可能通过 git URL 安装。安装脚本会确保 Git 存在，避免 `spawn git ENOENT` 错误。

---

## 🛠️ 故障排除

### npm EACCES 权限错误

**症状**：
```
npm ERR! Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
```

**原因**：npm 全局目录归 root 所有

**解决方案**：
```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### sharp 安装失败

**症状**：
```
npm ERR! sharp: Please add node-gyp to your dependencies
```

**解决方案 1**：使用预编译二进制
```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

**解决方案 2**：安装构建工具
```bash
# macOS
xcode-select --install
npm install -g node-gyp

# 然后重试安装
npm install -g openclaw@latest
```

### 安装后命令找不到

详见 [Node.js + npm PATH 配置](/zh-CN/install/node)

---

## 📝 下一步

- [安装指南](/zh-CN/install) - 选择适合你的安装方式
- [Node.js PATH 配置](/zh-CN/install/node) - 解决命令找不到问题
- [更新指南](/zh-CN/install/updating) - 保持系统最新
