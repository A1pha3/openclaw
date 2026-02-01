# OpenClaw 中文文档

欢迎使用 OpenClaw 中文文档。本文档将帮助您从零开始，逐步掌握 OpenClaw 的使用、开发和运维。

## 文档结构

本文档分为以下几个主要部分：

### 入门指南

适合刚接触 OpenClaw 的新用户，从安装到发送第一条消息。

- [快速入门](/zh-cn/start/quick-start) - 5分钟快速上手
- [安装指南](/zh-cn/start/installation) - 详细安装步骤
- [初次配置](/zh-cn/start/first-setup) - 完成基础配置
- [验证安装](/zh-cn/start/verification) - 确认一切正常

### 核心概念

深入理解 OpenClaw 的架构和工作原理。

- [系统架构](/zh-cn/concepts/architecture) - 整体架构概述
- [Gateway 网关](/zh-cn/concepts/gateway) - 核心网关系统
- [渠道系统](/zh-cn/concepts/channels-overview) - 消息渠道抽象
- [AI 代理](/zh-cn/concepts/agents) - 代理系统详解
- [会话管理](/zh-cn/concepts/sessions) - 会话与上下文
- [消息路由](/zh-cn/concepts/routing) - 智能消息路由

### 渠道集成

连接各种消息平台的详细指南。

- [渠道概述](/zh-cn/channels/index) - 支持的渠道列表
- [WhatsApp](/zh-cn/channels/whatsapp) - WhatsApp Web 集成
- [Telegram](/zh-cn/channels/telegram) - Telegram Bot 配置
- [Discord](/zh-cn/channels/discord) - Discord Bot 设置
- [Slack](/zh-cn/channels/slack) - Slack 应用集成
- [更多渠道](/zh-cn/channels/others) - 其他渠道配置

### 配置指南

全面的配置参考和最佳实践。

- [配置概述](/zh-cn/config/index) - 配置系统介绍
- [配置参考](/zh-cn/config/reference) - 完整配置项说明
- [配置示例](/zh-cn/config/examples) - 常见场景配置
- [模型配置](/zh-cn/config/models) - AI 模型设置
- [安全配置](/zh-cn/config/security) - 安全相关设置

### 开发者手册

面向开发者的技术文档。

- [开发概述](/zh-cn/developer/index) - 开发环境搭建
- [项目结构](/zh-cn/developer/project-structure) - 代码组织结构
- [插件开发](/zh-cn/developer/plugin-development) - 扩展 OpenClaw
- [API 参考](/zh-cn/developer/api-reference) - 接口文档
- [测试指南](/zh-cn/developer/testing) - 测试最佳实践
- [贡献指南](/zh-cn/developer/contributing) - 如何贡献代码

### 运维手册

生产环境部署和维护指南。

- [运维概述](/zh-cn/operations/index) - 运维最佳实践
- [部署指南](/zh-cn/operations/deployment) - 各平台部署
- [监控与日志](/zh-cn/operations/monitoring) - 系统监控
- [故障排除](/zh-cn/operations/troubleshooting) - 问题诊断
- [备份恢复](/zh-cn/operations/backup) - 数据保护
- [安全加固](/zh-cn/operations/security-hardening) - 安全最佳实践

## 快速链接

| 我想要... | 查看 |
|-----------|------|
| 快速开始使用 | [快速入门](/zh-cn/start/quick-start) |
| 连接 WhatsApp | [WhatsApp 配置](/zh-cn/channels/whatsapp) |
| 理解系统架构 | [系统架构](/zh-cn/concepts/architecture) |
| 开发插件 | [插件开发](/zh-cn/developer/plugin-development) |
| 部署到生产环境 | [部署指南](/zh-cn/operations/deployment) |
| 解决问题 | [故障排除](/zh-cn/operations/troubleshooting) |

## 关于 OpenClaw

OpenClaw 是一个强大的多渠道 AI 代理网关，支持：

- **多渠道集成**: WhatsApp、Telegram、Discord、Slack、iMessage 等
- **AI 代理桥接**: 集成 Pi 等 AI 编程代理
- **灵活配置**: JSON5 配置，支持热重载
- **插件系统**: 可扩展的插件架构
- **跨平台**: macOS、Linux、Windows (WSL2)

## 获取帮助

- **GitHub**: [openclaw/openclaw](https://github.com/openclaw/openclaw)
- **问题反馈**: [GitHub Issues](https://github.com/openclaw/openclaw/issues)
- **英文文档**: [docs.openclaw.ai](https://docs.openclaw.ai)

---

*"我们都只是在玩自己的提示词。"* — 一个可能嗑多了 token 的 AI
