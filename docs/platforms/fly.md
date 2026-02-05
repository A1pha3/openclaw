---
summary: "在 Fly.io 上部署 OpenClaw（云原生容器化部署方案）"
read_when:
  - 在 Fly.io 平台上部署 OpenClaw
  - 寻求高可用、自动扩缩容的云原生部署方案
  - 需要持久化存储和自动 HTTPS 配置
title: "Fly.io"
---

# OpenClaw 在 Fly.io 平台上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 Fly.io 平台的架构设计和服务特性
- 掌握通过 flyctl CLI 工具管理 Fly.io 应用的全流程
- 能够完成从应用创建到首次访问的完整部署
- 理解 Fly.io 的持久卷（Volume）存储机制

### 进阶目标（建议掌握）

- 配置生产级别的 `fly.toml` 配置文件
- 实现机密信息（Secrets）的安全管理
- 配置私有部署模式以增强安全性
- 掌握应用更新和滚动发布的最佳实践

### 专家目标（挑战）

- 设计基于 Fly.io 的多区域高可用部署架构
- 优化资源配额以平衡性能和成本
- 构建基于 Fly.io 的灾难恢复和业务连续性方案

---

## 核心概念解析

### 为什么选择 Fly.io？

Fly.io 是一个专为分布式应用设计的云平台，其核心优势使其非常适合部署 OpenClaw 网关：

**全球分布式部署**：Fly.io 在全球多个区域部署了边缘节点，可以将应用部署在靠近用户的地理位置。对于需要低延迟响应的 AI 助手应用，这种地理分布可以显著改善用户体验。

**自动 HTTPS**：Fly.io 自动为所有应用配置 Let's Encrypt 证书，用户无需手动处理证书申请、验证和续期。这大大简化了安全配置的复杂性。

**原生容器支持**：Fly.io 基于 Docker 容器技术，提供了完整的容器化部署体验。这种设计带来了：
- 环境一致性（开发、测试、生产环境相同）
- 隔离性（每个应用运行在独立的容器中）
- 可移植性（随时迁移到其他支持 Docker 的平台）

**持久存储支持**：通过 Volume 机制，Fly.io 允许容器访问持久化的文件系统。这对于 OpenClaw 存储配置、凭证和会话数据至关重要。

### Fly.io 架构深度解析

#### 应用与机器（Application vs Machine）

Fly.io 的架构中有两个核心概念需要理解：

**应用（Application）** 是逻辑上的部署单元，包含：
- 应用配置（`fly.toml`）
- 机密信息（Secrets）
- 部署历史记录
- 关联的机器实例

**机器（Machine）** 是实际运行容器的计算单元，每个机器具有：
- 独立的 CPU、内存资源
- 独立的文件系统（通过 Volume 挂载）
- 独立的网络接口

这种分离设计使得：
- 同一个应用可以运行多个机器实例（负载均衡）
- 机器可以独立启动、停止、重启
- 配置变更可以逐步应用到不同的机器

#### 网络架构

Fly.io 的网络架构设计如下：

```
用户请求
    │
    ▼
Fly.io 全球边缘网络
    │
    ▼
自动负载均衡器
    │
    ▼
某个机器实例（容器）
    │
    ▼
内部 Volume 存储
```

**关键特性**：
- **自动健康检查**：Fly.io 会定期检查应用健康状态，自动将流量路由到健康的实例
- **零停机部署**：更新时新实例先启动并通过健康检查，旧实例才停止
- **自动扩缩容**：根据流量自动调整实例数量（需要配置自动扩缩容策略）

---

## 快速上手路径

### 十分钟快速部署

```bash
# 步骤 1：安装 flyctl CLI
curl -L https://fly.io/install.sh | sh

# 步骤 2：登录 Fly.io
flyctl auth login

# 步骤 3：克隆 OpenClaw 仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 步骤 4：创建 Fly.io 应用
fly apps create my-openclaw

# 步骤 5：创建持久卷
fly volumes create openclaw_data --size 1 --region iad

# 步骤 6：配置机密信息
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# 步骤 7：部署应用
fly deploy
```

---

## 完整部署流程

### 第一步：环境准备

#### flyctl CLI 安装与配置

```bash
# 使用安装脚本安装
curl -L https://fly.io/install.sh | sh

# 验证安装
flyctl version

# 登录 Fly.io 账户
flyctl auth login

# 如果使用 GitHub，可以快速授权
flyctl auth login --github
```

#### 注册账户与免费层级

Fly.io 提供免费层级（Free Tier），包括：
- 有限的计算资源
- 单个共享 CPU 机器
- 有限的存储空间

对于测试和开发环境，免费层级通常足够。生产环境可能需要付费升级。

### 第二步：应用创建与配置

#### 创建应用

```bash
# 创建新应用（交互式）
fly apps create

# 或指定名称创建
fly apps create my-openclaw
```

#### 创建持久卷

持久卷用于存储 OpenClaw 的配置、凭证和会话数据：

```bash
# 创建 1GB 持久卷（通常足够）
fly volumes create openclaw_data --size 1 --region iad

# 查看已创建的卷
fly volumes list
```

**区域选择指南**：

| 区域代码 | 地理位置 | 推荐场景 |
|---------|---------|---------|
| `iad` | 弗吉尼亚（美国东部） | 北美用户首选 |
| `sjc` | 加利福尼亚（美国西部） | 西海岸用户 |
| `lhr` | 伦敦（英国） | 欧洲用户 |
| `hkg` | 香港 | 亚太用户 |
| `sin` | 新加坡 | 东南亚用户 |
| `nrt` | 东京（日本） | 日本用户 |

> **选择原则**：选择距离主要用户群体最近的区域，以获得最低的网络延迟。

### 第三步：fly.toml 配置详解

`fly.toml` 是 Fly.io 应用的配置文件，定义了应用的运行参数：

```toml
app = "my-openclaw"  # 应用名称
primary_region = "iad"  # 主要部署区域

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

#### 配置参数详解

**进程配置**：

```toml
[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"
```

| 参数 | 作用 | 推荐值 |
|-----|------|-------|
| `--allow-unconfigured` | 允许无配置文件启动 | 首次部署时使用，后续删除 |
| `--port 3000` | 监听端口 | 必须与 `internal_port` 一致 |
| `--bind lan` | 绑定到所有网络接口 | 让 Fly.io 代理能够访问 |

**资源配额**：

```toml
[[vm]]
  size = "shared-cpu-2x"  # 2 共享 CPU 核心
  memory = "2048mb"       # 2GB 内存
```

| 规格 | CPU | 内存 | 适用场景 |
|-----|-----|------|---------|
| `shared-cpu-1x` | 1 核 | 1GB | 测试环境 |
| `shared-cpu-2x` | 2 核 | 2GB | 生产环境推荐 |
| `shared-cpu-4x` | 4 核 | 4GB | 高负载场景 |
| `performance-1x` | 1 核 | 2GB | 需要保证性能 |

> **内存选择依据**：OpenClaw 网关本身占用内存不大（约 200-500MB），但运行大型 JavaScript 应用和 Node.js 堆空间需要额外内存。2GB 是生产环境的推荐配置。

**持久卷挂载**：

```toml
[mounts]
  source = "openclaw_data"   # 卷名称
  destination = "/data"       # 容器内挂载路径
```

### 第四步：机密信息安全配置

Fly.io 的 Secrets 系统用于存储敏感配置，如 API 密钥和访问令牌：

```bash
# 网关 Token（用于非本地回环绑定）
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# 模型提供商 API 密钥
fly secrets set ANTHROPIC_API_KEY=sk-ant-03...
fly secrets set OPENAI_API_KEY=sk-...

# 即时通讯渠道令牌
fly secrets set DISCORD_BOT_TOKEN=MTIzND...
fly secrets set TELEGRAM_BOT_TOKEN=123456:ABC...

# 查看所有机密信息
fly secrets list
```

> **安全最佳实践**：
> - 机密信息优先使用环境变量而非配置文件
> - 定期轮换 Token 和密钥
> - 不要在日志或错误消息中暴露机密信息
> - 使用强随机数生成 Token：`openssl rand -hex 32`

### 第五步：应用部署

```bash
# 首次部署（会构建 Docker 镜像）
fly deploy

# 后续部署（使用缓存的镜像，更快）
fly deploy
```

部署过程会执行以下步骤：
1. 读取 `fly.toml` 配置
2. 构建 Docker 镜像
3. 上传镜像到 Fly.io  registry
4. 启动新的机器实例
5. 执行健康检查
6. 自动切换流量到新实例

#### 部署验证

```bash
# 查看应用状态
fly status

# 查看实时日志
fly logs

# 查看部署历史
fly deploy history
```

成功部署的日志应包含：
```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

### 第六步：配置管理与初始设置

部署完成后，需要创建 OpenClaw 配置文件：

```bash
# SSH 连接到机器
fly ssh console
```

在远程 shell 中创建配置：

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": [
          "anthropic/claude-sonnet-4-5",
          "openai/gpt-4o"
        ]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": {
        "mode": "token",
        "provider": "anthropic"
      },
      "openai:default": {
        "mode": "token",
        "provider": "openai"
      }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "discord"
      }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": {
            "general": {
              "allow": true
            }
          },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**配置说明**：
- 配置路径：`/data/openclaw.json`（对应卷挂载的目录）
- 模型认证：从环境变量读取，无需在配置中重复配置
- Discord Token：同样从 `DISCORD_BOT_TOKEN` 环境变量读取

重启应用使配置生效：

```bash
exit
fly machine restart <machine-id>
```

### 第七步：访问控制界面

#### 通过浏览器访问

```bash
# 打开默认浏览器访问
fly open

# 或直接访问
# https://my-openclaw.fly.dev/
```

首次访问时会要求输入网关 Token（在 Secrets 中配置的 `OPENCLAW_GATEWAY_TOKEN`）。

#### 日志查看

```bash
# 实时日志
fly logs

# 最近日志（不追踪）
fly logs --no-tail
```

#### SSH 访问

```bash
# 获取交互式 shell
fly ssh console
```

---

## 私有部署模式（安全强化）

### 公有部署与私有部署的对比

默认情况下，Fly.io 会为应用分配公网 IP 地址，使应用可以通过 `*.fly.dev` 域名直接访问。这种设计虽然方便，但也意味着应用对整个互联网可见，可能面临：

- 端口扫描和侦察
- 自动化攻击工具扫描
- DDoS 攻击风险

**私有部署模式**通过不分配公网 IP，从网络层面消除这些风险：

| 方面 | 公有部署 | 私有部署 |
|-----|---------|---------|
| 可发现性 | 互联网可发现 | 仅内部网络可访问 |
| 直接攻击 | 可能 | 不可能 |
| 控制 UI 访问 | 直接浏览器 | 需通过代理/VPN |
| webhook 回调 | 直接接收 | 需要隧道 |

### 何时使用私有部署

**适合私有部署的场景**：
- 仅发起出站调用（如 AI 模型 API 调用）
- 使用 ngrok/Tailscale 隧道接收 webhook
- 通过 SSH、代理或 WireGuard 访问控制界面
- 对安全性有严格要求的环境

**不适合私有部署的场景**：
- 需要直接接收公网 webhook
- 希望直接通过浏览器访问控制 UI
- 对运维复杂度敏感的用户

### 私有部署配置步骤

#### 步骤一：使用私有配置模板

```bash
# 使用 fly.private.toml 部署
fly deploy -c fly.private.toml
```

`fly.private.toml` 与标准配置的主要区别是移除了 `[http_service]` 部分，不分配公网 IP。

#### 步骤二：释放公网 IP

```bash
# 查看当前分配的 IP
fly ips list -a my-openclaw

# 释放 IPv4
fly ips release <public-ipv4> -a my-openclaw

# 释放 IPv6
fly ips release <public-ipv6> -a my-openclaw
```

验证仅保留私有 IP：

```bash
fly ips list -a my-openclaw
```

应仅显示 `private` 类型的 IPv6 地址。

#### 步骤三：后续部署使用私有配置

```bash
# 每次部署时指定私有配置
fly deploy -c fly.private.toml
```

### 私有部署的访问方式

#### 方式一：本地端口转发（推荐）

```bash
# 将远程 3000 端口转发到本地
fly proxy 3000:3000 -a my-openclaw

# 然后在浏览器访问
# http://localhost:3000
```

#### 方式二：WireGuard VPN

```bash
# 创建 WireGuard 配置（一次性）
fly wireguard create

# 导入到 WireGuard 客户端
# 配置后可通过内部 IPv6 直接访问
# 示例：http://[fdaa:x:x:x:x::x]:3000
```

#### 方式三：SSH 访问

```bash
# 直接 SSH 到机器
fly ssh console -a my-openclaw
```

### 私有部署的 Webhook 处理

某些渠道（如 Twilio 电话、Telnyx）需要接收公网 webhook。以下是解决方案：

#### 方案一：ngrok 隧道

在 OpenClaw 配置中启用 ngrok 隧道：

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": {
            "provider": "ngrok"
          }
        }
      }
    }
  }
}
```

ngrok 会在容器内部运行，提供公网 webhook URL，而 Fly.io 应用本身保持私有。

#### 方案二：Tailscale Funnel

使用 Tailscale Funnel 暴露特定路径：

```bash
# 启用 Funnel（需要 Tailscale 账户）
tailscale funnel 443 --https=8080
```

---

## 生产环境优化

### 资源配额优化

根据实际负载调整资源配置：

```toml
[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"
```

| 场景 | CPU | 内存 | 说明 |
|-----|-----|------|------|
| 开发测试 | 1x | 1GB | 成本最低 |
| 个人使用 | 2x | 2GB | 推荐配置 |
| 高并发 | 4x | 4GB | 大流量场景 |

### 自动扩缩容配置

```toml
[http_service]
  auto_stop_machines = false    # 不自动停止机器
  auto_start_machines = true     # 流量增加时自动启动
  min_machines_running = 1       # 保持至少 1 台机器运行
```

### 监控与告警

```bash
# 查看应用指标
fly metrics

# 设置告警
fly alerts create --metric=cpu --threshold=80
```

---

## 更新与维护

### 常规更新流程

```bash
# 拉取最新代码
git pull

# 重新部署
fly deploy

# 验证状态
fly status
fly logs
```

### 更新机器命令

如果需要修改启动命令而不重新部署整个应用：

```bash
# 列出机器
fly machines list

# 更新单个机器
fly machine update <machine-id> \
  --command "node dist/index.js gateway --port 3000 --bind lan" \
  -y

# 或同时更新内存
fly machine update <machine-id> \
  --vm-memory 2048 \
  --command "node dist/index.js gateway --port 3000 --bind lan" \
  -y
```

> **注意**：`fly deploy` 后，机器命令可能会被 `fly.toml` 中的配置覆盖。如需持久保留自定义命令，请更新配置文件。

### 回滚操作

```bash
# 查看部署历史
fly deploy history

# 回滚到上一个版本
fly deploy --rollback
```

---

## 故障排查完全指南

### 症状一：健康检查失败

**现象**：部署后状态显示「unhealthy」，流量无法路由到实例

**诊断步骤**：

```bash
# 查看健康检查详情
fly doctor

# 查看详细日志
fly logs --no-pager | grep -i health

# 检查应用是否在正确端口监听
fly ssh console --command "netstat -tlnp | grep 3000"
```

**常见原因**：

| 原因 | 诊断命令 | 解决方案 |
|-----|---------|---------|
| 端口不匹配 | `netstat -tlnp` | 确保 `--port` 与 `internal_port` 一致 |
| 绑定地址错误 | 检查启动日志 | 使用 `--bind lan` |
| 应用崩溃 | `fly logs` | 检查错误堆栈 |

### 症状二：应用内存不足（OOM）

**现象**：容器频繁重启或被终止，日志显示 `SIGABRT`、`v8::internal::Runtime_AllocateInYoungGeneration` 或无声重启

**诊断步骤**：

```bash
# 查看内存使用趋势
fly metrics

# 检查容器退出码
fly machines list
```

**解决方案**：

```bash
# 增加内存配额
fly machine update <machine-id> --vm-memory 2048 -y

# 或更新 fly.toml 后重新部署
```

> **内存推荐**：512MB 太小，可能在日志详细输出或高负载时 OOM。2GB 是生产环境的推荐配置。

### 症状三：网关锁文件问题

**现象**：网关拒绝启动，报错「already running」

**原因**：容器重启后，锁文件仍然残留在持久卷上

**解决方案**：

```bash
# 删除锁文件
fly ssh console --command "rm -f /data/gateway.*.lock"

# 重启机器
fly machine restart <machine-id>
```

### 症状四：配置未生效

**现象**：修改 `openclaw.json` 后，行为没有变化

**诊断步骤**：

```bash
# 验证配置文件存在
fly ssh console --command "cat /data/openclaw.json"

# 检查配置路径
fly ssh console --command "ls -la /data/"
```

**解决方案**：

```bash
# 如果配置错误，删除后重新创建
fly ssh console --command "rm /data/openclaw.json"

# 重启应用
fly machine restart <machine-id>

# 重新创建配置（如前文所述）
```

### 症状五：状态未持久化

**现象**：重启后丢失凭证、会话或配置

**诊断步骤**：

```bash
# 检查卷挂载状态
fly volumes list

# 验证挂载点
fly ssh console --command "df -h /data"
```

**解决方案**：

确保 `fly.toml` 中正确配置了卷挂载：

```toml
[mounts]
  source = "openclaw_data"
  destination = "/data"
```

### 症状六：文件写入失败（通过 SSH）

**现象**：使用 `fly ssh console -C` 命令时，shell 重定向不起作用

**原因**：`fly ssh console -C` 命令不支持交互式 shell 重定向

**解决方案**：

```bash
# 方法一：使用 tee 管道
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# 方法二：使用 SFTP
fly sftp shell
> put /local/path/config.json /data/openclaw.json

# 如果文件已存在，先删除
fly ssh console --command "rm /data/openclaw.json"
```

---

## 适用场景深度分析

### 最适合的场景

**场景一：追求高可用性**

Fly.io 的全球分布式架构和自动健康检查使其非常适合需要高可用性的部署：
- 自动检测不健康实例并切换流量
- 多区域部署选项
- 零停机部署能力

**场景二：简化运维复杂度**

对于不希望深入了解服务器管理的用户：
- 自动 HTTPS 配置
- 简化的部署流程
- 托管式基础设施管理

**场景三：快速原型和测试**

开发阶段需要频繁部署和测试：
- 快速部署周期
- 易于回滚
- 隔离的测试环境

### 需要考虑其他方案的场景

**场景一：成本敏感型大规模部署**

虽然 Fly.io 的单个应用成本不高，但扩展到多个应用或高流量场景时：
- 考虑 Hetzner 等固定月费方案
- 自建 Kubernetes 集群

**场景二：需要完整控制权**

某些场景需要深入控制底层基础设施：
- 自定义网络配置
- 特定的防火墙规则
- 特殊的安全合规要求

**场景三：长期运行的大流量应用**

对于持续高负载的应用：
- 专用实例（Dedicated CPU）可能更经济
- 考虑传统云服务（AWS EC2、GCP Compute Engine）

---

## 成本分析与优化

### Fly.io 定价结构

Fly.io 的费用包括以下组成部分：

| 费用项目 | 计算方式 | 典型成本 |
|---------|---------|---------|
| 计算资源 | 按 CPU/内存/秒计费 | ~$10-15/月（推荐配置） |
| 持久卷 | 按 GB/月计费 | ~$0.15/GB/月 |
| 出站带宽 | 按 GB 计费 | 取决于使用量 |
| 私有 IP | 免费 | - |

### 成本优化策略

**策略一：合理选择区域**

不同区域的定价可能略有差异，选择最近的区域通常最优。

**策略二：按需扩展**

仅在需要时扩展资源：
- 开发环境使用最小配置
- 生产环境使用推荐配置

**策略三：利用免费层级**

测试和开发环境可以利用免费层级运行。

---

## 专家思维模型：本章总结

### 部署架构决策框架

```
应用需求评估
    │
    ├── 可用性要求
    │       ├── 高（99.9%+） ─────► 多区域部署 + 私有网络
    │       │
    │       └── 中等 ─────────────► 单区域部署
    │
    ├── 安全要求
    │       ├── 高 ───────────────► 私有部署 + VPN
    │       │
    │       └── 标准 ─────────────► 公有部署
    │
    └── 运维能力
            ├── 有限 ─────────────► 托管服务（Fly.io）
            │
            └── 充足 ─────────────► 自建/容器化（Hetzner）
```

### 核心设计原则回顾

1. **配置与状态分离**：配置通过 fly.toml 和 Secrets 管理，状态存储在 Volume 中
2. **健康检查驱动**：流量切换基于健康检查结果，保证服务连续性
3. **最小权限原则**：私有部署模式消除不必要的公网暴露面
4. **可观测性优先**：通过日志、指标实现问题快速定位

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 其他托管方案对比
- [Docker 容器化部署](/install/docker) — 通用 Docker 配置
- [OpenClaw 网关配置](/gateway/configuration) — 完整配置参考
- [Hetzner Docker 部署](/platforms/hetzner) — 容器化替代方案

### 外部资源

- [Fly.io 官方文档](https://fly.io/docs/)
- [Fly.io 定价页面](https://fly.io/docs/about/pricing/)
- [flyctl CLI 参考](https://fly.io/docs/reference/flyctl/)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 Fly.io 的应用与机器架构
- [ ] 理解健康检查在流量切换中的作用
- [ ] 了解公有部署与私有部署的安全差异

### 动手能力

- [ ] 能够完成 Fly.io 应用的创建和配置
- [ ] 能够正确配置持久卷和环境变量
- [ ] 能够执行应用部署和更新操作

### 问题解决

- [ ] 能够诊断健康检查失败问题
- [ ] 能够解决内存不足和锁文件问题
- [ ] 能够处理配置未生效的故障

### 进阶能力

- [ ] 能够设计多区域高可用部署架构
- [ ] 能够优化资源配置以平衡性能和成本
- [ ] 能够构建完整的监控和告警体系
