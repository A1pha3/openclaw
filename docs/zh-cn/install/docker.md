---
summary: "Docker 部署 OpenClaw：容器化网关和代理沙箱配置"
read_when:
  - 想要容器化部署网关
  - 验证 Docker 安装流程
  - 配置代理沙箱隔离
title: "Docker 部署"
---

# 🐳 Docker 部署

Docker 是**可选**的。仅在需要容器化网关或验证 Docker 流程时使用。

---

## 🤔 Docker 适合我吗？

| 场景 | 推荐 |
|------|------|
| 想要隔离的、可丢弃的网关环境 | ✅ 使用 Docker |
| 在没有本地安装的主机上运行 | ✅ 使用 Docker |
| 在自己机器上开发，追求最快的开发循环 | ❌ 使用普通安装 |

**沙箱说明**：代理沙箱也使用 Docker，但**不需要**整个网关在 Docker 中运行。详见 [沙箱配置](/zh-CN/gateway/sandboxing)。

本指南涵盖：
- **容器化网关**：完整的 OpenClaw 在 Docker 中运行
- **代理沙箱**：主机网关 + Docker 隔离的代理工具

---

## 📋 系统要求

- Docker Desktop（或 Docker Engine）+ Docker Compose v2
- 足够的磁盘空间用于镜像和日志

---

## 🚀 容器化网关（Docker Compose）

### 快速开始（推荐）

从 repo 根目录：

```bash
./docker-setup.sh
```

此脚本会：
1. 构建网关镜像
2. 运行引导向导
3. 打印可选的提供者设置提示
4. 通过 Docker Compose 启动网关
5. 生成网关令牌并写入 `.env`

### 环境变量选项

| 变量 | 说明 |
|------|------|
| `OPENCLAW_DOCKER_APT_PACKAGES` | 构建时安装额外的 apt 包 |
| `OPENCLAW_EXTRA_MOUNTS` | 添加额外的主机绑定挂载 |
| `OPENCLAW_HOME_VOLUME` | 在命名卷中持久化 `/home/node` |

### 完成后

1. 在浏览器中打开 `http://127.0.0.1:18789/`
2. 在 Control UI 中粘贴令牌（设置 → token）

配置和工作区写入主机：
- `~/.openclaw/`
- `~/.openclaw/workspace`

**在 VPS 上运行？** 参见 [Hetzner (Docker VPS)](/zh-CN/platforms/hetzner)。

### 手动流程（compose）

```bash
# 构建镜像
docker build -t openclaw:local -f Dockerfile .

# 运行引导
docker compose run --rm openclaw-cli onboard

# 启动网关
docker compose up -d openclaw-gateway
```

---

## 📁 额外挂载（可选）

如果需要将额外的主机目录挂载到容器中，在运行 `docker-setup.sh` 前设置 `OPENCLAW_EXTRA_MOUNTS`：

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

**格式**：逗号分隔的 Docker 绑定挂载列表。

**注意事项**：
- macOS/Windows 上的路径必须与 Docker Desktop 共享
- 修改后需重新运行 `docker-setup.sh`
- `docker-compose.extra.yml` 是自动生成的，不要手动编辑

---

## 💾 持久化容器主目录（可选）

如果希望 `/home/node` 在容器重建后保持，设置命名卷：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

**组合使用**：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro"
./docker-setup.sh
```

---

## 📦 安装额外系统包（可选）

如果需要镜像中有系统包（如构建工具或媒体库）：

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

这会在镜像构建时安装包，即使容器删除也会保留。

---

## 🔧 渠道配置（可选）

使用 CLI 容器配置渠道：

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

详见：[WhatsApp](/zh-CN/channels/whatsapp)、[Telegram](/zh-CN/channels/telegram)、[Discord](/zh-CN/channels/discord)

---

## 🩺 健康检查

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### E2E 冒烟测试

```bash
scripts/e2e/onboard-docker.sh
```

### QR 导入测试

```bash
pnpm test:docker:qr
```

---

## 🛡️ 代理沙箱（主机网关 + Docker 工具）

深入了解：[沙箱配置](/zh-CN/gateway/sandboxing)

### 工作原理

当启用 `agents.defaults.sandbox` 时，**非主会话**的工具在 Docker 容器内运行。网关保持在主机上，但工具执行是隔离的：

| 设置 | 说明 |
|------|------|
| `scope: "agent"` | 每个代理一个容器 + 工作区（默认） |
| `scope: "session"` | 每个会话一个隔离环境 |
| `scope: "shared"` | ⚠️ 所有会话共享一个容器（禁用隔离） |

### 默认行为

- **镜像**：`openclaw-sandbox:bookworm-slim`
- **每个代理一个容器**
- **网络**：默认 `none`（无出站）
- **自动清理**：空闲 > 24h 或 存在 > 7d

### 默认工具策略

| 允许 | 拒绝 |
|------|------|
| `exec`, `process`, `read`, `write`, `edit` | `browser`, `canvas`, `nodes` |
| `sessions_list`, `sessions_history` | `cron`, `discord`, `gateway` |
| `sessions_send`, `sessions_spawn`, `session_status` | |

### 启用沙箱

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
        workspaceAccess: "none",  // none | ro | rw
      }
    }
  }
}
```

### 构建默认沙箱镜像

```bash
scripts/sandbox-setup.sh
```

这会构建 `openclaw-sandbox:bookworm-slim`。

### 构建浏览器沙箱镜像

如果需要在沙箱中运行浏览器工具：

```bash
scripts/sandbox-browser-setup.sh
```

这会构建 `openclaw-sandbox-browser:bookworm-slim`，包含 Chromium + CDP。

**配置**：

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

## 🔒 沙箱安全配置

完整的安全配置选项：

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

---

## 📋 Docker 注意事项

- 网关绑定默认为 `lan`（容器使用）
- 会话存储在 `~/.openclaw/agents/<agentId>/sessions/`

---

## 🐛 故障排除

| 问题 | 解决方案 |
|------|----------|
| 镜像缺失 | 运行 `scripts/sandbox-setup.sh` |
| 容器未运行 | 按需自动创建，检查 Docker 服务 |
| 权限错误 | 设置 `docker.user` 匹配挂载目录的 UID:GID |
| 自定义工具找不到 | 设置 `docker.env.PATH` 或在 Dockerfile 中添加 `/etc/profile.d/` 脚本 |

---

## 📝 相关文档

- [安装指南](/zh-CN/install) - 其他安装选项
- [沙箱配置](/zh-CN/gateway/sandboxing) - 完整沙箱文档
- [网关配置](/zh-CN/gateway/configuration) - 所有配置选项
