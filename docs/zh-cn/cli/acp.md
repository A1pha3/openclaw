---
summary: "运行 ACP 桥接用于 IDE 集成"
read_when:
  - 设置基于 ACP 的 IDE 集成
  - 调试 ACP 会话路由到网关
title: "acp"
---

# openclaw acp

运行与 OpenClaw 网关通信的 ACP（Agent Client Protocol）桥接。

## 为什么需要 ACP

ACP 让 IDE 能够直接使用 OpenClaw 代理：

- **IDE 集成**：在编辑器中直接使用代理
- **会话同步**：IDE 会话与网关会话映射
- **代理选择**：针对不同任务使用不同代理
- **统一体验**：IDE、CLI、聊天使用相同的代理能力

## 工作原理

此命令通过 stdio 使用 ACP 协议与 IDE 通信，并通过 WebSocket 将提示转发到网关。它保持 ACP 会话与网关会话键的映射。

## 基本用法

```bash
# 本地网关
openclaw acp

# 远程网关
openclaw acp --url wss://gateway-host:18789 --token <token>

# 附加到现有会话键
openclaw acp --session agent:main:main

# 按标签附加（必须已存在）
openclaw acp --session-label "support inbox"

# 首次提示前重置会话
openclaw acp --session agent:main:main --reset-session
```

## ACP 客户端（调试）

使用内置 ACP 客户端在没有 IDE 的情况下测试桥接。它会启动 ACP 桥接并让你交互式输入提示。

```bash
# 基本客户端
openclaw acp client

# 指向远程网关
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# 覆盖服务器命令
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

## 如何使用

当 IDE（或其他客户端）使用 Agent Client Protocol 并且你想让它驱动 OpenClaw 网关会话时使用 ACP。

1. 确保网关正在运行（本地或远程）
2. 配置网关目标（配置或标志）
3. 将 IDE 配置为通过 stdio 运行 `openclaw acp`

### 持久化配置

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

### 直接运行（不写配置）

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

## 选择代理

ACP 不直接选择代理。它通过网关会话键路由。

使用代理作用域的会话键来定位特定代理：

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

每个 ACP 会话映射到一个网关会话键。一个代理可以有多个会话；除非你覆盖键或标签，否则 ACP 默认使用隔离的 `acp:<uuid>` 会话。

## Zed 编辑器设置

在 `~/.config/zed/settings.json` 中添加自定义 ACP 代理：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

### 指定特定网关或代理

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

在 Zed 中打开 Agent 面板并选择 "OpenClaw ACP" 开始对话。

## 会话映射

默认情况下，ACP 会话获得带有 `acp:` 前缀的隔离网关会话键。要重用已知会话，传递会话键或标签：

| 选项 | 说明 |
|------|------|
| `--session <key>` | 使用特定网关会话键 |
| `--session-label <label>` | 按标签解析现有会话 |
| `--reset-session` | 为该键创建新会话 ID（相同键，新转录） |

如果你的 ACP 客户端支持元数据，可以按会话覆盖：

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

了解更多关于会话键：[/zh-cn/concepts/session](/zh-cn/concepts/session)

## 选项

| 选项 | 说明 |
|------|------|
| `--url <url>` | 网关 WebSocket URL |
| `--token <token>` | 网关认证 token |
| `--password <password>` | 网关认证密码 |
| `--session <key>` | 默认会话键 |
| `--session-label <label>` | 要解析的默认会话标签 |
| `--require-existing` | 如果会话键/标签不存在则失败 |
| `--reset-session` | 首次使用前重置会话键 |
| `--no-prefix-cwd` | 不在提示前添加工作目录 |
| `--verbose, -v` | 详细日志到 stderr |

### acp client 选项

| 选项 | 说明 |
|------|------|
| `--cwd <dir>` | ACP 会话的工作目录 |
| `--server <command>` | ACP 服务器命令（默认：`openclaw`） |
| `--server-args <args...>` | 传递给 ACP 服务器的额外参数 |
| `--server-verbose` | 在 ACP 服务器上启用详细日志 |
| `--verbose, -v` | 详细客户端日志 |

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| IDE 无法连接 | 路径问题 | 确保 `openclaw` 在 PATH 中 |
| 会话未找到 | 会话不存在 | 移除 `--require-existing` |
| 认证失败 | Token 无效 | 检查网关认证 |
