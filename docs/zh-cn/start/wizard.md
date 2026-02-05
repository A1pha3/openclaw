---
summary: "CLI 引导向导：网关、工作区、渠道和技能的向导式配置"
read_when:
  - 运行或配置引导向导
  - 设置新机器
  - 理解向导流程和选项
title: "向导模式详解"
---

# 🧙 向导模式详解

**目标**：深入理解 OpenClaw 引导向导的设计原理，掌握所有配置选项的含义和使用场景。

本指南专为希望**完全理解**向导工作原理的用户设计。如果您只想快速完成配置，请直接运行 `openclaw onboard`，它会引导您完成所有步骤。

---

## 🎯 学习目标

完成本指南学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解向导的设计理念和核心价值
- [ ] 掌握快速开始和高级模式的区别
- [ ] 理解每个配置步骤的作用
- [ ] 完成向导配置并理解生成的配置文件

### 进阶目标（建议掌握）

- [ ] 理解守护进程的工作原理
- [ ] 掌握认证存储机制
- [ ] 能够在不同场景下选择最优配置方案
- [ ] 排查向导配置过程中的问题

---

## 💡 首先理解：为什么需要向导？

### 设计决策背景

```
问题：OpenClaw 是一个高度可配置的系统，如何让用户快速上手？

挑战：
  - 多个组件：网关、认证、渠道、技能
  - 每个组件都有多种配置选项
  - 直接编辑配置文件对新手门槛高

解决方案：设计交互式向导

核心价值：
  ✅ 降低门槛：通过问答式交互完成配置
  ✅ 确保安全：自动生成令牌、合理默认权限
  ✅ 验证配置：运行健康检查确保配置有效
  ✅ 最佳实践：遵循社区推荐的安全和性能设置
```

### 向导 vs 手动配置

| 对比维度 | 向导模式 | 手动配置 |
|----------|----------|----------|
| **易用性** | ⭐⭐⭐⭐⭐ 问答式交互 | ⭐⭐ 需要了解配置结构 |
| **灵活性** | ⭐⭐⭐ 预设选项 | ⭐⭐⭐⭐⭐ 完全自定义 |
| **安全性** | ⭐⭐⭐⭐ 自动安全设置 | ⭐⭐⭐ 依赖用户知识 |
| **学习价值** | ⭐⭐⭐ 理解基本概念 | ⭐⭐⭐⭐⭐ 深入理解系统 |
| **配置速度** | ⭐⭐⭐⭐⭐ 快速完成 | ⭐⭐ 需查找文档 |

> **💡 专家建议**：先用向导完成基础配置，然后根据需要手动微调。这是最效率的方式。

---

## 🚀 启动向导

### 主入口

```bash
# 完整引导（推荐首次使用）
openclaw onboard
```

### 快速体验（无需配置渠道）

如果只想快速体验，不配置渠道：

```bash
# 启动 Web 控制台（无需渠道配置）
openclaw dashboard

# 然后在浏览器中直接聊天
# 访问：http://127.0.0.1:18789/
```

### 重新配置

如果需要修改配置：

```bash
# 保留现有配置，仅运行向导
openclaw configure

# 完全重置后重新配置
openclaw onboard --reset
```

---

## ⚡ 快速开始 vs 高级模式

向导开始时会提供两种模式选择。

### 快速开始（推荐新手）

快速模式使用**合理默认配置**，适合大多数用户：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| **网关模式** | 本地 (loopback) | 仅本地访问，最高安全性 |
| **工作区** | ~/.openclaw/workspace | 默认工作区位置 |
| **端口** | 18789 | 默认网关端口 |
| **认证** | Token (自动生成) | 即使本地也启用令牌认证 |
| **Tailscale** | 关闭 | 默认不暴露到网络 |
| **DM 安全** | 配对模式 | 陌生人需手动批准 |

### 高级模式（完全控制）

高级模式暴露所有配置选项：

```
高级模式配置项：
  ├── 1. 模式选择
  │     ├── 本地模式
  │     └── 远程模式
  ├── 2. 工作区配置
  │     ├── 位置
  │     └── 引导文件
  ├── 3. 网关设置
  │     ├── 端口
  │     ├── 绑定地址
  │     ├── 认证模式
  │     └── Tailscale 暴露
  ├── 4. 渠道配置
  │     ├── Telegram
  │     ├── WhatsApp
  │     ├── Discord
  │     └── 其他渠道
  ├── 5. 守护进程
  │     ├── 安装类型
  │     └── 运行时选择
  └── 6. 技能选择
```

---

## 📋 向导执行内容

### 本地模式执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    向导执行流程                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 现有配置检测                                                 │
│     └── 检测 ~/.openclaw/openclaw.json 是否存在                  │
│                              ↓                                   │
│  2. 模型/认证配置                                                │
│     ├── OAuth 或 API Key 选择                                   │
│     └── 默认模型选择                                             │
│                              ↓                                   │
│  3. 工作区配置                                                   │
│     └── 工作区位置 + 引导文件创建                                │
│                              ↓                                   │
│  4. 网关配置                                                     │
│     ├── 端口、绑定地址                                           │
│     ├── 认证模式                                                 │
│     └── Tailscale 暴露                                          │
│                              ↓                                   │
│  5. 渠道配置                                                     │
│     ├── WhatsApp（二维码）                                      │
│     ├── Telegram（Bot Token）                                   │
│     ├── Discord（Bot Token）                                    │
│     └── 其他渠道                                                 │
│                              ↓                                   │
│  6. 守护进程安装                                                 │
│     ├── macOS: LaunchAgent                                      │
│     └── Linux: systemd 用户单元                                 │
│                              ↓                                   │
│  7. 健康检查                                                     │
│     └── 启动网关并运行 health 检查                                │
│                              ↓                                   │
│  8. 技能选择                                                     │
│     └── 推荐技能安装                                             │
│                              ↓                                   │
│  9. 完成                                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 远程模式

远程模式**仅配置本地客户端**，连接到其他地方的网关：

```
┌─────────────────────────────────────────────────────────────────┐
│                    远程模式配置                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  远程模式不会在远程主机上安装或更改任何内容！                    │
│                                                                 │
│  配置内容：                                                      │
│    ├── 1. 网关地址连接                                          │
│    │     └── 输入远程网关地址                                    │
│    ├── 2. 认证令牌                                              │
│    │     └── 输入连接令牌                                        │
│    └── 3. 验证连接                                              │
│          └── 测试连接是否正常                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔍 流程详情（本地模式）

### 1. 现有配置检测

向导首先检测是否已存在配置：

```
检测逻辑：
  ├── ~/.openclaw/openclaw.json 存在？
  │     ├── 是 → 选择 "保留 / 修改 / 重置"
  │     └── 否 → 创建新配置
  │
  └── 配置有效？
        ├── 是 → 进入下一步
        └── 否 → 停止并提示运行 openclaw doctor
```

**重置选项：**

| 重置范围 | 说明 | 命令参数 |
|----------|------|----------|
| 仅配置 | 只重置配置文件 | `--reset config` |
| 配置 + 凭证 | 重置配置和认证 | `--reset credentials` |
| 完全重置 | 全部删除，包括工作区 | `--reset all` |

> **⚠️ 注意**：重置使用 `trash` 命令（不会永久删除），方便恢复。

### 2. 模型/认证配置

认证配置是最重要的步骤之一。

#### Anthropic API Key（推荐）

```
配置方式：
  ├── 检查环境变量 ANTHROPIC_API_KEY
  │     ├── 存在 → 自动使用
  │     └── 不存在 → 提示输入
  └── 保存位置
        └── ~/.clawdbot/agents/<agentId>/agent/auth-profiles.json
```

#### Anthropic OAuth（Claude Code CLI 用户）

```
macOS 特殊处理：
  └── 检查 Keychain 中的 "Claude Code-credentials"
       └── 选择 "始终允许" 以免 launchd 启动时阻塞

Linux/Windows：
  └── 重用 ~/.claude/.credentials.json（如存在）
```

#### Anthropic Token

```
使用场景：
  └── 在任何机器上运行 `claude setup-token`
       └── 生成 token，可命名（空白 = 默认）
```

#### OpenAI API Key

```
配置方式：
  ├── 检查环境变量 OPENAI_API_KEY
  │     ├── 存在 → 自动使用
  │     └── 不存在 → 提示输入
  └── 保存位置
        └── ~/.clawdbot/.env
```

#### 其他提供商

| 提供商 | 配置方式 | 说明 |
|--------|----------|------|
| **OpenCode Zen** | API Key | 提示输入 `OPENCODE_API_KEY` |
| **Vercel AI Gateway** | API Key | 提示输入 `AI_GATEWAY_API_KEY` |
| **MiniMax M2.1** | 自动配置 | 配置自动写入 |
| **Synthetic** | API Key | 提示输入 `SYNTHETIC_API_KEY` |
| **Moonshot (Kimi)** | 自动配置 | 配置自动写入 |
| **跳过** | - | 暂不配置认证 |

#### 模型选择

```
选择逻辑：
  ├── 检测到的认证选项
  │     └── 从中选择默认模型
  │
  └── 或手动输入
        └── provider/model 格式
             └── 例如：anthropic/claude-opus-4-5
```

> **🔐 安全提醒**：向导会运行模型检查，如果配置的模型未知或缺少认证会发出警告。

### 3. 工作区配置

```
默认位置：~/.clawdbot/workspace

工作区结构：
  workspace/
  ├── AGENTS.md       # 代理行为定义
  ├── SOUL.md         # 代理个性
  ├── USER.md         # 用户信息
  ├── MEMORY.md       # 长期记忆
  ├── TOOLS.md        # 工具说明
  └── skills/         # 技能目录
```

### 4. 网关配置

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| **端口** | 网关监听端口 | 18789 |
| **绑定地址** | 网络接口绑定 | loopback (仅本地) |
| **认证模式** | 连接认证方式 | token |
| **Tailscale** | 是否通过 Tailscale 暴露 | 关闭 |

#### 认证建议

```
专家建议：即使使用 loopback（仅本地），也保持 Token 认证

原因：
  ✅ 本地 WS 客户端必须认证
  ✅ 防止未授权的本地进程连接
  ✅ 保持安全性一致性

何时可以禁用：
  └── 仅在完全信任每个本地进程时
       └── 不推荐，除非有特殊需求
```

### 5. 渠道配置

#### 支持的渠道

| 渠道 | 认证方式 | 配置复杂度 | 文档链接 |
|------|----------|------------|----------|
| **WhatsApp** | 二维码 | ⭐⭐ | [WhatsApp](/zh-CN/channels/whatsapp) |
| **Telegram** | Bot Token | ⭐⭐ | [Telegram](/zh-CN/channels/telegram) |
| **Discord** | Bot Token | ⭐⭐ | [Discord](/zh-CN/channels/discord) |
| **Google Chat** | 服务账户 | ⭐⭐⭐ | [Google Chat](/zh-CN/channels/googlechat) |
| **Mattermost** | Bot Token | ⭐⭐ | [Mattermost](/zh-CN/channels/mattermost) |
| **Signal** | signal-cli | ⭐⭐⭐⭐ | [Signal](/zh-CN/channels/signal) |
| **iMessage** | 本地 CLI | ⭐⭐⭐ | [iMessage](/zh-CN/channels/imessage) |

#### DM 安全模式

```
默认安全策略：配对模式

工作流程：
  1. 陌生人发送 DM
  2. OpenClaw 发送配对码给发送者
  3. 用户批准配对请求
       └── openclaw pairing approve <channel> <code>
  4. 发送者被添加到允许列表
  5. AI 正常回复

替代方案：
  └── 在配置中设置 allowFrom: ["*"] 允许所有 DM
       └── ⚠️ 不推荐，存在安全风险
```

### 6. 守护进程安装

#### macOS: LaunchAgent

```
配置位置：~/Library/LaunchAgents/com.openclaw.gateway.plist

启动时机：用户登录时

限制：
  └── 需要登录用户会话
  └── 无头模式需自定义 LaunchDaemon（向导不提供）
```

#### Linux: systemd 用户单元

```
配置位置：~/.config/systemd/user/openclaw-gateway.service

启动时机：用户登录后（配合 linger）

启用 linger：
  └── loginctl enable-linger <user>
       └── 允许服务在用户注销后继续运行

可能需要 sudo：
  └── 写入 /var/lib/systemd/linger
       └── 先尝试无 sudo，失败再提示
```

#### 运行时选择

| 运行时 | 推荐度 | 说明 |
|--------|--------|------|
| **Node.js** | ⭐⭐⭐⭐⭐ | **推荐**，WhatsApp/Telegram 必需 |
| **Bun** | ⭐⭐ | **不推荐**，存在兼容性问题 |

> **⚠️ 重要**：使用 WhatsApp 或 Telegram 渠道时，**必须**使用 Node.js 运行时。

### 7. 健康检查

```
检查内容：
  ├── 启动网关（如需要）
  ├── 检查网关可达性
  ├── 验证认证配置
  └── 检查渠道连接状态

输出示例：
  ✅ Gateway: healthy
  ✅ Auth: configured (Anthropic)
  ⚠️ Channel (whatsapp): not connected
```

### 8. 技能选择

```
技能配置步骤：
  ├── 1. 读取可用技能列表
  ├── 2. 检查技能要求
  │     ├── Node.js 版本要求
  │     ├── 系统依赖
  │     └── API Keys
  ├── 3. 选择技能
  │     ├── Web Search（推荐）
  │     ├── GitHub
  │     └── 其他
  └── 4. 安装技能
        ├── npm/pnpm 安装
        └── 可选依赖安装
```

### 9. 完成

```
完成检查清单：
  ✅ 网关配置完成
  ✅ 至少一个认证配置
  ✅ 至少一个渠道配置（如选择）
  ✅ 守护进程安装（如选择）
  ✅ 健康检查通过
```

---

## 🎓 理解向导背后的原理

### 配置文件生成

向导会生成 `~/.clawdbot/openclaw.json`：

```json5
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "type": "token",
      "token": "<auto-generated-uuid>"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4",
      "workspace": "~/.clawdbot/workspace"
    }
  },
  "channels": {
    "whatsapp": {
      "enabled": true
    },
    "telegram": {
      "enabled": true,
      "botToken": "<stored-in-credentials>"
    }
  }
}
```

### 守护进程原理

#### macOS LaunchAgent

```
┌─────────────────────────────────────────────────────────────────┐
│                    LaunchAgent 工作原理                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户登录                                                        │
│      ↓                                                          │
│  launchd 读取 ~/Library/LaunchAgents/                            │
│      ↓                                                          │
│  加载 com.openclaw.gateway.plist                                │
│      ↓                                                          │
│  启动 openclaw gateway (作为用户进程)                           │
│      ↓                                                          │
│  用户注销时                                                      │
│      └── 进程终止（除非配置 KeepAlive）                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Linux systemd

```
┌─────────────────────────────────────────────────────────────────┐
│                    systemd 用户单元工作原理                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户登录                                                        │
│      ↓                                                          │
│  systemd --user 启动                                             │
│      ↓                                                          │
│  加载 openclaw-gateway.service                                 │
│      ↓                                                          │
│  启动 openclaw gateway                                         │
│      ↓                                                          │
│  用户注销                                                        │
│      ├── 未启用 linger → 进程终止                              │
│      └── 已启用 linger → 进程继续运行                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 认证存储机制

```
┌─────────────────────────────────────────────────────────────────┐
│                    认证信息存储架构                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OAuth 认证                                                     │
│      └── ~/.clawdbot/credentials/oauth.json                     │
│           ├── access_token                                      │
│           ├── refresh_token                                     │
│           └── expiry_time                                       │
│                                                                 │
│  API Key 认证                                                   │
│      └── ~/.clawdbot/agents/<agentId>/agent/auth-profiles.json│
│           ├── anthropic                                        │
│           │     └── apiKey                                      │
│           └── openai                                            │
│                 └── apiKey                                       │
│                                                                 │
│  环境变量备份                                                   │
│      └── ~/.clawdbot/.env                                      │
│           ├── OPENAI_API_KEY                                   │
│           └── ANTHROPIC_API_KEY                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🐛 故障排除

### 向导卡住

**症状**：向导停止响应，没有错误信息。

**解决方案**：

```bash
# 1. 强制重置配置
openclaw onboard --reset

# 2. 查看详细日志
openclaw logs --lines 100

# 3. 检查系统资源
top -u $USER | grep node
```

### 配置验证失败

**症状**：向导提示配置无效。

**解决方案**：

```bash
# 1. 先运行诊断
openclaw doctor

# 2. 根据诊断结果修复问题

# 3. 重新运行向导
openclaw onboard
```

### 守护进程启动失败

#### macOS

```bash
# 检查 LaunchAgent 状态
launchctl list | grep openclaw

# 查看详细错误
openclaw logs

# 手动加载
launchctl load ~/Library/LaunchAgents/com.openclaw.gateway.plist

# 重新加载
launchctl unload ~/Library/LaunchAgents/com.openclaw.gateway.plist
launchctl load ~/Library/LaunchAgents/com.openclaw.gateway.plist
```

#### Linux

```bash
# 检查 systemd 状态
systemctl --user status openclaw-gateway

# 查看日志
journalctl --user -u openclaw-gateway -f

# 启用 linger
loginctl enable-linger $USER

# 重新启动服务
systemctl --user restart openclaw-gateway
```

### OAuth 授权失败

**症状**：浏览器打开后授权失败。

**解决方案**：

```bash
# 1. 删除旧的 OAuth 凭证
rm ~/.clawdbot/credentials/oauth.json

# 2. 清除浏览器缓存

# 3. 重新运行向导
openclaw onboard

# 4. 确保 localhost 回调未被阻止
```

---

## 📊 专家思维模型：配置决策框架

在向导过程中，使用以下决策框架：

```
┌─────────────────────────────────────────────────────────────────┐
│                    配置决策流程                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  问题 1：使用场景？                                             │
│           ↓                                                     │
│     个人使用 ←─┬─→ 团队/共享使用                                │
│           ↓                                                     │
│     本地模式                                                    │
│                                                                 │
│  问题 2：需要远程访问？                                         │
│           ↓                                                     │
│     是 ←─┬─→ 否                                                 │
│           ↓                                                     │
│     Tailscale/令牌认证                                          │
│     （增加复杂度，但更灵活）                                      │
│                                                                 │
│  问题 3：主要使用哪些渠道？                                     │
│           ↓                                                     │
│     WhatsApp + Telegram                                        │
│       ↓                                                         │
│     必须使用 Node.js 运行时                                     │
│                                                                 │
│  问题 4：需要 24/7 运行？                                       │
│           ↓                                                     │
│     是 ←─┬─→ 否                                                 │
│           ↓                                                     │
│     安装守护进程                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 知识点回顾

完成本指南后，您应该掌握以下技能：

| 技能 | 掌握程度 |
|------|----------|
| 向导工作原理 | ⭐⭐⭐⭐⭐ |
| 配置选项理解 | ⭐⭐⭐⭐ |
| 守护进程配置 | ⭐⭐⭐ |
| 认证存储机制 | ⭐⭐⭐⭐ |
| 故障排查 | ⭐⭐⭐ |

---

## 🎯 下一步学习

| 主题 | 文档链接 | 预计时间 |
|------|----------|----------|
| **网关概念** | [Gateway 详解](/zh-CN/concepts/gateway) | 20 分钟 |
| **认证机制** | [OAuth 概念](/zh-CN/concepts/oauth) | 15 分钟 |
| **渠道配置** | [Channels](/zh-CN/channels/index) | 45 分钟 |
| **CLI 参考** | [CLI 命令](/zh-CN/cli/index) | 30 分钟 |

---

## 📖 附录

### 附录 A：命令速查

| 命令 | 说明 |
|------|------|
| `openclaw onboard` | 启动向导 |
| `openclaw onboard --reset` | 重置并重新配置 |
| `openclaw configure` | 修改现有配置 |
| `openclaw dashboard` | 打开 Web 控制台 |
| `openclaw doctor` | 诊断配置问题 |
| `openclaw status` | 查看状态 |

### 附录 B：术语表

| 英文术语 | 中文术语 | 说明 |
|---------|---------|------|
| Wizard | 向导 | 交互式配置流程 |
| Onboarding | 引导配置 | 初始化设置流程 |
| LaunchAgent | 启动代理 | macOS 用户级服务 |
| systemd | 系统管理器 | Linux 服务管理器 |
| linger | 驻留 | 允许服务在注销后运行 |
| OAuth | 开放授权 | 开放认证协议 |

### 附录 C：相关资源

- **官方文档**：[docs.openclaw.ai](https://docs.openclaw.ai)
- **GitHub**：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- **社区支持**：Discord #support 频道

---

> **💡 提示**：遇到问题时，运行 `openclaw doctor` 获取诊断信息。大多数配置问题都可以通过此工具自动修复。
