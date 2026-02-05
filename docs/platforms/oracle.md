---
summary: "在 Oracle Cloud Infrastructure (OCI) 上部署 OpenClaw（永久免费 ARM 方案）"
read_when:
  - 在 Oracle Cloud 上部署 OpenClaw
  - 寻求永久免费的云托管方案
  - 需要强大的 ARM 计算资源
title: "Oracle Cloud"
---

# OpenClaw 在 Oracle Cloud Infrastructure 上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 Oracle Cloud Always Free 层的服务特性和限制
- 掌握通过 OCI 控制台创建 ARM 实例的完整流程
- 能够完成从账户注册到 OpenClaw 运行验证的端到端部署
- 理解 ARM64 架构的兼容性问题及其解决方案

### 进阶目标（建议掌握）

- 配置 Tailscale 实现安全的远程访问
- 实现 VCN 安全列表的最佳安全实践
- 掌握无公网 IP 的安全架构设计
- 针对 ARM 架构进行依赖兼容性配置

### 专家目标（挑战）

- 设计基于 OCI 的高可用多可用域部署
- 构建 ARM 架构应用的持续集成/持续部署流水线
- 实现基于 IAM 的精细权限控制体系

---

## 核心概念解析

### 为什么选择 Oracle Cloud？

Oracle Cloud Infrastructure (OCI) 提供业界唯一的「永久免费」云服务层，使其成为部署 OpenClaw 的独特选择：

**永久免费层的规格**：

| 资源类型 | 免费配额 | 市值估算 |
|---------|---------|---------|
| AMD/ARM 实例 | 2 个 OCPU, 1GB RAM | ~$10-15/月 |
| 块存储 | 200GB | ~$8/月 |
| 对象存储 | 10GB | ~$0.3/月 |
| 入站数据传输 | 每月 10TB | 价值~$50 |
| 出站数据传输 | 每月 10TB | 价值~$50 |

**总计**：永久免费层的综合价值约为 **$70-100/月**

**核心优势**：

1. **真正的永久免费**：不像其他云提供商的「免费试用」，OCI 的免费层在账户存在期间持续有效
2. **强大的 ARM 实例**：VM.Standard.A1.Flex 使用 Ampere Altra 处理器，提供企业级性能
3. **充足的资源**：2 OCPU 和 1GB RAM 对于 OpenClaw 网关来说足够使用
4. **全球数据中心**：多个区域可选，选择靠近用户的区域可降低延迟

### ARM64 架构深度解析

OCI 的免费层主要提供 ARM64 (AArch64) 实例，这带来了一些独特的考量：

**ARM 架构的优势**：

- **能效比高**：ARM 处理器通常比同性能 x86 处理器更省电
- **现代设计**：ARM 服务器处理器采用云计算优化的微架构
- **核心数量多**：免费层提供 2 OCPU，可升级到最多 4 OCPU

**ARM 架构的挑战**：

- **二进制兼容性**：部分仅提供 x86 编译的软件无法直接运行
- **生态成熟度**：某些工具和库的 ARM 支持可能不如 x86 完善
- **开发者工具**：少数闭源工具可能不支持 ARM

### 架构设计考量

**控制平面与执行平面分离**：

```
┌─────────────────────────────────────────────────────────┐
│                    OCI 实例                              │
│  ┌─────────────────────────────────────────────────┐  │
│  │              OpenClaw 网关 (ARM64)               │  │
│  │                                                 │  │
│  │  • 监听本地回环 (127.0.0.1:18789)              │  │
│  │  • 处理渠道连接和消息路由                       │  │
│  │  • 调用云端 AI 模型 API                         │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                        │
                        │ Tailscale 加密隧道
                        ▼
┌─────────────────────────────────────────────────────────┐
│                    Tailscale 网络                        │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────┐  │
│  │   你的电脑    │    │   你的手机   │    │  其他设备 │  │
│  └──────────────┘    └──────────────┘    └──────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 快速上手路径

### 经验者快速部署

```bash
# 步骤 1：创建 OCI 账户
# 访问 https://www.oracle.com/cloud/free/
# 完成注册（可能需要多次尝试，遇到限制可参考社区指南）

# 步骤 2：创建 ARM 实例
# Compute → Instances → Create Instance
# Name: openclaw
# Image: Ubuntu 24.04 (aarch64)
# Shape: VM.Standard.A1.Flex (2 OCPU, 12 GB RAM)
# SSH Key: 添加你的公钥

# 步骤 3：SSH 连接并配置
ssh ubuntu@INSTANCE_PUBLIC_IP

# 步骤 4：系统更新和依赖安装
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential

# 步骤 5：安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw

# 步骤 6：安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 步骤 7：配置网关
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token
openclaw config set gateway.tailscale.mode serve

# 步骤 8：重启服务
systemctl --user restart openclaw-gateway
```

---

## 完整部署流程

### 第一步：Oracle Cloud 账户注册

#### 注册流程

1. 访问 [Oracle Cloud 免费注册页面](https://www.oracle.com/cloud/free/)
2. 选择「Try Oracle Cloud Free Tier」
3. 完成电子邮件验证
4. 添加支付方式（需要信用卡验证，但不会扣费）
5. 完成手机验证码验证

#### 常见注册问题与解决方案

| 问题 | 解决方案 |
|-----|---------|
| 「Out of capacity」错误 | 稍后重试，或尝试不同的可用域 |
| 账户验证失败 | 检查信用卡信息，确保地址匹配 |
|地区限制 | 尝试使用 VPN 或联系支持 |

> **社区资源**：如果遇到注册困难，可参考 [社区注册指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)

### 第二步：创建 ARM 实例

#### 通过 OCI 控制台创建

1. 登录 [OCI 控制台](https://cloud.oracle.com/)
2. 导航到 **Compute → Instances**
3. 点击 **Create Instance**
4. 配置以下参数：

| 配置项 | 推荐值 | 说明 |
|-------|--------|------|
| Name | `openclaw` | 便于识别的名称 |
| Compartment | Root | 使用根目录或创建专用隔室 |
| Image | Ubuntu 24.04 | aarch64 架构 |
| Shape | VM.Standard.A1.Flex | ARM 免费实例 |
| OCPUs | 2 | 免费层上限 |
| Memory | 12 GB | 免费层上限 |
| Boot Volume | 50 GB | 默认 47GB，可扩展到 200GB |
| SSH Keys | 添加你的公钥 | 无密码认证 |

5. 点击 **Create**

#### 实例配置详解

**为什么选择 Ubuntu 24.04**：

- **长期支持**：Ubuntu 24.04 LTS 提供 5 年安全更新
- **ARM 优化**：Canonical 为 ARM 架构提供良好支持
- **软件兼容性**：大多数软件在 Ubuntu 上有最佳支持
- **社区资源**：丰富的故障排查资源和教程

**Shape 选择说明**：

VM.Standard.A1.Flex 是 OCI 的 ARM 实例类型：
- **Ampere Altra 处理器**：云原生设计的 ARM 服务器 CPU
- **灵活的资源配置**：可在 1-4 OCPU 和 6-64 GB RAM 之间调整
- **永久免费**：2 OCPU + 12 GB RAM 组合在免费层内

### 第三步：连接与系统初始化

#### SSH 连接

```bash
# 使用实例公网 IP 连接
ssh ubuntu@INSTANCE_PUBLIC_IP
```

> **连接提示**：首次连接时会显示主机密钥指纹，确认后继续。

#### 系统配置

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装构建工具（某些 npm 包需要编译）
sudo apt install -y build-essential

# 安装其他常用工具
sudo apt install -y curl wget git
```

> **build-essential 重要性**：这是包含 GCC 编译器、C 库等基础开发工具的元包。许多 Node.js 的原生模块（如 bcrypt、sharp）在安装时需要从源码编译，缺少 build-essential 会导致安装失败。

### 第四步：配置用户与主机名

```bash
# 设置主机名
sudo hostnamectl set-hostname openclaw

# 设置 ubuntu 用户密码（用于 Tailscale SSH）
sudo passwd ubuntu

# 启用用户登录保持
sudo loginctl enable-linger ubuntu
```

### 第五步：安装与配置 Tailscale

#### Tailscale 安装

```bash
# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 启动并连接
sudo tailscale up --ssh --hostname=openclaw
```

#### Tailscale 配置说明

| 参数 | 作用 |
|-----|------|
| `--ssh` | 启用 Tailscale SSH，无需公网 IP 即可 SSH |
| `--hostname=openclaw` | 在 Tailscale 网络中的设备名称 |

#### 验证 Tailscale

```bash
# 查看状态
tailscale status

# 确认 SSH 功能已启用
tailscale status | grep -q 'offers: ssh' && echo "SSH enabled"
```

> **后续连接方式**：之后可以通过 `ssh ubuntu@openclaw` 或 Tailscale IP 从任何 Tailscale 设备连接。

### 第六步：OpenClaw 安装

```bash
# 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 加载环境变量
source ~/.bashrc

# 验证安装
openclaw --version
```

#### 安装模式选择

安装过程中会询问「如何孵化你的机器人？」，选择「Do this later」稍后手动配置。

### 第七步：网关配置

#### 基础配置

```bash
# 网关绑定到本地回环（安全最佳实践）
openclaw config set gateway.bind loopback

# 启用 Token 认证
openclaw config set gateway.auth.mode token

# 生成网关 Token
openclaw doctor --generate-gateway-token
```

#### Tailscale Serve 配置

```bash
# 启用 Tailscale Serve（自动 HTTPS + 访问控制）
openclaw config set gateway.tailscale.mode serve

# 配置可信代理
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

# 重启网关服务
systemctl --user restart openclaw-gateway
```

### 第八步：验证部署

```bash
# 检查版本
openclaw --version

# 检查服务状态
systemctl --user status openclaw-gateway

# 检查 Tailscale Serve
tailscale serve status

# 测试本地响应
curl http://localhost:18789
```

---

## VCN 安全配置详解

### 为什么需要锁定 VCN

OCI 的 Virtual Cloud Network (VCN) 是实例的网络边界。默认情况下，VCN 可能开放了不必要的入站端口。锁定 VCN 可以：

- **消除公网攻击面**：阻止来自互联网的随机扫描和攻击
- **零信任网络**：仅允许已知来源的流量
- **合规要求**：满足安全审计和合规要求

### VCN 安全配置步骤

1. 导航到 **Networking → Virtual Cloud Networks**
2. 点击你的 VCN
3. 进入 **Security Lists** → **Default Security List**
4. 删除或修改入站规则

#### 推荐的安全配置

**删除所有入站规则**，仅保留：

| 类型 | 协议 | 端口 | 来源 | 原因 |
|-----|------|-----|------|------|
| Ingress | UDP | 41641 | 0.0.0.0/0 | Tailscale 需要的 WireGuard 端口 |

#### 保留出站规则

默认的出站规则（允许所有出站流量）保持不变。

### 零信任架构

```
┌─────────────────────────────────────────────────────────────┐
│                     OCI VCN 边缘                            │
│                                                              │
│   入站规则：仅 UDP 41641 (Tailscale)                        │
│   出站规则：允许所有                                         │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Ubuntu 主机                               │
│                                                              │
│   防火墙：默认阻止入站                                       │
│   SSH：仅 Tailscale SSH                                     │
│   网关：绑定到 127.0.0.1                                    │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw 网关                            │
│                                                              │
│   认证：Token 认证                                           │
│   访问：仅通过 Tailscale 网络                               │
└─────────────────────────────────────────────────────────────┘
```

### 传统安全措施是否仍需配置

| 传统措施 | 是否必需 | 说明 |
|---------|---------|------|
| UFW 防火墙 | 否 | VCN 在网络边缘阻止流量 |
| fail2ban | 否 | 端口 22 不对公网开放 |
| SSH 强化 | 否 | Tailscale SSH 不使用 sshd |
| 禁用 root 登录 | 否 | 使用 Tailscale 身份而非系统用户 |
| 仅 SSH 密钥 | 否 | Tailscale 使用 tailnet 身份 |

### 仍推荐的安全措施

```bash
# 凭证目录权限
chmod 700 ~/.openclaw

# 安全审计
openclaw security audit

# 系统更新
sudo apt update && sudo apt upgrade
```

### 安全姿态验证

```bash
# 确认无公网端口监听
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# 确认 Tailscale SSH 活跃
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# 可选：完全禁用传统 SSH
sudo systemctl disable --now ssh
```

---

## 远程访问架构

### Tailscale 访问模式

**模式一：Tailscale Serve（推荐）**

```
用户浏览器 → Tailscale Edge → openclaw.ts.net → 网关
    │
    └── 自动 HTTPS 证书
    └── Tailscale 身份认证
```

访问地址：`https://openclaw.<tailnet-name>.ts.net/`

**模式二：Tailscale SSH**

```bash
# 从任何 Tailscale 设备
ssh ubuntu@openclaw
```

**模式三：SSH 隧道（备用）**

```bash
# 如果 Tailscale Serve 有问题
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

### 无公网 IP 的优势

| 方面 | 有公网 IP | 无公网 IP |
|-----|----------|----------|
| 可发现性 | 互联网可扫描 | 仅 tailnet 内可访问 |
| 攻击面 | 大 | 最小 |
| 运维 | 需要防火墙规则 | 无需配置 |
| 成本 | 无额外成本 | 无额外成本 |
| 便利性 | 直接访问 | 需 Tailscale |

---

## ARM 架构兼容性指南

### 二进制兼容性验证

```bash
# 验证系统架构
uname -m

# 应输出：aarch64
```

### 兼容性良好的组件

| 组件 | ARM64 状态 | 说明 |
|-----|-----------|------|
| OpenClaw 核心 | ✅ 原生支持 | TypeScript，无架构依赖 |
| Node.js | ✅ 原生支持 | 官方提供 aarch64 包 |
| npm 包 | ✅ 大部分支持 | 纯 JavaScript 包通用 |
| WhatsApp (Baileys) | ✅ 原生支持 | 纯 JavaScript |
| Telegram | ✅ 原生支持 | 纯 JavaScript |

### 需要注意的组件

| 组件 | ARM64 状态 | 解决方案 |
|-----|-----------|---------|
| gog (Gmail CLI) | ⚠️ 需检查 | 查看是否有 linux-arm64 发布 |
| Chromium 浏览器 | ✅ 可安装 | `sudo apt install chromium-browser` |
| Puppeteer | ⚠️ 需配置 | 确保使用 ARM 兼容版本 |
| Go 二进制 | ⚠️ 需验证 | 检查是否有 arm64 构建 |

### ARM 编译技巧

```bash
# 如果某个工具需要从源码编译
git clone https://github.com/example/tool.git
cd tool

# 检查是否有 ARM 特定的构建说明
cat README.md

# 通常只需标准流程
make
sudo make install
```

---

## 故障排查完全指南

### 症状一：实例创建失败

**现象**：「Out of capacity」或创建超时

**诊断步骤**：

```bash
# 尝试不同的可用域
# OCI 控制台 → Compute → Availability Domains
```

**解决方案**：

1. **尝试不同的可用域**：免费层资源在某些区域可能紧张
2. **稍后重试**：高峰时段可能容量不足
3. **使用「Always Free」过滤器**：确保选择的是免费资源类型

### 症状二：Tailscale 连接失败

```bash
# 检查 Tailscale 状态
sudo tailscale status

# 查看日志
sudo journalctl -u tailscaled

# 重新连接
sudo tailscale up --ssh --hostname=openclaw --reset
```

### 症状三：网关无法启动

```bash
# 检查网关状态
openclaw gateway status

# 运行诊断
openclaw doctor --non-interactive

# 查看日志
journalctl --user -u openclaw-gateway -n 50
```

### 症状四：无法访问控制界面

```bash
# 验证 Tailscale Serve
tailscale serve status

# 检查本地网关
curl http://localhost:18789

# 重启服务
systemctl --user restart openclaw-gateway
```

### 症状五：ARM 二进制兼容性问题

```bash
# 验证架构
uname -m

# 如果输出不是 aarch64，需要重新选择 ARM 实例

# 对于缺失的二进制
# 1. 检查项目是否有 ARM 发布版本
# 2. 尝试从源码编译
# 3. 寻找替代工具
```

---

## 数据持久化与备份

### 持久化位置

所有 OpenClaw 数据存储在：

| 目录 | 内容 |
|-----|------|
| `~/.openclaw/` | 配置、凭证、会话 |
| `~/.openclaw/workspace/` | 工作区文件 |

### 备份策略

```bash
# 创建完整备份
tar -czvf openclaw-backup-$(date +%Y%m%d).tar.gz \
  ~/.openclaw ~/.openclaw/workspace

# 下载到本地
scp ubuntu@openclaw:/home/ubuntu/openclaw-backup-*.tar.gz .
```

---

## 适用场景深度分析

### 最适合的场景

**场景一：零成本运行**

对于预算有限但需要可靠云托管的用户：
- 永久免费，无需任何费用
- 2 OCPU + 12 GB RAM 配置足够运行 OpenClaw
- 200 GB 存储空间充裕

**场景二：开发者实验**

开发者可以使用免费资源进行实验和测试：
- 无成本试错
- 可以随时销毁重建
- 学习云原生部署技能

**场景三：长期个人使用**

对于计划长期运行个人 AI 助手的用户：
- 无月度费用担忧
- 资源足够个人使用
- 随时可以升级到付费层

### 需要考虑其他方案的场景

**场景一：新手用户**

如果你是云计算新手：
- OCI 的界面和概念相对复杂
- 注册流程可能遇到问题
- DigitalOcean 可能更友好

**场景二：需要 x86 工具**

如果依赖仅支持 x86 的特定工具：
- Oracle Cloud 也提供 x86 实例（但不是免费）
- 考虑 Hetzner、DigitalOcean 等

**场景三：追求极致简单**

如果希望最小化运维复杂度：
- Fly.io 提供更简化的部署体验
- 自建 Raspberry Pi 无需云账户管理

---

## 成本深度分析

### OCI 免费层 vs 其他方案

| 维度 | OCI Always Free | Hetzner CX22 | DigitalOcean |
|-----|----------------|--------------|--------------|
| 月费 | $0 | €3.79 (~$4) | $6 |
| CPU | 2 OCPU ARM | 2 vCPU x86 | 1 vCPU x86 |
| 内存 | 12 GB | 4 GB | 1 GB |
| 存储 | 200 GB | 40 GB SSD | 25 GB SSD |
| 公网 IP | 需付费添加 | 包含 | 包含 |
| 总价值/月 | ~$70 | ~$8 | ~$8 |

### 升级成本（如果需要）

如果免费层不够用，可以：

| 升级选项 | 成本 | 配置 |
|---------|------|------|
| 增加 ARM 资源 | $0.01/小时/OCPU | 最多再增加 2 OCPU |
| 添加公网 IP | ~$3/月 | 需要时 |
| x86 实例 | 按需计费 | 不是免费 |

---

## 专家思维模型：本章总结

### 零成本云部署决策

```
部署需求
    │
    ├── 是否愿意接受 ARM 架构？
    │       ├── 是 ─────► OCI Always Free ⭐ 推荐
    │       │
    │       └── 否 ─────► 考虑其他方案
    │                   ├── 预算极低 → Hetzner €3.79/月
    │                   └── 愿意付费 → DigitalOcean $6/月
    │
    └── 是否需要 x86 专用工具？
            ├── 是 ─────► 不适合 OCI
            └── 否 ─────► OCI 是最佳选择
```

### 安全架构设计原则

1. **网络隔离**：VCN + Tailscale 实现零公网暴露
2. **身份认证**：Token + Tailscale 身份双因素
3. **最小权限**：仅开放必要端口和服务
4. **纵深防御**：多层安全机制

### ARM 架构拥抱策略

1. **优先选择跨平台工具**
2. **验证关键依赖的 ARM 支持**
3. **建立 ARM 编译能力**
4. **保持对 x86 方案的了解（备选）**

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 所有托管方案对比
- [Tailscale 集成](/gateway/tailscale) — 远程访问配置
- [Gateway 远程访问](/gateway/remote) — 其他远程访问模式
- [DigitalOcean 部署](/platforms/digitalocean) — 付费替代方案

### 外部资源

- [Oracle Cloud 文档](https://docs.oracle.com/)
- [OCI Always Free 常见问题](https://www.oracle.com/cloud/free/faq/)
- [Tailscale 文档](https://tailscale.com/docs/)
- [社区注册指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 OCI Always Free 层的服务特性和限制
- [ ] 理解 ARM64 架构的优势和挑战
- [ ] 了解 VCN + Tailscale 的零公网安全架构

### 动手能力

- [ ] 能够完成 OCI 账户注册和 ARM 实例创建
- [ ] 能够配置 Tailscale 实现安全的远程访问
- [ ] 能够完成 OpenClaw 安装和基础配置

### 问题解决

- [ ] 能够诊断实例创建失败问题
- [ ] 能够处理 Tailscale 连接问题
- [ ] 能够解决 ARM 二进制兼容性问题

### 进阶能力

- [ ] 能够设计多可用域高可用架构
- [ ] 能够构建 ARM 应用的 CI/CD 流水线
- [ ] 能够实现基于 IAM 的精细权限控制
