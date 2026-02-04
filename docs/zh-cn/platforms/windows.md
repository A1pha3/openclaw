---
summary: "Windows (WSL2) 支持 + 伴侣应用状态"
read_when:
  - 在 Windows 上安装 OpenClaw
  - 查看 Windows 伴侣应用状态
title: "Windows (WSL2)"
---

# Windows (WSL2)

OpenClaw 在 Windows 上**推荐通过 WSL2**（建议使用 Ubuntu）运行。CLI 和网关在 Linux 环境内运行，这使运行时保持一致，并且工具兼容性更好（Node/Bun/pnpm、Linux 二进制文件、技能）。原生 Windows 安装未经测试，问题较多。

原生 Windows 伴侣应用正在计划中。

## 安装（WSL2）

- [快速开始](/zh-cn/start/getting-started)（在 WSL 内使用）
- [安装与更新](/zh-cn/install/updating)
- 官方 WSL2 指南（微软）：https://learn.microsoft.com/windows/wsl/install

## 网关

- [网关操作手册](/zh-cn/gateway)
- [配置](/zh-cn/gateway/configuration)

## 网关服务安装（CLI）

在 WSL2 内：

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

## 进阶：通过局域网暴露 WSL 服务（端口代理）

WSL 有自己的虚拟网络。如果其他机器需要访问 **WSL 内**运行的服务（SSH、本地 TTS 服务器或网关），你需要将 Windows 端口转发到当前的 WSL IP。WSL IP 在重启后会改变，因此你可能需要刷新转发规则。

示例（以管理员身份运行 PowerShell）：

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

通过 Windows 防火墙允许端口（一次性操作）：

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL 重启后刷新端口代理：

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

注意事项：

- 从其他机器 SSH 时目标是 **Windows 主机 IP**（例如：`ssh user@windows-host -p 2222`）。
- 远程节点必须指向**可达的**网关 URL（不是 `127.0.0.1`）；使用 `openclaw status --all` 确认。
- 使用 `listenaddress=0.0.0.0` 允许局域网访问；`127.0.0.1` 仅限本地访问。
- 如果你想自动化，可以注册一个计划任务在登录时运行刷新步骤。

## WSL2 安装分步指南

### 1) 安装 WSL2 + Ubuntu

打开 PowerShell（管理员）：

```powershell
wsl --install
# 或者明确选择发行版：
wsl --list --online
wsl --install -d Ubuntu-24.04
```

如果 Windows 要求重启，请重启。

### 2) 启用 systemd（网关安装所需）

在 WSL 终端中：

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

然后在 PowerShell 中：

```powershell
wsl --shutdown
```

重新打开 Ubuntu，然后验证：

```bash
systemctl --user status
```

### 3) 安装 OpenClaw（在 WSL 内）

在 WSL 内按照 Linux 快速开始流程：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行自动安装 UI 依赖
pnpm build
openclaw onboard
```

完整指南：[快速开始](/zh-cn/start/getting-started)

## Windows 伴侣应用

我们目前还没有 Windows 伴侣应用。如果你想帮助实现，欢迎贡献。

## 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| WSL 安装失败 | 确保 Windows 10 版本 2004+ 或 Windows 11，启用虚拟化 |
| systemd 不工作 | 确保 `/etc/wsl.conf` 配置正确并已运行 `wsl --shutdown` |
| 无法从局域网访问 | 检查防火墙规则和端口代理配置 |
| WSL IP 改变了 | 使用计划任务自动刷新端口代理规则 |
| 网关连接超时 | 确认 WSL IP 和端口代理配置匹配 |

## 为什么推荐 WSL2？

| 方面 | WSL2 | 原生 Windows |
|------|------|-------------|
| 工具兼容性 | 完整 Linux 生态 | 部分工具不可用 |
| 运行时一致性 | 与 Linux 服务器一致 | 需要特殊适配 |
| 性能 | 接近原生 Linux | 依赖 Windows 原生实现 |
| 社区支持 | 主要测试环境 | 最少测试覆盖 |
| 技能支持 | 完整支持 | 部分可能不工作 |

WSL2 是在 Windows 上运行 OpenClaw 的最佳方式，它提供了完整的 Linux 环境同时保持 Windows 作为宿主系统的便利性。
