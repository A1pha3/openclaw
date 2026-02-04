---
summary: "`openclaw approvals` 命令参考（网关或节点主机的执行批准）"
read_when:
  - 想要从 CLI 编辑执行批准
  - 需要管理网关或节点主机上的允许列表
title: "approvals"
---

# `openclaw approvals`

管理**本地主机**、**网关主机**或**节点主机**的执行批准。默认情况下，命令针对磁盘上的本地批准文件。使用 `--gateway` 针对网关，或使用 `--node` 针对特定节点。

## 为什么需要执行批准

执行批准是安全机制：

- **控制命令执行**：限制代理可以运行的命令
- **分层安全**：不同主机可以有不同的批准规则
- **审计追踪**：了解哪些命令被允许执行
- **灵活管理**：按代理、按路径精细控制

## 相关链接

- 执行批准：[Exec approvals](/zh-cn/tools/exec-approvals)
- 节点：[Nodes](/zh-cn/nodes)

## 常用命令

```bash
# 获取本地批准
openclaw approvals get

# 获取节点批准
openclaw approvals get --node <id|name|ip>

# 获取网关批准
openclaw approvals get --gateway
```

## 从文件替换批准

```bash
# 本地
openclaw approvals set --file ./exec-approvals.json

# 节点
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json

# 网关
openclaw approvals set --gateway --file ./exec-approvals.json
```

## 允许列表辅助

### 添加允许条目

```bash
# 添加路径模式
openclaw approvals allowlist add "~/Projects/**/bin/rg"

# 为特定代理和节点添加
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"

# 为所有代理添加
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"
```

### 移除允许条目

```bash
openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## 选项

| 选项 | 说明 |
|------|------|
| `--node <id>` | 目标节点（使用与 `openclaw nodes` 相同的解析器） |
| `--gateway` | 目标网关主机 |
| `--agent <id>` | 代理 ID（默认 `"*"` 应用于所有代理） |
| `--file <path>` | 批准配置文件路径 |

## 批准文件格式

```json
{
  "allowlist": [
    {
      "agent": "*",
      "pattern": "/usr/bin/*"
    },
    {
      "agent": "main",
      "pattern": "~/Projects/**/bin/*"
    }
  ],
  "denylist": [
    {
      "agent": "*",
      "pattern": "/usr/bin/rm"
    }
  ]
}
```

## 路径模式

支持的模式：

| 模式 | 说明 |
|------|------|
| `*` | 匹配任意字符（不含 `/`） |
| `**` | 匹配任意路径段 |
| `~` | 用户主目录 |
| 绝对路径 | 精确匹配 |

### 示例

```bash
# 允许特定目录下的所有命令
"~/Projects/**/bin/*"

# 允许特定命令
"/usr/bin/uptime"

# 允许所有 brew 安装的命令
"/opt/homebrew/bin/*"
```

## 说明

- `--node` 使用与 `openclaw nodes` 相同的解析器（id、名称、ip 或 id 前缀）
- `--agent` 默认为 `"*"`，应用于所有代理
- 节点主机必须公开 `system.execApprovals.get/set`（macOS 应用或无头节点主机）
- 批准文件按主机存储在 `~/.openclaw/exec-approvals.json`

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 节点不可访问 | 节点未连接 | 检查节点状态 |
| 设置失败 | 权限问题 | 检查节点权限 |
| 命令被拒绝 | 不在允许列表 | 添加到允许列表 |

## 最佳实践

1. **最小权限**：只允许必要的命令
2. **使用模式**：而非逐个添加命令
3. **代理分离**：不同代理有不同权限
4. **定期审计**：检查批准列表
