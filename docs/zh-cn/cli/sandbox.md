---
title: "sandbox"
summary: "管理沙箱容器并检查有效沙箱策略"
read_when:
  - 管理沙箱容器或调试沙箱/工具策略行为
---

# 沙箱 CLI

管理基于 Docker 的沙箱容器，用于隔离代理执行。

## 为什么需要沙箱

沙箱为代理执行提供安全隔离：

- **安全边界**：限制代理对主机系统的访问
- **环境隔离**：每个会话/代理有独立的运行环境
- **风险控制**：即使代理被恶意利用，影响也被限制在容器内

## 概述

OpenClaw 可以在隔离的 Docker 容器中运行代理以增强安全性。`sandbox` 命令帮助你管理这些容器，特别是在更新或配置更改后。

## 命令

### `openclaw sandbox explain`

检查**有效的**沙箱模式/作用域/工作区访问、沙箱工具策略和提权门控（带修复配置键路径）。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### `openclaw sandbox list`

列出所有沙箱容器及其状态和配置。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # 只列出浏览器容器
openclaw sandbox list --json     # JSON 输出
```

**输出包括：**

| 信息 | 说明 |
|------|------|
| 容器名称 | 容器标识 |
| 状态 | 运行中/已停止 |
| Docker 镜像 | 是否与配置匹配 |
| 年龄 | 创建以来的时间 |
| 空闲时间 | 上次使用以来的时间 |
| 关联会话/代理 | 所属的会话或代理 |

### `openclaw sandbox recreate`

移除沙箱容器以强制使用更新的镜像/配置重新创建。

```bash
openclaw sandbox recreate --all                # 重建所有容器
openclaw sandbox recreate --session main       # 特定会话
openclaw sandbox recreate --agent mybot        # 特定代理
openclaw sandbox recreate --browser            # 只重建浏览器容器
openclaw sandbox recreate --all --force        # 跳过确认
```

**选项：**

| 选项 | 说明 |
|------|------|
| `--all` | 重建所有沙箱容器 |
| `--session <key>` | 重建特定会话的容器 |
| `--agent <id>` | 重建特定代理的容器 |
| `--browser` | 只重建浏览器容器 |
| `--force` | 跳过确认提示 |

**重要**：容器在代理下次使用时会自动重新创建。

## 使用场景

### 更新 Docker 镜像后

```bash
# 拉取新镜像
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 更新配置使用新镜像
# 编辑配置：agents.defaults.sandbox.docker.image（或 agents.list[].sandbox.docker.image）

# 重建容器
openclaw sandbox recreate --all
```

### 更改沙箱配置后

```bash
# 编辑配置：agents.defaults.sandbox.*（或 agents.list[].sandbox.*）

# 重建以应用新配置
openclaw sandbox recreate --all
```

### 更改 setupCommand 后

```bash
openclaw sandbox recreate --all
# 或只重建一个代理：
openclaw sandbox recreate --agent family
```

### 只针对特定代理

```bash
# 只更新一个代理的容器
openclaw sandbox recreate --agent alfred
```

## 为什么需要这个命令？

**问题**：当你更新沙箱 Docker 镜像或配置时：

- 现有容器继续使用旧设置运行
- 容器只在闲置 24 小时后才被清理
- 经常使用的代理会无限期保持旧容器运行

**解决方案**：使用 `openclaw sandbox recreate` 强制移除旧容器。它们会在下次需要时使用当前设置自动重新创建。

**提示**：优先使用 `openclaw sandbox recreate` 而不是手动 `docker rm`。它使用网关的容器命名约定，避免在作用域/会话键更改时出现不匹配。

## 配置

沙箱设置在 `~/.openclaw/openclaw.json` 的 `agents.defaults.sandbox` 下（每个代理的覆盖在 `agents.list[].sandbox`）：

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",           // off, non-main, all
        "scope": "agent",        // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... 更多 Docker 选项
        },
        "prune": {
          "idleHours": 24,       // 闲置 24 小时后自动清理
          "maxAgeDays": 7        // 7 天后自动清理
        }
      }
    }
  }
}
```

### 沙箱模式

| 模式 | 说明 |
|------|------|
| `off` | 禁用沙箱 |
| `non-main` | 只对非主会话启用沙箱 |
| `all` | 所有会话都使用沙箱 |

### 作用域

| 作用域 | 说明 |
|--------|------|
| `session` | 每个会话一个容器 |
| `agent` | 每个代理一个容器 |
| `shared` | 共享容器 |

## 相关文档

- [沙箱文档](/zh-cn/gateway/sandboxing)
- [代理配置](/zh-cn/concepts/agent-workspace)
- [Doctor 命令](/zh-cn/gateway/doctor) - 检查沙箱设置
