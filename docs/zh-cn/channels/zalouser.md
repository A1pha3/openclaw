---
summary: "Zalo 个人账号渠道配置 - 通过 zca-cli（QR 登录）"
read_when:
  - 设置 Zalo 个人账号连接
  - 调试 Zalo 个人账号登录或消息流
title: "Zalo Personal"
---

# Zalo Personal（非官方）

**状态:** 实验性。此集成通过 `zca-cli` 自动化**个人 Zalo 账号**。

> **警告:** 这是非官方集成，可能导致账号被暂停或封禁。使用风险自负。

## 为什么需要这个

当 Zalo Bot API 无法满足需求时，你可能需要使用个人账号：

- **完整功能**: 访问个人账号的所有功能
- **群组支持**: Bot API 暂不支持群组，但个人账号支持
- **无审核**: 不需要通过 Zalo 的机器人审核流程

## 安装插件

Zalo Personal 作为插件提供，不包含在核心安装中。

```bash
# 从 npm 安装
openclaw plugins install @openclaw/zalouser

# 或从本地源码安装
openclaw plugins install ./extensions/zalouser
```

## 前置条件：zca-cli

网关机器上必须安装 `zca` 命令行工具。

```bash
# 验证安装
zca --version
```

如果缺失，请参考 `extensions/zalouser/README.md` 或 zca-cli 上游文档安装。

## 快速设置

### 1. 安装插件

见上文。

### 2. 登录（QR 码）

在网关机器上执行：

```bash
openclaw channels login --channel zalouser
```

用 Zalo 手机 App 扫描终端中显示的二维码。

### 3. 启用渠道

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

### 4. 重启网关

### 5. 完成配对

首次联系时批准配对码。

## 工作原理

- 使用 `zca listen` 接收入站消息
- 使用 `zca msg ...` 发送回复（文本/媒体/链接）
- 专为"个人账号"场景设计，适用于 Bot API 不可用的情况

## 命名说明

渠道 ID 为 `zalouser`，明确表示这是自动化**个人 Zalo 用户账号**（非官方）。`zalo` 保留给未来可能的官方 Zalo API 集成。

## 查找 ID（目录功能）

使用目录 CLI 发现联系人/群组及其 ID：

```bash
# 查看自己的信息
openclaw directory self --channel zalouser

# 搜索联系人
openclaw directory peers list --channel zalouser --query "姓名"

# 搜索群组
openclaw directory groups list --channel zalouser --query "工作"
```

## 限制

| 限制类型 | 说明 |
|---------|------|
| 文本长度 | 出站文本分块为约 2000 字符（Zalo 客户端限制） |
| 流式传输 | 默认禁用 |

## 访问控制

### 私信策略

`channels.zalouser.dmPolicy` 支持：

| 策略 | 说明 |
|------|------|
| `pairing`（默认） | 未知发送者收到配对码 |
| `allowlist` | 仅 `allowFrom` 中的用户可发消息 |
| `open` | 开放公共私信 |
| `disabled` | 忽略所有私信 |

`channels.zalouser.allowFrom` 接受用户 ID 或名称。向导会在可用时通过 `zca friend find` 将名称解析为 ID。

### 批准配对

```bash
openclaw pairing list zalouser
openclaw pairing approve zalouser <code>
```

## 群组访问

### 默认行为

`channels.zalouser.groupPolicy = "open"`（允许所有群组）。使用 `channels.defaults.groupPolicy` 可覆盖默认值。

### 白名单模式

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "工作群": { allow: true },
      },
    },
  },
}
```

### 禁用群组

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "disabled",
    },
  },
}
```

注意：启动时，OpenClaw 会将允许列表中的群组/用户名解析为 ID 并记录映射；无法解析的条目保持原样。

## 多账号配置

账号映射到 zca 配置文件：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
        personal: { enabled: true, profile: "personal" },
      },
    },
  },
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| `zca` not found | 未安装 zca-cli | 安装 zca-cli 并确保在网关进程的 PATH 中 |
| 登录状态丢失 | 会话过期 | 运行 `openclaw channels status --probe` 检查状态 |
| 登录状态丢失 | 需要重新登录 | 运行 `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser` |

## 安全提示

由于这是非官方集成：

- **使用专用账号**: 不要用主要个人账号
- **限制访问**: 使用 `allowlist` 或 `pairing` 策略
- **监控账号状态**: 定期检查账号是否正常

## 相关资源

- 配对流程: [配对](/zh-cn/start/pairing)
- 插件管理: [插件](/zh-cn/plugin)
- Zalo Bot API（官方）: [Zalo](/zh-cn/channels/zalo)
