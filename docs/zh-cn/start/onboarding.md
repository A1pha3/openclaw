---
summary: "Onboarding 流程详解 - 初始配置向导和工作区设置"
read_when:
  - 首次配置 OpenClaw
  - 理解向导流程
  - 配置 AI 认证和工作区
title: "Onboarding 流程"
---

# 🧙 Onboarding 流程

**目标**：理解 OpenClaw 的初始化配置流程，完成从安装到首次使用的完整配置。

本指南详细解释 Onboarding（引导配置）流程的设计理念、每个配置步骤的作用，以及如何根据不同场景选择最优配置方案。

---

## 🎯 学习目标

完成本指南学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 Onboarding 的设计理念和工作流程
- [ ] 完成完整的 Onboarding 配置
- [ ] 理解每个配置步骤的作用和选项含义
- [ ] 掌握工作区的创建和自定义

### 进阶目标（建议掌握）

- [ ] 使用非交互模式实现自动化配置
- [ ] 在不同场景下选择最优配置方案
- [ ] 管理工作区文件和模板
- [ ] 排查 Onboarding 过程中的问题

---

## 💡 首先理解：什么是 Onboarding？

### 设计决策背景

```
问题：如何让用户在几分钟内完成复杂的系统配置？

挑战：
  - OpenClaw 涉及多个组件：网关、认证、渠道、技能
  - 每个组件都有多种配置选项
  - 直接让用户阅读文档再手动配置，学习成本高

解决方案：设计 Onboarding 交互式向导

设计原则：
  ✅ 渐进式配置：先简单后复杂
  ✅ 合理默认值：大多数用户可以直接使用默认值
  ✅ 交互式反馈：每步都提供清晰反馈
  ✅ 可跳过/可重试：用户可以跳过不想配置的步骤
  ✅ 保留控制权：最终配置仍可手动修改
```

### Onboarding 执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    Onboarding 完整流程                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户运行命令                                                    │
│         ↓                                                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  步骤 1：网关配置                                 │        │
│  │  ├── 本地模式（默认）→ loopback:18789           │        │
│  │  └── 远程模式 → lan/tailnet:18789                │        │
│  └─────────────────────────────────────────────────┘        │
│         ↓                                                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  步骤 2：AI 认证配置                             │        │
│  │  ├── Anthropic API Key                           │        │
│  │  ├── Anthropic OAuth                             │        │
│  │  ├── OpenAI API Key                             │        │
│  │  └── 其他提供商                                  │        │
│  └─────────────────────────────────────────────────┘        │
│         ↓                                                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  步骤 3：消息渠道配置                            │        │
│  │  ├── WhatsApp（二维码登录）                      │        │
│  │  ├── Telegram（Bot Token）                      │        │
│  │  └── Discord（Bot Token）                       │        │
│  └─────────────────────────────────────────────────┘        │
│         ↓                                                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  步骤 4：技能选择                               │        │
│  │  ├── Web Search（推荐）                         │        │
│  │  ├── GitHub                                    │        │
│  │  └── 其他技能                                  │        │
│  └─────────────────────────────────────────────────┘        │
│         ↓                                                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  步骤 5：后台服务配置                           │        │
│  │  ├── 安装服务（推荐）→ 开机自启动               │        │
│  │  └── 手动管理                                   │        │
│  └─────────────────────────────────────────────────┘        │
│         ↓                                                       │
│  生成配置文件                                                   │
│         ↓                                                       │
│  提示用户启动网关或使用仪表盘                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Onboarding vs 手动配置

| 方式 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| **Onboarding** | 首次配置、快速上手 | 简单、有引导 | 定制能力有限 |
| **手动配置** | 高级用户、需要特定配置 | 完全控制 | 需要了解配置结构 |

> **💡 专家建议**：先用 Onboarding 完成基础配置，然后根据需要手动微调。这是最效率的方式。

---

## 🚀 启动 Onboarding

### 首次运行

```bash
# 完整引导（推荐）
openclaw onboard --install-daemon
```

此命令将启动完整的引导流程，包括后台服务安装。

### 命令参数详解

| 参数 | 说明 | 使用场景 |
|------|------|----------|
| `--install-daemon` | 安装后台服务 | 首次配置，推荐使用 |
| `--reset` | 重置所有配置后重新引导 | 想要完全重新配置 |
| `--non-interactive` | 非交互模式 | 自动化脚本、CI/CD |
| `--workspace <path>` | 指定工作区路径 | 自定义工作区位置 |
| `--config <path>` | 指定配置文件路径 | 使用预设配置 |
| `--skip-channels` | 跳过渠道配置 | 只想配置核心功能 |
| `--skip-daemon` | 不安装后台服务 | 手动管理网关 |

### 跳过 Onboarding

如果只想安装 CLI，稍后再配置：

```bash
# 方式一：安装时跳过引导
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard

# 方式二：先安装 CLI，后续再运行引导
npm install -g openclaw@latest

# 后续手动运行引导
openclaw onboard
```

---

## 📋 Onboarding 步骤详解

### 步骤 1：网关配置

```
问题：您希望如何运行网关？

选项：
  1. 本地模式（推荐）- 仅本地访问
  2. 远程模式 - 通过网络访问
```

#### 本地模式（默认推荐）

```
┌─────────────────────────────────────────────────────────────────┐
│                    本地模式配置                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  配置示例：                                                      │
│  ```json5                                                       │
│  {                                                              │
│    "gateway": {                                                 │
│      "bind": "loopback",  // 只监听本地接口                     │
│      "port": 18789        // 默认端口                           │
│    }                                                            │
│  }                                                              │
│  ```                                                            │
│                                                                 │
│  适用场景：                                                      │
│    ✅ 个人使用                                                    │
│    ✅ 不需要远程访问                                              │
│    ✅ 最高安全性（只能本地访问）                                  │
│                                                                 │
│  访问方式：                                                      │
│    - 浏览器访问：http://127.0.0.1:18789/                        │
│    - CLI 命令：所有命令正常工作                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 远程模式

```
┌─────────────────────────────────────────────────────────────────┐
│                    远程模式配置                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  配置示例：                                                      │
│  ```json5                                                       │
│  {                                                              │
│    "gateway": {                                                 │
│      "bind": "lan",      // 监听局域网                          │
│      "port": 18789                                             │
│    }                                                            │
│  }                                                              │
│  ```                                                            │
│                                                                 │
│  或使用 Tailscale：                                              │
│  ```json5                                                       │
│  {                                                              │
│    "gateway": {                                                 │
│      "bind": "tailnet",  // 监听 Tailscale 网络                 │
│      "port": 18789                                             │
│    }                                                            │
│  }                                                              │
│  ```                                                            │
│                                                                 │
│  适用场景：                                                      │
│    ✅ 多设备访问                                                  │
│    ✅ 远程服务器部署                                              │
│    ⚠️ 需要额外注意网络安全                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> **🔐 安全提醒**：远程模式需要配置令牌认证，参考[网关安全配置](/zh-CN/gateway/authentication)。

### 步骤 2：AI 认证配置

```
问题：如何配置 AI 模型访问？

选项：
  1. Anthropic API Key
  2. Anthropic OAuth（Claude Code 用户）
  3. OpenAI API Key
  4. OpenAI OAuth（Codex 用户）
  5. 其他提供商
```

#### API Key 方式

```
┌─────────────────────────────────────────────────────────────────┐
│                    API Key 认证方式                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  配置示例：                                                      │
│  ```bash                                                        │
│  # 在 Onboarding 中输入                                         │
│  export ANTHROPIC_API_KEY="sk-ant-api03-..."                   │
│  ```                                                            │
│                                                                 │
│  或直接设置：                                                    │
│  ```bash                                                        │
│  openclaw config set "agents.defaults.authProfiles.anthropic.apiKey" "sk-ant-..." │
│  ```                                                            │
│                                                                 │
│  凭证存储：                                                      │
│    ~/.clawdbot/agents/<agentId>/agent/auth-profiles.json       │
│                                                                 │
│  优点：                                                          │
│    ✅ 配置简单                                                    │
│    ✅ 无需浏览器                                                  │
│                                                                 │
│  缺点：                                                          │
│    ❌ 需要手动管理密钥                                            │
│    ❌ 密钥泄露 风险                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### OAuth 方式（推荐）

```
┌─────────────────────────────────────────────────────────────────┐
│                    OAuth 认证方式                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  认证流程：                                                      │
│    1. Onboarding 打开浏览器                                      │
│    2. 用户登录 Anthropic/OpenAI 账户                            │
│    3. 授权 OpenClaw 访问                                        │
│    4. 浏览器关闭，凭证自动保存                                    │
│                                                                 │
│  凭证存储：                                                      │
│    ~/.clawdbot/credentials/oauth.json                           │
│                                                                 │
│  优点：                                                          │
│    ✅ 更安全（不需要存储 API Key）                                │
│    ✅ 自动处理刷新令牌                                            │
│    ✅ 可随时撤销访问权限                                          │
│                                                                 │
│  缺点：                                                          │
│    ❌ 需要浏览器                                                  │
│    ❌ 首次配置稍复杂                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### AI 提供商对比

| 提供商 | 推荐度 | 适用场景 | 价格 |
|--------|--------|----------|------|
| **Anthropic (Claude)** | ⭐⭐⭐⭐⭐ | 通用场景、长上下文 | 中等 |
| **OpenAI (GPT)** | ⭐⭐⭐⭐ | 通用场景、插件生态 | 中等 |
| **Moonshot (Kimi)** | ⭐⭐⭐ | 中文场景 | 较低 |
| **GLM (智谱)** | ⭐⭐⭐ | 中文场景 | 较低 |
| **Ollama (本地)** | ⭐⭐⭐ | 离线场景 | 免费 |

### 步骤 3：消息渠道配置

```
问题：您想配置哪些消息渠道？

选项：
  ✓ WhatsApp（扫码登录）
  ✓ Telegram（Bot Token）
  ✓ Discord（Bot Token）
  ○ Slack
  ○ Signal
  ○ iMessage（仅 macOS）
```

#### WhatsApp 渠道

```
┌─────────────────────────────────────────────────────────────────┐
│                    WhatsApp 配置                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  认证方式：二维码扫描                                            │
│                                                                 │
│  配置步骤：                                                      │
│    1. 在 Onboarding 中选择 WhatsApp                             │
│    2. 终端显示二维码                                              │
│    3. 手机 WhatsApp → 设置 → 已关联设备                          │
│    4. 扫描二维码                                                 │
│                                                                 │
│  工作原理：                                                      │
│    OpenClaw 使用 Baileys 库连接 WhatsApp Web                    │
│    凭证保存在 ~/.clawdbot/credentials/whatsapp/                 │
│                                                                 │
│  注意事项：                                                       │
│    ⚠️ 需要手机保持网络连接                                        │
│    ⚠️ 每次重启可能需要重新扫码（会话过期）                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Telegram 渠道

```
┌─────────────────────────────────────────────────────────────────┐
│                    Telegram 配置                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  认证方式：Bot Token                                             │
│                                                                 │
│  创建机器人：                                                     │
│    1. 搜索 @BotFather                                           │
│    2. 发送 /newbot                                              │
│    3. 输入名称和用户名                                           │
│    4. 保存 Bot Token                                            │
│                                                                 │
│  Token 格式：                                                    │
│    123456789:ABCdefGHIjklMNOpqrsTUVwxyz                         │
│                                                                 │
│  配置命令：                                                      │
│    openclaw config set channels.telegram.botToken "YOUR_TOKEN"  │
│                                                                 │
│  优点：                                                          │
│    ✅ 不依赖手机                                                  │
│    ✅ 24/7 在线                                                  │
│    ✅ Bot API 功能丰富                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Discord 渠道

```
┌─────────────────────────────────────────────────────────────────┐
│                    Discord 配置                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  认证方式：Bot Token                                             │
│                                                                 │
│  创建应用：                                                      │
│    1. 访问 Discord Developer Portal                              │
│    2. 创建新 Application                                        │
│    3. 添加 Bot 用户                                              │
│    4. 获取 Bot Token                                            │
│    5. 配置权限并邀请到服务器                                      │
│                                                                 │
│  必要权限：                                                      │
│    - Send Messages                                              │
│    - Read Message History                                       │
│    - Manage Messages（可选）                                     │
│    - Embed Links                                                │
│                                                                 │
│  配置命令：                                                      │
│    openclaw config set channels.discord.token "YOUR_TOKEN"      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 步骤 4：技能选择

```
问题：您想启用哪些技能？

选项：
  ✓ Web Search（网页搜索）- 推荐
  ✓ GitHub（代码仓库访问）
  ○ 文件读写
  ○ 命令执行
  ○ 浏览器控制
```

#### 推荐技能说明

| 技能 | 推荐度 | 说明 |
|------|--------|------|
| **Web Search** | ⭐⭐⭐⭐⭐ | 实时信息获取，最常用 |
| **GitHub** | ⭐⭐⭐⭐ | 代码仓库访问和搜索 |
| **Browser** | ⭐⭐⭐⭐ | 网页自动化和截图 |
| **File System** | ⭐⭐⭐ | 本地文件读写 |
| **Exec** | ⭐⭐⭐ | 命令执行（需注意安全） |

### 步骤 5：后台服务配置

```
问题：是否安装后台服务？

选项：
  1. 是（推荐）- 网关随系统启动
  2. 否 - 手动启动网关
```

#### 后台服务对比

| 平台 | 服务类型 | 安装命令 |
|------|----------|----------|
| macOS | LaunchAgent | `openclaw onboard --install-daemon` |
| Linux | systemd 用户单元 | `openclaw onboard --install-daemon` |
| Windows | 服务（WSL2） | `openclaw onboard --install-daemon` |

---

## 📁 工作区配置

### 工作区目录结构

Onboarding 会创建默认工作区：

```
~/.clawdbot/workspace/
├── AGENTS.md         # 代理行为指南
├── SOUL.md           # 代理个性定义
├── USER.md           # 用户信息
├── MEMORY.md        # 长期记忆
├── TOOLS.md          # 工具说明
└── skills/           # 技能目录
    └── <skill-name>/
        └── SKILL.md
```

### 工作区文件说明

| 文件 | 用途 | 说明 |
|------|------|------|
| `AGENTS.md` | 代理行为 | 定义 AI 的工作方式、系统提示和约束 |
| `SOUL.md` | 代理个性 | 定义 AI 的性格、沟通风格和价值观 |
| `USER.md` | 用户信息 | 告诉 AI 关于您的信息（姓名、工作、偏好） |
| `MEMORY.md` | 长期记忆 | AI 应该长期记住的重要信息 |
| `TOOLS.md` | 工具说明 | 自定义工具使用指南和示例 |

### 自定义工作区

```bash
# 在 Onboarding 中指定
openclaw onboard --workspace ~/my-openclaw-workspace

# 或手动配置
openclaw config set agents.defaults.workspace ~/my-workspace

# 工作区结构
~/my-openclaw-workspace/
├── AGENTS.md         # 代理行为
├── SOUL.md          # 个性定义
├── USER.md          # 用户信息
├── MEMORY.md        # 长期记忆
├── projects/        # 项目目录
│   ├── project-a/
│   └── project-b/
└── skills/          # 自定义技能
```

> **💡 专家提示**：可以将工作区放在云同步目录（如 Dropbox、iCloud），实现多设备共享配置。

---

## 🔄 重新运行 Onboarding

### 何时需要重新运行？

| 场景 | 是否需要重新运行 Onboarding |
|------|----------------------------|
| 更改网关配置 | ✅ 需要 |
| 更新 AI 认证 | ✅ 需要 |
| 添加/删除渠道 | ✅ 需要 |
| 调整技能配置 | ❌ 可手动 |
| 更改工作区 | ✅ 需要 |
| 微调配置项 | ❌ 使用 CLI |

### 重新运行命令

```bash
# 完整重新引导（重置所有配置）
openclaw onboard --reset

# 保留现有配置，仅运行向导
openclaw onboard
```

### 备份配置

建议在重新运行前备份：

```bash
# 备份主配置
cp ~/.clawdbot/openclaw.json ~/.clawdbot/openclaw.json.backup

# 备份工作区
cp -r ~/.clawdbot/workspace ~/workspace.backup

# 备份凭证（谨慎！）
cp -r ~/.clawdbot/credentials ~/credentials.backup
```

---

## 🖥️ 非交互模式

适用于自动化脚本、CI/CD 或批量部署场景。

### 使用配置文件

```bash
openclaw onboard --non-interactive --config ~/openclaw-config.json
```

### 配置文件示例

```json5
{
  "gateway": {
    "bind": "loopback",
    "port": 18789
  },
  "auth": {
    "provider": "anthropic",
    "apiKey": "${ANTHROPIC_API_KEY}"
  },
  "channels": {
    "whatsapp": {
      "enabled": true
    },
    "telegram": {
      "enabled": true,
      "botToken": "${TELEGRAM_BOT_TOKEN}"
    },
    "discord": {
      "enabled": false
    }
  },
  "skills": {
    "webSearch": {
      "enabled": true
    },
    "github": {
      "enabled": true
    }
  },
  "daemon": {
    "install": true
  }
}
```

### 环境变量注入

```bash
# 使用环境变量
export ANTHROPIC_API_KEY="sk-ant-..."
export TELEGRAM_BOT_TOKEN="123456:..."

# 运行非交互 Onboarding
openclaw onboard --non-interactive --config ~/openclaw-config.json
```

---

## 📊 Onboarding 状态检查

### 配置状态命令

```bash
# 查看整体配置状态
openclaw status

# 检查认证状态
openclaw health

# 查看渠道状态
openclaw channels status

# 查看服务状态
openclaw service status
```

### 常见状态解读

| 状态 | 说明 | 解决方案 |
|------|------|----------|
| ✅ Configured | 已配置完成 | 可以开始使用 |
| ⚠️ No auth | 未配置 AI 认证 | 运行 `openclaw onboard` |
| ❌ Gateway not running | 网关未运行 | `openclaw gateway` 或 `openclaw service start` |
| ⚠️ Channel disconnected | 渠道已断开 | 检查网络或重新登录 |
| ❌ Auth expired | 认证已过期 | 重新认证 |

---

## 🐛 故障排除

### Onboarding 失败

```bash
# 查看详细错误信息
openclaw doctor

# 检查日志
openclaw logs --lines 100
```

### 常见问题与解决方案

#### 问题一：API Key 无效

**症状**：认证失败，提示 API Key 错误。

**解决方案**：

```bash
# 1. 检查 API Key 是否正确
echo $ANTHROPIC_API_KEY

# 2. 重新运行 Onboarding
openclaw onboard --reset

# 3. 或手动重新设置
openclaw config unset "agents.defaults.authProfiles.anthropic.apiKey"
openclaw config set "agents.defaults.authProfiles.anthropic.apiKey" "sk-ant-new-key..."
```

#### 问题二：WhatsApp 扫码无响应

**症状**：扫码后终端无响应或显示超时。

**解决方案**：

```bash
# 1. 检查网络连接
curl -I https://web.whatsapp.com

# 2. 退出当前会话
openclaw channels logout whatsapp

# 3. 重新登录
openclaw channels login whatsapp

# 4. 确保手机和电脑在同一网络
```

#### 问题三：OAuth 授权失败

**症状**：浏览器打开后授权失败或无响应。

**解决方案**：

```bash
# 1. 删除旧的凭证文件
rm ~/.clawdbot/credentials/oauth.json

# 2. 清除浏览器缓存或使用无痕模式

# 3. 重新运行 Onboarding
openclaw onboard

# 4. 确保没有防火墙阻止 localhost 回调
```

#### 问题四：后台服务无法启动

**症状**：服务状态显示 failed。

**解决方案**：

```bash
# 1. 查看详细错误
journalctl --user -u openclaw-gateway -e  # Linux
# 或
openclaw logs

# 2. 手动运行网关排查
openclaw gateway --verbose

# 3. 检查端口是否被占用
lsof -i :18789
```

---

## 📝 专家最佳实践

### 推荐配置流程

```
专家推荐的配置路径：

1. 首次使用
   └── Onboarding（--install-daemon）→ 基础配置完成

2. 日常使用
   └── 通过 CLI 微调配置 → 满足特定需求

3. 重大变更
   └── 备份 → Onboarding --reset → 重新配置

4. 问题排查
   └── openclaw doctor → 诊断问题
```

### 工作区管理最佳实践

```
┌─────────────────────────────────────────────────────────────────┐
│                    推荐工作区结构                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ~/openclaw-workspace/                                         │
│  ├── AGENTS.md         # 保持简洁，只定义核心行为               │
│  ├── SOUL.md          # 根据个人喜好定制                        │
│  ├── USER.md          # 定期更新个人信息                         │
│  ├── MEMORY.md        # 重要信息手动管理                        │
│  ├── projects/        # 按项目组织                              │
│  │   ├── work/        # 工作相关                                │
│  │   └── personal/   # 个人项目                                │
│  └── skills/          # 自定义技能                              │
│      ├── my-skills/   # 自己编写的技能                          │
│      └── notes/       # 技能使用笔记                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 配置版本控制

建议将配置文件纳入版本控制（敏感信息除外）：

```bash
# 初始化 git 仓库
cd ~/.clawdbot/workspace
git init
git add .
git commit -m "Initial OpenClaw workspace"

# .gitignore 示例
# credentials/
# sessions/
# *secret*
# *.log
```

---

## 📊 专家思维模型：配置决策框架

在 Onboarding 过程中，使用以下决策框架：

```
┌─────────────────────────────────────────────────────────────────┐
│                    Onboarding 决策流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  问题 1：谁来使用？                                               │
│           ↓                                                     │
│     个人使用 ←─┬─→ 团队/多人使用                                  │
│           ↓                                                     │
│     本地模式                                                    │
│                                                                 │
│  问题 2：需要远程访问吗？                                         │
│           ↓                                                     │
│     是 ←─┬─→ 否                                                  │
│           ↓                                                     │
│     远程模式 + 令牌认证                                          │
│                                                                 │
│  问题 3：使用哪个 AI 提供商？                                     │
│           ↓                                                     │
│     Anthropic（推荐）                                            │
│       ↓                                                         │
│     有 Claude Code？ → OAuth（更安全）                           │
│       ↓                                                         │
│     没有？ → API Key                                            │
│                                                                 │
│  问题 4：需要哪些渠道？                                           │
│           ↓                                                     │
│     WhatsApp（最常用）                                           │
│     Telegram（替代方案）                                          │
│     Discord（社区用）                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 知识点回顾

完成本指南后，您应该掌握以下技能：

| 技能 | 掌握程度 |
|------|----------|
| Onboarding 流程理解 | ⭐⭐⭐⭐⭐ |
| 网关配置选择 | ⭐⭐⭐⭐ |
| AI 认证配置 | ⭐⭐⭐⭐ |
| 渠道配置 | ⭐⭐⭐⭐ |
| 工作区管理 | ⭐⭐⭐ |
| 后台服务配置 | ⭐⭐⭐⭐ |

---

## 🎯 下一步学习

| 主题 | 文档链接 | 预计时间 |
|------|----------|----------|
| **网关详解** | [Gateway 概念](/zh-CN/concepts/gateway) | 20 分钟 |
| **AI 提供商** | [Providers](/zh-CN/providers/index) | 30 分钟 |
| **渠道配置** | [Channels](/zh-CN/channels/index) | 45 分钟 |
| **技能系统** | [Skills](/zh-CN/tools/skills) | 30 分钟 |

---

## 📖 附录

### 附录 A：命令速查

| 命令 | 说明 |
|------|------|
| `openclaw onboard` | 运行引导配置 |
| `openclaw onboard --reset` | 重置后重新引导 |
| `openclaw onboard --install-daemon` | 安装后台服务 |
| `openclaw onboard --non-interactive` | 非交互模式 |
| `openclaw status` | 查看配置状态 |
| `openclaw health` | 健康检查 |
| `openclaw doctor` | 诊断问题 |

### 附录 B：术语表

| 英文术语 | 中文术语 | 说明 |
|---------|---------|------|
| Onboarding | 引导配置 | 初始化配置流程 |
| Gateway | 网关 | 核心控制平面 |
| Workspace | 工作区 | 配置和运行时目录 |
| OAuth | OAuth | 开放授权协议 |
| API Key | API 密钥 | 程序化访问凭证 |
| Daemon | 后台服务 | 长期运行的系统服务 |

### 附录 C：相关资源

- **官方文档**：[docs.openclaw.ai](https://docs.openclaw.ai)
- **GitHub**：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- **社区支持**：Discord #support 频道

---

> **💡 提示**：Onboarding 让复杂的配置变得简单。如果遇到问题，运行 `openclaw doctor` 获取诊断信息。
