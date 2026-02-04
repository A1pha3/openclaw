---
summary: "`openclaw devices` 命令参考（设备配对 + token 轮换/撤销）"
read_when:
  - 正在批准设备配对请求
  - 需要轮换或撤销设备 token
title: "devices"
---

# `openclaw devices`

管理设备配对请求和设备作用域的 token。

## 为什么需要设备管理

设备是 OpenClaw 网络的端点：

- **跨设备协作**：让多个设备（Mac、iPhone、Android）协同工作
- **安全控制**：精细控制每个设备的访问权限
- **Token 管理**：安全地轮换和撤销访问凭据
- **审计追踪**：了解哪些设备连接到你的网关

## 命令

### 列出设备

```bash
openclaw devices list
openclaw devices list --json
```

列出待处理的配对请求和已配对的设备。

**输出示例**：

```
Paired Devices:
  ID              Name          Role      Last Seen
  ──              ────          ────      ─────────
  dev_abc123      MacBook Pro   operator  2 minutes ago
  dev_def456      iPhone 15     node      5 minutes ago

Pending Requests:
  ID              Device        Requested
  ──              ──────        ─────────
  req_xyz789      iPad Air      10 minutes ago
```

### 批准配对请求

```bash
openclaw devices approve <requestId>
```

批准待处理的设备配对请求。

```bash
openclaw devices approve req_xyz789
```

**输出**：

```
✓ Approved device pairing request
  Device: iPad Air
  ID: dev_ghi012
  Role: node
```

### 拒绝配对请求

```bash
openclaw devices reject <requestId>
```

拒绝待处理的设备配对请求。

```bash
openclaw devices reject req_xyz789
```

**输出**：

```
✓ Rejected device pairing request
  Request ID: req_xyz789
```

### 轮换设备 Token

```bash
openclaw devices rotate --device <id> --role <role> [--scope <scope...>]
```

为特定角色轮换设备 token（可选择更新作用域）。

```bash
# 轮换 operator token
openclaw devices rotate --device dev_abc123 --role operator

# 轮换并更新作用域
openclaw devices rotate --device dev_abc123 --role operator \
  --scope operator.read --scope operator.write
```

**输出**：

```
✓ Rotated device token
  Device: dev_abc123
  Role: operator
  New Token: eyJhbGc...（敏感信息，请安全保存）
```

**警告**：Token 轮换会返回新的 token。请像对待密码一样保护它。

### 撤销设备 Token

```bash
openclaw devices revoke --device <id> --role <role>
```

撤销特定角色的设备 token。

```bash
openclaw devices revoke --device dev_def456 --role node
```

**输出**：

```
✓ Revoked device token
  Device: dev_def456
  Role: node
```

## 通用选项

| 选项 | 说明 |
|------|------|
| `--url <url>` | 网关 WebSocket URL（配置了 `gateway.remote.url` 时默认使用） |
| `--token <token>` | 网关 token（如果需要） |
| `--password <password>` | 网关密码（密码认证） |
| `--timeout <ms>` | RPC 超时 |
| `--json` | JSON 输出（推荐用于脚本） |

## 角色和作用域

### 角色

| 角色 | 说明 |
|------|------|
| `operator` | 完全控制权限，用于管理设备 |
| `node` | 节点设备，执行本地操作 |

### 常用作用域

| 作用域 | 说明 |
|--------|------|
| `operator.read` | 读取操作权限 |
| `operator.write` | 写入操作权限 |
| `operator.admin` | 管理员权限 |
| `operator.pairing` | 配对管理权限 |
| `node.camera` | 相机访问权限 |
| `node.screen` | 屏幕访问权限 |
| `node.system` | 系统命令权限 |

## 权限要求

这些命令需要 `operator.pairing`（或 `operator.admin`）作用域。

## 配对流程

1. **新设备请求配对**：设备通过 Bonjour 或手动输入网关地址发起配对
2. **查看待处理请求**：`openclaw devices list`
3. **批准或拒绝**：`openclaw devices approve <id>` 或 `reject`
4. **设备获得 token**：配对成功后设备自动获得访问 token
5. **设备连接网关**：设备使用 token 连接并注册为节点

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 看不到配对请求 | 网络问题 | 确保设备在同一网络或可访问网关 |
| 批准失败 | 请求已过期 | 让设备重新发起配对 |
| Token 无效 | 已被撤销或过期 | 重新配对设备 |
| 权限不足 | 缺少作用域 | 使用具有 `operator.pairing` 权限的 token |

## 安全最佳实践

1. **定期轮换 Token**：定期轮换敏感设备的 token
2. **最小权限原则**：只授予必要的作用域
3. **监控设备活动**：定期检查已连接设备列表
4. **撤销未使用设备**：移除不再使用的设备
5. **安全存储 Token**：新 token 只显示一次，请安全保存
