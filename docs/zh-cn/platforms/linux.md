---
summary: "Linux 支持 + 伴侣应用状态"
read_when:
  - 查看 Linux 伴侣应用状态
  - 规划平台覆盖或贡献
title: "Linux 应用"
---

# Linux 应用

网关在 Linux 上完全支持。**Node 是推荐的运行时**。
不推荐使用 Bun 运行网关（WhatsApp/Telegram 有 bug）。

原生 Linux 伴侣应用正在计划中。如果你想帮助构建，欢迎贡献。

## 新手快速路径（VPS）

1. 安装 Node 22+
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. 从你的笔记本：`ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. 打开 `http://127.0.0.1:18789/` 并粘贴你的令牌

分步 VPS 指南：[exe.dev](/zh-cn/platforms/exe-dev)

## 安装

- [快速开始](/zh-cn/start/getting-started)
- [安装与更新](/zh-cn/install/updating)
- 可选流程：[Bun（实验性）](/zh-cn/install/bun)、[Nix](/zh-cn/install/nix)、[Docker](/zh-cn/install/docker)

## 网关

- [网关操作手册](/zh-cn/gateway)
- [配置](/zh-cn/gateway/configuration)

## 网关服务安装（CLI）

使用以下任一方式：

```
openclaw onboard --install-daemon
```

或者：

```
openclaw gateway install
```

或者：

```
openclaw configure
```

出现提示时选择 **Gateway service**。

修复/迁移：

```
openclaw doctor
```

## 系统控制（systemd 用户单元）

OpenClaw 默认安装 systemd **用户**服务。对于共享或始终在线的服务器，使用**系统**服务。完整的单元示例和指南在 [网关操作手册](/zh-cn/gateway) 中。

最小设置：

创建 `~/.config/systemd/user/openclaw-gateway[-<profile>].service`：

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

启用它：

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

## 用户服务 vs 系统服务

| 方面 | 用户服务 | 系统服务 |
|------|----------|----------|
| 权限范围 | 当前用户 | 系统级 |
| 启动时机 | 用户登录后 | 系统启动时 |
| 配置位置 | `~/.config/systemd/user/` | `/etc/systemd/system/` |
| 管理命令 | `systemctl --user` | `sudo systemctl` |
| 适用场景 | 个人工作站 | 服务器/共享环境 |

## 常见管理命令

```bash
# 查看服务状态
systemctl --user status openclaw-gateway

# 重启服务
systemctl --user restart openclaw-gateway

# 查看日志
journalctl --user -u openclaw-gateway -f

# 停止服务
systemctl --user stop openclaw-gateway

# 禁用自启动
systemctl --user disable openclaw-gateway
```

## 推荐的 Linux 发行版

| 发行版 | 支持状态 | 备注 |
|--------|----------|------|
| Ubuntu 22.04+ | 完全支持 | 首选，社区测试最多 |
| Debian 12+ | 完全支持 | 稳定可靠 |
| Fedora 38+ | 支持 | 较新的软件包 |
| Arch Linux | 支持 | 滚动更新 |
| Alpine | 部分支持 | Docker 镜像常用 |

## 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| 服务启动失败 | 检查 `journalctl --user -u openclaw-gateway` |
| Node 版本过低 | 使用 nvm 或 n 安装 Node 22+ |
| 端口被占用 | `ss -ltnp \| grep 18789` 查找占用进程 |
| 权限问题 | 确保配置文件所有者正确 |
| systemd 不可用 | 某些容器环境可能不支持，直接运行即可 |
