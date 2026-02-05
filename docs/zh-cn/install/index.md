---
summary: "OpenClaw 安装指南：推荐安装脚本、全局安装或源码安装的完整安装指南"
read_when:
  - 安装 OpenClaw
  - 想要从 GitHub 安装
  - 选择安装方式
title: "安装指南"
---

# 📦 安装指南

本文档介绍多种安装 OpenClaw 的方法，从最简单到最灵活。无论你是新手还是开发者，都能找到适合自己的安装方式。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的安装方式及其适用场景
- [ ] 能够使用安装脚本完成快速安装
- [ ] 掌握 Node.js 环境要求和检查方法
- [ ] 理解 npm 全局安装的流程

### 进阶目标（建议掌握）

- [ ] 能够从源码安装 OpenClaw
- [ ] 理解安装选项的含义并根据需求选择
- [ ] 掌握 PATH 配置和问题解决
- [ ] 能够配置安装后的验证

### 专家目标（挑战）

- [ ] 能够为团队设计标准化安装流程
- [ ] 理解安装脚本的工作原理
- [ ] 能够定制安装脚本
- [ ] 掌握多环境部署的安装策略

---

## 🚀 第一部分：快速选择指南

### 1.1 安装方式对比

| 你的情况 | 推荐方式 | 命令 | 复杂度 |
|---------|---------|------|--------|
| **只想快速开始** | 安装脚本（推荐） | `curl -fsSL https://openclaw.ai/install.sh \| bash` | ⭐ |
| **已有 Node 环境** | npm 全局安装 | `npm install -g openclaw@latest` | ⭐⭐ |
| **开发者/贡献者** | 源码安装 | `git clone ...` | ⭐⭐⭐ |
| **服务器部署** | Docker | 见 [Docker 安装](/zh-CN/install/docker) | ⭐⭐⭐ |
| **Nix 用户** | Nix 包管理器 | 见 [Nix 安装](/zh-CN/install/nix) | ⭐⭐ |

**选择决策树**：

```
需要快速开始？
    │
    ├── 是 → 想要自动化配置？
    │           ├── 是 → 安装脚本
    │           └── 否 → npm 全局安装
    │
    └── 否 → 需要修改源码？
                ├── 是 → 源码安装
                └── 否 → 需要环境隔离？
                            ├── 是 → Docker
                            └── 否 → npm 全局安装
```

---

## 🔧 第二部分：推荐方式——安装脚本

### 2.1 为什么推荐安装脚本？

**安装脚本的自动化能力**：

| 自动化项 | 说明 | 手动操作对比 |
|----------|------|--------------|
| ✅ CLI 工具安装 | 自动下载并安装 | 需要手动执行 npm install |
| ✅ 引导向导运行 | 自动启动交互式配置 | 需要手动运行 onboard |
| ✅ 环境配置 | 自动配置 PATH 等 | 需要手动编辑配置文件 |
| ✅ 服务安装 | 可选安装后台服务 | 需要手动配置 launchd/systemd |

**安装脚本的核心价值**：

1. **降低门槛**：无需了解 Node.js 或命令行细节
2. **减少错误**：自动化流程避免手动配置错误
3. **一致性**：确保所有用户的安装过程一致
4. **可恢复**：支持重新运行进行修复

### 2.2 安装命令

**macOS / Linux**：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell)**：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 2.3 安装后步骤

如果跳过了引导向导，手动运行：

```bash
openclaw onboard --install-daemon
```

**引导向导的作用**：

| 步骤 | 内容 | 可跳过 |
|------|------|--------|
| 1 | 欢迎和简介 | ❌ |
| 2 | 模型认证配置 | ✅ |
| 3 | 工作区设置 | ✅ |
| 4 | 可选功能选择 | ✅ |

### 2.4 安装脚本选项

**查看所有选项**：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
```

**常用选项**：

```bash
# 跳过引导（仅安装 CLI）
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
# 配置安装参数
export OPENCLAW_INSTALL_METHOD=npm        # npm 或 git
export OPENCLAW_GIT_DIR=~/openclaw        # 源码目录
export OPENCLAW_NO_PROMPT=1               # 禁用提示
export OPENCLAW_DRY_RUN=1                 # 干运行
export OPENCLAW_NO_ONBOARD=1              # 跳过引导

# 执行安装
curl -fsSL https://openclaw.ai/install.sh | bash
```

---

## 📋 第三部分：系统要求

### 3.1 必需组件

| 组件 | 最低版本 | 说明 |
|------|---------|------|
| **Node.js** | >= 22 | 运行时环境 |
| **操作系统** | - | macOS、Linux、Windows WSL2 |
| **包管理器** | - | npm（内置）或 pnpm（源码构建） |

### 3.2 检查系统环境

```bash
# 检查 Node 版本
node -v

# 检查 npm
npm -v

# 推荐：安装 pnpm
npm install -g pnpm
```

**版本要求说明**：

| 版本 | 状态 | 建议 |
|------|------|------|
| < 20 | ❌ 不支持 | 升级到 Node.js 22+ |
| 20.x | ⚠️ 最低支持 | 建议升级 |
| 22.x | ✅ 推荐 | 当前最佳选择 |
| 23.x | ✅ 支持 | 新版本特性 |

### 3.3 平台特别说明

**macOS**：

| 场景 | 要求 |
|------|------|
| 仅 CLI + 网关 | Node.js 即可 |
| 构建 App | 需要 Xcode / Command Line Tools |

**Windows**：

| 场景 | 要求 |
|------|------|
| 原生 Windows | ⚠️ 未经测试，兼容性差 |
| WSL2 | ✅ 推荐（Ubuntu） |

**专家建议**：强烈推荐使用 WSL2（Windows Subsystem for Linux 2）来运行 OpenClaw，而不是原生 Windows。WSL2 提供了与 Linux 几乎一致的运行环境。

---

## 🛠️ 第四部分：方式一——npm 全局安装

### 4.1 适用场景

npm 全局安装适合以下用户：

| 用户类型 | 推荐程度 | 理由 |
|----------|----------|------|
| 已有 Node.js 环境 | ⭐⭐⭐⭐⭐ | 最简单 |
| 偶尔使用 | ⭐⭐⭐⭐ | 无需复杂配置 |
| 开发者 | ⭐⭐⭐ | 便于更新 |

### 4.2 安装命令

```bash
npm install -g openclaw@latest
```

或使用 pnpm：

```bash
pnpm add -g openclaw@latest
```

### 4.3 常见问题：sharp 安装失败

**问题表现**：

```
npm ERR! sharp installation failed
```

**解决方案**：

```bash
# 强制使用预编译二进制文件（跳过本地编译）
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

**macOS 额外方案**：

```bash
# 安装 Xcode Command Line Tools
xcode-select --install

# 安装 node-gyp
npm install -g node-gyp
```

**原理说明**：sharp 是一个图像处理库，在某些环境下需要从源码编译。`SHARP_IGNORE_GLOBAL_LIBVIPS=1` 环境变量告诉 sharp 使用预编译的二进制文件，避免编译过程。

### 4.4 安装后配置

```bash
# 运行引导向导
openclaw onboard --install-daemon
```

---

## 💻 第五部分：方式二——源码安装

### 5.1 适用场景

源码安装适合以下用户：

| 用户类型 | 推荐程度 | 理由 |
|----------|----------|------|
| 开发者 | ⭐⭐⭐⭐⭐ | 可以修改代码 |
| 贡献者 | ⭐⭐⭐⭐⭐ | 需要提交 PR |
| 定制需求 | ⭐⭐⭐⭐ | 需要深度定制 |

### 5.2 克隆仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 5.3 安装依赖

```bash
pnpm install
```

**为什么使用 pnpm**：

| 特性 | pnpm | npm | yarn |
|------|------|-----|------|
| 安装速度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 磁盘占用 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| 依赖结构 | 扁平但隔离 | 完全扁平 | 完全扁平 |

### 5.4 构建 UI

```bash
pnpm ui:build
```

**说明**：首次运行会自动安装 UI 依赖。

### 5.5 构建项目

```bash
pnpm build
```

**说明**：此命令同时会打包 A2UI 资源。

### 5.6 仅打包 A2UI（如果需要）

```bash
pnpm canvas:a2ui:bundle
```

### 5.7 运行引导

```bash
openclaw onboard --install-daemon
```

**说明**：没有全局安装时，使用 `pnpm openclaw ...` 运行命令。

### 5.8 从仓库运行网关

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

---

## 🐳 第六部分：其他安装选项

### 6.1 Docker 安装

**适用场景**：

| 场景 | 推荐程度 | 理由 |
|------|----------|------|
| 服务器部署 | ⭐⭐⭐⭐⭐ | 环境隔离 |
| 验证流程 | ⭐⭐⭐⭐⭐ | 可复现 |
| 隔离测试 | ⭐⭐⭐⭐ | 可丢弃 |

**详细指南**：[Docker 安装](/zh-CN/install/docker)

### 6.2 Nix 安装

**适用场景**：

| 场景 | 推荐程度 | 理由 |
|------|----------|------|
| NixOS 用户 | ⭐⭐⭐⭐⭐ | 原生支持 |
| 可复现构建 | ⭐⭐⭐⭐⭐ | 声明式配置 |
| 依赖隔离 | ⭐⭐⭐⭐ | 干净环境 |

**详细指南**：[Nix 安装](/zh-CN/install/nix)

### 6.3 Ansible 安装

**适用场景**：

| 场景 | 推荐程度 | 理由 |
|------|----------|------|
| 批量部署 | ⭐⭐⭐⭐⭐ | 自动化 |
| 多服务器 | ⭐⭐⭐⭐⭐ | 一键部署 |
| 团队使用 | ⭐⭐⭐⭐ | 标准化 |

**详细指南**：[Ansible 安装](/zh-CN/install/ansible)

### 6.4 Bun 安装（仅 CLI）

**⚠️ 警告**：Bun 不推荐用于网关运行。

| 场景 | 推荐程度 | 理由 |
|------|----------|------|
| CLI 命令 | ⭐⭐⭐⭐ | 速度快 |
| 网关运行 | ⭐ | 兼容性问题 |
| WhatsApp/Telegram | ❌ | 存在运行时错误 |

**详细指南**：[Bun 安装](/zh-CN/install/bun)

---

## 🔧 第七部分：安装方式对比

### 7.1 功能对比表

| 特性 | 安装脚本 | npm 全局 | 源码 | Docker | Nix |
|------|---------|---------|------|--------|-----|
| **复杂度** | ⭐ | ⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| **自动配置** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **可修改源码** | ❌ | ❌ | ✅ | ❌ | ❌ |
| **环境隔离** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **版本锁定** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **适合生产** | ✅ | ✅ | ⚠️ | ✅ | ✅ |
| **适合开发** | ✅ | ✅ | ✅⭐⭐⭐ | ❌ | ⚠️ |

### 7.2 选择指南

| 你的需求 | 推荐选择 |
|----------|----------|
| 快速体验 | 安装脚本 |
| 日常使用 | npm 全局安装 |
| 开发贡献 | 源码安装 |
| 服务器部署 | Docker |
| Nix 系统 | Nix |
| 批量部署 | Ansible |

---

## 🩺 第八部分：安装后检查清单

### 8.1 验证步骤

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

### 8.2 预期输出

| 命令 | 预期结果 | 正常表现 |
|------|----------|----------|
| `doctor` | 健康报告 | 无严重警告 |
| `status` | 网关状态 | Running |
| `health` | 健康检查 | PASS |
| `dashboard` | 打开浏览器 | 正常显示 |

---

## 🐛 第九部分：故障排除——命令找不到

### 9.1 诊断步骤

```bash
# 检查 Node 和 npm
node -v
npm -v

# 查看 npm 全局安装路径
npm prefix -g

# 检查 PATH
echo "$PATH"
```

### 9.2 问题：路径不在 PATH 中

**诊断方法**：将 `npm prefix -g` 的输出与 `echo "$PATH"` 对比。

**解决方案**：

**zsh（macOS 默认）**：

```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**bash**：

```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Windows**：

1. 运行 `npm prefix -g` 获取路径
2. 打开「系统属性」→「环境变量」
3. 在「用户变量」中编辑 `PATH`
4. 添加 npm 全局路径
5. 重启 PowerShell / CMD

### 9.3 常见输出示例

| 系统 | npm prefix -g 输出 | 需要的 PATH 路径 |
|------|-------------------|------------------|
| macOS (Homebrew) | `/opt/homebrew` | `/opt/homebrew/bin` |
| macOS (nvm) | `~/.nvm/versions/node/v22.x.x` | `~/.nvm/versions/node/v22.x.x/bin` |
| Linux | `/usr` | `/usr/bin` |
| Linux (用户级) | `~/.npm-global` | `~/.npm-global/bin` |

---

## 🔄 第十部分：安装方法——npm vs git

### 10.1 npm 方法（默认）

```bash
npm install -g openclaw@latest
```

**特点**：

| 特性 | 说明 |
|------|------|
| 安装来源 | npm 注册表 |
| 版本 | 已发布的稳定版本 |
| 更新 | `npm update` |
| 适合 | 大多数用户 |

### 10.2 git 方法

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

**特点**：

| 特性 | 说明 |
|------|------|
| 安装来源 | GitHub 源码 |
| 版本 | 最新开发版本 |
| 更新 | `git pull` |
| 适合 | 开发者、贡献者 |

### 10.3 切换安装方式

**从 npm 切换到 git**：

```bash
# 卸载 npm 版本
npm uninstall -g openclaw

# 重新用 git 方式安装
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

**从 git 切换到 npm**：

```bash
# 删除源码目录
rm -rf ~/openclaw

# 重新用 npm 方式安装
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method npm
```

---

## 📝 第十一部分：下一步

### 11.1 安装完成后

| 步骤 | 内容 | 链接 |
|------|------|------|
| 1 | 新手上路 - 30分钟快速入门 | [新手上路](/zh-CN/start/getting-started) |
| 2 | 配置向导 - 详细理解引导流程 | [配置向导](/zh-CN/start/wizard) |
| 3 | 更新指南 - 保持系统最新 | [更新指南](/zh-CN/install/updating) |
| 4 | 迁移指南 - 换新机器时的数据迁移 | [迁移指南](/zh-CN/install/migrating) |

### 11.2 验证清单

- [ ] `openclaw --version` 正常显示版本
- [ ] `openclaw status` 显示网关运行中
- [ ] `openclaw dashboard` 能打开 Web 界面
- [ ] 能够发送测试消息并收到回复

---

## 🗑️ 第十二部分：卸载

需要卸载？查看 [卸载指南](/zh-CN/install/uninstall)。

**卸载步骤**：

```bash
# 停止网关
openclaw gateway stop

# 卸载 CLI
npm uninstall -g openclaw

# 删除数据（谨慎！）
rm -rf ~/.openclaw
```

---

## 🎓 章节总结

### 学习目标完成检查

#### 基础目标（必掌握）

- [ ] 理解 OpenClaw 的安装方式及其适用场景
- [ ] 能够使用安装脚本完成快速安装
- [ ] 掌握 Node.js 环境要求和检查方法
- [ ] 理解 npm 全局安装的流程

#### 进阶目标（建议掌握）

- [ ] 能够从源码安装 OpenClaw
- [ ] 理解安装选项的含义并根据需求选择
- [ ] 掌握 PATH 配置和问题解决
- [ ] 能够配置安装后的验证

#### 专家目标（挑战）

- [ ] 能够为团队设计标准化安装流程
- [ ] 理解安装脚本的工作原理
- [ ] 能够定制安装脚本
- [ ] 掌握多环境部署的安装策略

### 安装方式选择决策树

```
安装 OpenClaw
  │
  ├── 快速体验？
  │       ├── 是 → 选择安装脚本
  │       └── 否 → 进入下一步
  │
  ├── 是开发者/贡献者？
  │       ├── 是 → 选择源码安装
  │       └── 否 → 进入下一步
  │
  ├── 需要环境隔离？
  │       ├── 是 → 选择 Docker
  │       └── 否 → 选择 npm 全局安装
  │
  └── NixOS 用户？
          ├── 是 → 选择 Nix
          └── 否 → 默认选择
```

### 专家建议

| 场景 | 建议 |
|------|------|
| 首次安装 | 使用安装脚本，省心省力 |
| 日常使用 | npm 全局安装，便于更新 |
| 开发使用 | 源码安装，可修改代码 |
| 服务器部署 | Docker，隔离性好 |
| 团队部署 | Ansible，自动化程度高 |

---

**遇到问题？** 查看 [常见问题](/zh-CN/help/faq) 或访问 [GitHub Issues](https://github.com/openclaw/openclaw/issues)。
