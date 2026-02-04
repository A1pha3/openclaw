---
summary: "执行审批、白名单和沙箱逃逸提示"
read_when:
  - 配置执行审批或白名单
  - 在 macOS 应用中实现执行审批 UX
  - 审查沙箱逃逸提示及其影响
title: "执行审批"
---

# 执行审批

执行审批是**伴侣应用/节点主机的安全机制**，用于让沙箱化的智能体在真实主机（`gateway` 或 `node`）上运行命令。可以把它想象成一个安全联锁：只有当策略 + 白名单 +（可选的）用户审批都同意时，命令才会被允许执行。

执行审批是在工具策略和提权门控**之上**的额外保护层（除非提权设置为 `full`，这会跳过审批）。

有效策略取 `tools.exec.*` 和审批默认值中**更严格的一个**；如果审批字段省略，则使用 `tools.exec` 的值。

如果伴侣应用 UI **不可用**，任何需要提示的请求都由**询问回退**策略决定（默认：拒绝）。

## 适用范围

执行审批在执行主机上本地强制执行：

| 主机类型 | 位置 |
|----------|------|
| **gateway 主机** | Gateway 机器上的 `openclaw` 进程 |
| **node 主机** | 节点运行器（macOS 伴侣应用或无头节点主机） |

**macOS 分工：**

- **节点主机服务**通过本地 IPC 将 `system.run` 转发给 **macOS 应用**
- **macOS 应用**强制执行审批 + 在 UI 上下文中执行命令

## 设置和存储

审批配置存储在执行主机上的本地 JSON 文件中：

`~/.openclaw/exec-approvals.json`

**示例结构：**

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 策略选项

### 安全级别（`exec.security`）

| 级别 | 说明 |
|------|------|
| **deny** | 阻止所有宿主执行请求 |
| **allowlist** | 仅允许白名单中的命令 |
| **full** | 允许所有（相当于提权模式） |

### 询问模式（`exec.ask`）

| 模式 | 说明 |
|------|------|
| **off** | 从不提示 |
| **on-miss** | 仅在白名单不匹配时提示 |
| **always** | 每条命令都提示 |

### 询问回退（`askFallback`）

如果需要提示但无法访问 UI，回退策略决定：

| 模式 | 说明 |
|------|------|
| **deny** | 阻止 |
| **allowlist** | 仅在白名单匹配时允许 |
| **full** | 允许 |

## 白名单（按智能体）

白名单是**按智能体**配置的。如果存在多个智能体，在 macOS 应用中切换要编辑的智能体。

模式是**大小写不敏感的 glob 匹配**。模式应解析为**二进制路径**（仅基本名称的条目会被忽略）。

旧的 `agents.default` 条目在加载时会迁移到 `agents.main`。

**示例模式：**

```
~/Projects/**/bin/bird
~/.local/bin/*
/opt/homebrew/bin/rg
```

**每个白名单条目跟踪：**

| 字段 | 说明 |
|------|------|
| **id** | 稳定 UUID，用于 UI 标识（可选） |
| **lastUsedAt** | 最后使用时间戳 |
| **lastUsedCommand** | 最后使用的命令 |
| **lastResolvedPath** | 最后解析的路径 |

## 自动允许技能 CLI

启用**自动允许技能 CLI** 后，已知技能引用的可执行文件在节点上（macOS 节点或无头节点主机）被视为白名单。这通过 Gateway RPC 使用 `skills.bins` 获取技能 bin 列表。如果想要严格的手动白名单，禁用此选项。

## 安全二进制（仅 stdin）

`tools.exec.safeBins` 定义了一小组**仅 stdin** 的二进制文件（例如 `jq`），可以在白名单模式下运行**无需**显式白名单条目。安全二进制拒绝位置文件参数和类路径 token，因此只能操作输入流。

Shell 链接和重定向在白名单模式下不自动允许。

当每个顶级段都满足白名单（包括安全二进制或技能自动允许）时，允许 Shell 链接（`&&`、`||`、`;`）。重定向在白名单模式下不支持。

**默认安全二进制**：`jq`、`grep`、`cut`、`sort`、`uniq`、`head`、`tail`、`tr`、`wc`

## 控制 UI 编辑

使用**控制 UI → 节点 → 执行审批**卡片编辑默认值、按智能体覆盖和白名单。

选择范围（默认值或特定智能体），调整策略，添加/删除白名单模式，然后**保存**。UI 显示每个模式的**最后使用**元数据，方便保持列表整洁。

目标选择器选择 **Gateway**（本地审批）或**节点**。节点必须通告 `system.execApprovals.get/set`（macOS 应用或无头节点主机）。如果节点尚未通告执行审批，直接编辑其本地 `~/.openclaw/exec-approvals.json`。

**CLI**：`openclaw approvals` 支持 gateway 或节点编辑（参见 [审批 CLI](/cli/approvals)）。

## 审批流程

需要提示时，Gateway 向操作员客户端广播 `exec.approval.requested`。控制 UI 和 macOS 应用通过 `exec.approval.resolve` 解决它，然后 Gateway 将批准的请求转发给节点主机。

需要审批时，exec 工具立即返回一个审批 ID。使用该 ID 关联后续系统事件（`Exec finished` / `Exec denied`）。如果超时前没有收到决定，请求被视为审批超时并作为拒绝原因呈现。

**确认对话框包含：**

- 命令 + 参数
- 工作目录
- 智能体 ID
- 解析的可执行文件路径
- 主机 + 策略元数据

**操作选项：**

| 操作 | 说明 |
|------|------|
| **允许一次** | 立即运行 |
| **始终允许** | 添加到白名单 + 运行 |
| **拒绝** | 阻止 |

## 审批转发到聊天渠道

你可以将执行审批提示转发到任何聊天渠道（包括插件渠道），并用 `/approve` 批准。这使用标准的出站投递管道。

**配置示例：**

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // 子字符串或正则表达式
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

**在聊天中回复：**

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

## macOS IPC 流程

```
Gateway -> 节点服务 (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac 应用 (UI + 审批 + system.run)
```

**安全说明：**

- Unix socket 模式 `0600`，token 存储在 `exec-approvals.json`
- 同 UID 对等检查
- 挑战/响应（nonce + HMAC token + 请求哈希）+ 短 TTL

## 系统事件

执行生命周期作为系统消息呈现：

| 事件 | 说明 |
|------|------|
| `Exec running` | 仅当命令超过运行通知阈值时 |
| `Exec finished` | 命令完成 |
| `Exec denied` | 命令被拒绝 |

这些在节点报告事件后发布到智能体的会话。Gateway 主机执行审批在命令完成时（以及可选地运行超过阈值时）发出相同的生命周期事件。

审批门控的执行复用审批 ID 作为这些消息中的 `runId`，便于关联。

## 影响和建议

| 建议 | 说明 |
|------|------|
| 谨慎使用 `full` | 这很强大；尽可能优先使用白名单 |
| 使用 `ask` 保持知情 | 在允许快速审批的同时保持知情 |
| 按智能体隔离白名单 | 防止一个智能体的审批泄漏到其他智能体 |

**重要说明：**

- 审批仅适用于来自**授权发送者**的宿主执行请求。未授权发送者无法发出 `/exec`
- `/exec security=full` 是授权操作员的会话级便利功能，设计上跳过审批
- 要硬阻止宿主执行，将审批安全级别设为 `deny` 或通过工具策略拒绝 `exec` 工具

## 相关文档

- [执行工具](/tools/exec) - 命令执行详情
- [提权模式](/tools/elevated) - 提权访问配置
- [技能系统](/tools/skills) - 技能和二进制引用
