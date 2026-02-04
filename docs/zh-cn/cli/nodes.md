---
summary: "CLI 参考 - `openclaw nodes` (列出/状态/批准/调用节点，摄像头/画布/屏幕)"
read_when:
  - 管理已配对的节点（摄像头、屏幕、画布）
  - 需要批准请求或调用节点命令
title: "nodes"
---

# `openclaw nodes`

管理已配对的节点（设备）并调用节点能力。

## 为什么需要这个命令

节点扩展了代理的感知和行动能力：

- **摄像头**：拍照、录像
- **屏幕**：截图、录屏
- **画布**：实时可视化界面
- **系统**：在远程机器上执行命令

## 相关文档

- 节点概述：[节点](/zh-cn/nodes)
- 摄像头：[摄像头节点](/zh-cn/nodes/camera)
- 图片：[图片节点](/zh-cn/nodes/images)

## 通用选项

| 选项 | 说明 |
|------|------|
| `--url` | 网关 WebSocket URL |
| `--token` | 网关令牌 |
| `--timeout` | 请求超时 |
| `--json` | JSON 格式输出 |

## 常用命令

```bash
# 列出所有节点
openclaw nodes list

# 只显示当前已连接的节点
openclaw nodes list --connected

# 显示最近 24 小时内连接过的节点
openclaw nodes list --last-connected 24h

# 查看待批准的节点
openclaw nodes pending

# 批准节点配对请求
openclaw nodes approve <requestId>

# 查看节点状态
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

### 输出说明

`nodes list` 显示待批准和已配对的节点表格。已配对行包含最近连接时间（Last Connect）。

| 过滤选项 | 说明 |
|----------|------|
| `--connected` | 只显示当前已连接的节点 |
| `--last-connected <duration>` | 显示指定时间内连接过的节点（如 `24h`、`7d`） |

## 调用和运行

### invoke - 直接调用

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
```

invoke 选项：

| 选项 | 说明 |
|------|------|
| `--params <json>` | JSON 对象字符串（默认 `{}`） |
| `--invoke-timeout <ms>` | 节点调用超时（默认 15000） |
| `--idempotency-key <key>` | 可选的幂等键 |

### run - 执行命令

```bash
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

`nodes run` 镜像模型的 exec 行为（默认值 + 审批）：

- 读取 `tools.exec.*`（以及 `agents.list[].tools.exec.*` 覆盖）
- 使用 exec 审批（`exec.approval.request`）后再调用 `system.run`
- 当设置了 `tools.exec.node` 时可省略 `--node`
- 需要通告 `system.run` 的节点（macOS 伴侣应用或无头节点主机）

run 选项：

| 选项 | 说明 |
|------|------|
| `--cwd <path>` | 工作目录 |
| `--env <key=val>` | 环境变量覆盖（可重复） |
| `--command-timeout <ms>` | 命令超时 |
| `--invoke-timeout <ms>` | 节点调用超时（默认 30000） |
| `--needs-screen-recording` | 需要屏幕录制权限 |
| `--raw <command>` | 运行 shell 字符串（`/bin/sh -lc` 或 `cmd.exe /c`） |
| `--agent <id>` | 代理作用域的审批/白名单（默认使用配置的代理） |
| `--ask <off\|on-miss\|always>` | 覆盖询问模式 |
| `--security <deny\|allowlist\|full>` | 覆盖安全模式 |

## 使用场景

### 管理设备配对

```bash
# 查看待批准的设备
openclaw nodes pending

# 批准新设备
openclaw nodes approve abc123
```

### 远程执行命令

```bash
# 在特定节点运行命令
openclaw nodes run --node my-mac --raw "ls -la"

# 需要屏幕权限的命令
openclaw nodes run --node my-mac --needs-screen-recording --raw "screencapture screenshot.png"
```

### 调用摄像头

```bash
# 拍照
openclaw nodes invoke --node iphone --command camera.snap

# 带参数调用
openclaw nodes invoke --node iphone --command camera.snap --params '{"quality":"high"}'
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 节点未显示 | 确保节点应用已启动并连接到同一网关 |
| 配对失败 | 检查网络连接和配对码是否正确 |
| 命令超时 | 增加 `--invoke-timeout` 值 |
| 权限错误 | 在节点设备上授予所需权限 |
