---
summary: "Node.js + npm PATH 配置：版本要求、PATH 设置和全局安装问题解决"
read_when:
  - 安装了 OpenClaw 但找不到命令
  - 设置新机器的 Node.js/npm 环境
  - npm install -g 遇到权限或 PATH 问题
title: "Node.js + npm PATH 配置"
---

# 🔧 Node.js + npm PATH 配置

OpenClaw 的运行时基线是 **Node.js 22+**。

如果你成功运行了 `npm install -g openclaw@latest`，但之后看到 `openclaw: command not found`，这几乎总是 **PATH** 问题：npm 存放全局二进制文件的目录不在你的 shell PATH 中。

---

## 🔍 快速诊断

运行以下命令检查环境：

```bash
# 检查 Node 版本（需要 >= 22）
node -v

# 检查 npm 版本
npm -v

# 查看 npm 全局安装路径
npm prefix -g

# 检查当前 PATH
echo "$PATH"
```

### 判断问题

将 `npm prefix -g` 的输出与 `echo "$PATH"` 对比：

| 系统 | 需要在 PATH 中的路径 |
|------|---------------------|
| macOS / Linux | `$(npm prefix -g)/bin` |
| Windows | `$(npm prefix -g)` |

如果这个路径**不在** PATH 中，你的 shell 就找不到全局 npm 二进制文件（包括 `openclaw`）。

---

## ✅ 解决方案：添加 npm 全局目录到 PATH

### 第一步：找到全局 npm 前缀

```bash
npm prefix -g
```

常见输出：
- macOS (Homebrew): `/opt/homebrew` 或 `/usr/local`
- macOS (nvm): `~/.nvm/versions/node/v22.x.x`
- Linux: `/usr` 或 `~/.npm-global`
- Windows: `C:\Users\<用户名>\AppData\Roaming\npm`

### 第二步：添加到 shell 配置文件

**zsh（macOS 默认）：**

```bash
# 添加到 ~/.zshrc
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc

# 重新加载配置
source ~/.zshrc
```

**bash：**

```bash
# 添加到 ~/.bashrc
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc

# 重新加载配置
source ~/.bashrc
```

**Windows：**

1. 运行 `npm prefix -g` 获取路径
2. 打开"系统属性" → "环境变量"
3. 在"用户变量"中编辑 `PATH`
4. 添加 npm 全局路径
5. 重启 PowerShell / CMD

### 第三步：验证

打开**新的**终端窗口（或运行 `rehash` / `hash -r`），然后：

```bash
openclaw --version
```

---

## 🐧 Linux 特别问题：避免 sudo npm install

### 问题

在某些 Linux 系统上（尤其是通过系统包管理器或 NodeSource 安装 Node 后），npm 全局前缀指向 root 拥有的目录，导致：

```
npm ERR! Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
```

### 解决方案：切换到用户目录

**不要**使用 `sudo npm install -g`！这会造成更多权限问题。

正确做法：

```bash
# 创建用户级别的 npm 全局目录
mkdir -p "$HOME/.npm-global"

# 配置 npm 使用该目录
npm config set prefix "$HOME/.npm-global"

# 添加到 PATH
export PATH="$HOME/.npm-global/bin:$PATH"
```

将 `export PATH=...` 行添加到你的 shell 配置文件（`~/.bashrc` 或 `~/.zshrc`）以持久化。

### 验证配置

```bash
# 确认 prefix 已更改
npm config get prefix
# 应该输出: /home/<用户名>/.npm-global

# 现在可以安全安装
npm install -g openclaw@latest

# 验证
openclaw --version
```

---

## 📦 推荐的 Node.js 安装方式

选择正确的安装方式可以避免大多数 PATH 和权限问题：

### macOS

| 方式 | 优点 | 缺点 |
|------|------|------|
| **Homebrew**（推荐） | 自动配置 PATH，易于更新 | 需要先安装 Homebrew |
| **官方安装包** | 简单直接 | 更新不太方便 |
| **版本管理器** (nvm/fnm) | 支持多版本切换 | 配置稍复杂 |

```bash
# Homebrew 安装
brew install node

# 验证
node -v  # 应该 >= 22
```

### Linux

| 方式 | 优点 | 缺点 |
|------|------|------|
| **版本管理器** (nvm/fnm) | 灵活，用户级安装 | 每个 shell 需初始化 |
| **NodeSource** | 系统级，自动更新 | 可能有权限问题 |
| **Snap/Flatpak** | 隔离安装 | 可能版本较旧 |

```bash
# nvm 安装（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22

# 验证
node -v
```

### Windows

| 方式 | 优点 | 缺点 |
|------|------|------|
| **官方安装包** | 简单，自动配置 PATH | 更新需手动 |
| **winget** | 命令行管理 | 需要 Windows 10+ |
| **nvm-windows** | 多版本管理 | 配置稍复杂 |

```powershell
# winget 安装
winget install OpenJS.NodeJS.LTS

# 验证
node -v
```

---

## ⚠️ 版本管理器注意事项

如果你使用版本管理器（nvm/fnm/asdf 等），确保它在你**日常使用的 shell** 中正确初始化。

### 常见问题

**问题**：在 zsh 中安装了 nvm，但 PATH 只在 bash 中生效。

**解决**：确保在 `~/.zshrc` 中有 nvm 初始化代码：

```bash
# ~/.zshrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

**问题**：运行安装脚本时 Node 版本不对。

**解决**：在运行脚本前确认版本：

```bash
node -v  # 确认是 22+
npm prefix -g  # 确认路径正确
```

---

## 🔍 完整诊断流程

如果 `openclaw` 命令仍然找不到，按以下步骤排查：

```bash
# 1. 确认 Node 版本
node -v
# 预期: v22.x.x 或更高

# 2. 确认 npm 可用
npm -v

# 3. 查看全局前缀
npm prefix -g
# 记下这个路径

# 4. 检查 openclaw 是否在该目录
ls "$(npm prefix -g)/bin/openclaw"
# 如果存在，说明安装成功

# 5. 检查 PATH
echo "$PATH" | tr ':' '\n' | grep -E "npm|node"
# 确认步骤 3 的路径在这里

# 6. 如果不在 PATH，手动添加
export PATH="$(npm prefix -g)/bin:$PATH"

# 7. 测试
openclaw --version
```

---

## 📝 下一步

PATH 配置正确后：

1. **[新手上路](/zh-CN/start/getting-started)** - 开始使用 OpenClaw
2. **[安装指南](/zh-CN/install)** - 其他安装选项
3. **[更新指南](/zh-CN/install/updating)** - 保持系统最新
