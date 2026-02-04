---
summary: "Nostr 渠道配置 - 通过 NIP-04 加密私信连接"
read_when:
  - 想让 OpenClaw 接收 Nostr 私信
  - 设置去中心化消息渠道
title: "Nostr"
---

# Nostr

**状态:** 可选插件（默认禁用）。

Nostr 是一个去中心化的社交网络协议。此渠道让 OpenClaw 能够通过 NIP-04 接收和回复加密私信（DM）。

## 为什么需要这个

- **去中心化**: 不依赖单一服务商，数据由你控制
- **抗审查**: 使用多个中继服务器，不会被单点封禁
- **隐私优先**: 私信采用端到端加密
- **Web3 原生**: 与区块链/加密货币社区高度契合

## 安装插件

### 通过引导向导（推荐）

运行 `openclaw onboard` 或 `openclaw channels add`，选择 Nostr 后会提示安装插件。

### 手动安装

```bash
openclaw plugins install @openclaw/nostr
```

使用本地开发版本：

```bash
openclaw plugins install --link <openclaw-repo>/extensions/nostr
```

安装或启用插件后需重启网关。

## 快速设置

### 1. 生成 Nostr 密钥对（如果没有）

```bash
# 使用 nak 工具
nak key generate
```

### 2. 添加配置

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

### 3. 设置环境变量

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

### 4. 重启网关

## 配置参考

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `privateKey` | string | 必填 | 私钥，`nsec` 或 64 位十六进制格式 |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | 中继服务器 URL（WebSocket） |
| `dmPolicy` | string | `pairing` | 私信访问策略 |
| `allowFrom` | string[] | `[]` | 允许的发送者公钥列表 |
| `enabled` | boolean | `true` | 启用/禁用渠道 |
| `name` | string | - | 显示名称 |
| `profile` | object | - | NIP-01 个人资料元数据 |

## 个人资料设置

个人资料作为 NIP-01 `kind:0` 事件发布。可以在控制界面（渠道 -> Nostr -> 个人资料）管理，或直接在配置中设置。

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "个人助手机器人",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

注意：
- 个人资料 URL 必须使用 `https://`
- 从中继导入会合并字段，保留本地覆盖

## 访问控制

### 私信策略

| 策略 | 说明 |
|------|------|
| `pairing`（默认） | 未知发送者收到配对码 |
| `allowlist` | 仅 `allowFrom` 中的公钥可私信 |
| `open` | 开放公共私信（需要 `allowFrom: ["*"]`） |
| `disabled` | 忽略所有私信 |

### 白名单示例

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## 密钥格式

支持的格式：
- **私钥**: `nsec...` 或 64 位十六进制
- **公钥（`allowFrom`）**: `npub...` 或十六进制

## 中继服务器配置

默认使用 `relay.damus.io` 和 `nos.lol`。

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

配置建议：
- 使用 2-3 个中继保证冗余
- 避免过多中继（延迟、重复）
- 付费中继可提高可靠性
- 本地中继适合测试（`ws://localhost:7777`）

## 协议支持

| NIP | 状态 | 说明 |
|-----|------|------|
| NIP-01 | ✅ 支持 | 基础事件格式 + 个人资料元数据 |
| NIP-04 | ✅ 支持 | 加密私信（`kind:4`） |
| NIP-17 | 🔜 计划中 | 礼物包装私信 |
| NIP-44 | 🔜 计划中 | 版本化加密 |

## 测试

### 使用本地中继

```bash
# 启动 strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### 手动测试

1. 从日志获取机器人公钥（npub）
2. 打开 Nostr 客户端（Damus、Amethyst 等）
3. 向机器人公钥发送私信
4. 验证收到回复

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 收不到消息 | 私钥无效 | 验证私钥格式正确 |
| 收不到消息 | 中继不可达 | 检查 URL 和网络连接 |
| 收不到消息 | 渠道被禁用 | 确认 `enabled` 不是 `false` |
| 发不出回复 | 中继拒绝写入 | 检查中继是否接受写入 |
| 发不出回复 | 网络问题 | 检查出站连接 |
| 收到重复回复 | 多中继同步 | 正常现象，消息会按事件 ID 去重 |

## 安全提示

- **永远不要**提交私钥到代码仓库
- 使用环境变量存储密钥
- 生产环境建议使用 `allowlist` 策略

## 当前限制（MVP）

- 仅支持私信（不支持群聊）
- 不支持媒体附件
- 仅支持 NIP-04（NIP-17 礼物包装计划中）

## 相关资源

- 配对流程: [配对](/zh-cn/start/pairing)
- 插件管理: [插件](/zh-cn/plugin)
