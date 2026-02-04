---
summary: "完全卸载 OpenClaw：CLI、服务、状态和工作区"
read_when:
  - 想要从机器上移除 OpenClaw
  - 卸载后网关服务仍在运行
title: "卸载指南"
---

# 🗑️ 卸载指南

两种卸载路径：

- **简单路径**：如果 `openclaw` 命令仍可用
- **手动服务移除**：如果 CLI 已卸载但服务仍在运行

---

## ✨ 简单路径（推荐）

如果 `openclaw` 命令仍然可用，使用内置卸载器：

```bash
openclaw uninstall
```

### 完全卸载（非交互式）

适用于自动化或使用 npx：

```bash
openclaw uninstall --all --yes --non-interactive

# 或通过 npx
npx -y openclaw uninstall --all --yes --non-interactive
```

---

## 📝 手动卸载步骤

如果不想使用内置卸载器，可以手动执行以下步骤：

### 1. 停止网关服务

```bash
openclaw gateway stop
```

### 2. 卸载网关服务

这会移除 launchd/systemd/schtasks 配置：

```bash
openclaw gateway uninstall
```

### 3. 删除状态和配置

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

如果你设置了 `OPENCLAW_CONFIG_PATH` 到状态目录外的位置，也删除该文件。

### 4. 删除工作区（可选）

这会删除你的代理文件（记忆、笔记等）：

```bash
rm -rf ~/.openclaw/workspace
```

⚠️ **警告**：工作区包含你的自定义配置和记忆文件，删除前请确认备份。

### 5. 移除 CLI 安装

根据你的安装方式选择：

```bash
# npm 安装
npm rm -g openclaw

# pnpm 安装
pnpm remove -g openclaw

# bun 安装
bun remove -g openclaw
```

### 6. 删除 macOS App（如果安装了）

```bash
rm -rf /Applications/OpenClaw.app
```

---

## ⚠️ 注意事项

### 多 profile 场景

如果你使用了 profile（`--profile` 或 `OPENCLAW_PROFILE`），对每个状态目录重复步骤 3：

```bash
# 默认状态目录
rm -rf ~/.openclaw

# 其他 profile 的状态目录
rm -rf ~/.openclaw-work
rm -rf ~/.openclaw-personal
```

### 远程模式场景

如果使用远程模式，状态目录位于**网关主机**上，需要在那里执行步骤 1-4。

---

## 🔧 手动服务移除（CLI 不可用时）

如果网关服务仍在运行但 `openclaw` 命令不存在，使用以下方法：

### macOS（launchd）

默认服务标签是 `bot.molt.gateway`（或 `bot.molt.<profile>`）：

```bash
# 停止并卸载服务
launchctl bootout gui/$UID/bot.molt.gateway

# 删除配置文件
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

**旧版兼容**：如果存在 `com.openclaw.*` 的 plist 文件，也一并删除：

```bash
rm -f ~/Library/LaunchAgents/com.openclaw.*.plist
```

**多 profile**：将标签和文件名中的 `gateway` 替换为 `<profile>`。

### Linux（systemd 用户服务）

默认服务名是 `openclaw-gateway.service`（或 `openclaw-gateway-<profile>.service`）：

```bash
# 禁用并停止服务
systemctl --user disable --now openclaw-gateway.service

# 删除服务文件
rm -f ~/.config/systemd/user/openclaw-gateway.service

# 重新加载配置
systemctl --user daemon-reload
```

### Windows（计划任务）

默认任务名是 `OpenClaw Gateway`（或 `OpenClaw Gateway (<profile>)`）：

```powershell
# 删除计划任务
schtasks /Delete /F /TN "OpenClaw Gateway"

# 删除启动脚本
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

**多 profile**：删除对应的任务名和 `~\.openclaw-<profile>\gateway.cmd`。

---

## 📋 安装类型对比

### 正常安装（install.sh / npm / pnpm / bun）

如果你使用了 `https://openclaw.ai/install.sh` 或 `install.ps1`，CLI 是通过 `npm install -g openclaw@latest` 安装的。

卸载命令：

```bash
npm rm -g openclaw
# 或
pnpm remove -g openclaw
# 或
bun remove -g openclaw
```

### 源码安装（git clone）

如果你从 repo checkout 运行（`git clone` + `openclaw ...` / `bun run openclaw ...`）：

1. **先卸载网关服务**（使用简单路径或手动服务移除）
2. 删除 repo 目录
3. 按上述步骤删除状态和工作区

---

## ✅ 验证卸载完成

```bash
# CLI 已移除
which openclaw
# 应该无输出

# 服务已停止（macOS）
launchctl list | grep -i openclaw
# 应该无输出

# 服务已停止（Linux）
systemctl --user list-units | grep -i openclaw
# 应该无输出

# 状态目录已删除
ls ~/.openclaw
# 应该报错：No such file or directory
```

---

## 🔄 重新安装

如果将来想重新安装：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

详见 [安装指南](/zh-CN/install)。
