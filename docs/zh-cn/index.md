---
summary: "OpenClaw 中文文档主页 - 从零基础到精通的完整学习路径"
read_when:
  - 寻找中文文档入口
  - 了解 OpenClaw 是什么
  - 决定从哪里开始学习
title: "OpenClaw 中文文档"
---

# 🦞 OpenClaw 中文文档

欢迎来到 OpenClaw 中文文档！本文档专为**中文用户**和**完全新手**设计，帮助你从零开始，逐步掌握这个强大的 AI 助手系统。

> **什么是 OpenClaw？** 
> 
> OpenClaw 是**你的个人 AI 助手网关**，连接 WhatsApp、Telegram、Discord 等你常用的聊天应用，与 Claude、GPT 等 AI 大脑对接，让 AI 通过你熟悉的聊天界面为你服务。

---

## 🎯 新手起步（推荐路径）

**完全不懂？从这里开始！**

| 步骤 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 1 | [**新手上路**](/zh-CN/start/getting-started) | 30分钟 | 完成安装并发送第一条消息 |
| 2 | [**向导模式详解**](/zh-CN/start/wizard) | 20分钟 | 理解配置流程和原理 |
| 3 | [**安装指南**](/zh-CN/install) | 15分钟 | 掌握多种安装方式 |
| 4 | [**常见问题**](/zh-CN/help/faq) | 随时查阅 | 解决疑惑 |

**快速导航**：
- 🚀 [5分钟快速入门](/zh-CN/start/quick-start)
- ❓ [遇到问题？查看故障排除](/zh-CN/help/troubleshooting)
- 💬 [加入 Discord 社区](https://discord.gg/clawd)

---

## 📚 文档结构

### 🚀 入门指南 (Start)

适合刚接触 OpenClaw 的新用户，从安装到发送第一条消息。

- [**新手上路**](/zh-CN/start/getting-started) - **新手必读！** 从零到第一条消息的完整指南
- [**快速入门**](/zh-CN/start/quick-start) - 5分钟快速上手（已有基础者）
- [**安装指南**](/zh-CN/start/installation) - 详细安装步骤和环境准备
- [**向导模式详解**](/zh-CN/start/wizard) - 深入理解引导向导
- [核心概念](/zh-CN/concepts/architecture) - 系统架构理解

### 🧠 核心概念 (Concepts)

深入理解 OpenClaw 的**架构和工作原理**。

**架构理解：**
- [**系统架构**](/zh-CN/concepts/architecture) - **必读！** 整体架构和设计哲学
- [**Gateway 网关**](/zh-CN/concepts/gateway) - 核心控制中心详解
- [**AI 代理**](/zh-CN/concepts/agents) - 代理系统工作原理
- [**会话管理**](/zh-CN/concepts/sessions) - 会话与上下文机制
- [**消息路由**](/zh-CN/concepts/routing) - 智能消息分发

**深入机制：**
- [代理循环](/zh-CN/concepts/agents) - 消息处理流程
- [上下文管理](/zh-CN/concepts/sessions) - 会话与上下文机制
- [记忆系统](/zh-CN/concepts/memory) - 短期与长期记忆
- [流式响应](/zh-CN/concepts/streaming) - 实时回复机制
- [模型配置](/zh-CN/concepts/models) - AI 模型选择策略

### 📱 渠道集成 (Channels)

连接各种消息平台的详细指南。

- [**渠道概述**](/zh-CN/channels) - 所有支持的渠道对比
- [**WhatsApp**](/zh-CN/channels/whatsapp) - 最热门的移动消息
- [**Telegram**](/zh-CN/channels/telegram) - 简单快速的 Bot 集成
- [**Discord**](/zh-CN/channels/discord) - 社区和团队协作
- [**Slack**](/zh-CN/channels/slack) - 工作场景首选
- [**Signal**](/zh-CN/channels/signal) - 隐私优先
- [**iMessage**](/zh-CN/channels/imessage) - Apple 生态
- [**Matrix**](/zh-CN/channels/matrix) - 去中心化

### ⚙️ 配置指南 (Config)

全面的配置参考和最佳实践。

- [**配置概述**](/zh-CN/config) - 配置系统入门
- [**配置参考**](/zh-CN/config/reference) - 所有配置项详解
- [**配置示例**](/zh-CN/config/examples) - 常见场景配置模板

### 🛠️ 命令行工具 (CLI)

掌握 `openclaw` 命令行工具的所有功能。

**核心命令：**
- [CLI 概述](/zh-CN/cli) - 命令结构和帮助
- [onboard](/zh-CN/cli/onboard) - 引导配置
- [gateway](/zh-CN/cli/gateway) - 网关管理
- [config](/zh-CN/cli/config) - 配置操作
- [status](/zh-CN/cli/status) - 状态查看
- [doctor](/zh-CN/cli/doctor) - 诊断工具

**渠道管理：**
- [channels](/zh-CN/cli/channels) - 渠道管理
- [pairing](/zh-CN/cli/pairing) - 配对管理
- [message](/zh-CN/cli/message) - 发送消息

**代理管理：**
- [agent/agents](/zh-CN/cli/agents) - 代理管理
- [sessions](/zh-CN/cli/sessions) - 会话管理
- [memory](/zh-CN/cli/memory) - 记忆管理

### 🔧 工具与技能 (Tools)

扩展 AI 能力的工具系统。

- [**技能概述**](/zh-CN/tools) - 技能系统入门
- [浏览器工具](/zh-CN/tools/browser) - 网页自动化
- [执行工具](/zh-CN/tools/exec) - 命令执行
- [Web 工具](/zh-CN/tools/web) - 网页搜索和抓取
- [思考模式](/zh-CN/tools/thinking) - 深度思考
- [子代理](/zh-CN/tools/subagents) - 任务分发

### 🌐 网关与协议 (Gateway)

掌握核心网关的配置和运维。

**基础配置：**
- [**网关概述**](/zh-CN/gateway) - 网关系统入门
- [**配置详解**](/zh-CN/concepts/gateway) - 所有配置选项

**网络与连接：**
- [远程访问](/zh-CN/gateway/remote) - 从外部连接
- [Tailscale 集成](/zh-CN/gateway/tailscale) - 安全组网

**运维：**
- [后台服务](/zh-CN/gateway/background-process) - 守护进程
- [健康检查](/zh-CN/gateway/health) - 监控健康
- [日志系统](/zh-CN/gateway/logging) - 日志管理
- [诊断工具](/zh-CN/gateway/doctor) - 问题诊断

### 🤖 AI 提供商 (Providers)

配置各种 AI 模型提供商。

- [**提供商概述**](/zh-CN/providers) - 所有支持的 AI 服务
- [**Anthropic (Claude)**](/zh-CN/providers/anthropic) - 最推荐的 AI
- [**OpenAI (GPT)**](/zh-CN/providers/openai) - GPT 系列
- [**OpenRouter**](/zh-CN/providers/openrouter) - 多模型聚合
- [**Moonshot (Kimi)**](/zh-CN/providers/moonshot) - 国产大模型
- [**GLM**](/zh-CN/providers/glm) - 智谱 AI
- [**MiniMax**](/zh-CN/providers/minimax) - MiniMax 模型
- [**Ollama**](/zh-CN/providers/ollama) - 本地模型

### 💻 部署平台 (Platforms)

在不同平台上运行 OpenClaw。

**操作系统：**
- [**macOS**](/zh-CN/platforms/macos) - Mac 用户指南
- [**Linux**](/zh-CN/platforms/linux) - Linux 部署
- [**Windows (WSL2)**](/zh-CN/platforms/windows) - Windows 用户
- [**iOS**](/zh-CN/platforms/ios) - 移动节点
- [**Android**](/zh-CN/platforms/android) - 移动节点

**云服务：**
- [Docker](/zh-CN/install/docker)
- [Fly.io](/zh-CN/platforms/fly)
- [树莓派](/zh-CN/platforms/raspberry-pi)

### 📱 移动节点 (Nodes)

将手机变为 AI 助手节点。

- [**节点概述**](/zh-CN/nodes) - 移动设备集成
- [音频节点](/zh-CN/nodes/audio) - 语音输入
- [相机节点](/zh-CN/nodes/camera) - 图像理解
- [位置服务](/zh-CN/nodes/location-command) - 地理信息
- [语音对话](/zh-CN/nodes/talk) - 语音交互
- [语音唤醒](/zh-CN/nodes/voicewake) - 免提唤醒

### ⏰ 自动化 (Automation)

定时任务和自动化工作流。

- [**Cron 任务**](/zh-CN/automation/cron-jobs) - 定时执行
- [Webhook](/zh-CN/automation/webhook) - HTTP 触发

### 🖥️ Web 界面 (Web)

浏览器控制界面。

- [**Web 概述**](/zh-CN/web) - Web 界面介绍
- [**仪表盘**](/zh-CN/web/dashboard) - 浏览器聊天

### 👨‍💻 开发者文档 (Developer)

面向开发者的技术文档。

- [**开发概述**](/zh-CN/developer) - 环境搭建
- [**项目结构**](/zh-CN/developer/project-structure) - 代码组织
- [**插件开发**](/zh-CN/developer/plugin-development) - 扩展系统
- [**测试指南**](/zh-CN/developer/testing) - 测试最佳实践
- [**贡献指南**](/zh-CN/developer/contributing) - 如何贡献

### 🔧 运维手册 (Operations)

生产环境部署和维护。

- [**运维概述**](/zh-CN/operations) - 最佳实践
- [**部署指南**](/zh-CN/operations/deployment) - 生产部署
- [**监控与日志**](/zh-CN/operations/monitoring) - 系统监控
- [**故障排除**](/zh-CN/operations/troubleshooting) - 问题诊断

### ❓ 帮助中心 (Help)

快速解决问题。

- [**常见问题**](/zh-CN/help/faq) - **随时查阅！** 最常见的问题和解答
- [**故障排除**](/zh-CN/help/troubleshooting) - 系统诊断和修复
- [帮助首页](/zh-CN/help)

---

## 🗺️ 学习路径推荐

### 🔰 完全新手

1. [新手上路](/zh-CN/start/getting-started) - 理解基础并安装
2. [系统架构](/zh-CN/concepts/architecture) - 理解系统如何工作
3. [配置一个渠道](/zh-CN/channels/telegram) - 推荐 Telegram（最简单）
4. [常见问题](/zh-CN/help/faq) - 解决疑惑

### 💻 开发者

1. [开发概述](/zh-CN/developer)
2. [项目结构](/zh-CN/developer/project-structure)
3. [插件开发](/zh-CN/developer/plugin-development)
4. [贡献指南](/zh-CN/developer/contributing)

### 🚀 生产部署

1. [运维概述](/zh-CN/operations)
2. [部署指南](/zh-CN/operations/deployment)
3. [监控与日志](/zh-CN/operations/monitoring)
4. [故障排除](/zh-CN/operations/troubleshooting)

---

## 🆘 获取帮助

**遇到问题？**

1. 🔍 [常见问题](/zh-CN/help/faq) - 80% 的问题已有答案
2. 🔧 [故障排除](/zh-CN/help/troubleshooting) - 系统诊断指南
3. 💬 [Discord 社区](https://discord.gg/clawd) - 实时交流
4. 🐛 [GitHub Issues](https://github.com/openclaw/openclaw/issues) - 提交问题

---

## 📝 关于本文档

本文档是官方英文文档的中文翻译和增强版本，专为中文用户优化：

- ✅ 深入浅出的原理解释
- ✅ 丰富的中文类比和例子
- ✅ 完整的配置示例
- ✅ 常见问题详细解答
- ✅ 从新手到专家的学习路径

**英文文档**：[docs.openclaw.ai](https://docs.openclaw.ai)

---

> 💡 **提示**：文档持续更新中！如果发现错误或有建议，欢迎通过 GitHub Issues 反馈。
> 
> *"最好的学习方式就是动手做。安装它，使用它，你会理解的。"*
