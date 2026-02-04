---
summary: "在沙箱化 macOS 虚拟机中运行 OpenClaw（本地或托管），需要隔离或 iMessage 时使用"
read_when:
  - 你想将 OpenClaw 与主 macOS 环境隔离
  - 你想在沙箱中实现 iMessage 集成（BlueBubbles）
  - 你想要一个可重置、可克隆的 macOS 环境
  - 你想比较本地和托管 macOS VM 选项
title: "macOS 虚拟机"
---

# 在 macOS 虚拟机上运行 OpenClaw（沙箱化）

## 推荐默认方案（大多数用户）

- **小型 Linux VPS** 用于始终在线的网关，成本低。参见 [VPS 托管](/zh-cn/vps)。
- **专用硬件**（Mac mini 或 Linux 机器）如果你想完全控制并拥有**住宅 IP** 用于浏览器自动化。许多网站会阻止数据中心 IP，所以本地浏览通常效果更好。
- **混合方案：** 将网关放在便宜的 VPS 上，当需要浏览器/UI 自动化时将 Mac 作为**节点**连接。参见 [节点](/zh-cn/nodes) 和 [网关远程](/zh-cn/gateway/remote)。

当你特别需要 macOS 专有功能（iMessage/BlueBubbles）或想与日常使用的 Mac 严格隔离时，使用 macOS VM。

## macOS VM 选项

### 在 Apple Silicon Mac 上运行本地 VM（Lume）

使用 [Lume](https://cua.ai/docs/lume) 在现有 Apple Silicon Mac 上的沙箱化 macOS VM 中运行 OpenClaw。

这给你：

- 完整隔离的 macOS 环境（宿主机保持干净）
- 通过 BlueBubbles 支持 iMessage（Linux/Windows 上不可能）
- 通过克隆 VM 即时重置
- 无需额外硬件或云成本

### 托管 Mac 提供商（云）

如果你想在云中使用 macOS，托管 Mac 提供商也可以：

- [MacStadium](https://www.macstadium.com/)（托管 Mac）
- 其他托管 Mac 供应商也可以；按照他们的 VM + SSH 文档操作

一旦你有了 macOS VM 的 SSH 访问，继续下面的第 6 步。

---

## 快速路径（Lume，有经验的用户）

1. 安装 Lume
2. `lume create openclaw --os macos --ipsw latest`
3. 完成设置助手，启用远程登录（SSH）
4. `lume run openclaw --no-display`
5. SSH 进入，安装 OpenClaw，配置频道
6. 完成

---

## 你需要什么（Lume）

- Apple Silicon Mac（M1/M2/M3/M4）
- 宿主机上运行 macOS Sequoia 或更高版本
- 每个 VM 约 60 GB 可用磁盘空间
- 约 20 分钟

---

## 1) 安装 Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

如果 `~/.local/bin` 不在你的 PATH 中：

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

验证：

```bash
lume --version
```

文档：[Lume 安装](https://cua.ai/docs/lume/guide/getting-started/installation)

---

## 2) 创建 macOS VM

```bash
lume create openclaw --os macos --ipsw latest
```

这会下载 macOS 并创建 VM。VNC 窗口会自动打开。

注意：下载可能需要一些时间，取决于你的网络连接。

---

## 3) 完成设置助手

在 VNC 窗口中：

1. 选择语言和地区
2. 跳过 Apple ID（如果之后想用 iMessage 可以登录）
3. 创建用户账户（记住用户名和密码）
4. 跳过所有可选功能

设置完成后，启用 SSH：

1. 打开 系统设置 → 通用 → 共享
2. 启用"远程登录"

---

## 4) 获取 VM 的 IP 地址

```bash
lume get openclaw
```

查找 IP 地址（通常是 `192.168.64.x`）。

---

## 5) SSH 进入 VM

```bash
ssh youruser@192.168.64.X
```

将 `youruser` 替换为你创建的账户，IP 替换为你 VM 的 IP。

---

## 6) 安装 OpenClaw

在 VM 内：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

按照引导提示设置你的模型提供商（Anthropic、OpenAI 等）。

---

## 7) 配置频道

编辑配置文件：

```bash
nano ~/.openclaw/openclaw.json
```

添加你的频道：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

然后登录 WhatsApp（扫描二维码）：

```bash
openclaw channels login
```

---

## 8) 无头运行 VM

停止 VM 并在无显示模式下重启：

```bash
lume stop openclaw
lume run openclaw --no-display
```

VM 在后台运行。OpenClaw 的守护进程保持网关运行。

检查状态：

```bash
ssh youruser@192.168.64.X "openclaw status"
```

---

## 额外功能：iMessage 集成

这是在 macOS 上运行的杀手级功能。使用 [BlueBubbles](https://bluebubbles.app) 将 iMessage 添加到 OpenClaw。

在 VM 内：

1. 从 bluebubbles.app 下载 BlueBubbles
2. 使用你的 Apple ID 登录
3. 启用 Web API 并设置密码
4. 将 BlueBubbles webhooks 指向你的网关（例如：`https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）

添加到你的 OpenClaw 配置：

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

重启网关。现在你的助手可以发送和接收 iMessage。

完整设置详情：[BlueBubbles 频道](/zh-cn/channels/bluebubbles)

---

## 保存黄金镜像

在进一步自定义之前，快照你的干净状态：

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

随时重置：

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

---

## 7×24 运行

保持 VM 运行通过：

- 保持 Mac 插电
- 在 系统设置 → 节能 中禁用睡眠
- 如需要使用 `caffeinate`

对于真正的始终在线，考虑专用 Mac mini 或小型 VPS。参见 [VPS 托管](/zh-cn/vps)。

---

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 无法 SSH 进入 VM | 检查 VM 系统设置中是否启用了"远程登录" |
| VM IP 不显示 | 等待 VM 完全启动，再次运行 `lume get openclaw` |
| 找不到 Lume 命令 | 将 `~/.local/bin` 添加到 PATH |
| WhatsApp 二维码扫描失败 | 确保运行 `openclaw channels login` 时在 VM 中（不是宿主机） |
| VM 性能差 | 确保宿主机有足够的 RAM 和磁盘空间 |
| 网络连接问题 | 检查宿主机防火墙设置 |

---

## 方案对比

| 方面 | 本地 VM (Lume) | 托管 Mac | Linux VPS |
|------|---------------|----------|-----------|
| 成本 | 无额外成本 | 月费 | 低月费 |
| iMessage 支持 | ✅ | ✅ | ❌ |
| 隔离性 | 高 | 高 | 高 |
| 始终在线 | 需要宿主机开启 | ✅ | ✅ |
| 住宅 IP | ✅ | ❌ | ❌ |
| 设置复杂度 | 中 | 低 | 低 |

---

## 相关文档

- [VPS 托管](/zh-cn/vps)
- [节点](/zh-cn/nodes)
- [网关远程](/zh-cn/gateway/remote)
- [BlueBubbles 频道](/zh-cn/channels/bluebubbles)
- [Lume 快速开始](https://cua.ai/docs/lume/guide/getting-started/quickstart)
- [Lume CLI 参考](https://cua.ai/docs/lume/reference/cli-reference)
- [无人值守 VM 设置](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup)（高级）
- [Docker 沙箱化](/zh-cn/install/docker)（替代隔离方案）
