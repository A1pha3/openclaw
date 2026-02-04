---
summary: "在一台主机上运行多个 OpenClaw 网关（隔离、端口和配置文件）"
read_when:
  - "在同一台机器上运行多个网关"
  - "你需要每个网关的隔离配置/状态/端口"
title: "多个网关"
---

# 多个网关（同一主机）

大多数设置应该使用一个网关，因为单个网关可以处理多个消息连接和代理。如果你需要更强的隔离或冗余（例如，救援机器人），使用隔离的配置文件/端口运行独立的网关。

## 隔离清单（必需）

- `OPENCLAW_CONFIG_PATH` — 每个实例的配置文件
- `OPENCLAW_STATE_DIR` — 每个实例的会话、凭据、缓存
- `agents.defaults.workspace` — 每个实例的工作区根目录
- `gateway.port`（或 `--port`）— 每个实例唯一
- 派生端口（浏览器/画布）不能重叠

如果这些共享，你会遇到配置竞争和端口冲突。

## 推荐：配置文件（`--profile`）

配置文件自动作用域 `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` 并为服务名称添加后缀。

```bash
# 主
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# 救援
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

每个配置文件的服务：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## 救援机器人指南

在同一主机上运行第二个网关，有自己的：

- 配置文件
- 状态目录
- 工作区
- 基础端口（加上派生端口）

这将救援机器人与主机器人隔离，以便在主机器人宕机时可以进行调试或应用配置更改。

端口间距：在基础端口之间保留至少 20 个端口，这样派生的浏览器/画布/CDP 端口永远不会碰撞。

### 如何安装（救援机器人）

```bash
# 主机器人（现有或新的，不带 --profile 参数）
# 运行在端口 18789 + Chrome CDC/画布/... 端口
openclaw onboard
openclaw gateway install

# 救援机器人（隔离的配置文件 + 端口）
openclaw --profile rescue onboard
# 注意：
# - 工作区名称将按默认后缀 -rescue
# - 端口应至少为 18789 + 20 端口，
#   最好选择完全不同的基础端口，如 19789，
# - 其余入职流程与正常相同

# 要安装服务（如果入职期间没有自动发生）
openclaw --profile rescue gateway install
```

## 端口映射（派生）

基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）。

- 浏览器控制服务端口 = base + 2（仅环回）
- `canvasHost.port = base + 4`
- 浏览器配置文件 CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配

如果你在配置或环境中覆盖了这些，你必须保持每个实例唯一。

## 浏览器/CDP 注意事项（常见陷阱）

- **不要**将 `browser.cdpUrl` 固定到多个实例上的相同值。
- 每个实例需要自己的浏览器控制端口和 CDP 范围（从其网关端口派生）。
- 如果你需要显式 CDP 端口，为每个实例设置 `browser.profiles.<name>.cdpPort`。
- 远程 Chrome：使用 `browser.profiles.<name>.cdpUrl`（每个配置文件，每个实例）。

## 手动环境示例

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## 快速检查

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
