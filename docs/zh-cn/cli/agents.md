---
summary: "`openclaw agents` 命令参考（列出/添加/删除/设置身份）"
read_when:
  - 想要配置多个隔离的代理（工作区 + 路由 + 认证）
title: "agents"
---

# `openclaw agents`

管理隔离的代理实例（独立的工作区、认证和路由配置）。

## 为什么需要多代理

在实际使用中，你可能需要：

- **工作/生活分离**：工作代理有访问公司工具的权限，个人代理更加轻松随意
- **不同人格**：一个代理专注于编程，另一个擅长写作
- **多租户场景**：为家庭成员或团队成员配置独立的代理
- **测试隔离**：开发新技能时使用独立的测试代理

## 相关链接

- 多代理路由：[Multi-Agent Routing](/zh-cn/concepts/multi-agent)
- 代理工作区：[Agent workspace](/zh-cn/concepts/agent-workspace)

## 基本命令

```bash
# 列出所有代理
openclaw agents list

# 添加新代理
openclaw agents add work --workspace ~/.openclaw/workspace-work

# 从 IDENTITY.md 加载身份
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity

# 手动设置头像
openclaw agents set-identity --agent main --avatar avatars/openclaw.png

# 删除代理
openclaw agents delete work
```

## 身份文件

每个代理工作区可以在根目录包含一个 `IDENTITY.md` 文件：

- 示例路径：`~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` 从工作区根目录读取（或使用 `--identity-file` 指定路径）

头像路径相对于工作区根目录解析。

## 设置身份

`set-identity` 命令将字段写入 `agents.list[].identity`：

| 字段 | 说明 |
|------|------|
| `name` | 代理显示名称 |
| `theme` | 主题描述 |
| `emoji` | 代理表情符号 |
| `avatar` | 头像路径（相对路径、URL 或 data URI） |

### 从 IDENTITY.md 加载

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

### 手动指定字段

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

## 配置示例

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
      {
        id: "work",
        workspace: "~/.openclaw/workspace-work",
        identity: {
          name: "工作助手",
          theme: "professional",
          emoji: "💼",
        },
      },
    ],
  },
}
```

## 多代理路由

配置多个代理后，可以设置路由规则将不同的消息来源路由到不同的代理：

```json5
{
  routing: {
    rules: [
      // 工作相关的 Slack 消息路由到工作代理
      { channel: "slack", agent: "work" },
      // 其他消息使用默认代理
      { channel: "*", agent: "main" },
    ],
  },
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 代理未找到 | ID 拼写错误 | `openclaw agents list` 查看所有代理 |
| 身份未更新 | 缓存问题 | 重启网关 |
| 头像不显示 | 路径错误 | 确保相对于工作区根目录 |
| 工作区冲突 | 多代理共享工作区 | 每个代理使用独立工作区 |
