---
summary: 欢迎使用 OpenClaw 入门指南，涵盖产品介绍、目标用户、学习路径、快速开始步骤、系统要求和获取帮助
read_when:
  - 首次了解 OpenClaw 时
  - 新用户开始使用前
  - 选择学习路径时
title: 入门指南
version: "2026.2.17"
last_updated: "2026-03-05"
---

# 入门指南

欢迎使用 OpenClaw！本部分将指导您从零开始，逐步完成 OpenClaw 的安装、配置和使用。

## 什么是 OpenClaw？

OpenClaw 是一个强大的多渠道 AI 代理网关，可以将 AI 代理连接到您常用的消息平台。

**核心功能：**

- 🔗 **多渠道连接**: WhatsApp、Telegram、Discord、Slack、iMessage 等
- 🤖 **AI 代理桥接**: 与 Pi 等 AI 编程代理无缝集成
- 🔧 **灵活配置**: JSON5 格式，支持环境变量和配置包含
- 🧩 **插件扩展**: 通过插件添加更多渠道和功能
- 🖥️ **跨平台**: 支持 macOS、Linux、Windows (WSL2)

## 适合谁使用？

| 用户类型 | 使用场景 |
|----------|----------|
| 个人用户 | 通过手机消息应用与 AI 助手对话 |
| 开发者 | 构建基于 AI 的聊天机器人和自动化 |
| 团队 | 在 Slack/Teams 中集成 AI 助手 |
| 企业 | 多代理、多渠道的 AI 服务部署 |

## 学习路径

### 新手入门

如果您是第一次使用 OpenClaw：

1. **[快速入门](/start/quick-start)** - 5 分钟完成安装和基础配置
2. **[安装指南](/start/installation)** - 详细的安装选项说明
3. **[WhatsApp 配置](/channels/whatsapp)** - 连接最常用的消息渠道

### 进阶学习

理解 OpenClaw 的工作原理：

1. **[系统架构](/concepts/architecture)** - 整体架构概述
2. **[Gateway 网关](/concepts/gateway)** - 核心服务详解
3. **[AI 代理](/concepts/agents)** - 代理配置和管理
4. **[配置参考](/config/reference)** - 完整配置选项

### 高级使用

发挥 OpenClaw 的全部潜力：

1. **[消息路由](/concepts/routing)** - 多代理智能路由
2. **[插件开发](/developer/plugin-development)** - 扩展 OpenClaw
3. **[运维指南](/operations/index)** - 生产环境最佳实践

## 快速开始

### 1. 安装

```bash
# 使用安装脚本（推荐）
curl -fsSL https://openclaw.ai/install.sh | bash

# 或使用 npm
npm install -g openclaw@latest
```

### 2. 配置

```bash
openclaw onboard --install-daemon
```

### 3. 连接 WhatsApp

```bash
openclaw channels login
```

扫描二维码完成连接。

### 4. 开始对话

打开浏览器控制台：

```bash
openclaw dashboard
```

或直接访问 http://127.0.0.1:18789/

## 系统要求

| 组件 | 要求 |
|------|------|
| Node.js | ≥ 22.12.0 |
| 操作系统 | macOS 12+ / Ubuntu 20.04+ / Windows 10+ (WSL2) |
| 内存 | 512MB（推荐 2GB+） |
| 磁盘 | 500MB（推荐 2GB+） |

## 获取帮助

- 📖 **[故障排除](/operations/troubleshooting)** - 常见问题解答
- 🩺 **诊断工具**: `openclaw doctor`
- 📋 **状态检查**: `openclaw status --all`
- 🐛 **问题反馈**: [GitHub Issues](https://github.com/openclaw/openclaw/issues)

## 文档导航

| 章节 | 说明 |
|------|------|
| [入门指南](/start/quick-start) | 安装和基础配置 |
| [核心概念](/concepts/architecture) | 架构和原理 |
| [渠道集成](/channels/index) | 连接消息平台 |
| [配置指南](/config/index) | 配置选项详解 |
| [开发者手册](/developer/index) | 开发和扩展 |
| [运维手册](/operations/index) | 部署和维护 |

---

准备好了吗？让我们开始吧！

👉 [快速入门](/start/quick-start)

---

## 💡 最佳实践

### 新手上路建议

**第一次使用 OpenClaw？** 遵循以下建议可以快速上手并避免常见问题：

1. **从 WhatsApp 开始** - WhatsApp 配置最简单，文档最完善
2. **使用备用号码** - 避免使用主用手机号码，推荐使用 eSIM 或备用手机
3. **完成新手引导** - `openclaw onboard` 会自动配置大部分设置
4. **先测试再部署** - 使用 `openclaw message send` 测试消息发送
5. **阅读故障排除** - 遇到问题先查看 [故障排除](/operations/troubleshooting)

### 配置管理最佳实践

**配置文件管理**：

```bash
# 1. 定期备份配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak.$(date +%Y%m%d)

# 2. 使用 doctor 检查配置
openclaw doctor

# 3. 验证配置语法
openclaw config get
```

**环境变量管理**：

```bash
# 推荐：使用 .env 文件管理敏感信息
# ~/.openclaw/.env
ANTHROPIC_API_KEY=sk-ant-xxx
TELEGRAM_BOT_TOKEN=xxx

# 在配置文件中引用
# openclaw.json:
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

### 安全最佳实践

**令牌和密钥管理**：

| 做法 | 推荐 | 不推荐 |
|------|------|--------|
| API 密钥存储 | 环境变量或凭据存储 | 硬编码在配置文件 |
| Gateway 令牌 | 使用强随机令牌 | 使用默认令牌 |
| 配置文件权限 | `chmod 600` | 全局可读 |
| 备份配置 | 加密备份 | 明文备份 |

**访问控制**：

```json5
{
  // 启用网关认证
  gateway: {
    auth: {
      token: "${GATEWAY_TOKEN}"  // 使用环境变量
    }
  },
  
  // 配置白名单
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]  // 只允许信任的号码
    }
  }
}
```

### 性能优化建议

**消息队列优化**：

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 500,    // 减少收集窗口（默认 1000ms）
      cap: 10             // 减小批量大小（默认 20）
    }
  }
}
```

**会话历史管理**：

```json5
{
  agents: {
    defaults: {
      historyLimit: {
        messages: 50,      // 减少历史消息数（默认 100）
        days: 7,           // 限制历史保存天数（默认 30）
        maxTokens: 50000   // 限制 token 数量
      }
    }
  }
}
```

### 监控和日志

**日志级别建议**：

| 环境 | 推荐级别 | 说明 |
|------|---------|------|
| 开发 | `debug` | 查看详细调试信息 |
| 测试 | `info` | 记录关键操作 |
| 生产 | `warn` | 仅记录警告和错误 |

**监控命令**：

```bash
# 实时日志
openclaw logs --follow

# 查看错误
openclaw logs --level error --since "1 hour"

# 健康检查
openclaw health

# 渠道状态
openclaw channels status --probe
```

### 常见问题预防

**定期维护任务**：

```bash
# 每周：检查服务状态
openclaw status

# 每月：清理旧会话
openclaw sessions prune --older-than 30d

# 每月：更新配置备份
cp ~/.openclaw/openclaw.json ~/.openclaw/backup.$(date +%Y%m).json

# 每季度：检查并更新
openclaw doctor
```

**避免常见错误**：

1. ❌ **不要**在配置文件中硬编码 API 密钥
2. ❌ **不要**使用默认 Gateway 令牌
3. ❌ **不要**忽略 `openclaw doctor` 的警告
4. ✅ **要**定期备份配置文件
5. ✅ **要**使用环境变量管理敏感信息
6. ✅ **要**定期查看日志和监控

---

## 📝 变更历史

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 2026.2.17 | 2026-03-05 | 添加版本标注和最佳实践章节 |
| 2026.2.17 | 2026-02-04 | 初始翻译版本 |
