---
summary: "安全更新 OpenClaw：全局安装或源码安装的更新方式，以及回滚策略"
read_when:
  - 更新 OpenClaw
  - 更新后出现问题需要回滚
title: "更新指南"
---

# 🔄 更新指南

OpenClaw 发展迅速（尚未达到"1.0"）。请将更新视为基础设施变更：**更新 → 运行检查 → 重启 → 验证**。

---

## 🚀 推荐：重新运行网站安装器

**首选**的更新方式是重新运行安装器。它会：
- 检测现有安装
- 原地升级
- 需要时运行 `openclaw doctor`

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### 选项说明

| 场景 | 命令 |
|------|------|
| 跳过引导向导 | `curl -fsSL https://openclaw.ai/install.sh \| bash -s -- --no-onboard` |
| 源码安装更新 | `curl -fsSL https://openclaw.ai/install.sh \| bash -s -- --install-method git --no-onboard` |

**源码安装注意**：安装器只在 repo 干净时执行 `git pull --rebase`。

---

## 📋 更新前准备

### 1. 确认安装方式

| 安装方式 | 特征 |
|---------|------|
| **全局安装**（npm/pnpm） | 通过 `npm install -g` 或 `pnpm add -g` 安装 |
| **源码安装**（git clone） | 有 `.git` 目录 |

### 2. 确认网关运行方式

| 运行方式 | 特征 |
|---------|------|
| **前台终端** | 手动运行 `openclaw gateway` |
| **后台服务** | launchd（macOS）/ systemd（Linux）管理 |

### 3. 备份重要文件（推荐）

```bash
# 配置文件
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak

# 凭据（敏感）
cp -r ~/.openclaw/credentials ~/.openclaw/credentials.bak

# 工作区
cp -r ~/.openclaw/workspace ~/.openclaw/workspace.bak
```

---

## 📦 更新方法

### 方法一：全局安装更新

```bash
# npm
npm i -g openclaw@latest

# 或 pnpm
pnpm add -g openclaw@latest
```

⚠️ **不推荐** Bun 作为网关运行时（WhatsApp/Telegram 存在兼容问题）。

### 方法二：切换更新渠道

```bash
# 切换到 beta 渠道
openclaw update --channel beta

# 切换到 dev 渠道（最新开发版）
openclaw update --channel dev

# 切换回 stable 渠道
openclaw update --channel stable
```

详见 [开发渠道](/zh-CN/install/development-channels)。

### 方法三：`openclaw update` 命令

对于**源码安装**（git checkout），推荐使用：

```bash
openclaw update
```

此命令执行安全更新流程：
1. ✅ 要求干净的工作目录
2. ✅ 切换到选定渠道（标签或分支）
3. ✅ Fetch + rebase 到上游（dev 渠道）
4. ✅ 安装依赖、构建、构建 Control UI
5. ✅ 运行 `openclaw doctor`
6. ✅ 默认重启网关（使用 `--no-restart` 跳过）

### 方法四：Control UI / RPC 更新

Control UI 有 **Update & Restart** 功能（RPC: `update.run`）：

1. 运行与 `openclaw update` 相同的源码更新流程
2. 写入重启哨兵，包含结构化报告
3. 重启网关并向最后活跃的会话发送报告

如果 rebase 失败，网关会中止并在不应用更新的情况下重启。

### 方法五：手动源码更新

从 repo checkout 目录：

```bash
git pull
pnpm install
pnpm build
pnpm ui:build  # 首次运行会自动安装 UI 依赖
openclaw doctor
openclaw health
```

---

## 🩺 必须运行：`openclaw doctor`

Doctor 是"安全更新"命令。它执行：修复 + 迁移 + 警告。

**注意**：如果是**源码安装**，`openclaw doctor` 会首先询问是否运行 `openclaw update`。

### Doctor 的功能

| 功能 | 说明 |
|------|------|
| 配置迁移 | 迁移已弃用的配置键和旧配置文件位置 |
| DM 策略审计 | 警告有风险的"open"设置 |
| 网关健康检查 | 检查并可选重启 |
| 服务迁移 | 将旧版网关服务迁移到当前版本 |
| Linux lingering | 确保 systemd 用户 lingering（网关在登出后继续运行） |

详见 [Doctor 命令](/zh-CN/gateway/doctor)。

---

## 🔄 网关启动/停止/重启

```bash
# 查看状态
openclaw gateway status

# 停止网关
openclaw gateway stop

# 重启网关
openclaw gateway restart

# 手动启动（指定端口）
openclaw gateway --port 18789

# 查看日志
openclaw logs --follow
```

### 系统服务管理

**macOS（launchd）**：
```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
```

**Linux（systemd）**：
```bash
systemctl --user restart openclaw-gateway.service
```

**Windows（WSL2）**：
```bash
systemctl --user restart openclaw-gateway.service
```

运行手册和服务标签详情：[网关运行手册](/zh-CN/gateway)

---

## ⏪ 回滚/锁定版本

### 全局安装：锁定特定版本

安装已知可用的版本（将 `<version>` 替换为上一个工作版本）：

```bash
# npm
npm i -g openclaw@<version>

# 或 pnpm
pnpm add -g openclaw@<version>
```

**查看当前发布版本**：
```bash
npm view openclaw version
```

然后重启并运行 doctor：
```bash
openclaw doctor
openclaw gateway restart
```

### 源码安装：按日期锁定

选择特定日期的提交（例如：2026-01-01 的 main 状态）：

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

重新安装依赖并重启：

```bash
pnpm install
pnpm build
openclaw gateway restart
```

**恢复到最新版本**：
```bash
git checkout main
git pull
```

---

## 🔔 更新提示设置

npm 安装的网关会在启动时检查更新并显示提示。

**禁用更新检查**：
```json5
{
  update: {
    checkOnStart: false
  }
}
```

---

## 🆘 遇到问题？

1. **再次运行 `openclaw doctor`** 并仔细阅读输出（它通常会告诉你如何修复）
2. **查看**：[故障排除](/zh-CN/gateway/troubleshooting)
3. **Discord 求助**：https://discord.gg/clawd

---

## 📝 相关文档

- [开发渠道](/zh-CN/install/development-channels) - stable/beta/dev 渠道说明
- [迁移指南](/zh-CN/install/migrating) - 迁移到新机器
- [Doctor 命令](/zh-CN/gateway/doctor) - 详细了解 doctor
- [卸载指南](/zh-CN/install/uninstall) - 完全移除 OpenClaw
