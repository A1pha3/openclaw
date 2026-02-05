---
summary: "在 Google Cloud Platform (GCP) 上部署 OpenClaw（生产级虚拟机部署方案）"
read_when:
  - 在 GCP Compute Engine 上部署 OpenClaw
  - 寻求生产级别、高可用的云端部署方案
  - 需要完整控制持久化、依赖和重启行为
title: "GCP"
---

# OpenClaw 在 Google Cloud Platform 上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 GCP Compute Engine 的核心服务特性和计费模式
- 掌握通过 gcloud CLI 工具管理 GCP 虚拟机的完整流程
- 能够完成从项目创建到 OpenClaw 运行的端到端部署
- 理解 Docker 容器化部署的基本概念和优势

### 进阶目标（建议掌握）

- 配置生产级别的 Docker Compose 环境
- 实现配置文件、凭证和工作区的持久化存储
- 掌握二进制依赖的镜像构建技术
- 配置 SSH 隧道实现安全的远程访问

### 专家目标（挑战）

- 设计基于 GCP 的多区域高可用部署架构
- 构建基于 Cloud IAM 的精细权限控制体系
- 实现自动化运维和监控告警体系

---

## 核心概念解析

### 为什么选择 Google Cloud Platform？

Google Cloud Platform 是全球领先的云服务提供商之一，其基础设施和服务对于部署 OpenClaw 网关具有以下优势：

**全球网络基础设施**：GCP 拥有覆盖全球的数据中心网络，选择靠近用户群体的区域可以提供低延迟的网络体验。此外，GCP 的内部网络采用 Jupiter 光纤互连，提供极高的带宽和可靠性。

**成熟的生态系统**：作为业界领先的云平台，GCP 提供了丰富的配套服务：
- Cloud Logging：集中式日志管理
- Cloud Monitoring：全面的监控和告警
- Cloud IAM：精细的权限控制
- Cloud Storage：可靠的持久化存储

**按需付费的灵活性**：GCP 采用按秒计费模式，仅为实际使用的资源付费。这使得：
- 小规模部署可以控制成本
- 可以根据需求动态调整资源
- 无需预付大额费用

### Docker 容器化部署的设计考量

在 GCP 上使用 Docker 部署 OpenClaw 涉及以下核心设计决策：

**容器与虚拟机的对比**：

| 维度 | 传统虚拟机 | Docker 容器 |
|-----|----------|------------|
| 启动速度 | 分钟级 | 秒级 |
| 资源隔离 | 硬件级 | 进程级 |
| 操作系统 | 完整 OS | 共享内核 |
| 镜像大小 | GB 级 | MB 级 |
| 一致性 | 依赖配置管理 | 镜像即环境 |
| 升级方式 | 原地更新 | 替换容器 |

**为什么选择容器化**：

1. **环境一致性**：开发、测试、生产环境使用完全相同的镜像，消除「在我机器上是好的」问题
2. **依赖隔离**：OpenClaw 及其所有依赖打包在容器中，不与主机系统产生冲突
3. **快速恢复**：容器崩溃后可以秒级重启
4. **简化部署**：通过镜像版本管理实现可重复的部署流程

### 持久化存储的设计原则

Docker 容器本质上是**临时性**的，任何写入容器文件系统的数据在容器销毁后都会丢失。因此，必须将需要持久化的数据挂载到主机或网络存储上。

**OpenClaw 需要持久化的数据**：

| 数据类型 | 存储位置 | 重要性 | 丢失后果 |
|---------|---------|-------|---------|
| 网关配置 | `~/.openclaw/` | 高 | 需要重新配置所有设置 |
| 模型认证 | `~/.openclaw/` | 高 | 需要重新登录所有模型提供商 |
| 技能配置 | `~/.openclaw/skills/` | 中 | 技能设置丢失 |
| 工作区 | `~/.openclaw/workspace/` | 高 | AI 助手的记忆和上下文丢失 |
| WhatsApp 会话 | `~/.openclaw/` | 高 | 需要重新扫码登录 |
| Gmail 密钥环 | `~/.openclaw/` | 高 | 需要重新配置 Gmail 访问 |

---

## 快速上手路径

### 经验者快速部署

```bash
# 步骤 1：创建 GCP 项目并启用 Compute Engine API
gcloud projects create my-openclaw --name="OpenClaw Gateway"
gcloud config set project my-openclaw
gcloud services enable compute.googleapis.com

# 步骤 2：创建 Compute Engine 虚拟机
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud

# 步骤 3：SSH 连接并安装 Docker
gcloud compute ssh openclaw-gateway --zone=us-central1-a
curl -fsSL https://get.docker.com | sudo sh

# 步骤 4：克隆 OpenClaw 并构建镜像
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 步骤 5：配置持久化存储和 Docker Compose
mkdir -p ~/.openclaw ~/.openclaw/workspace
# 配置 .env 和 docker-compose.yml

# 步骤 6：构建并启动容器
docker compose build
docker compose up -d openclaw-gateway
```

---

## 完整部署流程

### 第一步：gcloud CLI 环境准备

#### 安装与初始化

gcloud CLI 是管理 GCP 资源的官方命令行工具：

```bash
# macOS 安装
brew install google-cloud-sdk

# Linux 安装
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Windows 安装
# 下载安装程序：https://cloud.google.com/sdk/docs/install

# 初始化（打开浏览器进行身份验证）
gcloud init

# 或使用服务账户进行非交互式认证
gcloud auth activate-service-account --key-file=/path/to/key.json
```

#### 验证安装

```bash
# 检查版本
gcloud version

# 确认当前配置
gcloud config list

# 列出可用的计算区域
gcloud compute zones list
```

### 第二步：GCP 项目创建与配置

#### 创建新项目

```bash
# 创建项目
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"

# 设置当前项目
gcloud config set project my-openclaw-project
```

> **项目命名规范**：建议使用描述性名称和项目 ID，如 `my-openclaw`、`openclaw-production` 等，便于在多个环境间区分。

#### 启用必要的 API

部署 Compute Engine 虚拟机需要启用以下 API：

```bash
# 启用 Compute Engine API（必需）
gcloud services enable compute.googleapis.com

# 启用其他可能需要的 API
gcloud services enable container.googleapis.com    # 如果使用 GKE
gcloud services enable secretmanager.googleapis.com # 如果使用 Secret Manager
```

> **API 启用提示**：首次启用 API 可能需要几分钟时间，GCP 会自动处理依赖关系。

#### 配置计费

重要：Compute Engine 虚拟机需要关联计费账户才能创建。

1. 访问 [GCP Console Billing](https://console.cloud.google.com/billing)
2. 创建或选择计费账户
3. 将计费账户链接到项目

### 第三步：虚拟机实例创建

#### 机器类型选择

| 类型 | CPU | 内存 | 月成本（预估） | 适用场景 |
|-----|-----|------|---------------|---------|
| e2-micro | 2 共享 vCPU | 1GB | 免费层级 | 测试环境 |
| e2-small | 2 vCPU | 2GB | ~$12/月 | 生产环境推荐 |
| e2-medium | 2 vCPU | 4GB | ~$24/月 | 高负载场景 |
| n2-standard-2 | 2 vCPU | 8GB | ~$50/月 | 内存密集型 |

> **选择建议**：对于 OpenClaw 网关，e2-small 通常足够。如果遇到 OOM（内存不足），可以升级到 e2-medium。

#### 创建虚拟机

```bash
# 使用 gcloud CLI 创建
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-ssd \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --service-account=default \
  --scopes=cloud-platform
```

#### 配置参数详解

| 参数 | 说明 | 推荐值 |
|-----|------|-------|
| `--zone` | 计算区域 | 选择靠近用户的位置 |
| `--machine-type` | 机器规格 | e2-small（推荐）或 e2-medium |
| `--boot-disk-size` | 启动磁盘大小 | 20GB 起步 |
| `--boot-disk-type` | 启动磁盘类型 | pd-ssd（SSD 更快） |
| `--image-family` | 操作系统 | debian-12 或 ubuntu-22.04-lts |
| `--service-account` | 服务账户 | default 或自定义 |
| `--scopes` | API 访问范围 | cloud-platform |

> **区域选择**：对于中国用户，可选择 `asia-east1`（台湾）、`asia-northeast1`（东京）或 `asia-south1`（孟买）等亚洲区域。

#### 通过 GCP Console 创建（可选）

如果不习惯命令行，可以通过 Web 控制台创建：

1. 访问 [Compute Engine Instances](https://console.cloud.google.com/compute/instances)
2. 点击「Create Instance」
3. 配置各项参数
4. 点击「Create」

### 第四步：SSH 连接与初始配置

#### SSH 连接

```bash
# 使用 gcloud SSH（推荐，自动处理密钥）
gcloud compute ssh openclaw-gateway --zone=us-central1-a

# 或使用传统的 SSH 命令（需要配置 SSH 密钥）
ssh -i ~/.ssh/google_compute_engine username@external-ip
```

> **SSH 密钥提示**：首次连接时，gcloud 会自动生成 SSH 密钥并添加到项目元数据。后续连接无需额外配置。

#### 系统初始化

```bash
# 更新软件包
sudo apt-get update
sudo apt-get upgrade -y

# 安装基础工具
sudo apt-get install -y git curl ca-certificates

# 安装 Docker
curl -fsSL https://get.docker.com | sudo sh

# 将当前用户添加到 docker 组（免 sudo 运行 docker）
sudo usermod -aG docker $USER

# 退出并重新登录使组权限生效
exit
```

#### 重新连接

```bash
# 重新 SSH 连接
gcloud compute ssh openclaw-gateway --zone=us-central1-a

# 验证 Docker 安装
docker --version
docker compose version
```

### 第五步：OpenClaw 仓库准备

```bash
# 克隆 OpenClaw 源码
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 第六步：持久化目录配置

Docker 容器中的数据在容器销毁后会丢失，因此必须将需要持久化的数据挂载到主机文件系统：

```bash
# 创建持久化目录
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace

# 确保目录权限正确
chmod -R 700 ~/.openclaw
```

### 第七步：环境变量配置

创建 `.env` 文件存储敏感配置：

```bash
cat > .env << 'EOF'
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-to-strong-random-token
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-to-strong-password
XDG_CONFIG_HOME=/home/node/.openclaw
EOF
```

#### 生成强随机 Token

```bash
# 生成 32 字节十六进制随机字符串（约 64 字符）
openssl rand -hex 32
```

> **安全提示**：不要将 `.env` 文件提交到版本控制系统。建议将此文件添加到 `.gitignore`。

### 第八步：Docker Compose 配置

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
      # 推荐：仅绑定到本地回环，通过 SSH 隧道访问
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # 可选：如果需要公网访问（需配置防火墙）
      # - "0.0.0.0:${OPENCLAW_GATEWAY_PORT}:18789"

      # 可选：Canvas 主机端口（仅当需要 iOS/Android 节点时）
      # - "18793:18793"
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

#### 配置说明

**网络配置**：
- `127.0.0.1:18789:18789`：仅允许本地访问，强制通过 SSH 隧道访问
- `0.0.0.0:18789:18789`：允许公网访问（需配置 GCP 防火墙规则）

**卷挂载**：
- 配置文件挂载到容器内的 `/home/node/.openclaw/`
- 工作区挂载到容器内的 `/home/node/.openclaw/workspace/`

### 第九步：二进制依赖镜像构建

**核心原则**：所有外部二进制依赖必须在镜像构建时安装，运行时安装的任何内容都会在容器重启后丢失。

#### 为什么必须在构建时安装

Docker 容器使用分层文件系统：
- 构建时写入的层被固化在镜像中
- 运行时写入的层在容器销毁后丢失
- Volume 挂载仅覆盖特定路径

因此，必须通过 Dockerfile 在构建时安装所有依赖。

#### Dockerfile 示例

```dockerfile
FROM node:22-bookworm

# 安装系统依赖
RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 安装 Gmail CLI (gog)
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 安装 Google Places CLI (goplaces)
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 安装 WhatsApp CLI (wacli)
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 复制项目文件
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

# 启用 pnpm
RUN corepack enable
RUN pnpm install --frozen-lockfile

# 构建项目
COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### 第十步：构建与启动

```bash
# 构建 Docker 镜像
docker compose build

# 启动容器
docker compose up -d openclaw-gateway

# 验证容器运行状态
docker compose ps

# 查看日志
docker compose logs -f openclaw-gateway
```

#### 验证二进制依赖

```bash
# 检查二进制文件是否存在
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

预期输出：
```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

#### 验证网关运行

查看日志，确认看到以下输出：
```
[gateway] listening on ws://0.0.0.0:18789
```

### 第十一步：远程访问配置

由于 OpenClaw 网关仅绑定到本地回环（127.0.0.1），需要通过 SSH 隧道从本地机器访问：

```bash
# 在本地机器上执行
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

然后在浏览器中访问：`http://127.0.0.1:18789/`

输入在 `.env` 中配置的 `OPENCLAW_GATEWAY_TOKEN` 进行认证。

---

## 数据持久化架构详解

### 持久化存储全景图

```
┌─────────────────────────────────────────────────────────────┐
│                      Docker 容器                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              OpenClaw 网关进程                         │  │
│  │                                                      │  │
│  │  /home/node/.openclaw/          /home/node/.openclaw/ │  │
│  │    ├─ openclaw.json              workspace/           │  │
│  │    ├─ credentials/                ├─ SOUL.md         │  │
│  │    │   ├─ anthropic.json           ├─ memory/         │  │
│  │    │   └─ openai.json             └─ skills/         │  │
│  │    └─ skills/                                          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                        │
                        │  Volume 挂载
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    GCP 持久磁盘                              │
│                                                              │
│   /home/<username>/.openclaw/                                │
│   /home/<username>/.openclaw/workspace/                       │
└─────────────────────────────────────────────────────────────┘
```

### 持久化组件详细说明

| 组件 | 容器内路径 | 主机路径 | 持久化机制 | 丢失影响 |
|-----|-----------|---------|-----------|---------|
| 网关配置 | `/home/node/.openclaw/` | `~/.openclaw/` | Volume 挂载 | 需要重新配置 |
| 模型认证 | `/home/node/.openclaw/` | `~/.openclaw/` | Volume 挂载 | 需要重新登录 |
| 技能配置 | `/home/node/.openclaw/skills/` | `~/.openclaw/skills/` | Volume 挂载 | 技能设置丢失 |
| 工作区 | `/home/node/.openclaw/workspace/` | `~/.openclaw/workspace/` | Volume 挂载 | AI 记忆丢失 |
| WhatsApp 会话 | `/home/node/.openclaw/` | `~/.openclaw/` | Volume 挂载 | 需要重新扫码 |
| Gmail 密钥环 | `/home/node/.openclaw/` + 环境变量 | `~/.openclaw/` | Volume + 密码 | 需要重新配置 |
| 二进制依赖 | `/usr/local/bin/` | 镜像层 | 镜像构建 | 需重新构建镜像 |
| Node.js 运行时 | 容器文件系统 | 镜像层 | 镜像构建 | 需重新构建镜像 |

---

## 故障排查完全指南

### 症状一：SSH 连接被拒绝

**现象**：SSH 连接失败或超时

**诊断步骤**：

```bash
# 检查实例状态
gcloud compute instances describe openclaw-gateway --zone=us-central1-a

# 检查防火墙规则
gcloud compute firewall-rules list --filter="allowed[].ports:22"
```

**常见原因与解决方案**：

| 原因 | 解决方案 |
|-----|---------|
| SSH 密钥未传播 | 等待 1-2 分钟后重试 |
| 防火墙阻止 | 添加或修改防火墙规则 |
| 实例已停止 | 通过控制台启动实例 |
| 区域错误 | 确认使用了正确的区域 |

### 症状二：OS Login 认证问题

**现象**：无法通过 gcloud SSH 连接

**诊断步骤**：

```bash
# 检查 OS Login 配置
gcloud compute os-login describe-profile

# 检查 IAM 权限
gcloud projects get-iam-policy my-openclaw-project
```

**解决方案**：

确保账户有以下 IAM 角色之一：
- Compute OS Admin Login
- Compute OS Login

### 症状三：内存不足（OOM）

**现象**：容器反复崩溃或重启

**诊断步骤**：

```bash
# 在虚拟机上检查内存使用
free -h

# 查看系统日志中的 OOM 事件
dmesg | grep -i oom

# 检查容器资源使用
docker stats
```

**解决方案**：

升级机器类型：

```bash
# 停止实例
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# 更改机器类型
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-medium

# 启动实例
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

### 症状四：容器无法启动

**现象**：docker compose up 失败或容器退出

**诊断步骤**：

```bash
# 查看详细错误日志
docker compose logs openclaw-gateway

# 检查容器配置
docker compose config

# 检查端口冲突
lsof -i :18789
```

**常见原因**：

- 端口被占用
- 配置文件语法错误
- 权限问题
- Volume 挂载失败

### 症状五：二进制依赖缺失

**现象**：技能运行时报告找不到命令

**诊断步骤**：

```bash
# 检查二进制是否在镜像中
docker compose exec openclaw-gateway which <binary-name>

# 检查二进制是否存在并有执行权限
docker compose exec openclaw-gateway ls -la /usr/local/bin/<binary-name>
```

**解决方案**：

1. 更新 Dockerfile 添加缺失的二进制安装
2. 重新构建镜像
3. 重新部署

---

## 安全最佳实践

### 服务账户最小权限原则

创建专用的服务账户而不是使用默认账户：

```bash
# 创建服务账户
gcloud iam service-accounts create openclaw-deploy \
  --display-name="OpenClaw Deployment"

# 分配最小必要权限
gcloud projects add-iam-policy-binding my-openclaw-project \
  --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
  --role="roles/compute.instanceAdmin.v1"
```

### 网络安全

#### 防火墙规则

```bash
# 允许 SSH（仅限特定 IP）
gcloud compute firewall-rules create allow-ssh \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=openclaw

# 拒绝所有其他入站流量
gcloud compute firewall-rules deny-all \
  --deny=tcp:0-65535,udp:0-65535 \
  --target-tags=openclaw
```

### 数据加密

GCP 默认对磁盘数据进行加密：
- 静态加密：使用 Google 管理的密钥或客户管理密钥（CMEK）
- 传输加密：所有数据传输使用 TLS 加密

---

## 成本优化策略

### 资源选择建议

| 场景 | 推荐配置 | 月成本估算 |
|-----|---------|-----------|
| 开发测试 | e2-micro | 免费 |
| 个人使用 | e2-small | ~$12 |
| 生产环境 | e2-small | ~$12 |
| 高负载 | e2-medium | ~$24 |

### 成本监控

```bash
# 查看当前计费情况
gcloud billing budgets describe

# 设置预算告警
gcloud billing budgets create \
  --billing-account=<account-id> \
  --display-name="OpenClaw Budget" \
  --budget-amount=50USD \
  --threshold-rule=percent=0.5
```

---

## 适用场景深度分析

### 最适合的场景

**场景一：企业级部署**

GCP 的成熟生态和合规认证使其非常适合企业环境：
- 完整的审计日志
- 精细的 IAM 权限控制
- 与 Google Workspace 集成

**场景二：混合云架构**

如果已有 GCP 基础设施，添加 OpenClaw 是自然的选择：
- 共享 VPC 网络
- 统一的身份认证
- 集中的日志管理

**场景三：大规模运维**

对于需要管理多个 OpenClaw 实例的场景：
- 使用 Terraform 进行基础设施即代码管理
- 使用 Cloud Monitoring 进行集中监控
- 使用 Cloud Logging 进行日志分析

### 需要考虑其他方案的场景

**场景一：成本敏感型**

如果预算有限：
- 考虑 Hetzner（固定月费更低）
- 考虑 Oracle Cloud Always Free
- 考虑自建硬件

**场景二：简化运维需求**

如果不需要 GCP 的高级功能：
- 考虑 Fly.io（自动 HTTPS，更简化）
- 考虑 DigitalOcean（更简单的 UI）

**场景三：本地部署要求**

如果有数据本地化要求：
- 考虑 Raspberry Pi（本地部署）
- 考虑自建机房

---

## 专家思维模型：本章总结

### 云平台选型决策矩阵

```
需求评估
    │
    ├── 规模需求
    │       ├── 小规模（1-5 用户） ─────► 成本优先：Hetzner / Oracle
    │       │
    │       └── 中大规模 ──────────────► 功能优先：GCP / AWS
    │
    ├── 运维能力
    │       ├── 有限 ───────────────────► 托管服务：Fly.io
    │       │
    │       └── 充足 ───────────────────► 自建/容器化：GCP / Hetzner
    │
    └── 合规要求
            ├── 高 ──────────────────────► 主流云：GCP / AWS / Azure
            │
            └── 一般 ───────────────────► 任意方案
```

### 架构设计核心原则

1. **不可变基础设施**：通过镜像重建而不是原地更新
2. **配置与代码分离**：使用环境变量和 Volume 管理配置
3. **最小权限**：服务账户只授予必要权限
4. **可观测性**：日志、指标、追踪缺一不可

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 其他托管方案对比
- [Docker 容器化部署](/install/docker) — 通用 Docker 配置
- [OpenClaw 网关配置](/gateway/configuration) — 完整配置参考
- [Hetzner Docker 部署](/platforms/hetzner) — 容器化替代方案

### 外部资源

- [GCP Compute Engine 文档](https://cloud.google.com/compute/docs)
- [gcloud CLI 参考](https://cloud.google.com/sdk/gcloud)
- [Docker 官方文档](https://docs.docker.com/)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 Docker 容器化部署的核心优势
- [ ] 理解 GCP 的项目、区域和机器类型概念
- [ ] 了解持久化存储的设计原则

### 动手能力

- [ ] 能够创建和配置 GCP Compute Engine 虚拟机
- [ ] 能够构建包含所有依赖的 Docker 镜像
- [ ] 能够配置 Docker Compose 环境并启动服务

### 问题解决

- [ ] 能够诊断 SSH 连接和认证问题
- [ ] 能够处理内存不足和容器启动失败
- [ ] 能够解决二进制依赖缺失问题

### 进阶能力

- [ ] 能够设计基于 GCP 的高可用部署架构
- [ ] 能够配置 Cloud IAM 权限控制
- [ ] 能够构建完整的监控和成本优化体系
