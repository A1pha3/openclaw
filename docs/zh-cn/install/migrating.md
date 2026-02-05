---
summary: "将 OpenClaw 从一台机器迁移到另一台：保留会话、认证和渠道登录状态的完整指南"
read_when:
  - 将 OpenClaw 迁移到新电脑或服务器
  - 想要保留会话记录、认证信息和渠道登录
title: "迁移指南"
---

# 🚚 迁移指南

本文档帮助你将 OpenClaw 网关从一台机器迁移到另一台，**无需重新进行引导配置**。掌握正确的迁移方法可以节省大量配置时间，并保留所有重要数据。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解迁移的核心原理和数据结构
- [ ] 能够执行完整的迁移流程
- [ ] 掌握备份和恢复的最佳实践
- [ ] 理解需要迁移的具体内容

### 进阶目标（建议掌握）

- [ ] 能够处理多 profile 和自定义状态目录
- [ ] 掌握权限问题的诊断和解决
- [ ] 理解远程/本地模式的区别
- [ ] 能够设计团队的迁移检查清单

### 专家目标（挑战）

- [ ] 能够建立自动化的迁移流程
- [ ] 掌握增量迁移和同步策略
- [ ] 理解不同网络环境下的迁移方案
- [ ] 能够制定团队的备份恢复策略

---

## 📖 第一部分：迁移原理

### 1.1 核心原理

迁移的核心很简单：**复制状态目录**。

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

### 1.2 常见的「坑」

| 坑点 | 问题 | 避免方法 |
|------|------|----------|
| 配置文件位置 | 不知道实际使用哪个配置目录 | 迁移前运行 `openclaw status` |
| 权限问题 | 文件权限不正确导致无法读取 | 迁移后运行 `chown` |
| 不完整复制 | 只复制了配置文件 | 复制整个 `.openclaw` 目录 |
| Profile 混淆 | 使用了多个 profile | 确认迁移正确的 profile |

---

## 📋 第二部分：迁移前准备

### 2.1 确认状态目录

大多数安装使用默认位置，但配置方式不同，位置可能不同：

| 配置方式 | 状态目录位置 |
|---------|-------------|
| 默认 | `~/.openclaw/` |
| `--profile <name>` | `~/.openclaw-<profile>/` |
| `OPENCLAW_STATE_DIR=/custom/path` | `/custom/path/` |

**确认命令**：

```bash
# 在旧机器上运行
openclaw status
```

查看输出中的 `OPENCLAW_STATE_DIR` 或 profile 信息。

**多网关注意**：如果你运行多个网关，对每个 profile 重复检查。

### 2.2 确认工作区

常见位置：

| 位置 | 说明 |
|------|------|
| `~/.openclaw/workspace/` | 推荐位置（默认） |
| 自定义目录 | 你创建的任何目录 |

**工作区包含**：`MEMORY.md`、`USER.md`、`AGENTS.md` 和 `memory/*.md` 等文件。

### 2.3 了解迁移内容

**完整迁移（复制状态目录 + 工作区）保留**：

| 内容 | 说明 | 重要性 |
|------|------|--------|
| 网关配置 | `openclaw.json` | ⭐⭐⭐⭐⭐ |
| 认证信息 | API 密钥、OAuth 令牌 | ⭐⭐⭐⭐⭐ |
| 会话历史 | 所有对话记录 | ⭐⭐⭐⭐ |
| 代理状态 | 每个代理的运行状态 | ⭐⭐⭐ |
| 渠道状态 | WhatsApp 登录、Telegram 会话等 | ⭐⭐⭐⭐⭐ |
| 工作区文件 | 记忆、技能笔记等 | ⭐⭐⭐⭐ |

**仅复制工作区（如通过 Git）不保留**：

| 内容 | 说明 |
|------|------|
| ❌ 会话历史 | 存储在 `$OPENCLAW_STATE_DIR/agents/` |
| ❌ 认证凭据 | 存储在 `$OPENCLAW_STATE_DIR/credentials/` |
| ❌ 渠道登录状态 | 存储在各自的凭证目录 |

---

## 🔄 第三部分：迁移步骤

### 3.1 步骤 0：备份（旧机器）

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

### 3.2 步骤 1：在新机器安装 OpenClaw

在**新机器**上安装 CLI（如需要也安装 Node）：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**详细指南**：[安装指南](/zh-CN/install)

**注意**：这时如果引导向导创建了新的 `~/.openclaw/`，没关系——下一步会覆盖它。

### 3.3 步骤 2：复制状态目录 + 工作区

复制以下内容到新机器：

| 内容 | 来源 | 目标 |
|------|------|------|
| 状态目录 | `$OPENCLAW_STATE_DIR`（默认 `~/.openclaw/`） | `~/.openclaw/` |
| 工作区 | 默认 `~/.openclaw/workspace/` | 相同位置 |

**常用方法**：

```bash
# 方法 1：scp 传输归档文件
scp user@old-machine:~/openclaw-state.tgz .
tar -xzf openclaw-state.tgz -C ~

# 方法 2：rsync 同步（保留权限）
rsync -avz user@old-machine:~/.openclaw/ ~/.openclaw/

# 方法 3：外部存储设备
# 复制归档文件到新机器
```

**复制后检查**：

| 检查项 | 命令 | 预期 |
|--------|------|------|
| 隐藏目录包含 | `ls -la ~/.openclaw/` | 显示 `.openclaw` 目录 |
| 文件权限 | `ls -la ~/.openclaw/` | 当前用户有读写权限 |

### 3.4 步骤 3：运行 Doctor（迁移修复）

在**新机器**上：

```bash
openclaw doctor
```

**Doctor 的作用**：

| 功能 | 说明 |
|------|------|
| 修复服务配置 | 重建服务文件 |
| 应用配置迁移 | 更新旧版配置格式 |
| 警告不匹配 | 提示潜在问题 |

**然后启动网关**：

```bash
openclaw gateway restart
openclaw status
```

---

## ⚠️ 第四部分：常见问题与解决

### 4.1 问题：profile/状态目录不匹配

**症状**：

| 症状 | 说明 |
|------|------|
| 配置更改不生效 | 使用了错误的配置文件 |
| 渠道丢失/已登出 | 凭证不在正确位置 |
| 会话历史为空 | 旧的会话数据未复制 |

**原因**：旧网关使用了 profile 或自定义 `OPENCLAW_STATE_DIR`，新网关使用了不同的位置。

**解决**：使用迁移时的**相同** profile/状态目录运行网关：

```bash
# 如果旧机器使用 --profile work
openclaw --profile work gateway restart

# 然后运行 doctor
openclaw --profile work doctor
```

### 4.2 问题：只复制了 openclaw.json

**症状**：渠道登出、会话丢失

**原因**：`openclaw.json` 不够。很多状态存储在：

| 目录 | 存储内容 |
|------|----------|
| `$OPENCLAW_STATE_DIR/credentials/` | 认证信息 |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/...` | 代理数据 |

**解决**：**始终迁移整个** `$OPENCLAW_STATE_DIR` 目录。

### 4.3 问题：权限/所有权错误

**症状**：网关无法读取凭据/会话

**原因**：以 root 复制或更换了用户

**解决**：确保状态目录和工作区归运行网关的用户所有：

```bash
chown -R $(whoami):$(whoami) ~/.openclaw
```

### 4.4 问题：远程/本地模式混淆

**场景**：

| 组件 | 说明 |
|------|------|
| UI（WebUI/TUI） | 连接到**远程**网关 |
| 网关主机 | 拥有会话存储和工作区 |

**注意**：迁移你的笔记本电脑**不会**移动远程网关的状态。

**解决**：如果使用远程模式，迁移**网关主机**而非客户端。

### 4.5 问题：备份中的敏感信息

`$OPENCLAW_STATE_DIR` 包含敏感信息：

| 敏感内容 | 说明 |
|----------|------|
| API 密钥 | 模型访问凭证 |
| OAuth 令牌 | 渠道认证 |
| WhatsApp 凭据 | 会话数据 |

**最佳实践**：

| 实践 | 说明 |
|------|------|
| 🔒 加密存储备份 | 使用 GPG 或类似工具 |
| 🚫 避免不安全渠道传输 | 使用 scp 而不是明文传输 |
| 🔄 如怀疑泄露，轮换密钥 | 重新生成 API 密钥 |

---

## ✅ 第五部分：验证清单

### 5.1 迁移后检查

在新机器上确认：

| 检查项 | 命令 | 预期结果 |
|--------|------|----------|
| 网关运行中 | `openclaw status` | Running |
| 渠道已连接 | `openclaw channels status` | Connected |
| 仪表盘正常 | `openclaw dashboard` | 正常显示现有会话 |
| 工作区文件存在 | `ls ~/.openclaw/workspace/` | 列出所有文件 |

### 5.2 WhatsApp 特别检查

WhatsApp 最容易出问题。如果需要重新登录：

```bash
openclaw channels login
```

扫描二维码重新链接设备。

---

## 🎓 章节总结

### 学习目标完成检查

#### 基础目标（必掌握）

- [ ] 理解迁移的核心原理和数据结构
- [ ] 能够执行完整的迁移流程
- [ ] 掌握备份和恢复的最佳实践
- [ ] 理解需要迁移的具体内容

#### 进阶目标（建议掌握）

- [ ] 能够处理多 profile 和自定义状态目录
- [ ] 掌握权限问题的诊断和解决
- [ ] 理解远程/本地模式的区别
- [ ] 能够设计团队的迁移检查清单

#### 专家目标（挑战）

- [ ] 能够建立自动化的迁移流程
- [ ] 掌握增量迁移和同步策略
- [ ] 理解不同网络环境下的迁移方案
- [ ] 能够制定团队的备份恢复策略

### 迁移流程检查清单

| 步骤 | 操作 | 检查 |
|------|------|------|
| 0 | 停止旧网关 | `openclaw gateway stop` |
| 0 | 创建备份 | `tar -czf openclaw-state.tgz .openclaw` |
| 1 | 新机器安装 CLI | `curl -fsSL https://openclaw.ai/install.sh | bash` |
| 2 | 复制状态目录 | `rsync` 或 `scp` |
| 2 | 复制工作区 | 同上 |
| 3 | 运行 doctor | `openclaw doctor` |
| 3 | 重启网关 | `openclaw gateway restart` |
| 5 | 验证状态 | `openclaw status` |

### 专家建议

| 场景 | 建议 |
|------|------|
| 定期迁移 | 每次迁移前完整备份 |
| 紧急迁移 | 先停止网关，再快速复制 |
| 跨平台迁移 | 注意文件权限（Linux vs macOS） |
| 敏感环境 | 加密传输和存储 |

---

## 📚 相关文档

- [Doctor 命令](/zh-CN/gateway/doctor) - 了解 doctor 的所有功能
- [网关故障排除](/zh-CN/gateway/troubleshooting) - 更多问题解决
- [常见问题](/zh-CN/help/faq) - 数据存储位置等

---

**需要帮助？** 加入 [Discord 社区](https://discord.gg/clawd) 获取支持。
