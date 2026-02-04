---
summary: "`openclaw uninstall` 命令参考（移除网关服务 + 本地数据）"
read_when:
  - 想要移除网关服务和/或本地状态
  - 想要先预演
title: "uninstall"
---

# `openclaw uninstall`

卸载网关服务和本地数据（CLI 保留）。

## 为什么需要这个命令

完全卸载 OpenClaw 时使用：

- **移除服务**：停止并删除后台服务
- **清理数据**：删除所有本地配置和数据
- **干净卸载**：为重新安装准备干净环境

## 基本用法

```bash
# 交互式卸载（会提示确认）
openclaw uninstall

# 卸载所有内容并自动确认
openclaw uninstall --all --yes

# 预演模式
openclaw uninstall --dry-run
```

## 选项

| 选项 | 说明 |
|------|------|
| `--all` | 删除所有本地数据 |
| `--yes` | 跳过确认提示 |
| `--dry-run` | 预演模式，只显示会执行什么 |
| `--keep-config` | 保留配置文件 |
| `--keep-data` | 保留数据文件 |

## 执行的操作

`openclaw uninstall` 会：

1. **停止网关服务**（如果正在运行）
2. **移除服务配置**
   - macOS: 移除 launchd plist
   - Linux: 移除 systemd unit
3. **删除本地数据**（如果指定 `--all`）
   - 配置文件
   - 凭据
   - 会话数据
   - 工作区

## 示例

### 预演查看

```bash
$ openclaw uninstall --dry-run
Would perform:
  - Stop gateway service
  - Remove launchd plist: ~/Library/LaunchAgents/ai.openclaw.gateway.plist
  
With --all would also remove:
  - ~/.openclaw/ (all data)
```

### 只移除服务

```bash
openclaw uninstall --yes
```

服务被移除，但配置和数据保留。

### 完全卸载

```bash
openclaw uninstall --all --yes
```

移除服务和所有本地数据。

### 保留配置

```bash
openclaw uninstall --all --keep-config --yes
```

删除数据但保留配置，便于重新安装。

## 不会删除

- **CLI 程序本身**：需要使用包管理器卸载
  ```bash
  npm uninstall -g openclaw
  # 或
  pnpm remove -g openclaw
  ```
- **macOS 应用**：需要手动删除 OpenClaw.app

## 重新安装

卸载后重新安装：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 服务未停止 | 权限问题 | 手动停止服务 |
| 文件未删除 | 权限或锁定 | 手动删除 `~/.openclaw/` |
| plist 未移除 | launchd 问题 | 手动移除 plist |

## 手动清理

如果自动卸载失败，手动清理：

```bash
# 停止服务 (macOS)
launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.gateway.plist

# 删除 plist
rm ~/Library/LaunchAgents/ai.openclaw.gateway.plist

# 删除数据
rm -rf ~/.openclaw/

# 卸载 CLI
npm uninstall -g openclaw
```
