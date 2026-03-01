---
summary: 欢迎使用 OpenClaw 入门指南，涵盖产品介绍、目标用户、学习路径、快速开始步骤、系统要求和获取帮助
read_when:
  - 首次了解 OpenClaw 时
  - 新用户开始使用前
  - 选择学习路径时
title: 入门指南
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

1. **[快速入门](/zh-CN/start/quick-start)** - 5 分钟完成安装和基础配置
2. **[安装指南](/zh-CN/start/installation)** - 详细的安装选项说明
3. **[WhatsApp 配置](/zh-CN/channels/whatsapp)** - 连接最常用的消息渠道

### 进阶学习

理解 OpenClaw 的工作原理：

1. **[系统架构](/zh-CN/concepts/architecture)** - 整体架构概述
2. **[Gateway 网关](/zh-CN/concepts/gateway)** - 核心服务详解
3. **[AI 代理](/zh-CN/concepts/agents)** - 代理配置和管理
4. **[配置参考](/zh-CN/config/reference)** - 完整配置选项

### 高级使用

发挥 OpenClaw 的全部潜力：

1. **[消息路由](/zh-CN/concepts/routing)** - 多代理智能路由
2. **[插件开发](/zh-CN/developer/plugin-development)** - 扩展 OpenClaw
3. **[运维指南](/zh-CN/operations/index)** - 生产环境最佳实践

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

- 📖 **[故障排除](/zh-CN/operations/troubleshooting)** - 常见问题解答
- 🩺 **诊断工具**: `openclaw doctor`
- 📋 **状态检查**: `openclaw status --all`
- 🐛 **问题反馈**: [GitHub Issues](https://github.com/openclaw/openclaw/issues)

## 文档导航

| 章节 | 说明 |
|------|------|
| [入门指南](/zh-CN/start/quick-start) | 安装和基础配置 |
| [核心概念](/zh-CN/concepts/architecture) | 架构和原理 |
| [渠道集成](/zh-CN/channels/index) | 连接消息平台 |
| [配置指南](/zh-CN/config/index) | 配置选项详解 |
| [开发者手册](/zh-CN/developer/index) | 开发和扩展 |
| [运维手册](/zh-CN/operations/index) | 部署和维护 |

---

准备好了吗？让我们开始吧！

👉 [快速入门](/zh-CN/start/quick-start)
