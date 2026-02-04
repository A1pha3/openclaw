---
summary: "将 OpenClaw 从一台机器迁移到另一台：保留会话、认证和渠道登录状态"
read_when:
  - 将 OpenClaw 迁移到新电脑或服务器
  - 想要保留会话记录、认证信息和渠道登录（如 WhatsApp）
title: "迁移指南"
---

# 🚚 迁移指南

本指南帮助你将 OpenClaw 网关从一台机器迁移到另一台，**无需重新进行引导配置**。

## 🎯 迁移原理

迁移的核心很简单：

```
旧机器                        新机器
┌─────────────────┐          ┌─────────────────┐
│ ~/.openclaw/    │  ──复制──▶ │ ~/.openclaw/    │
│ ├── openclaw.json│          │ ├── openclaw.json│
│ ├── credentials/│          │ ├── credentials/│
│ ├── agents/     │          │ ├── agents/     │
│ └── workspace/  │          │ └── workspace/  │
└─────────────────┘          └─────────────────┘
```

但有一些常见的"坑"需要注意：**配置文件位置**、**权限**和**不完整复制**。

---

## 📋 迁移前准备

### 1. 确认你的状态目录

大多数安装使用默认位置：

- **状态目录**: `~/.openclaw/`

但如果你使用了以下选项，位置可能不同：

| 配置方式 | 状态目录位置 |
|---------|-------------|
| 默认 | `~/.openclaw/` |
| `--profile <name>` | `~/.openclaw-<profile>/` |
| `OPENCLAW_STATE_DIR=/custom/path` | `/custom/path/` |

**不确定？** 在**旧机器**上运行：

```bash
openclaw status
```

查看输出中的 `OPENCLAW_STATE_DIR` 或 profile 信息。如果你运行多个网关，对每个 profile 重复检查。

### 2. 确认你的工作区

常见位置：

- `~/.openclaw/workspace/`（推荐位置）
- 你创建的自定义目录

工作区是存放 `MEMORY.md`、`USER.md` 和 `memory/*.md` 等文件的地方。

### 3. 了解迁移内容

**完整迁移（复制状态目录 + 工作区）保留：**

| 内容 | 说明 |
|------|------|
| 网关配置 | `openclaw.json` |
| 认证信息 | API 密钥、OAuth 令牌 |
| 会话历史 | 所有对话记录 |
| 代理状态 | 每个代理的运行状态 |
| 渠道状态 | WhatsApp 登录、Telegram 会话等 |
| 工作区文件 | 记忆、技能笔记等 |

**仅复制工作区（如通过 Git）不保留：**

- ❌ 会话历史
- ❌ 认证凭据
- ❌ 渠道登录状态

这些都存储在 `$OPENCLAW_STATE_DIR` 中。

---

## 🔄 迁移步骤

### 步骤 0：备份（旧机器）

**首先停止网关**，确保文件不会在复制过程中变化：

```bash
openclaw gateway stop
```

**创建备份归档**（推荐）：

```bash
# 进入用户目录
cd ~

# 打包状态目录
tar -czf openclaw-state.tgz .openclaw

# 如果工作区不在状态目录内，单独打包
tar -czf openclaw-workspace.tgz .openclaw/workspace
```

**多 profile 场景**：如果有 `~/.openclaw-main`、`~/.openclaw-work` 等，分别打包。

### 步骤 1：在新机器安装 OpenClaw

在**新机器**上安装 CLI（如需要也安装 Node）：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

详见：[安装指南](/zh-CN/install)

这时如果引导向导创建了新的 `~/.openclaw/`，没关系——下一步会覆盖它。

### 步骤 2：复制状态目录 + 工作区

复制以下内容到新机器：

- `$OPENCLAW_STATE_DIR`（默认 `~/.openclaw/`）
- 你的工作区（默认 `~/.openclaw/workspace/`）

**常用方法**：

```bash
# 方法 1：scp 传输归档文件
scp user@old-machine:~/openclaw-state.tgz .
tar -xzf openclaw-state.tgz -C ~

# 方法 2：rsync 同步（保留权限）
rsync -avz user@old-machine:~/.openclaw/ ~/.openclaw/

# 方法 3：外部存储设备
```

**复制后检查**：

- ✅ 隐藏目录已包含（如 `.openclaw/`）
- ✅ 文件权限正确（运行网关的用户有读写权限）

### 步骤 3：运行 Doctor（迁移修复）

在**新机器**上：

```bash
openclaw doctor
```

Doctor 是"安全修复"命令。它会：
- 修复服务配置
- 应用配置迁移
- 警告任何不匹配

然后启动网关：

```bash
openclaw gateway restart
openclaw status
```

---

## ⚠️ 常见问题与解决

### 问题：profile/状态目录不匹配

**症状**：
- 配置更改不生效
- 渠道丢失/已登出
- 会话历史为空

**原因**：旧网关使用了 profile 或自定义 `OPENCLAW_STATE_DIR`，新网关使用了不同的位置。

**解决**：使用迁移时的**相同** profile/状态目录运行网关：

```bash
# 如果旧机器使用 --profile work
openclaw --profile work gateway restart

# 然后运行 doctor
openclaw --profile work doctor
```

### 问题：只复制了 openclaw.json

**症状**：渠道登出、会话丢失

**原因**：`openclaw.json` 不够。很多状态存储在：
- `$OPENCLAW_STATE_DIR/credentials/` — 认证信息
- `$OPENCLAW_STATE_DIR/agents/<agentId>/...` — 代理数据

**解决**：**始终迁移整个** `$OPENCLAW_STATE_DIR` 目录。

### 问题：权限/所有权错误

**症状**：网关无法读取凭据/会话

**原因**：以 root 复制或更换了用户

**解决**：确保状态目录和工作区归运行网关的用户所有：

```bash
chown -R $(whoami):$(whoami) ~/.openclaw
```

### 问题：远程/本地模式混淆

**场景**：
- 你的 UI（WebUI/TUI）连接到**远程**网关
- 远程主机拥有会话存储和工作区
- 迁移你的笔记本电脑**不会**移动远程网关的状态

**解决**：如果使用远程模式，迁移**网关主机**而非客户端。

### 问题：备份中的敏感信息

`$OPENCLAW_STATE_DIR` 包含敏感信息：
- API 密钥
- OAuth 令牌
- WhatsApp 凭据

**最佳实践**：
- 🔒 加密存储备份
- 🚫 避免通过不安全渠道传输
- 🔄 如怀疑泄露，轮换密钥

---

## ✅ 验证清单

在新机器上确认：

| 检查项 | 命令 |
|--------|------|
| 网关运行中 | `openclaw status` |
| 渠道已连接 | `openclaw channels status` |
| 仪表盘正常 | `openclaw dashboard`（检查现有会话） |
| 工作区文件存在 | `ls ~/.openclaw/workspace/` |

### WhatsApp 特别检查

WhatsApp 最容易出问题。如果需要重新登录：

```bash
openclaw channels login
```

扫描二维码重新链接设备。

---

## 📖 相关文档

- [Doctor 命令](/zh-CN/gateway/doctor) - 了解 doctor 的所有功能
- [网关故障排除](/zh-CN/gateway/troubleshooting) - 更多问题解决
- [常见问题](/zh-CN/help/faq) - 数据存储位置等
