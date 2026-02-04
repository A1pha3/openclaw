---
summary: "OpenClaw 在 DigitalOcean 上（简单的付费 VPS 选项）"
read_when:
  - 在 DigitalOcean 上设置 OpenClaw
  - 寻找 OpenClaw 的廉价 VPS 托管
title: "DigitalOcean"
---

# OpenClaw 在 DigitalOcean 上

## 目标

在 DigitalOcean 上运行持久的 OpenClaw Gateway，费用为 **$6/月**（使用预留定价则为 $4/月）。

如果您想要 $0/月的选项，并且不介意 ARM 和特定提供商的设置，请参阅 [Oracle Cloud 指南](/zh-CN/platforms/oracle)。

## 成本比较（2026 年）

| 提供商       | 方案             | 规格                     | 价格/月     | 说明                                   |
| ------------ | ---------------- | ------------------------ | ----------- | -------------------------------------- |
| Oracle Cloud | Always Free ARM  | 最高 4 OCPU，24GB RAM    | $0          | ARM，容量有限/注册有一些问题           |
| Hetzner      | CX22             | 2 vCPU，4GB RAM          | €3.79 (~$4) | 最便宜的付费选项                       |
| DigitalOcean | Basic            | 1 vCPU，1GB RAM          | $6          | 易用的界面，优秀的文档                 |
| Vultr        | Cloud Compute    | 1 vCPU，1GB RAM          | $6          | 很多地理位置                           |
| Linode       | Nanode           | 1 vCPU，1GB RAM          | $5          | 现在属于 Akamai                        |

**选择提供商：**

- DigitalOcean：最简单的用户体验 + 可预测的设置（本指南）
- Hetzner：良好的性价比（参见 [Hetzner 指南](/zh-CN/platforms/hetzner)）
- Oracle Cloud：可以是 $0/月，但更加挑剔且仅限 ARM（参见 [Oracle 指南](/zh-CN/platforms/oracle)）

---

## 前置要求

- DigitalOcean 账户（[注册可获得 $200 免费额度](https://m.do.co/c/signup)）
- SSH 密钥对（或者愿意使用密码认证）
- 约 20 分钟

## 1) 创建 Droplet

1. 登录 [DigitalOcean](https://cloud.digitalocean.com/)
2. 点击 **Create → Droplets**
3. 选择：
   - **Region：** 离您最近（或您的用户最近）的区域
   - **Image：** Ubuntu 24.04 LTS
   - **Size：** Basic → Regular → **$6/mo**（1 vCPU，1GB RAM，25GB SSD）
   - **Authentication：** SSH 密钥（推荐）或密码
4. 点击 **Create Droplet**
5. 记下 IP 地址

## 2) 通过 SSH 连接

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) 安装 OpenClaw

```bash
# 更新系统
apt update && apt upgrade -y

# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 验证
openclaw --version
```

## 4) 运行入门向导

```bash
openclaw onboard --install-daemon
```

向导将引导您完成：

- 模型认证（API 密钥或 OAuth）
- 频道设置（Telegram、WhatsApp、Discord 等）
- Gateway 令牌（自动生成）
- 守护进程安装（systemd）

## 5) 验证 Gateway

```bash
# 检查状态
openclaw status

# 检查服务
systemctl --user status openclaw-gateway.service

# 查看日志
journalctl --user -u openclaw-gateway.service -f
```

## 6) 访问控制面板

Gateway 默认绑定到 loopback。要访问控制 UI：

**选项 A：SSH 隧道（推荐）**

```bash
# 从本地机器
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# 然后打开：http://localhost:18789
```

**选项 B：Tailscale Serve（HTTPS，仅限 loopback）**

```bash
# 在 droplet 上
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# 配置 Gateway 使用 Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

打开：`https://<magicdns>/`

说明：

- Serve 保持 Gateway 仅限 loopback，并通过 Tailscale 标识头进行认证。
- 如果需要令牌/密码认证，请设置 `gateway.auth.allowTailscale: false` 或使用 `gateway.auth.mode: "password"`。

**选项 C：Tailnet 绑定（不使用 Serve）**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

打开：`http://<tailscale-ip>:18789`（需要令牌）。

## 7) 连接您的频道

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# 扫描二维码
```

有关其他提供商，请参阅 [频道](/zh-CN/channels)。

---

## 1GB RAM 的优化

$6 的 droplet 只有 1GB RAM。为了保持流畅运行：

### 添加交换空间（推荐）

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### 使用更轻量的模型

如果遇到内存不足（OOM），请考虑：

- 使用基于 API 的模型（Claude、GPT）而不是本地模型
- 将 `agents.defaults.model.primary` 设置为更小的模型

### 监控内存

```bash
free -h
htop
```

---

## 持久化

所有状态保存在：

- `~/.openclaw/` — 配置、凭据、会话数据
- `~/.openclaw/workspace/` — 工作区（SOUL.md、内存等）

这些数据在重启后仍然存在。定期备份：

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

---

## Oracle Cloud 免费替代方案

Oracle Cloud 提供 **Always Free** ARM 实例，其性能远超此处的任何付费选项 — 费用为 $0/月。

| 您获得的内容     | 规格                     |
| ---------------- | ------------------------ |
| **4 OCPUs**      | ARM Ampere A1            |
| **24GB RAM**     | 充足                     |
| **200GB 存储**   | 块存储卷                 |
| **永久免费**     | 不收取信用卡费用         |

**注意事项：**

- 注册可能有些挑剔（如果失败请重试）
- ARM 架构 — 大部分功能正常，但某些二进制文件需要 ARM 版本

有关完整的设置指南，请参阅 [Oracle Cloud](/zh-CN/platforms/oracle)。有关注册提示和注册过程的故障排除，请参阅此[社区指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)。

---

## 故障排除

### Gateway 无法启动

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### 端口已被占用

```bash
lsof -i :18789
kill <PID>
```

### 内存不足

```bash
# 检查内存
free -h

# 添加更多交换空间
# 或者升级到 $12/mo 的 droplet（2GB RAM）
```

---

## 另请参阅

- [Hetzner 指南](/zh-CN/platforms/hetzner) — 更便宜、更强大
- [Docker 安装](/zh-CN/install/docker) — 容器化设置
- [Tailscale](/zh-CN/gateway/tailscale) — 安全的远程访问
- [配置](/zh-CN/gateway/configuration) — 完整配置参考
