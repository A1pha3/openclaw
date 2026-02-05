---
summary: "在 exe.dev 虚拟机上部署 OpenClaw（VM + HTTPS 代理方案）"
read_when:
  - 你需要一个经济实惠的 Linux 主机来运行 OpenClaw 网关
  - 你需要在不自行管理 VPS 的情况下远程访问控制界面
  - 你希望利用 HTTPS 代理简化安全配置
title: "exe.dev"
---

# OpenClaw 在 exe.dev 平台上的部署指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 exe.dev 平台的核心架构和价值定位
- 掌握通过 Shelley 代理实现自动化部署的方法
- 能够完成从虚拟机创建到 OpenClaw 运行验证的完整流程
- 理解 HTTPS 代理模式与直接暴露模式的差异

### 进阶目标（建议掌握）

- 配置 nginx 反向代理以支持 WebSocket 连接
- 理解 exe.dev 平台的端口转发和 HTTPS 终止机制
- 实现安全的多因素认证访问控制
- 掌握手动部署流程以应对自动化失败场景

### 专家目标（挑战）

- 设计基于 exe.dev 的高可用部署架构
- 优化 nginx 配置以提升 WebSocket 连接稳定性
- 构建完整的监控和告警体系

---

## 核心概念解析

### exe.dev 平台架构设计

exe.dev 是一个现代化的虚拟机托管平台，其核心设计理念围绕以下几个关键特性展开：

**简化运维负担**：传统的 VPS 部署需要用户自行处理操作系统安装、安全更新、网络配置、SSL 证书管理等繁琐任务。exe.dev 通过预置的「exeuntu」操作系统镜像和自动化工具链，将这些复杂性封装在平台层面，让用户可以专注于应用本身。

**内置 HTTPS 代理**：这是 exe.dev 与传统 VPS 最显著的差异。exe.dev 自动为每个虚拟机分配 `*.exe.xyz` 域名，并在平台层面处理 SSL 证书的申请、部署和续期。用户无需了解 Let's Encrypt、证书管理、Nginx 配置等细节，即可获得开箱即用的 HTTPS 访问体验。

**端口转发机制**：exe.dev 将虚拟机的 8000 端口自动映射到公网的 443 端口（HTTPS），同时支持 WebSocket 协议的透明转发。这种设计既保证了安全性（用户无需开放随机高端口），又简化了网络配置（无需配置防火墙规则）。

### Shelley 代理的工作原理

Shelley 是 exe.dev 平台的智能代理系统，其设计目标是在最小化人工干预的前提下，完成复杂的软件部署任务。其工作流程如下：

```
用户通过浏览器访问 exe.dev → 输入配置参数（认证密钥、API Token 等）
    │
    ▼
Shelley 代理连接到目标虚拟机
    │
    ▼
解析部署指令 → 执行环境检查 → 安装依赖 → 配置服务 → 验证运行状态
    │
    ▼
返回部署结果（如遇错误，提供诊断信息和修复建议）
```

**自动化部署的优势**：
- 减少人为配置错误
- 确保部署步骤的完整性和顺序正确
- 提供一致的可重复部署体验
- 支持快速回滚和环境重建

### HTTPS 代理模式的技术优势

在传统 VPS 部署模式下，获取可信的 HTTPS 证书需要以下步骤：

1. 配置 DNS 记录指向服务器公网 IP
2. 安装 certbot 或类似工具
3. 配置 Let's Encrypt 客户端
4. 执行域名验证（HTTP-01 或 DNS-01 挑战）
5. 配置 Nginx/Apache 反向代理
6. 设置证书自动续期任务

exe.dev 的 HTTPS 代理模式将这些步骤完全自动化：

- **零配置证书**：平台自动处理证书申请和续期
- **即时生效**：域名解析完成后即可使用 HTTPS
- **安全传输**：所有流量在平台层面加密
- **无需开放端口**：无需在防火墙中开放 80/443 端口

---

## 快速上手路径

### 方案一：Shelley 代理自动化部署（推荐新手）

如果你希望以最少的配置步骤完成部署，Shelley 自动化部署是最佳选择：

1. 访问 [https://exe.new/openclaw](https://exe.new/openclaw)
2. 填写必要的认证信息（API 密钥、Token 等）
3. 点击「Agent」按钮，等待 Shelley 完成部署
4. 验证服务运行状态
5. 开始使用 OpenClaw

### 方案二：手动部署流程

对于需要自定义配置或 Shelley 部署失败的场景，以下是完整的手动部署步骤：

```bash
# 步骤 1：创建虚拟机
ssh exe.dev new

# 步骤 2：连接到虚拟机
ssh <vm-name>.exe.xyz

# 步骤 3：安装系统依赖
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl

# 步骤 4：安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 步骤 5：配置 nginx 反向代理
# 详细配置见下一章节

# 步骤 6：验证部署
openclaw status
openclaw health
```

---

## 完整部署流程

### 第一步：虚拟机创建与连接

#### 创建新虚拟机

exe.dev 提供了简洁的命令行工具来创建和管理虚拟机：

```bash
# 创建新虚拟机（使用默认配置）
ssh exe.dev new

# 创建时指定虚拟机名称
ssh exe.dev new my-openclaw
```

创建完成后，系统会返回虚拟机的连接信息，包括主机名和临时访问凭证。

#### 建立 SSH 连接

```bash
# 使用分配的域名连接
ssh <vm-name>.exe.xyz

# 或使用 IP 地址连接（如果域名解析有问题）
ssh <username>@<ip-address>
```

> **部署模式选择**：强烈建议将虚拟机保持在**有状态（stateful）**模式。OpenClaw 的所有配置、会话数据和工作区文件都存储在 `~/.openclaw/` 目录下。如果虚拟机被重建，这些数据将会丢失。在 exe.dev 平台上，stateful 模式是默认设置，除非有明确理由，否则不要切换到 stateless 模式。

### 第二步：系统环境准备

#### 安装必要软件包

```bash
# 更新软件包索引
sudo apt-get update

# 安装基础工具
sudo apt-get install -y git curl jq ca-certificates openssl

# 验证安装
git --version
curl --version
jq --version
openssl version
```

| 软件包 | 用途 |
|-------|------|
| git | 版本控制，用于克隆 OpenClaw 源码仓库 |
| curl | HTTP 客户端，用于下载安装脚本和 API 调用 |
| jq | JSON 处理器，用于解析配置和处理 API 响应 |
| ca-certificates | 根证书包，确保 HTTPS 连接的安全性 |
| openssl | 加密工具，用于生成安全令牌和证书管理 |

### 第三步：OpenClaw 安装

#### 标准一键安装

```bash
# 执行官方安装脚本
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装脚本会自动完成以下步骤：

1. 检测系统环境和 Node.js 版本
2. 安装或升级 OpenClaw CLI 工具
3. 创建必要的目录结构
4. 配置环境变量
5. 初始化基本配置

#### 验证安装结果

```bash
# 检查 OpenClaw 版本
openclaw --version

# 检查服务状态
openclaw status

# 查看健康检查结果
openclaw health
```

### 第四步：Nginx 反向代理配置

虽然 exe.dev 平台会自动将 8000 端口映射到 HTTPS，但 OpenClaw 网关默认监听的是 18789 端口。配置 nginx 反向代理的目的是将外部 HTTPS 请求转发到内部的 OpenClaw 服务。

#### 配置文件结构

exe.dev 虚拟机使用典型的 Nginx 配置结构：

- 主配置文件：`/etc/nginx/nginx.conf`
- 站点配置目录：`/etc/nginx/sites-enabled/`
- 可用站点目录：`/etc/nginx/sites-available/`

#### 创建反向代理配置

编辑 `/etc/nginx/sites-enabled/default` 文件：

```nginx
server {
    # 监听配置
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    # OpenClaw 网关反向代理配置
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket 支持（关键配置）
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 标准代理头信息
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置（支持长连接的 WebSocket）
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

#### 配置说明

**WebSocket 支持配置**：

```nginx
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

这两行配置是 WebSocket 协议升级的关键。HTTP/1.1 的 Keep-Alive 连接无法直接用于 WebSocket，客户端需要发送一个「协议升级」请求，将连接从 HTTP 升级为 WebSocket。Nginx 必须正确处理这些头部信息，否则 WebSocket 连接会失败。

**超时时间设置**：

```nginx
proxy_read_timeout 86400s;  # 86400秒 = 24小时
proxy_send_timeout 86400s;
```

WebSocket 是长连接协议，客户端和服务器之间的连接可能保持数小时甚至数天不中断。如果不设置足够的超时时间，Nginx 会在连接空闲一段时间后主动关闭连接，导致 OpenClaw 的实时消息功能失效。

#### 验证配置并重启

```bash
# 检查配置语法
sudo nginx -t

# 重启 nginx 服务
sudo systemctl restart nginx

# 查看服务状态
sudo systemctl status nginx
```

### 第五步：访问控制与授权

#### 获取网关 Token

OpenClaw 在首次启动时会生成一个随机 Token，用于控制界面的身份认证。获取 Token 的方式如下：

```bash
# 查看当前 Token（如果已配置）
openclaw config get gateway.auth.token

# 生成新的 Token
openclaw doctor --generate-gateway-token
```

#### 访问控制界面

在浏览器中访问：`https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL`

将 `YOUR-TOKEN-FROM-TERMINAL` 替换为实际获取的 Token 值。

#### 设备配对

对于需要连接到网关的设备（如 CLI、桌面应用、移动端节点），需要执行配对流程：

```bash
# 在网关主机上查看配对请求
openclaw devices list

# 批准配对请求
openclaw device approve <request-id>
```

> **安全提示**：网关 Token 相当于超级管理员密码，拥有 Token 的人可以完全控制 OpenClaw 网关及其所有连接的渠道。请务必：
> - 不要在公开渠道分享 Token
> - 定期轮换 Token
> - 考虑使用 exe.dev 的访问控制功能限制管理权限

---

## 远程访问架构详解

### exe.dev 的网络架构

exe.dev 平台的网络架构设计旨在简化安全访问的复杂性：

```
用户浏览器
    │
    │ HTTPS (443端口)
    ▼
exe.dev 边缘服务器
    │
    │ 内部转发 (8000端口)
    ▼
Nginx 反向代理
    │
    │ HTTP (本地回环)
    ▼
OpenClaw 网关 (18789端口)
```

**各层组件职责**：

| 组件 | 职责 | 安全性保障 |
|-----|------|-----------|
| exe.dev 边缘服务器 | SSL 终止、域名解析、端口转发 | 处理 TLS 加密，隐藏内部网络结构 |
| Nginx 反向代理 | 请求路由、WebSocket 支持、负载均衡 | 提供额外的访问控制层 |
| OpenClaw 网关 | 业务逻辑处理、渠道连接管理 | Token 认证、权限控制 |

### 内置认证机制

exe.dev 平台提供了可选的内置认证功能，默认通过电子邮件进行身份验证：

- **访问控制**：所有访问 `*.exe.xyz` 域名的请求都需要通过 exe.dev 的身份验证
- **访问日志**：管理员可以在 exe.dev 控制台查看访问记录
- **权限管理**：可以配置哪些用户可以访问特定的虚拟机

> **与 OpenClaw Token 的关系**：exe.dev 的认证和 OpenClaw 的 Token 认证是两个独立的机制。exe.dev 认证控制「谁能连接到虚拟机」，而 OpenClaw Token 认证控制「谁能访问 OpenClaw 功能」。建议两者都启用，以实现纵深防御。

---

## 更新与维护

### 手动更新流程

```bash
# 更新 OpenClaw 到最新版本
npm i -g openclaw@latest

# 运行健康检查
openclaw doctor

# 重启网关服务
openclaw gateway restart

# 验证服务状态
openclaw health
```

### Shelley 自动化更新

对于使用 Shelley 部署的用户，可以通过重新运行自动化脚本来完成更新：

1. 访问 [https://exe.new/openclaw](https://exe.new/openclaw)
2. 输入相同的配置参数
3. Shelley 会自动检测现有安装并执行增量更新

### 更新最佳实践

1. **备份数据**：执行更新前，备份 `~/.openclaw/` 目录
2. **查看变更日志**：了解新版本的变更内容和潜在的破坏性变化
3. **测试环境验证**：生产环境更新前，先在测试环境验证兼容性
4. **制定回滚计划**：如果更新后出现问题，能够快速回滚到之前的版本

---

## 故障排查完全指南

### 症状一：无法通过 HTTPS 访问

**现象**：浏览器返回「无法访问此网站」或证书错误

**诊断步骤**：

```bash
# 1. 检查 exe.dev 端口转发状态
ssh exe.dev status <vm-name>

# 2. 检查虚拟机是否在线
ssh <vm-name>.exe.xyz "uptime"

# 3. 检查内部服务是否正常运行
ssh <vm-name>.exe.xyz "curl -s http://localhost:18789"

# 4. 检查 Nginx 日志
ssh <vm-name>.exe.xyz "sudo tail -n 50 /var/log/nginx/error.log"
```

**常见原因与解决方案**：

| 原因 | 解决方案 |
|-----|---------|
| 虚拟机已停止 | 通过 exe.dev 控制台启动虚拟机 |
| 端口转发未配置 | 联系 exe.dev 支持或检查账户设置 |
| Nginx 未运行 | `sudo systemctl restart nginx` |
| 防火墙阻止连接 | 检查 iptables 或 nftables 配置 |

### 症状二：WebSocket 连接失败

**现象**：控制界面加载后显示「连接已断开」或无法实时接收消息

**诊断步骤**：

```bash
# 1. 测试 WebSocket 连接
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" \
  http://localhost:18789/ws

# 2. 检查 Nginx 配置中的 WebSocket 头部
grep -A 5 "proxy_set_header Upgrade" /etc/nginx/sites-enabled/default

# 3. 检查 OpenClaw 网关的 WebSocket 端点
openclaw config get gateway.bind
```

**解决方案**：

1. 确认 Nginx 配置中包含 WebSocket 支持的头部设置
2. 检查 `proxy_read_timeout` 和 `proxy_send_timeout` 是否设置足够的值
3. 验证 OpenClaw 网关是否绑定到正确的网络接口（`127.0.0.1` 表示仅本地回环）

### 症状三：Token 认证失败

**现象**：输入正确的 Token 后仍无法访问控制界面

**诊断步骤**：

```bash
# 1. 验证 Token 配置
openclaw config get gateway.auth.mode

# 2. 检查是否存在 Token
openclaw config get gateway.auth.token

# 3. 查看认证日志
journalctl -u openclaw -n 100 | grep -i auth
```

**解决方案**：

1. 如果 Token 为空，重新生成：`openclaw doctor --generate-gateway-token`
2. 如果认证模式设置错误，重置为 Token 模式：`openclaw config set gateway.auth.mode token`
3. 重启网关服务使配置生效：`openclaw gateway restart`

### 症状四：Nginx 配置错误

**现象**：`sudo nginx -t` 返回配置语法错误

**诊断步骤**：

```bash
# 检查具体错误信息
sudo nginx -t 2>&1

# 查看错误行号附近的配置
sudo sed -n '1,200p' /etc/nginx/sites-enabled/default
```

**常见语法错误**：

```nginx
# 错误示例：分号缺失
proxy_pass http://127.0.0.1:18789  # 缺少分号

# 正确写法
proxy_pass http://127.0.0.1:18789;

# 错误示例：大括号不匹配
server {
    listen 80 default_server
}  # 缺少分号

# 正确写法
server {
    listen 80 default_server;
}
```

---

## 适用场景深度分析

### 最适合的场景

**场景一：快速原型验证**

当你需要快速验证 OpenClaw 的功能时，exe.dev 的自动化部署可以在数分钟内完成整个环境搭建：
- 无需了解 Linux 系统管理
- 无需配置 SSL 证书
- 无需处理网络防火墙

**场景二：轻量级个人使用**

对于个人用户而言，exe.dev 提供了一个简单、可靠的托管环境：
- 成本可控（按使用量计费或固定月费）
- 平台负责运维
- 随时可以迁移到其他平台

**场景三：团队协作入口**

当团队成员需要访问 OpenClaw 控制界面时：
- exe.dev 的内置认证提供了基本的访问控制
- 统一的访问入口（`*.exe.xyz`）
- 无需在每个成员电脑上配置 SSH 隧道

### 需要考虑其他方案的场景

**场景一：大规模部署**

如果需要在多个环境部署或需要高级定制：
- 考虑使用 Hetzner、GCP 等提供完整控制权的平台
- 自动化工具如 Terraform、Ansible 可以更好地管理基础设施即代码

**场景二：强合规要求**

如果业务有严格的数据驻留或安全合规要求：
- exe.dev 是第三方平台，数据经过其基础设施
- 对于高度敏感的数据，考虑自建 VPS 或本地部署

**场景三：成本敏感型大规模使用**

如果预期有较高的资源消耗：
- exe.dev 的按需计费模式可能导致费用快速上升
- 固定月费的 VPS 方案（如 Hetzner）可能更经济

---

## 成本与方案对比

### exe.dev 定价模式

exe.dev 采用灵活的定价模式，具体费用取决于：

- 虚拟机规格（CPU、内存、存储）
- 运行时间
- 带宽消耗

详细的定价信息请参考 [exe.dev 官方网站](https://exe.dev)。

### 与其他方案的对比

| 维度 | exe.dev | 自建 VPS (Hetzner) | 云服务器 (AWS/GCP) |
|-----|---------|-------------------|-------------------|
| 初始配置复杂度 | 低 | 中 | 高 |
| SSL 证书管理 | 自动 | 手动 | 半自动 |
| 运维负担 | 低 | 中 | 高 |
| 成本可控性 | 中（按需） | 高（固定月费） | 低（易超支） |
| 定制灵活性 | 中 | 高 | 高 |
| 学习曲线 | 缓 | 中 | 陡 |

---

## 专家思维模型：本章总结

### 平台选型决策框架

选择部署平台时，需要综合考虑以下因素：

```
评估维度
    │
    ├── 运维能力 ────────► 能力强 → 自建 VPS
    │                              能力弱 → 托管平台 (exe.dev)
    │
    ├── 成本结构 ────────► 稳定负载 → 固定月费 VPS
    │                              波动负载 → 按需计费
    │
    ├── 合规要求 ────────► 高要求 → 本地部署
    │                              一般要求 → 主流云平台
    │
    └── 扩展需求 ────────► 可能扩展 → 选择扩展性好的方案
                               固定需求 → 性价比优先
```

### 架构设计考量

1. **安全分层**：exe.dev 提供网络层安全，OpenClaw 提供应用层认证
2. **故障隔离**：单个虚拟机故障不影响其他服务
3. **可迁移性**：保持配置与数据的可移植性，避免平台锁定

---

## 扩展阅读与参考资源

### 相关文档

- [VPS 托管综合指南](/vps) — 其他托管方案对比
- [Linux 通用部署指南](/platforms/linux) — 通用 Linux 系统设置
- [Hetzner Docker 部署](/platforms/hetzner) — 容器化部署方案
- [OpenClaw 网关配置](/gateway/configuration) — 完整配置参考
- [Tailscale 远程访问](/gateway/tailscale) — 安全远程访问

### 外部资源

- [exe.dev 官方网站](https://exe.dev)
- [exe.dev 文档](https://docs.exe.dev)
- [Nginx 官方文档](https://nginx.org/en/docs/)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

---

## 自检清单

完成本章节学习后，请确认你已掌握以下能力：

### 概念理解

- [ ] 能够解释 exe.dev 平台的核心架构设计
- [ ] 理解 HTTPS 代理模式的工作原理
- [ ] 了解 WebSocket 协议升级的机制

### 动手能力

- [ ] 能够通过 Shelley 自动化完成 OpenClaw 部署
- [ ] 能够手动配置 nginx 反向代理
- [ ] 能够诊断常见的连接和认证问题

### 问题解决

- [ ] 能够识别并解决 WebSocket 连接失败问题
- [ ] 能够处理 Token 认证相关的故障
- [ ] 能够优化 Nginx 配置以提升连接稳定性

### 进阶能力

- [ ] 能够设计基于 exe.dev 的多环境部署架构
- [ ] 能够构建完整的监控和告警体系
- [ ] 能够制定灾难恢复计划
