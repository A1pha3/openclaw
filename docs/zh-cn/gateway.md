---
summary: "网关配置概述 - Gateway 系统入门"
read_when:
  - 了解 Gateway 网关系统
  - 配置网关服务
  - 理解网关工作原理
title: "网关概述"
---

# 🌐 网关概述

Gateway（网关）是 OpenClaw 的核心组件，是所有消息渠道和 AI 代理的中央控制中心。

---

## 🎯 什么是 Gateway？

Gateway 是一个**长期运行的守护进程**，负责：

- 📨 **消息路由**：接收和转发所有渠道消息
- 🔐 **认证管理**：验证客户端和设备身份
- 💬 **渠道连接**：维护 WhatsApp、Telegram 等连接
- 🤖 **代理协调**：管理 AI 代理会话
- 🌐 **WebSocket 服务**：提供实时通信接口

---

## 🏗️ 架构位置

```
用户消息 (WhatsApp/Telegram/Discord)
           │
           ▼
    ┌──────────────┐
    │   Gateway    │ ← 核心控制中心
    │   网关       │
    └──────────────┘
           │
     ┌─────┴─────┐
     ▼           ▼
  AI 代理     客户端
(Claude)    (CLI/Web)
```

---

## 📁 配置位置

网关配置在 `~/.openclaw/openclaw.json` 的 `gateway` 部分：

```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    auth: {
      type: "token",
      token: "your-token"
    }
  }
}
```

---

## 🚀 快速开始

### 启动网关

```bash
# 前台运行（调试）
openclaw gateway run --verbose

# 后台服务
openclaw gateway start
```

### 检查状态

```bash
openclaw gateway status
openclaw health
```

---

## 📚 文档导航

### 基础配置
- [**配置详解**](/zh-CN/config/reference) - 所有配置选项
- [**配置示例**](/zh-CN/config/examples) - 实用模板
- [**认证机制**](/zh-CN/concepts/architecture#安全模型) - 安全配置

### 网络与连接
- [远程访问](/zh-CN/operations/deployment) - 从外部连接
- [配对流程](/zh-CN/start/pairing) - 设备配对

### 运维
- [后台服务](/zh-CN/operations/monitoring) - 守护进程
- [健康检查](/zh-CN/operations/monitoring) - 监控健康
- [故障排除](/zh-CN/help/troubleshooting) - 问题解决

---

## 🔧 关键特性

| 特性 | 说明 |
|------|------|
| **单例模式** | 每主机推荐只运行一个 Gateway |
| **本地优先** | 默认绑定 `127.0.0.1`（环回）|
| **WebSocket** | 实时双向通信 |
| **热重载** | 配置变更后自动重启 |
| **Token 认证** | 支持网关令牌保护 |

---

## 🆘 常见问题

**网关无法启动？**
- 检查端口是否被占用：`lsof -i :18789`
- 查看配置是否有效：`openclaw doctor`
- 检查日志：`openclaw logs`

**连接被拒绝？**
- 确认网关正在运行：`openclaw gateway status`
- 检查防火墙设置
- 验证 token 是否正确

---

## 📖 相关文档

- [系统架构](/zh-CN/concepts/architecture) - 整体架构
- [配置参考](/zh-CN/config/reference) - 完整配置
- [故障排除](/zh-CN/help/troubleshooting) - 问题解决
