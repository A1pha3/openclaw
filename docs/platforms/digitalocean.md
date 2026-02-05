---
summary: "在 DigitalOcean 上部署 OpenClaw（简单可靠的 VPS 托管方案）"
read_when:
  - 在 DigitalOcean 上部署 OpenClaw
  - 寻求简单易用的 VPS 托管方案
  - 需要可靠的网络和良好的文档支持
title: "DigitalOcean"
---

# OpenClaw 在 DigitalOcean 平台上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 DigitalOcean 平台的定位和服务特性
- 掌握通过云控制台和 CLI 创建 Droplet 的完整流程
- 能够完成从虚拟机创建到 OpenClaw 运行验证的端到端部署
- 理解 SSH 隧道和 Tailscale 远程访问的配置方法

### 进阶目标（建议掌握）

- 针对 1GB 内存进行性能优化
- 配置 Swap 空间和轻量级模型
- 实现 Tailscale Serve 或 Tailnet Bind 远程访问
- 掌握系统维护和故障排查方法

### 专家目标（挑战）

- 设计基于 DigitalOcean 的成本优化方案
- 构建高可用架构和灾难恢复计划
- 实现自动化运维和监控体系

---

## 核心概念解析

### 为什么选择 DigitalOcean？

DigitalOcean 是专注于开发者和小型团队的云服务平台，其核心优势使其成为部署 OpenClaw 的理想选择：

**简洁至上的设计理念**：DigitalOcean 的设计哲学是「简单」—— 提供直观的用户界面、清晰的文档和完善的 API。相比 AWS、GCP 等复杂的企业云平台，DigitalOcean 让用户可以在几分钟内完成从注册到部署的全流程。

**透明的定价模式**：DigitalOcean 采用固定月费模式，费用清晰可预测：

| Droplet 类型 | CPU | 内存 | 存储 | 月费 |
|-------------|-----|------|-----|------|
| Basic | 1 vCPU | 1GB | 25GB SSD | $6 |
| Basic | 1 vCPU | 2GB | 50GB SSD | $12 |
| Basic | 2 vCPU | 2GB | 60GB SSD | $18 |
| Basic | 2 vCPU | 4GB | 80GB SSD | $24 |

**可靠的基础设施**：
- 自主开发的管理平台和监控系统
- 全球 14 个数据中心
- SSD 存储，高 IOPS 性能
- 99.99% 可用性 SLA

### 与其他平台的对比

| 维度 | DigitalOcean | Oracle Cloud | Hetzner | 自建 Raspberry Pi |
|-----|--------------|--------------|---------|-------------------|
| 入门难度 | 低 | 中 | 低 | 中 |
| 每月成本 | $6+ | $0 (ARM) | €3.79+ | ~$5 (电费) |
| ARM 支持 | 否 | 是 | 否 | 原生 |
| 文档质量 | 高 | 中 | 中 | N/A |
| API 完整性 | 高 | 中 | 中 | N/A |
| 免费层级 | $200/60天 | Always Free | 无 | N/A |

### 架构设计考量

**控制平面与执行平面分离**：与所有云托管方案一样，DigitalOcean 上的 OpenClaw 部署遵循控制平面与执行平面分离的架构。OpenClaw 网关运行在 Droplet 上，而实际的 AI 模型推理通过 API 调用云端服务完成。

**访问模式选择**：

| 访问模式 | 安全性 | 便利性 | 适用场景 |
|---------|-------|-------|---------|
| SSH 隧道 | 高 | 中 | 个人使用 |
| Tailscale Serve | 高 | 高 | 日常使用 |
| Tailnet Bind | 高 | 中 | 开发者访问 |

---

## 快速上手路径

### 十分钟快速部署

```bash
# 步骤 1：创建 Droplet
# 访问 https://cloud.digitalocean.com/
# Create → Droplets
# 选择：Ubuntu 24.04 LTS, Basic $6/mo, SSH Key

# 步骤 2：SSH 连接
ssh root@YOUR_DROPLET_IP

# 步骤 3：安装 OpenClaw
apt update && apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
curl -fsSL https://openclaw.ai/install.sh | bash

# 步骤 4：运行安装向导
openclaw onboard --install-daemon

# 步骤 5：配置远程访问（本地执行）
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# 步骤 6：在浏览器访问
# http://localhost:18789
```

---

## 完整部署流程

### 第一步：账户创建与配置

#### 注册 DigitalOcean 账户

1. 访问 [DigitalOcean 注册页面](https://m.do.co/c/signup)
2. 使用 GitHub 账户快速注册（推荐）
3. 完成邮箱验证和手机验证
4. 添加支付方式（支持信用卡和 PayPal）

#### 创建 SSH 密钥（推荐）

相比密码认证，SSH 密钥更加安全便捷：

```bash
# 本地生成 SSH 密钥（如果尚未创建）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 查看公钥内容
cat ~/.ssh/id_ed25519.pub
```

在 DigitalOcean 控制台添加 SSH 密钥：
1. 进入 Settings → Security
2. 点击「Add SSH Key」
3. 粘贴公钥内容
4. 保存

### 第二步：Droplet 创建与配置

#### 通过控制台创建

1. 登录 [DigitalOcean 控制台](https://cloud.digitalocean.com/)
2. 点击顶部导航的「Create」→「Droplets」
3. 配置以下选项：

| 配置项 | 选择/值 | 说明 |
|-------|--------|------|
| Region | 距离你最近的区域 | 影响网络延迟 |
| Image | Ubuntu 24.04 LTS | 长期支持版本，稳定可靠 |
| Size | Basic $6/mo | 1 vCPU, 1GB RAM, 25GB SSD |
| Authentication | SSH Key | 选择已添加的密钥 |
| Hostname | openclaw-gateway | 便于识别 |

4. 点击「Create Droplet」

#### 通过 doctl CLI 创建

```bash
# 安装 doctl
curl -sL https://github.com/digitalocean/doctl/releases/download/v1.104.0/doctl-1.104.0-linux-amd64.tar.gz | tar -xz
sudo mv doctl /usr/local/bin

# 认证
doctl auth init

# 创建 Droplet
doctl compute droplet create openclaw-gateway \
  --region nyc1 \
  --image ubuntu-24-04-lts \
  --size s-1vcpu-1gb \
  --ssh-keys <your-ssh-key-id>
```

### 第三步：SSH 连接与系统初始化

#### 建立 SSH 连接

```bash
# 使用 IP 地址连接
ssh root@DROPLET_IP

# 或使用主机名（如果配置了 DNS）
ssh root@openclaw-gateway
```

#### 系统配置

```bash
# 更新软件包
apt update && apt upgrade -y

# 安装基础工具
apt install -y curl wget git

# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# 验证安装
node --version
npm --version
```

### 第四步：OpenClaw 安装

```bash
# 执行官方安装脚本
curl -fsSL https://openclaw.ai/install.sh | bash

# 验证安装
openclaw --version
```

### 第五步：安装向导配置

```bash
# 启动安装向导
openclaw onboard --install-daemon
```

安装向导会引导完成以下配置：

1. **模型认证**：选择 API Keys 或 OAuth 模式
2. **即时通讯渠道**：配置 Telegram、WhatsApp、Discord 等
3. **网关 Token**：自动生成或使用现有 Token
4. **后台服务**：配置 systemd 服务实现开机自启

### 第六步：服务验证

```bash
# 检查 OpenClaw 状态
openclaw status

# 检查 systemd 服务状态
systemctl --user status openclaw-gateway.service

# 查看实时日志
journalctl --user -u openclaw-gateway.service -f
```

成功启动的标志：
- `openclaw status` 显示「Running」
- systemd 服务状态为「active (running)」
- 日志中包含 `[gateway] listening on ws://127.0.0.1:18789`

---

## 远程访问配置详解

### 方案一：SSH 隧道（推荐用于开发测试）

SSH 隧道是最简单、最安全的远程访问方式：

```bash
# 在本地机器上执行
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP
```

**工作原理**：

```
本地浏览器 → localhost:18789 → SSH 隧道 → Droplet:18789 → OpenClaw 网关
```

保持隧道连接后，在浏览器访问：`http://localhost:18789`

### 方案二：Tailscale Serve（推荐用于日常使用）

Tailscale 提供更便捷的 HTTPS 访问方式：

#### 安装 Tailscale

```bash
# 在 Droplet 上安装
curl -fsSL https://tailscale.com/install.sh | sh

# 启动并连接
sudo tailscale up
```

#### 配置 Serve

```bash
# 配置 OpenClaw 使用 Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

**访问方式**：打开浏览器访问 `https://<machine-name>.<tailnet-name>.ts.net/`

**Serve 模式特性**：
- 自动 HTTPS 证书
- 通过 Tailscale 身份验证
- 网关仍绑定到本地回环
- 无需开放公网端口

#### Tailnet Bind 模式（替代方案）

如果不想使用 Serve，可以使用 Tailnet Bind：

```bash
# 绑定到 Tailnet 接口
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

访问方式：`http://<tailscale-ip>:18789`（需要 Token 认证）

---

## 1GB 内存优化指南

### 内存压力分析

DigitalOcean 最便宜的 Droplet 只有 1GB 内存。在低内存环境下运行 OpenClaw 需要一些优化措施。

### 优化策略一：配置 Swap 空间

Swap 空间在物理内存不足时提供缓冲：

```bash
# 创建 2GB Swap 文件
fallocate -l 2G /swapfile

# 设置权限（仅 root 可读写）
chmod 600 /swapfile

# 格式化为 Swap
mkswap /swapfile

# 启用 Swap
swapon /swapfile

# 验证状态
swapon --show
free -h

# 永久生效
echo '/swapfile none swap sw 0 0' >> /etc/fstab

# 优化 Swappiness
echo 'vm.swappiness=10' >> /etc/sysctl.conf
sysctl -p
```

### 优化策略二：选择轻量级模型

在配置文件中使用更小的模型：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-haiku-4-20250514",
        "fallbacks": [
          "anthropic/claude-sonnet-4-20250514",
          "openai/gpt-4o-mini"
        ]
      }
    }
  }
}
```

### 优化策略三：监控内存使用

```bash
# 查看内存使用
free -h

# 实时监控
htop

# 检查是否有内存泄漏
docker stats  # 如果使用 Docker
```

---

## 数据持久化与备份

### 持久化存储位置

所有 OpenClaw 数据存储在以下目录：

| 目录 | 内容 |
|-----|------|
| `~/.openclaw/` | 配置文件、凭证、会话数据 |
| `~/.openclaw/workspace/` | 工作区文件、记忆、工具配置 |

### 备份策略

#### 手动备份

```bash
# 创建备份
tar -czvf openclaw-backup-$(date +%Y%m%d).tar.gz ~/.openclaw ~/.openclaw/workspace

# 下载到本地
scp root@DROPLET_IP:/root/openclaw-backup-*.tar.gz .
```

#### 自动化备份

使用 cron 实现每日自动备份：

```bash
# 编辑 crontab
crontab -e

# 添加每日备份任务（凌晨 3 点执行）
0 3 * * * tar -czvf /root/openclaw-backup-$(date +\%Y\%m\%d).tar.gz ~/.openclaw ~/.openclaw/workspace > /dev/null 2>&1
```

---

## 故障排查完全指南

### 症状一：网关无法启动

**诊断步骤**：

```bash
# 检查网关状态
openclaw gateway status

# 运行健康检查
openclaw doctor --non-interactive

# 查看详细日志
journalctl --user -u openclaw --no-pager -n 100
```

**常见原因与解决方案**：

| 原因 | 解决方案 |
|-----|---------|
| 端口被占用 | `lsof -i :18789`，终止占用进程 |
| 配置文件错误 | 检查 `~/.openclaw/openclaw.json` 语法 |
| 权限问题 | 确保 `~/.openclaw` 权限正确 |
| Token 缺失 | 运行 `openclaw doctor --generate-gateway-token` |

### 症状二：端口冲突

```bash
# 检查端口占用
lsof -i :18789

# 查看占用进程
kill <PID>

# 或者使用 fuser
fuser -k 18789/tcp
```

### 症状三：内存不足

**诊断步骤**：

```bash
# 检查内存使用
free -h

# 查看系统日志
dmesg | grep -i oom

# 检查 OpenClaw 进程
ps aux | grep openclaw
```

**解决方案**：

1. 添加 Swap 空间（如前所述）
2. 升级到 $12/mo Droplet（2GB 内存）
3. 禁用不必要的服务
4. 使用更小的模型配置

### 症状四：无法通过 SSH 隧道访问

```bash
# 检查 SSH 连接
ssh -v root@DROPLET_IP

# 检查 Droplet 是否在线
ping DROPLET_IP

# 检查 OpenClaw 是否运行
ssh root@DROPLET_IP "systemctl --user status openclaw-gateway"
```

---

## Oracle Cloud Always Free 替代方案

如果成本是主要考虑因素，Oracle Cloud 提供免费的 ARM 实例：

### Oracle Cloud 优势

| 规格 | DigitalOcean | Oracle Cloud |
|-----|--------------|--------------|
| CPU | 1 vCPU | 2 OCPU (ARM) |
| 内存 | 1GB | 1GB |
| 存储 | 25GB | 47GB |
| 月费 | $6 | $0 |

### 注意事项

- 需要 ARM 版本的支持（OpenClaw 核心支持良好）
- 注册流程可能较为复杂（可能需要多次尝试）
- 某些 x86 专有工具可能不支持

完整指南请参考 [Oracle Cloud 部署文档](/platforms/oracle)。

---

## 适用场景深度分析

### 最适合的场景

**场景一：追求简单可靠**

DigitalOcean 的简洁设计和优秀文档使其成为首选：
- 快速上手，无需复杂配置
- 透明的定价，无隐藏费用
- 完善的 API，支持自动化

**场景二：开发者个人使用**

对于开发者部署个人 AI 助手：
- $6/月成本可接受
- 可以随时销毁重建
- 支持 SSH 密钥认证

**场景二：小型团队协作**

对于小型团队的协作使用：
- 易于管理的界面
- Tailscale 支持便捷访问
- 支持多用户配置

### 需要考虑其他方案的场景

**场景一：追求零成本**

如果完全不想付费：
- Oracle Cloud Always Free ARM（需接受 ARM 限制）
- 自建 Raspberry Pi（一次性硬件成本）

**场景二：需要更高性能**

如果 1GB 内存不够用：
- 升级到 $12-24/月 Droplet
- 考虑 Hetzner €3.79/月（相同价格更多资源）

**场景三：需要 x86 架构**

如果依赖仅支持 x86 的工具：
- Oracle Cloud 提供 x86 选项（但不是免费）
- Hetzner 提供 x86 VPS
- GCP、AWS 提供 x86 实例

---

## 成本对比分析

### 主流 VPS 提供商对比（2026年）

| 提供商 | 基础配置 | 月费 | 年费 | 特点 |
|-------|---------|------|------|------|
| Oracle Cloud | 2 OCPU/1GB | $0 | $0 | 免费但需 ARM |
| Hetzner | 2 vCPU/4GB | €3.79 | €45.48 | 性价比之王 |
| DigitalOcean | 1 vCPU/1GB | $6 | $72 | 简单易用 |
| Vultr | 1 vCPU/1GB | $6 | $72 | 多位置可选 |
| Linode | 1 vCPU/1GB | $5 | $60 | 现属 Akamai |
| GCP | 2 vCPU/2GB | ~$12 | ~$144 | 功能丰富 |

### 成本优化建议

1. **按需升级**：从 1GB 开始，内存不足时再升级
2. **年度支付**：部分提供商提供年度折扣
3. **关注促销**：新用户通常有免费额度
4. **监控使用**：避免意外的费用增长

---

## 专家思维模型：本章总结

### 平台选型决策框架

```
部署需求评估
    │
    ├── 预算考量
    │       ├── $0 ─────────────────────► Oracle Cloud / 自建
    │       │
    │       ├── $3-6/月 ───────────────► Hetzner / Vultr
    │       │
    │       └── $6+/月 ─────────────────► DigitalOcean / GCP
    │
    ├── 运维能力
    │       ├── 新手 ───────────────────► DigitalOcean
    │       │
    │       └── 有经验 ─────────────────► Hetzner / 自建
    │
    └── 特殊需求
            ├── ARM 需求 ───────────────► Oracle Cloud
            │
            ├── x86 需求 ───────────────► Hetzner / DO / GCP
            │
            └── 免费需求 ───────────────► Oracle Cloud Always Free
```

### 架构设计核心原则

1. **最小化攻击面**：SSH 隧道或 Tailscale 访问，不开放公网
2. **数据安全**：定期备份，测试恢复流程
3. **成本意识**：从最小配置开始，按需升级
4. **可观测性**：建立监控和告警机制

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 所有托管方案对比
- [Hetzner Docker 部署](/platforms/hetzner) — 容器化部署方案
- [Oracle Cloud 部署](/platforms/oracle) — 免费 ARM 方案
- [Tailscale 集成](/gateway/tailscale) — 远程访问配置

### 外部资源

- [DigitalOcean 官方文档](https://docs.digitalocean.com/)
- [DigitalOcean Pricing](https://www.digitalocean.com/pricing/)
- [doctl CLI 文档](https://docs.digitalocean.com/reference/doctl/)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 DigitalOcean 平台的定位和优势
- [ ] 理解 SSH 隧道和 Tailscale 访问的工作原理
- [ ] 了解 1GB 内存环境的优化策略

### 动手能力

- [ ] 能够创建和配置 DigitalOcean Droplet
- [ ] 能够完成 OpenClaw 安装和基础配置
- [ ] 能够配置 SSH 隧道和 Tailscale 访问

### 问题解决

- [ ] 能够诊断网关启动失败问题
- [ ] 能够处理端口冲突和内存不足
- [ ] 能够进行系统备份和恢复

### 进阶能力

- [ ] 能够设计成本优化方案
- [ ] 能够构建高可用架构
- [ ] 能够实现自动化运维体系
