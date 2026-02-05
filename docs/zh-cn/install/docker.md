---
summary: "Docker 部署 OpenClaw：容器化网关和代理沙箱配置的完整指南"
read_when:
  - 想要容器化部署网关
  - 验证 Docker 安装流程
  - 配置代理沙箱隔离
title: "Docker 部署指南"
---

# 🐳 Docker 部署指南

本文档介绍如何使用 Docker 部署 OpenClaw，包括容器化网关和代理沙箱配置。Docker 部署适用于需要环境隔离、可复现部署或服务器运行的场景。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 Docker 部署的适用场景
- [ ] 能够使用 docker-setup.sh 完成快速部署
- [ ] 掌握容器化网关的基本配置
- [ ] 理解代理沙箱的概念和作用

### 进阶目标（建议掌握）

- [ ] 能够配置额外的挂载和持久化
- [ ] 掌握沙箱安全配置选项
- [ ] 能够构建自定义沙箱镜像
- [ ] 理解 Docker 网络和权限配置

### 专家目标（挑战）

- [ ] 能够设计生产级的 Docker 部署架构
- [ ] 掌握多容器编排和负载均衡
- [ ] 能够优化容器性能和资源使用
- [ ] 建立容器安全基线和监控策略

---

## 🤔 第一部分：Docker 适合我吗？

### 1.1 适用场景分析

| 场景 | 推荐 | 理由 |
|------|------|------|
| 隔离的、可丢弃的网关环境 | ✅ Docker | 环境隔离，随时重建 |
| 没有本地安装的主机上运行 | ✅ Docker | 只需 Docker Engine |
| 在自己机器上开发，追求最快开发循环 | ❌ 普通安装 | Docker 增加开销 |
| 生产环境部署 | ✅ Docker | 一致性、可复现 |
| CI/CD 集成测试 | ✅ Docker | 环境可控 |

### 1.2 两种 Docker 使用方式

| 类型 | 说明 | 网关位置 |
|------|------|----------|
| **容器化网关** | 完整的 OpenClaw 在 Docker 中运行 | 容器内 |
| **代理沙箱** | 主机网关 + Docker 隔离的代理工具 | 主机上 |

**沙箱说明**：代理沙箱使用 Docker 实现隔离，但**不需要**整个网关在 Docker 中运行。

### 1.3 Docker vs 普通安装对比

| 对比维度 | Docker 部署 | 普通安装 |
|----------|-------------|----------|
| 环境隔离 | ✅ 完全隔离 | ❌ 与主机共享 |
| 环境一致性 | ✅ 保证一致 | ❌ 依赖主机环境 |
| 资源占用 | ⚠️ 有额外开销 | ✅ 原生性能 |
| 更新方式 | 重建容器 | 更新包 |
| 数据持久化 | 需要挂载 | 直接存储 |
| 学习曲线 | ⚠️ 较陡 | ⭐ 简单 |

---

## 📋 第二部分：系统要求

### 2.1 必需组件

| 组件 | 要求 | 说明 |
|------|------|------|
| Docker Desktop 或 Docker Engine | 最新版 | 容器运行时 |
| Docker Compose | v2+ | 编排工具 |
| 磁盘空间 | 10GB+ | 镜像和日志 |
| 内存 | 4GB+ | 容器运行 |

### 2.2 检查 Docker 环境

```bash
# 检查 Docker 版本
docker --version

# 检查 Docker Compose 版本
docker compose version

# 验证 Docker 运行
docker run hello-world
```

---

## 🚀 第三部分：容器化网关（Docker Compose）

### 3.1 快速开始（推荐）

从仓库根目录运行：

```bash
./docker-setup.sh
```

**此脚本会**：

| 步骤 | 操作 |
|------|------|
| 1 | 构建网关镜像 |
| 2 | 运行引导向导 |
| 3 | 打印可选的提供者设置提示 |
| 4 | 通过 Docker Compose 启动网关 |
| 5 | 生成网关令牌并写入 `.env` |

### 3.2 完成后验证

```bash
# 在浏览器中打开
http://127.0.0.1:18789/

# 在 Control UI 中粘贴令牌
# 设置 → token
```

### 3.3 持久化目录

配置和工作区写入主机位置：

| 主机目录 | 容器路径 | 用途 |
|----------|----------|------|
| `~/.openclaw/` | `/home/node/.openclaw/` | 配置和凭证 |
| `~/.openclaw/workspace` | `/home/node/.openclaw/workspace/` | 工作区 |

**VPS 部署**：参见 [Hetzner (Docker VPS) 文档](/zh-CN/platforms/hetzner)。

### 3.4 环境变量选项

| 变量 | 说明 | 示例 |
|------|------|------|
| `OPENCLAW_DOCKER_APT_PACKAGES` | 构建时安装额外的 apt 包 | `ffmpeg build-essential` |
| `OPENCLAW_EXTRA_MOUNTS` | 添加额外的主机绑定挂载 | 参见下方 |
| `OPENCLAW_HOME_VOLUME` | 在命名卷中持久化 `/home/node` | `openclaw_home` |

### 3.5 手动流程（compose）

```bash
# 构建镜像
docker build -t openclaw:local -f Dockerfile .

# 运行引导
docker compose run --rm openclaw-cli onboard

# 启动网关
docker compose up -d openclaw-gateway
```

---

## 📁 第四部分：额外挂载（可选）

### 4.1 配置额外挂载

如果需要将额外的主机目录挂载到容器中，在运行 `docker-setup.sh` 前设置：

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

### 4.2 格式说明

**逗号分隔**的 Docker 绑定挂载列表，格式为：

```
<host_path>:<container_path>:<mode>
```

| 模式 | 说明 |
|------|------|
| `ro` | 只读 |
| `rw` | 可读写（默认） |

### 4.3 注意事项

| 注意事项 | 说明 |
|----------|------|
| macOS/Windows | 路径必须与 Docker Desktop 共享 |
| 修改后 | 需重新运行 `docker-setup.sh` |
| 配置位置 | `docker-compose.extra.yml` 是自动生成的 |

---

## 💾 第五部分：持久化容器主目录（可选）

### 5.1 使用命名卷

如果希望 `/home/node` 在容器重建后保持，设置命名卷：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

### 5.2 组合使用

```bash
# 持久化 + 额外挂载
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro"
./docker-setup.sh
```

### 5.3 数据管理命令

```bash
# 查看卷
docker volume ls | grep openclaw

# 查看卷详情
docker volume inspect openclaw_home

# 删除卷（谨慎！会丢失数据）
docker volume rm openclaw_home
```

---

## 📦 第六部分：安装额外系统包（可选）

### 6.1 安装构建工具

如果需要镜像中有系统包（如构建工具或媒体库）：

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

### 6.2 常用包列表

| 包名 | 用途 |
|------|------|
| `ffmpeg` | 媒体处理 |
| `build-essential` | C/C++ 编译 |
| `git` | 版本控制 |
| `curl` | 网络工具 |
| `vim` | 文本编辑 |

### 6.3 安装时机

这些包在**镜像构建时**安装，即使容器删除也会保留。

---

## 🔧 第七部分：渠道配置（可选）

### 7.1 使用 CLI 容器配置

**WhatsApp（扫码）**：

```bash
docker compose run --rm openclaw-cli channels login
```

**Telegram（机器人令牌）**：

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

**Discord（机器人令牌）**：

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

### 7.2 详细文档

| 渠道 | 文档链接 |
|------|----------|
| WhatsApp | [WhatsApp 文档](/zh-CN/channels/whatsapp) |
| Telegram | [Telegram 文档](/zh-CN/channels/telegram) |
| Discord | [Discord 文档](/zh-CN/channels/discord) |

---

## 🩺 第八部分：健康检查

### 8.1 命令行健康检查

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### 8.2 E2E 冒烟测试

```bash
scripts/e2e/onboard-docker.sh
```

### 8.3 QR 导入测试

```bash
pnpm test:docker:qr
```

---

## 🛡️ 第九部分：代理沙箱（主机网关 + Docker 工具）

### 9.1 沙箱概述

当启用 `agents.defaults.sandbox` 时，**非主会话**的工具在 Docker 容器内运行。网关保持在主机上，但工具执行是隔离的。

**详细文档**：[沙箱配置](/zh-CN/gateway/sandboxing)

### 9.2 沙箱范围配置

| 设置 | 说明 |
|------|------|
| `scope: "agent"` | 每个代理一个容器 + 工作区（默认） |
| `scope: "session"` | 每个会话一个隔离环境 |
| `scope: "shared"` | ⚠️ 所有会话共享一个容器（禁用隔离） |

### 9.3 默认行为

| 配置项 | 默认值 |
|--------|--------|
| 镜像 | `openclaw-sandbox:bookworm-slim` |
| 容器策略 | 每个代理一个容器 |
| 网络 | 默认 `none`（无出站） |
| 自动清理 | 空闲 > 24h 或存在 > 7d |

### 9.4 默认工具策略

| 允许 | 拒绝 |
|------|------|
| `exec`, `process`, `read`, `write`, `edit` | `browser`, `canvas`, `nodes` |
| `sessions_list`, `sessions_history` | `cron`, `discord`, `gateway` |
| `sessions_send`, `sessions_spawn`, `session_status` | |

### 9.5 启用沙箱配置

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",    // off | non-main | all
        scope: "agent",      // session | agent | shared
        workspaceAccess: "none",  // none | ro | rw
      }
    }
  }
}
```

---

## 🔧 第十部分：构建沙箱镜像

### 10.1 构建默认沙箱镜像

```bash
scripts/sandbox-setup.sh
```

**生成镜像**：`openclaw-sandbox:bookworm-slim`

### 10.2 构建浏览器沙箱镜像

如果需要在沙箱中运行浏览器工具：

```bash
scripts/sandbox-browser-setup.sh
```

**生成镜像**：`openclaw-sandbox-browser:bookworm-slim`（包含 Chromium + CDP）

**配置启用**：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

---

## 🔒 第十一部分：沙箱安全配置

### 11.1 完整配置示例

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "agent",
        workspaceAccess: "none",
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          }
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7
        }
      }
    }
  }
}
```

### 11.2 配置项说明

| 配置项 | 说明 | 安全级别 |
|--------|------|----------|
| `network: "none"` | 禁用网络 | ⭐⭐⭐⭐⭐ |
| `capDrop: ["ALL"]` | 移除所有权限 | ⭐⭐⭐⭐⭐ |
| `readOnlyRoot` | 根文件系统只读 | ⭐⭐⭐⭐ |
| `tmpfs` | 使用临时文件系统 | ⭐⭐⭐ |
| `memory` | 内存限制 | ⭐⭐⭐ |
| `cpus` | CPU 限制 | ⭐⭐⭐ |
| `pidsLimit` | 进程数限制 | ⭐⭐⭐ |
| `user` | 非 root 用户运行 | ⭐⭐⭐⭐ |

---

## 📋 第十二部分：Docker 注意事项

### 12.1 网关绑定

- 默认绑定为 `lan`（容器使用）
- 如果需要本机访问，配置 bind 为 `loopback`

### 12.2 会话存储

会话存储在 `~/.openclaw/agents/<agentId>/sessions/`

---

## 🐛 第十三部分：故障排除

### 13.1 常见问题与解决

| 问题 | 解决方案 |
|------|----------|
| 镜像缺失 | 运行 `scripts/sandbox-setup.sh` |
| 容器未运行 | 检查 Docker 服务，按需自动创建 |
| 权限错误 | 设置 `docker.user` 匹配挂载目录的 UID:GID |
| 自定义工具找不到 | 设置 `docker.env.PATH` 或在 Dockerfile 中添加 |

### 13.2 诊断命令

```bash
# 查看容器状态
docker compose ps

# 查看容器日志
docker compose logs -f openclaw-gateway

# 进入容器排查
docker compose exec openclaw-gateway bash
```

---

## 📝 第十四部分：相关文档

| 文档 | 说明 |
|------|------|
| [安装指南](/zh-CN/install) | 其他安装选项 |
| [沙箱配置](/zh-CN/gateway/sandboxing) | 完整沙箱文档 |
| [网关配置](/zh-CN/gateway/configuration) | 所有配置选项 |

---

## 🎓 章节总结

### 学习目标完成检查

#### 基础目标（必掌握）

- [ ] 理解 Docker 部署的适用场景
- [ ] 能够使用 docker-setup.sh 完成快速部署
- [ ] 掌握容器化网关的基本配置
- [ ] 理解代理沙箱的概念和作用

#### 进阶目标（建议掌握）

- [ ] 能够配置额外的挂载和持久化
- [ ] 掌握沙箱安全配置选项
- [ ] 能够构建自定义沙箱镜像
- [ ] 理解 Docker 网络和权限配置

#### 专家目标（挑战）

- [ ] 能够设计生产级的 Docker 部署架构
- [ ] 掌握多容器编排和负载均衡
- [ ] 能够优化容器性能和资源使用
- [ ] 建立容器安全基线和监控策略

### Docker 部署决策树

```
需要 Docker 部署？
    │
    ├── 容器化网关？
    │       ├── 是 → ./docker-setup.sh
    │       └── 否 → 进入下一步
    │
    ├── 只需要代理沙箱？
    │       ├── 是 → 配置 sandbox 选项
    │       └── 否 → 进入下一步
    │
    └── 需要自定义镜像？
            ├── 是 → 编写 Dockerfile
            └── 否 → 使用默认镜像
```

### 专家建议

| 场景 | 建议 |
|------|------|
| 快速部署 | 使用 `./docker-setup.sh` |
| 开发测试 | 主机网关 + 沙箱 |
| 生产环境 | 容器化网关 + 沙箱 |
| 安全敏感 | 严格的网络和权限配置 |
| 多租户 | scope: "session" |

---

**需要帮助？** 查看 [常见问题](/zh-CN/help/faq) 或加入 [Discord 社区](https://discord.gg/clawd)。
