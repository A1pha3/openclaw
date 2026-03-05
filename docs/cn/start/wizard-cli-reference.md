---
summary: "CLI 配置向导完整参考手册，包含认证/模型设置、输出内容和内部机制详解"
read_when:
  - 需要 openclaw onboard 的详细行为说明
  - 调试配置结果或集成配置客户端
title: "CLI 配置向导完整参考"
sidebarTitle: "CLI 完整参考"
---

# CLI 配置向导完整参考

本文档是 `openclaw onboard` 命令的完整参考手册。快速入门指南请参考 [CLI 配置向导](/zh-CN/start/wizard)。

---

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解配置向导的完整执行流程和每个步骤的作用
- [ ] 掌握本地模式和远程模式的区别与使用场景
- [ ] 了解所有认证和模型配置选项及其优先级

### 进阶目标（建议掌握）

- [ ] 理解配置文件的内部结构和存储位置
- [ ] 掌握自动化脚本集成的方法
- [ ] 能够调试和解决配置问题

---

## 📋 配置向导功能概览

### 本地模式（默认）

本地模式是完整的配置流程，引导您完成以下内容：

```
┌─────────────────────────────────────────────────────────────┐
│                    本地配置流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 模型和认证设置                                           │
│     ├── OpenAI Code 订阅 OAuth                              │
│     ├── Anthropic API Key 或设置令牌                        │
│     └── MiniMax、GLM、Moonshot、AI Gateway 等选项          │
│                                                             │
│  2. 工作区位置和引导文件                                     │
│                                                             │
│  3. 网关设置（端口、绑定、认证、Tailscale）                   │
│                                                             │
│  4. 渠道和提供商                                             │
│     ├── Telegram、WhatsApp、Discord                         │
│     ├── Google Chat、Mattermost 插件                        │
│     └── Signal                                              │
│                                                             │
│  5. 后台服务安装                                             │
│     ├── macOS: LaunchAgent                                  │
│     └── Linux/WSL2: systemd 用户单元                         │
│                                                             │
│  6. 健康检查                                                │
│                                                             │
│  7. 技能设置                                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 远程模式

远程模式将当前机器配置为连接到其他地方运行的网关。

> **⚠️ 重要说明**：远程模式不会在远程主机上安装或修改任何内容。

**配置内容**：

| 配置项 | 说明 |
|--------|------|
| 远程网关 URL | `ws://...` 格式的 WebSocket 地址 |
| 认证令牌 | 如果远程网关需要认证（推荐） |

**网络要求**：

- 如果网关仅绑定 loopback，使用 SSH 隧道或 tailnet
- 服务发现提示：
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)

---

## 🔄 本地模式详细流程

### 步骤 1：现有配置检测

当检测到 `~/.openclaw/openclaw.json` 已存在时：

```
配置文件检测
├── 保留（Keep）- 保持现有配置不变
├── 修改（Modify）- 在现有基础上调整
└── 重置（Reset）- 清除配置重新开始
```

**重置范围选项**：

| 重置类型 | 清除内容 |
|---------|----------|
| **仅配置** | 配置文件 |
| **配置 + 凭证 + 会话** | 配置、认证信息、会话历史 |
| **完全重置** | 以上所有 + 工作区 |

**命令行参数**：

- `--reset`: 默认重置范围为 `config+creds+sessions`
- `--reset-scope full`: 同时移除工作区

> **💡 专家提示**：重新运行向导不会删除任何内容，除非您明确选择 Reset（或传递 `--reset` 参数）。

**配置验证**：

- 如果配置无效或包含遗留的配置键，向导会停止并要求您先运行 `openclaw doctor`
- 重置使用 `trash` 命令（而非直接删除），提供安全回收机制

---

### 步骤 2：模型和认证

完整的认证和模型选项矩阵请参考 [认证和模型选项](#认证和模型选项) 章节。

---

### 步骤 3：工作区（Workspace）

**默认位置**：`~/.openclaw/workspace`（可配置）

**工作区内容**：

- 首次运行引导仪式所需的种子文件
- AI 代理的工作目录和资源

**工作区布局**：详见 [代理工作区](/zh-CN/concepts/agent-workspace)。

---

### 步骤 4：网关配置

向导会提示配置以下内容：

| 配置项 | 说明 | 推荐设置 |
|--------|------|----------|
| **端口** | 网关监听端口 | 默认 18789 |
| **绑定地址** | 网关监听的网络接口 | loopback 仅本地访问 |
| **认证模式** | 令牌认证开关 | **推荐启用** |
| **Tailscale** | 通过 Tailscale 暴露 | 按需启用 |

**安全建议**：

> **🔐 安全最佳实践**：
> - 即使是 loopback 绑定，也建议保持令牌认证启用，这样本地 WebSocket 客户端也必须经过认证
> - 只有在完全信任所有本地进程时才禁用认证
> - 非 loopback 绑定**必须**启用认证

---

### 步骤 5：渠道配置

向导支持配置以下消息渠道：

| 渠道 | 认证方式 | 复杂度 |
|------|----------|--------|
| [WhatsApp](/zh-CN/channels/whatsapp) | 可选 QR 扫码登录 | ⭐⭐ |
| [Telegram](/zh-CN/channels/telegram) | Bot Token | ⭐ |
| [Discord](/zh-CN/channels/discord) | Bot Token | ⭐ |
| [Google Chat](/zh-CN/channels/googlechat) | 服务账号 JSON + Webhook 受众 | ⭐⭐ |
| [Mattermost 插件](/zh-CN/channels/mattermost) | Bot Token + 基础 URL | ⭐⭐ |
| [Signal](/zh-CN/channels/signal) | 可选 `signal-cli` 安装 + 账号配置 | ⭐⭐⭐ |
| [BlueBubbles](/zh-CN/channels/bluebubbles) | 服务器 URL + 密码 + Webhook | ⭐⭐ |
| [iMessage](/zh-CN/channels/imessage) | 遗留 `imsg` CLI 路径 + 数据库访问 | ⭐⭐⭐ |

**私信（DM）安全**：

- 默认使用配对（pairing）机制
- 首次发送私信时会发送一个验证码
- 通过以下方式批准：
  ```bash
  openclaw pairing approve <channel> <code>
  ```
- 或使用允许列表替代

---

### 步骤 6：后台服务安装

### macOS：LaunchAgent

- 要求用户登录会话激活
- 无头环境使用自定义 LaunchDaemon（不随附提供）

### Linux 和 Windows (WSL2)：systemd 用户单元

- 向导尝试执行 `loginctl enable-linger <user>`，使网关在登出后保持运行
- 可能提示输入 sudo（写入 `/var/lib/systemd/linger`）；会先尝试无 sudo 执行

**运行时选择**：

| 运行时 | 推荐度 | 说明 |
|--------|--------|------|
| **Node** | ✅ 推荐 | WhatsApp 和 Telegram 的必需运行时 |
| **Bun** | ⚠️ 不推荐 | 存在已知的兼容性问题 |

---

### 步骤 7：健康检查

- 启动网关（如需要）并运行 `openclaw health`
- `openclaw status --deep` 在状态输出中添加网关健康探测

---

### 步骤 8：技能设置

向导会：

1. 读取可用技能并检查依赖要求
2. 让您选择节点管理器：npm 或 pnpm（不推荐 bun）
3. 安装可选依赖（某些在 macOS 上使用 Homebrew）

---

### 步骤 9：完成

- 显示摘要和后续步骤
- 包括 iOS、Android 和 macOS App 选项

---

## 💡 无头环境和服务器提示

### 无 GUI 环境处理

- 如果未检测到 GUI，向导会打印 SSH 端口转发指令，而非打开浏览器
- 如果 Control UI 资源缺失，向导会尝试构建
  - 回退命令：`pnpm ui:build`（自动安装 UI 依赖）

### OAuth 认证迁移

在无浏览器的服务器上完成 OAuth 认证：

1. **在有浏览器的机器上完成 OAuth**
2. **复制凭证文件**：
   ```bash
   scp ~/.openclaw/credentials/oauth.json user@server:~/.openclaw/credentials/
   ```
3. **在服务器上使用已复制的凭证**

> **💡 专家提示**：您也可以设置 `OPENCLAW_STATE_DIR` 环境变量来自定义状态目录位置。

---

## 🔐 认证和模型选项

<details>
<summary><strong>Anthropic API Key</strong></summary>

- 如果存在 `ANTHROPIC_API_KEY` 环境变量则自动使用
- 否则提示输入密钥
- 保存密钥供后台服务使用

</details>

<details>
<summary><strong>Anthropic OAuth (Claude Code CLI)</strong></summary>

**macOS**：
- 检查钥匙串项 "Claude Code-credentials"

**Linux 和 Windows**：
- 复用 `~/.claude/.credentials.json`（如存在）

**macOS 特别提示**：
> 选择"始终允许"，避免 launchd 启动时阻塞。

</details>

<details>
<summary><strong>Anthropic 设置令牌（setup-token）</strong></summary>

1. 在任何机器上运行 `claude setup-token`
2. 复制生成的令牌
3. 在向导中粘贴令牌
4. 可选择命名（空白使用默认名称）

</details>

<details>
<summary><strong>OpenAI Code 订阅（复用 Codex CLI）</strong></summary>

- 如果 `~/.codex/auth.json` 存在，向导可以复用它

</details>

<details>
<summary><strong>OpenAI Code 订阅（OAuth）</strong></summary>

- 浏览器流程，粘贴 `code#state`
- 当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.3-codex`

</details>

<details>
<summary><strong>OpenAI API Key</strong></summary>

- 如果存在 `OPENAI_API_KEY` 环境变量则自动使用
- 否则提示输入密钥
- 将凭证存储在认证配置文件中
- 当模型未设置、为 `openai/*` 或 `openai-codex/*` 时，将 `agents.defaults.model` 设置为 `openai/gpt-5.1-codex`

</details>

<details>
<summary><strong>xAI (Grok) API Key</strong></summary>

- 提示输入 `XAI_API_KEY`
- 将 xAI 配置为模型提供商

</details>

<details>
<summary><strong>OpenCode Zen</strong></summary>

- 提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）
- 设置 URL：[opencode.ai/auth](https://opencode.ai/auth)

</details>

<details>
<summary><strong>通用 API Key</strong></summary>

- 为您存储密钥

</details>

<details>
<summary><strong>Vercel AI Gateway</strong></summary>

- 提示输入 `AI_GATEWAY_API_KEY`
- 详情：[Vercel AI Gateway](/zh-CN/providers/vercel-ai-gateway)

</details>

<details>
<summary><strong>Cloudflare AI Gateway</strong></summary>

- 提示输入账户 ID、网关 ID 和 `CLOUDFLARE_AI_GATEWAY_API_KEY`
- 详情：[Cloudflare AI Gateway](/zh-CN/providers/cloudflare-ai-gateway)

</details>

<details>
<summary><strong>MiniMax M2.5</strong></summary>

- 配置自动写入
- 详情：[MiniMax](/zh-CN/providers/minimax)

</details>

<summary>
<strong>Synthetic（Anthropic 兼容）</strong>

- 提示输入 `SYNTHETIC_API_KEY`
- 详情：[Synthetic](/zh-CN/providers/synthetic)

</details>

<details>
<summary><strong>Moonshot 和 Kimi Coding</strong></summary>

- Moonshot (Kimi K2) 和 Kimi Coding 配置自动写入
- 详情：[Moonshot AI (Kimi + Kimi Coding)](/zh-CN/providers/moonshot)

</details>

<details>
<summary><strong>自定义提供商</strong></summary>

支持 OpenAI 兼容和 Anthropic 兼容的端点。

**交互式配置支持的 API Key 存储方式**：
- **立即粘贴 API Key**（明文存储）
- **使用密钥引用**（环境变量引用或配置的提供商引用，带预检验证）

**非交互式参数**：
- `--auth-choice custom-api-key`
- `--custom-base-url`
- `--custom-model-id`
- `--custom-api-key`（可选；回退到 `CUSTOM_API_KEY`）
- `--custom-provider-id`（可选）
- `--custom-compatibility <openai|anthropic>`（可选；默认 `openai`）

</details>

<details>
<summary><strong>跳过</strong></summary>

- 保持认证未配置状态

</details>

### 模型行为

- 从检测到的选项中选择默认模型，或手动输入提供商和模型
- 向导运行模型检查，如果配置的模型未知或缺少认证则发出警告

### 凭证和配置文件路径

| 凭证类型 | 存储路径 |
|---------|----------|
| **OAuth 凭证** | `~/.openclaw/credentials/oauth.json` |
| **认证配置**（API Keys + OAuth） | `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` |

### API Key 存储模式

**默认配置行为**：将 API Key 以明文形式持久化存储在认证配置文件中。

**引用模式**：`--secret-input-mode ref` 启用引用模式而非明文密钥存储。

**交互式配置**中，您可以选择：
- **环境变量引用**（例如 `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`）
- **配置的提供商引用**（`file` 或 `exec`）+ 提供商别名 + id

**引用模式的预检验证**：
- **环境变量引用**：验证变量名 + 当前配置环境中的非空值
- **提供商引用**：验证提供商配置并解析请求的 id
- 如果预检失败，配置向导会显示错误并允许重试

**非交互式模式**下，`--secret-input-mode ref` 仅支持环境变量后端：
- 在配置流程环境中设置提供商环境变量
- 内联密钥参数（例如 `--openai-api-key`）要求必须设置环境变量；否则配置快速失败
- 对于自定义提供商，非交互式 `ref` 模式将 `models.providers.<id>.apiKey` 存储为 `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`
- 在这种自定义提供商情况下，`--custom-api-key` 要求必须设置 `CUSTOM_API_KEY`；否则配置快速失败

**现有明文配置**：继续不受影响地正常工作。

---

## 📂 输出和内部机制

### 典型配置文件字段

`~/.openclaw/openclaw.json` 中的典型字段：

| 字段 | 说明 |
|------|------|
| `agents.defaults.workspace` | 默认工作区路径 |
| `agents.defaults.model` / `models.providers` | 默认模型或提供商配置 |
| `tools.profile` | 工具配置文件（本地配置默认为 `"messaging"`；保留现有显式值） |
| `gateway.*` | 网关配置（mode, bind, auth, tailscale） |
| `session.dmScope` | 私信会话范围（本地配置默认为 `per-channel-peer`；保留现有显式值） |
| `channels.telegram.botToken` | Telegram Bot Token |
| `channels.discord.token` | Discord Bot Token |
| `channels.signal.*` | Signal 渠道配置 |
| `channels.imessage.*` | iMessage 渠道配置 |
| `channels.*.allowlist` | 渠道允许列表（Slack、Discord、Matrix、Microsoft Teams） |
| `skills.install.nodeManager` | 技能节点管理器 |
| `wizard.lastRunAt` | 向导最后运行时间 |
| `wizard.lastRunVersion` | 向导最后运行时的版本 |
| `wizard.lastRunCommit` | 向导最后运行时的提交 |
| `wizard.lastRunCommand` | 向导最后运行的命令 |
| `wizard.lastRunMode` | 向导最后运行的模式 |

### 其他存储位置

| 内容 | 存储位置 |
|------|----------|
| **代理列表** | `openclaw agents add` 写入 `agents.list[]` 和可选的 `bindings` |
| **WhatsApp 凭证** | `~/.openclaw/credentials/whatsapp/<accountId>/` |
| **会话** | `~/.openclaw/agents/<agentId>/sessions/` |

> **💡 提示**：某些渠道以插件形式提供。在配置过程中选择时，向导会提示在渠道配置之前安装插件（npm 或本地路径）。

### 网关向导 RPC

客户端（macOS App 和 Control UI）可以渲染步骤而无需重新实现配置逻辑：

| RPC 方法 | 说明 |
|---------|------|
| `wizard.start` | 启动向导 |
| `wizard.next` | 下一步 |
| `wizard.cancel` | 取消向导 |
| `wizard.status` | 获取向导状态 |

### Signal 设置行为

| 步骤 | 说明 |
|------|------|
| **下载** | 下载适当的发布资产 |
| **存储** | 存储到 `~/.openclaw/tools/signal-cli/<version>/` |
| **配置** | 在配置中写入 `channels.signal.cliPath` |
| **JVM 构建** | 需要 Java 21 |
| **原生构建** | 优先使用（如可用） |
| **Windows** | 使用 WSL2，在 WSL 内遵循 Linux signal-cli 流程 |

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| **配置中心** | [CLI 配置向导](/zh-CN/start/wizard) |
| **自动化和脚本** | [CLI 自动化](/zh-CN/start/wizard-cli-automation) |
| **命令参考** | [`openclaw onboard`](/zh-CN/cli/onboard) |

---

## 🎯 知识点回顾

完成本文档学习后，您应该掌握以下核心概念：

| 概念 | 理解程度 |
|------|----------|
| **配置向导流程** | ⭐⭐⭐ 完全理解 |
| **认证选项** | ⭐⭐⭐ 知道如何选择 |
| **配置文件结构** | ⭐⭐ 了解主要字段 |
| **远程模式** | ⭐⭐ 知道如何使用 |
| **内部机制** | ⭐ 了解基本原理 |

---

## ⚠️ 常见问题与解决方案

### 问题一：配置向导检测到无效配置

**症状**：向导停止并提示运行 `openclaw doctor`

**解决方案**：

```bash
# 1. 运行诊断工具
openclaw doctor

# 2. 尝试自动修复
openclaw doctor --fix

# 3. 查看详细问题
openclaw doctor --verbose
```

### 问题二：重置后数据丢失

**症状**：选择 Reset 后重要配置被清除

**预防措施**：

- 重置前备份配置：`cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup`
- 使用"保留"或"修改"而非"重置"
- 理解重置范围选项

### 问题三：OAuth 认证在服务器上失败

**症状**：无浏览器环境无法完成 OAuth

**解决方案**：

1. 在有浏览器的机器上完成 OAuth
2. 复制 `~/.openclaw/credentials/oauth.json` 到服务器
3. 确保文件权限正确：`chmod 600 ~/.openclaw/credentials/oauth.json`

---

> **💡 提示**：文档持续更新中！如果发现错误或有改进建议，欢迎通过 GitHub Issues 反馈。
