---
summary: "在 Hetzner 上部署 OpenClaw（Docker 容器化高性价比方案）"
read_when:
  - 在 Hetzner 上部署 OpenClaw
  - 寻求最高性价比的欧洲 VPS 方案
  - 需要 Docker 容器化部署的完整控制
title: "Hetzner"
---

# OpenClaw 在 Hetzner 平台上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 Hetzner 平台的定位和核心优势
- 掌握通过 Hetzner Cloud Console 创建 VPS 的完整流程
- 能够完成 Docker 环境搭建和容器化部署
- 理解 Docker 容器与主机之间的存储和网络架构

### 进阶目标（建议掌握）

- 配置生产级别的 Docker Compose 环境
- 实现配置文件、二进制依赖和数据的完整持久化
- 掌握 SSH 隧道访问的配置方法
- 针对 Hetzner 网络进行安全加固

### 专家目标（挑战）

- 设计基于 Hetzner 的高可用多服务器架构
- 构建 Docker 镜像的 CI/CD 自动化流水线
- 实现完整的监控、告警和日志聚合体系

---

## 核心概念解析

### 为什么选择 Hetzner？

Hetzner 是总部位于德国的云服务提供商，以其**极高的性价比**和**简洁透明的定价**在开发者社区中享有盛誉：

**价格优势对比（2026年）**：

| 提供商 | 基础配置 | 月费 | 年费 | 每美元获得 |
|-------|---------|------|------|-----------|
| Hetzner | 2 vCPU / 4 GB / 40 GB SSD | €3.79 | €45.48 | 基准 |
| DigitalOcean | 1 vCPU / 1 GB / 25 GB SSD | $6 | $72 | 0.67x |
| Vultr | 1 vCPU / 1 GB / 25 GB SSD | $6 | $72 | 0.67x |
| Linode | 1 vCPU / 1 GB / 25 GB SSD | $5 | $60 | 0.79x |
| Oracle Cloud | 2 OCPU / 1 GB / 200 GB | $0 | $0 | 免费 |

> Hetzner 的 €3.79/月 配置相当于或优于其他平台 $6-12/月的配置

**Hetzner 的核心优势**：

1. **卓越的性价比**：相同价格获得翻倍的资源
2. **德国品质**：企业级硬件和数据中心可靠性
3. **简洁定价**：无隐藏费用，按月付费可随时取消
4. **欧洲位置**：对于欧洲用户或面向欧洲用户的应用有低延迟优势
5. **root 权限完整**：完整的 root 访问权限，无平台限制

### Docker 容器化部署的设计考量

在 Hetzner 上使用 Docker 部署 OpenClaw 涉及以下核心设计决策：

**为什么选择 Docker**：

1. **环境一致性**：开发、测试、生产环境使用相同镜像
2. **依赖隔离**：OpenClaw 及其依赖与主机系统隔离
3. **快速部署**：容器启动只需秒级
4. **易于迁移**：容器镜像可在任何支持 Docker 的平台运行
5. **资源限制**：可以精确控制容器的 CPU 和内存使用

**容器与主机的架构关系**：

```
┌─────────────────────────────────────────────────────────────┐
│                      Hetzner VPS                            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   Docker 运行时                       │   │
│  │                                                      │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │           OpenClaw 网关容器                    │   │   │
│  │  │                                              │   │   │
│  │  │  • /home/node/.openclaw/ ←──┐              │   │   │
│  │  │  • /home/node/.openclaw/workspace ←─┤      │   │   │
│  │  │  • /usr/local/bin/ ←──────────────┤      │   │   │
│  │  │                                   │      │   │   │
│  │  └───────────────────────────────────┼──────┘   │   │
│  │                                      │           │   │
│  └──────────────────────────────────────┼───────────┘   │
│                                         │               │
│              Volume 挂载                │               │
│              SSH 隧道                   ▼               │
│                                         │               │
│  ┌─────────────────────────────────────┴───────────┐   │
│  │              主机文件系统                              │   │
│  │                                                    │   │
│  │  /root/.openclaw/          /root/.openclaw/       │   │
│  │  /root/.openclaw/workspace/                        │   │
│  │                                                    │   │
│  └────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 持久化存储设计原则

Docker 容器中的数据在容器销毁后会丢失，必须通过 Volume 挂载将需要持久化的数据存储到主机文件系统：

| 数据类型 | 容器内路径 | 主机路径 | 持久化机制 | 丢失后果 |
|---------|-----------|---------|-----------|---------|
| 网关配置 | `/home/node/.openclaw/` | `/root/.openclaw/` | Volume | 需重新配置 |
| 凭证信息 | `/home/node/.openclaw/` | `/root/.openclaw/` | Volume | 需重新登录 |
| 工作区 | `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Volume | AI 记忆丢失 |
| 二进制依赖 | `/usr/local/bin/` | 镜像层 | 构建时安装 | 需重新构建镜像 |

---

## 快速上手路径

### 经验者快速部署

```bash
# 步骤 1：注册 Hetzner 账户
# 访问 https://www.hetzner.com/cloud
# 完成注册（需要信用卡或 PayPal）

# 步骤 2：创建项目
# 在 Cloud Console 中创建新项目

# 步骤 3：创建服务器
# Add Server
# Choose: Ubuntu 22.04
# Choose: CX22 (2 vCPU, 4 GB, 40 GB)
# SSH Key: 添加你的公钥
# Enable backups: 可选

# 步骤 4：SSH 连接
ssh root@SERVER_IP

# 步骤 5：安装 Docker
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh

# 步骤 6：克隆 OpenClaw
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 步骤 7：配置持久化
mkdir -p /root/.openclaw /root/.openclaw/workspace
chown -R 1000:1000 /root/.openclaw /root/.openclaw/workspace

# 步骤 8：创建 .env 文件
cat > .env << 'EOF'
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-to-strong-token
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace
GOG_KEYRING_PASSWORD=change-me
XDG_CONFIG_HOME=/home/node/.openclaw
EOF

# 步骤 9：配置 docker-compose.yml
cat > docker-compose.yml << 'EOF'
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      ["node", "dist/index.js", "gateway", "--bind", "lan", "--port", "18789"]
EOF

# 步骤 10：构建并启动
docker compose build
docker compose up -d openclaw-gateway

# 步骤 11：验证
docker compose logs -f openclaw-gateway
```

---

## 完整部署流程

### 第一步：Hetzner 账户创建与配置

#### 注册流程

1. 访问 [Hetzner Cloud 注册页面](https://www.hetzner.com/cloud)
2. 点击「Sign Up」
3. 完成邮箱验证
4. 添加支付方式（信用卡或 PayPal）
5. 完成身份验证

#### 项目创建

1. 登录 [Hetzner Cloud Console](https://console.hetzner.cloud)
2. 点击「New Project」
3. 输入项目名称（如「OpenClaw」）
4. 创建项目

### 第二步：SSH 密钥配置

#### 本地生成 SSH 密钥

```bash
# 生成 ED25519 密钥（推荐）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 查看公钥
cat ~/.ssh/id_ed25519.pub
```

#### 在 Hetzner 控制台添加密钥

1. 进入项目
2. 点击「SSH Keys」
3. 点击「Add SSH Key」
4. 粘贴公钥内容
5. 输入密钥名称
6. 保存

### 第三步：创建服务器

#### 通过控制台创建

1. 进入项目，点击「Add Server」
2. 配置以下选项：

| 配置项 | 选择 | 说明 |
|-------|------|------|
| Location | Falkenstein (欧洲) 或 Nuremberg | 选择靠近用户的位置 |
| Image | Ubuntu 22.04 LTS | 长期支持版本 |
| Type | Cloud | 标准云服务器 |
| Architecture | x86 | 标准配置 |
| Server | CX22 | 2 vCPU / 4 GB / 40 GB SSD |
| Enable backups | 可选 | 月费增加约 20% |

3. 选择已添加的 SSH Key
4. 点击「Create & Buy Now」

#### 通过 hcloud CLI 创建

```bash
# 安装 hcloud
curl -sL https://github.com/hetznercloud/cli/releases/download/v1.38.0/hcloud-linux-amd64.tar.gz | tar -xz
sudo mv hcloud /usr/local/bin

# 认证
hcloud context create openclaw

# 创建服务器
hcloud server create \
  --name openclaw \
  --type cx22 \
  --image ubuntu-22.04 \
  --location fsn1 \
  --ssh-key your-ssh-key-name
```

### 第四步：SSH 连接与系统初始化

#### 建立连接

```bash
# 使用 IP 地址连接
ssh root@SERVER_IP

# 验证主机密钥（首次连接时）
```

#### 系统配置

```bash
# 更新软件包
apt-get update
apt-get upgrade -y

# 安装基础工具
apt-get install -y git curl ca-certificates
```

### 第五步：Docker 环境搭建

#### Docker 安装

```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 验证安装
docker --version
docker compose version
```

### 第六步：OpenClaw 仓库准备

```bash
# 克隆源码
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 第七步：持久化目录配置

```bash
# 创建持久化目录
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# 设置所有权为容器用户（UID 1000）
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

> **为什么设置 UID 1000**：Docker 容器中的 Node.js 默认以 UID 1000 的非 root 用户运行。确保主机目录对该用户有读写权限。

### 第八步：环境变量配置

创建 `.env` 文件：

```bash
cat > .env << 'EOF'
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace
GOG_KEYRING_PASSWORD=$(openssl rand -hex 32)
XDG_CONFIG_HOME=/home/node/.openclaw
EOF

# 重新生成实际 Token
export OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)
export GOG_KEYRING_PASSWORD=$(openssl rand -hex 32)
```

> **安全提醒**：不要将 `.env` 文件提交到版本控制。

### 第九步：Docker Compose 配置

创建 `docker-compose.yml`：

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # 推荐：仅本地访问，通过 SSH 隧道
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

### 第十步：二进制依赖镜像构建

#### Dockerfile 自定义

创建或修改 `Dockerfile`：

```dockerfile
FROM node:22-bookworm

# 安装系统依赖
RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Gmail CLI (gog)
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Google Places CLI (goplaces)
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# WhatsApp CLI (wacli)
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 项目构建
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### 第十一步：构建与启动

```bash
# 构建镜像
docker compose build

# 启动容器
docker compose up -d openclaw-gateway

# 查看状态
docker compose ps

# 实时日志
docker compose logs -f openclaw-gateway
```

#### 验证部署

查看日志确认以下输出：

```
[gateway] listening on ws://0.0.0.0:18789
```

### 第十二步：远程访问配置

由于网关绑定到本地回环，需要通过 SSH 隧道从本地访问：

```bash
# 本地机器上执行
ssh -N -L 18789:127.0.0.1:18789 root@SERVER_IP
```

在浏览器访问：`http://127.0.0.1:18789/`

---

## 数据持久化架构详解

### 持久化全景图

```
┌─────────────────────────────────────────────────────────────────┐
│                      Docker 容器                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   OpenClaw 网关进程                        │  │
│  │                                                            │  │
│  │  /home/node/.openclaw/         /home/node/.openclaw/      │  │
│  │    ├─ openclaw.json              workspace/              │  │
│  │    ├─ credentials/                  ├─ SOUL.md            │  │
│  │    │   ├─ anthropic.json           ├─ memory/            │  │
│  │    │   └─ openai.json              └─ skills/            │  │
│  │    └─ skills/                                             │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Volume 挂载
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Hetzner VPS                               │
│                                                                  │
│  /root/.openclaw/                                               │
│  /root/.openclaw/workspace/                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 持久化组件详细说明

| 组件 | 容器路径 | 主机路径 | 机制 | 重要性 | 丢失后果 |
|-----|---------|---------|------|-------|---------|
| 网关配置 | `/home/node/.openclaw/` | `/root/.openclaw/` | Volume | 高 | 需重新配置 |
| 凭证 | `/home/node/.openclaw/` | `/root/.openclaw/` | Volume | 高 | 需重新登录 |
| 工作区 | `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Volume | 高 | AI 记忆丢失 |
| 二进制 | `/usr/local/bin/` | 镜像层 | 构建时安装 | 中 | 需重新构建 |
| Node 运行时 | 容器文件系统 | 镜像层 | 构建时安装 | 中 | 需重新构建 |

---

## 故障排查完全指南

### 症状一：容器无法启动

**诊断步骤**：

```bash
# 查看容器日志
docker compose logs openclaw-gateway

# 检查容器配置
docker compose config

# 检查端口占用
lsof -i :18789
```

**常见原因与解决方案**：

| 原因 | 解决方案 |
|-----|---------|
| 端口被占用 | 终止占用进程或修改端口 |
| 配置文件错误 | 检查 `docker-compose.yml` 语法 |
| Volume 权限问题 | 确保目录权限正确（chown 1000:1000） |
| 镜像构建失败 | 重新运行 `docker compose build` |

### 症状二：二进制依赖缺失

**诊断步骤**：

```bash
# 检查二进制是否存在
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli

# 检查权限
docker compose exec openclaw-gateway ls -la /usr/local/bin/
```

**解决方案**：

1. 更新 Dockerfile 添加缺失的安装步骤
2. 重新构建镜像
3. 重新部署容器

### 症状三：无法通过 SSH 隧道访问

**诊断步骤**：

```bash
# 检查服务器是否在线
ping SERVER_IP

# 检查 SSH 连接
ssh -v root@SERVER_IP

# 检查容器状态
docker compose ps

# 检查本地端口转发
netstat -tlnp | grep 18789
```

### 症状四：数据未持久化

**诊断步骤**：

```bash
# 检查 Volume 挂载
docker inspect openclaw-openclaw-gateway-1 | grep -A 10 "Mounts"

# 验证主机目录
ls -la /root/.openclaw/
```

**解决方案**：

确保 `docker-compose.yml` 中的 Volume 路径正确，且主机目录存在。

---

## 安全加固指南

### SSH 安全配置

#### 禁用密码认证

```bash
# 编辑 SSH 配置
nano /etc/ssh/sshd_config

# 确保以下设置
PasswordAuthentication no
PermitRootLogin prohibit-block

# 重启 SSH
systemctl restart sshd
```

#### 使用 Fail2Ban（可选）

如果公网 SSH 端口保持开放：

```bash
# 安装 fail2ban
apt-get install -y fail2ban

# 启动服务
systemctl enable fail2ban
systemctl start fail2ban
```

### 防火墙配置

```bash
# 安装 ufw
apt-get install -y ufw

# 设置默认策略
ufw default deny incoming
ufw default allow outgoing

# 允许 SSH（仅你的 IP）
ufw allow from YOUR_IP to any port 22

# 启用防火墙
ufw enable
```

### Hetzner 网络安全组

Hetzner 提供内置的防火墙功能：

1. 进入 Hetzner Cloud Console
2. 选择你的服务器
3. 点击「Firewall」
4. 创建防火墙规则

**推荐规则**：

| 方向 | 协议 | 端口 | 来源 | 描述 |
|-----|------|-----|------|------|
| 入站 | TCP | 22 | YOUR_IP | SSH |
| 入站 | ICMP | - | 0.0.0.0/0 | Ping |

---

## 成本优化策略

### Hetzner 定价结构

| 项目 | 费用 | 说明 |
|-----|------|------|
| CX22 服务器 | €3.49/月 | 2 vCPU, 4 GB RAM, 40 GB SSD |
| IPv4 地址 | €1.90/月 | 如果需要额外 IP |
| 备份 | ~€0.70/月 | 服务器费用的 20% |
| 出站流量 | €0.01/GB | 前 1TB 免费 |

### 成本优化建议

1. **选择合适的服务器**：CX22 对大多数用户足够
2. **使用单一 IP**：默认配置包含一个 IPv4
3. **按需启用备份**：仅在需要时启用
4. **监控流量使用**：避免意外的超额费用

---

## 适用场景深度分析

### 最适合的场景

**场景一：追求最高性价比**

对于预算敏感但需要可靠云托管的用户：
- €3.79/月获得 2 vCPU / 4 GB / 40 GB SSD
- 欧洲位置，适合欧洲用户或目标市场
- 完整的 root 权限，无平台限制

**场景二：开发者个人使用**

对于开发者部署个人 AI 助手：
- 资源充足（4 GB RAM）
- 可以运行多个服务
- Docker 容器化便于实验

**场景三：容器化部署偏好**

对于熟悉 Docker 的用户：
- 完整的容器化控制
- 自定义 Dockerfile
- 灵活的资源配置

### 需要考虑其他方案的场景

**场景一：新手用户**

如果你是云计算新手：
- DigitalOcean 可能更友好（更简单的界面）
- Fly.io 更简化（自动 HTTPS）

**场景二：需要美国位置**

如果你的用户主要在美国：
- Hetzner 服务器在欧洲，可能有延迟
- DigitalOcean/Fly.io 在美国有更多数据中心

**场景三：需要免费方案**

如果完全不想付费：
- Oracle Cloud Always Free ARM（需接受 ARM）
- 自建 Raspberry Pi

---

## 专家思维模型：本章总结

### 平台选型决策框架

```
部署需求评估
    │
    ├── 预算
    │       ├── $0 ─────────────────────► Oracle Cloud / 自建
    │       │
    │       ├── €3-5/月 ───────────────► Hetzner ⭐ 推荐
    │       │
    │       └── $6+/月 ─────────────────► DigitalOcean / Fly.io
    │
    ├── 位置偏好
    │       ├── 欧洲 ───────────────────► Hetzner
    │       │
    │       ├── 美国 ───────────────────► DigitalOcean / Vultr
    │       │
    │       └── 亚洲 ───────────────────► 选择亚洲区域的提供商
    │
    └── 技术偏好
            ├── Docker 容器 ─────────────► Hetzner / GCP
            │
            ├── 托管简化 ────────────────► Fly.io
            │
            └── 简单 VPS ────────────────► DigitalOcean
```

### Docker 容器化最佳实践

1. **构建时安装所有依赖**：运行时安装会丢失
2. **Volume 持久化关键数据**：配置、凭证、工作区
3. **最小化镜像大小**：使用多阶段构建
4. **健康检查配置**：实现自愈能力
5. **日志聚合**：集中管理日志

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 所有托管方案对比
- [Docker 容器化部署](/install/docker) — 通用 Docker 配置
- [GCP Docker 部署](/platforms/gcp) — GCP 容器化方案
- [OpenClaw 网关配置](/gateway/configuration) — 完整配置参考

### 外部资源

- [Hetzner 官方文档](https://docs.hetzner.cloud/)
- [Docker 官方文档](https://docs.docker.com/)
- [Docker Compose 文档](https://docs.docker.com/compose/)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 Hetzner 平台的性价比优势
- [ ] 理解 Docker 容器化部署的架构设计
- [ ] 了解持久化存储的设计原则

### 动手能力

- [ ] 能够创建和配置 Hetzner VPS
- [ ] 能够搭建 Docker 环境并部署 OpenClaw
- [ ] 能够配置 SSH 隧道实现远程访问

### 问题解决

- [ ] 能够诊断容器启动失败问题
- [ ] 能够处理二进制依赖缺失问题
- [ ] 能够解决数据持久化问题

### 进阶能力

- [ ] 能够设计高可用多服务器架构
- [ ] 能够构建 Docker CI/CD 流水线
- [ ] 能够实现完整的监控告警体系
