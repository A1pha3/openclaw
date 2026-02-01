---
summary: "开发者文档入口 - 开发环境搭建和贡献指南"
read_when:
  - 开始开发 OpenClaw
  - 了解项目结构
  - 贡献代码
title: "开发者概述"
---

# 👨‍💻 开发者概述

欢迎参与 OpenClaw 开发！本文档帮助你搭建开发环境并了解项目结构。

---

## 🎯 开发环境要求

- **Node.js** >= 22
- **pnpm**（推荐）或 npm
- **Git**

---

## 🚀 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 2. 安装依赖

```bash
pnpm install
```

### 3. 构建项目

```bash
pnpm ui:build
pnpm build
```

### 4. 运行开发版本

```bash
pnpm openclaw onboard
```

---

## 📁 项目结构

```
openclaw/
├── src/              # 核心源码
│   ├── agents/       # 代理系统
│   ├── channels/     # 渠道实现
│   ├── cli/          # CLI 工具
│   ├── gateway/      # 网关服务
│   └── ...
├── extensions/       # 插件扩展
├── docs/             # 文档
└── skills/           # 技能工具
```

---

## 📚 开发文档

| 文档 | 内容 |
|------|------|
| [**项目结构**](/zh-CN/developer/project-structure) | 代码组织详解 |
| [**插件开发**](/zh-CN/developer/plugin-development) | 创建扩展 |
| [**测试指南**](/zh-CN/developer/testing) | 测试最佳实践 |
| [**贡献指南**](/zh-CN/developer/contributing) | 如何贡献代码 |

---

## 🔧 开发命令

```bash
# 运行测试
pnpm test

# 代码检查
pnpm lint

# 格式代码
pnpm format

# 构建
pnpm build
```

---

## 🆘 开发支持

- [GitHub Issues](https://github.com/openclaw/openclaw/issues)
- [Discord 社区](https://discord.gg/clawd)

---

## 📖 相关文档

- [项目结构](/zh-CN/developer/project-structure)
- [插件开发](/zh-CN/developer/plugin-development)
- [贡献指南](/zh-CN/developer/contributing)
