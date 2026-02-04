---
summary: "`openclaw node` 命令参考（无头节点主机）"
read_when:
  - 运行无头节点主机
  - 为 system.run 配对非 macOS 节点
title: "node"
---

# `openclaw node`

运行连接到网关 WebSocket 并在此机器上公开 `system.run` / `system.which` 的**无头节点主机**。

## 为什么使用节点主机

当你想让代理**在其他机器上运行命令**而不需要在那里安装完整的 macOS 伴侣应用时，使用节点主机。

### 常见用例

- 在远程 Linux/Windows 机器上运行命令（构建服务器、实验室机器、NAS）
- 保持网关上的执行**沙盒化**，但将批准的运行委托给其他主机
- 为自动化或 CI 节点提供轻量级、无头的执行目标

执行仍然受节点主机上的**执行批准**和每代理允许列表保护，因此你可以保持命令访问范围明确。

## 浏览器代理（零配置）

如果节点上未禁用 `browser.enabled`，节点主机会自动公开浏览器代理。这让代理可以在该节点上使用浏览器自动化，无需额外配置。

如果需要在节点上禁用：

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## 运行（前台）

```bash
openclaw node run --host <gateway-host> --port 18789
```

### 选项

| 选项 | 说明 |
|------|------|
| `--host <host>` | 网关 WebSocket 主机（默认：`127.0.0.1`） |
| `--port <port>` | 网关 WebSocket 端口（默认：`18789`） |
| `--tls` | 使用 TLS 连接网关 |
| `--tls-fingerprint <sha256>` | 期望的 TLS 证书指纹（sha256） |
| `--node-id <id>` | 覆盖节点 ID（清除配对 token） |
| `--display-name <name>` | 覆盖节点显示名称 |

## 服务（后台）

将无头节点主机安装为用户服务。

```bash
openclaw node install --host <gateway-host> --port 18789
```

### 安装选项

| 选项 | 说明 |
|------|------|
| `--host <host>` | 网关 WebSocket 主机 |
| `--port <port>` | 网关 WebSocket 端口 |
| `--tls` | 使用 TLS 连接 |
| `--tls-fingerprint <sha256>` | TLS 证书指纹 |
| `--node-id <id>` | 覆盖节点 ID |
| `--display-name <name>` | 覆盖显示名称 |
| `--runtime <runtime>` | 服务运行时（`node` 或 `bun`） |
| `--force` | 如果已安装则重新安装/覆盖 |

### 管理服务

```bash
openclaw node status    # 查看状态
openclaw node stop      # 停止服务
openclaw node restart   # 重启服务
openclaw node uninstall # 卸载服务
```

服务命令接受 `--json` 用于机器可读输出。

## 配对

首次连接会在网关上创建一个待处理的节点配对请求。通过以下方式批准：

```bash
openclaw nodes pending          # 查看待处理请求
openclaw nodes approve <requestId>  # 批准请求
```

节点主机将其节点 ID、token、显示名称和网关连接信息存储在 `~/.openclaw/node.json`。

## 执行批准

`system.run` 受本地执行批准控制：

- 配置文件：`~/.openclaw/exec-approvals.json`
- 文档：[Exec approvals](/zh-cn/tools/exec-approvals)
- 从网关编辑：`openclaw approvals --node <id|name|ip>`

## 配置示例

节点配置存储在 `~/.openclaw/node.json`：

```json
{
  "nodeId": "node_abc123",
  "token": "eyJhbGc...",
  "displayName": "Build Server",
  "gateway": {
    "host": "gateway.example.com",
    "port": 18789,
    "tls": true
  }
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 连接失败 | 网关不可达 | 检查网络和防火墙 |
| 配对被拒绝 | 未批准 | 在网关上批准配对请求 |
| 命令执行失败 | 不在允许列表 | 添加到执行批准 |
| TLS 错误 | 证书不匹配 | 检查 fingerprint |

## 最佳实践

1. **使用 TLS**：生产环境总是启用 TLS
2. **最小权限**：只批准必要的命令
3. **监控状态**：定期检查节点状态
4. **安全网络**：使用 VPN 或 Tailscale
