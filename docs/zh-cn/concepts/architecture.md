# 系统架构

本文档详细介绍 OpenClaw 的系统架构，帮助您深入理解各组件的职责和交互方式。

## 架构概述

OpenClaw 采用中心化网关架构，通过 Gateway（网关）统一管理所有消息渠道和 AI 代理的通信。

```
消息渠道                         网关                          客户端
┌─────────────┐              ┌─────────────────┐         ┌─────────────┐
│  WhatsApp   │──┐           │                 │    ┌────│  Pi Agent   │
│  Telegram   │──┤           │                 │    │    │   (RPC)     │
│  Discord    │──┼───────────│    Gateway      │────┤    └─────────────┘
│  Slack      │──┤   HTTP/   │  ws://127.0.0.1 │    │    ┌─────────────┐
│  iMessage   │──┤    WS     │    :18789       │    ├────│  CLI 工具    │
│  更多插件... │──┘           │                 │    │    └─────────────┘
└─────────────┘              │                 │    │    ┌─────────────┐
                             └─────────────────┘    ├────│  macOS App  │
                                    │               │    └─────────────┘
                                    │               │    ┌─────────────┐
                             ┌──────┴──────┐        ├────│  iOS Node   │
                             │ Canvas Host │        │    └─────────────┘
                             │ :18793      │        │    ┌─────────────┐
                             └─────────────┘        └────│ Android Node│
                                                         └─────────────┘
```

## 核心组件

### 1. Gateway（网关）

Gateway 是 OpenClaw 的核心组件，负责：

- **消息路由**: 接收来自各渠道的消息，路由到正确的 AI 代理
- **会话管理**: 维护用户会话状态和上下文
- **渠道连接**: 管理与各消息平台的连接
- **WebSocket 服务**: 提供实时通信接口
- **HTTP API**: 提供 RESTful 和 RPC 接口

**启动网关:**

```bash
openclaw gateway --port 18789
```

**网关特性:**

| 特性 | 说明 |
|------|------|
| 单例运行 | 每个主机推荐只运行一个 Gateway |
| 本地优先 | 默认绑定到 `127.0.0.1`（环回地址） |
| 令牌认证 | 支持网关令牌保护 |
| 热重载 | 配置变更后自动重启 |

### 2. 消息渠道（Channels）

消息渠道是 OpenClaw 与外部消息平台的桥接层。

**内置渠道:**

| 渠道 | 协议 | 说明 |
|------|------|------|
| WhatsApp | Baileys (Web) | 通过 WhatsApp Web 协议 |
| Telegram | Bot API | 支持轮询和 Webhook |
| Discord | Gateway + REST | 使用 discord.js |
| Slack | Socket Mode | 实时事件订阅 |
| iMessage | imsg CLI | macOS 本地集成 |
| Google Chat | Webhook | 服务账号认证 |

**插件渠道（extensions）:**

| 渠道 | 说明 |
|------|------|
| Mattermost | 开源团队协作平台 |
| Matrix | 去中心化通信协议 |
| Microsoft Teams | 企业协作平台 |
| Signal | 安全通信 |
| Line | 亚洲流行通信应用 |
| Nostr | 去中心化社交协议 |
| Twitch | 直播平台聊天 |
| Zalo | 越南流行通信应用 |

### 3. AI 代理（Agents）

AI 代理负责处理消息并生成回复。

**代理系统特性:**

- **多代理支持**: 可配置多个独立代理，各有独立工作区
- **会话隔离**: 每个对话有独立的会话上下文
- **工具调用**: 支持文件操作、浏览器、命令执行等工具
- **沙箱模式**: 可在 Docker 容器中隔离运行

**代理工作流:**

```
入站消息 → 路由匹配 → 代理选择 → 会话加载 → AI 处理 → 回复发送
                ↓
           工具调用（可选）
                ↓
           继续处理
```

### 4. 会话系统（Sessions）

会话系统管理对话的上下文和历史。

**会话类型:**

| 类型 | 说明 | 会话键示例 |
|------|------|------------|
| DM | 私聊 | `agent:main:whatsapp:dm:+15551234567` |
| Group | 群聊 | `agent:main:whatsapp:group:120363...@g.us` |
| Channel | 频道 | `agent:main:discord:channel:1234567890` |

**会话合并:**

默认情况下，同一用户的多个私聊会话合并到 `main` 会话，共享上下文。

### 5. 插件系统（Plugins）

OpenClaw 支持通过插件扩展功能。

**插件类型:**

- **渠道插件**: 添加新的消息平台支持
- **技能插件**: 添加新的 AI 工具和能力
- **存储插件**: 自定义数据存储后端

**插件目录:**

```
extensions/
├── mattermost/     # Mattermost 渠道
├── matrix/         # Matrix 渠道
├── msteams/        # Microsoft Teams
├── memory-lancedb/ # 向量存储
└── voice-call/     # 语音通话
```

## 数据流

### 入站消息流

```
1. 消息平台 → 渠道适配器
2. 渠道适配器 → Gateway 消息队列
3. 消息队列 → 路由引擎
4. 路由引擎 → 代理选择
5. 代理处理 → 生成回复
6. 回复 → 渠道适配器 → 消息平台
```

### 出站消息流

```
1. CLI/API/工具 → Gateway RPC
2. Gateway → 渠道选择
3. 渠道适配器 → 消息平台
4. 消息平台 → 目标用户
```

## 网络模型

### 本地模式（默认）

```
Gateway: ws://127.0.0.1:18789
Canvas:  http://127.0.0.1:18793
```

适用于：
- 个人使用
- 开发测试
- 单机部署

### 远程模式

```
Gateway: wss://gateway.example.com:18789
Canvas:  https://gateway.example.com:18793
```

访问方式：
- SSH 隧道
- Tailscale/VPN
- 反向代理

## 安全模型

### 认证层次

1. **网关令牌**: 保护 Gateway WebSocket 和 API
2. **渠道认证**: 各平台 Bot Token
3. **AI 认证**: OpenAI/Anthropic API 密钥或 OAuth
4. **配对系统**: 未知用户需要配对审批

### 沙箱隔离

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "session"
      }
    }
  }
}
```

沙箱模式：
- `off`: 无沙箱，完全访问
- `non-main`: 非主会话使用沙箱
- `all`: 所有会话使用沙箱

## 配置系统

### 配置层次

```
环境变量 (最高优先级)
    ↓
配置文件 (~/.clawdbot/openclaw.json)
    ↓
默认值 (最低优先级)
```

### 配置热重载

配置变更后，Gateway 会自动检测并重启：

```bash
# 修改配置
openclaw config set channels.telegram.enabled true

# Gateway 自动重启应用变更
```

## 扩展架构

### 水平扩展

多实例部署：

```
┌──────────────┐     ┌──────────────┐
│  Gateway A   │     │  Gateway B   │
│  (实例 1)    │     │  (实例 2)    │
│  WhatsApp A  │     │  WhatsApp B  │
│  端口 19001  │     │  端口 19002  │
└──────────────┘     └──────────────┘
```

### 高可用部署

```
┌─────────────────────────────────────┐
│           负载均衡器                │
└─────────────────┬───────────────────┘
         ┌────────┴────────┐
         ↓                 ↓
   ┌──────────┐      ┌──────────┐
   │ Gateway  │      │ Gateway  │
   │ Primary  │      │ Standby  │
   └──────────┘      └──────────┘
         ↓                 ↓
   ┌─────────────────────────────┐
   │      共享存储/数据库         │
   └─────────────────────────────┘
```

## 监控与可观测性

### 健康检查

```bash
openclaw health
```

### 状态查询

```bash
openclaw status --all
```

### 日志系统

日志文件位置：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

配置日志级别：

```json
{
  "logging": {
    "level": "info",
    "consoleLevel": "debug"
  }
}
```

## 下一步

- [Gateway 网关](/zh-CN/concepts/gateway) - 深入了解网关
- [消息路由](/zh-CN/concepts/routing) - 理解路由机制
- [会话管理](/zh-CN/concepts/sessions) - 会话系统详解
- [配置参考](/zh-CN/config/reference) - 完整配置选项
