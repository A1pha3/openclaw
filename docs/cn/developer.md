---
summary: "开发者完整指南——OpenClaw 开发环境搭建、项目结构解析、插件开发和代码贡献指南"
read_when:
  - 开始 OpenClaw 本地开发
  - 理解项目代码结构
  - 开发自定义插件和扩展
  - 为 OpenClaw 项目贡献代码
title: "开发者概述"
---

# 👨‍💻 开发者概述

欢迎参与 OpenClaw 开发！OpenClaw 是一个开源项目，欢迎开发者参与贡献无论是修复 bug、添加新功能还是改进文档。本文档将帮助你搭建开发环境、理解项目结构、掌握开发流程，并顺利参与到 OpenClaw 的开发工作中。通过阅读本文档，你将能够本地运行开发版本、阅读和理解核心代码、开发自定义插件，并按照社区规范贡献代码。

> **开源协作的价值**
>
> OpenClaw 的愿景是构建最强大的个人 AI 助手，这个目标需要社区的共同努力才能实现。作为开发者，你将直接参与到这一激动人心的项目中：你的代码将被全球用户使用；你的创意将塑造产品的未来；你的贡献将被社区认可和铭记。让我们一起构建更好的 OpenClaw！

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 搭建完整的本地开发环境
- 从源码编译和运行 OpenClaw
- 理解项目的目录结构和代码组织
- 运行测试和代码检查

### 进阶目标（建议掌握）

- 开发自定义渠道插件
- 开发自定义工具和技能
- 理解核心模块的实现原理
- 阅读和理解 API 文档

### 专家目标（挑战）

- 为核心模块贡献代码
- 设计新功能和架构改进
- 审查他人的 Pull Request
- 参与项目管理决策

---

## 🗺️ 学习路径

根据你的经验水平和贡献目标，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速上手** | 首次参与开发 | 1 小时 | 环境搭建、运行开发版本 |
| **日常开发** | 常规功能开发 | 2 小时 | 代码结构、开发流程 |
| **插件开发** | 开发扩展功能 | 4 小时 | 插件系统、自定义开发 |
| **核心贡献** | 贡献核心代码 | 8 小时 | 核心模块、架构设计 |

---

## 🧠 原理解释：OpenClaw 项目架构

### 项目整体结构

OpenClaw 采用 Monorepo 架构，将核心功能、插件和工具整合在一个代码库中：

```
openclaw/
├── src/                          # 核心源代码
│   ├── agents/                   # AI 代理系统
│   │   ├── core/               # 代理核心逻辑
│   │   ├── runtime/            # 运行时环境
│   │   └── tools/              # 内置工具实现
│   │
│   ├── channels/               # 消息渠道实现
│   │   ├── telegram/          # Telegram 渠道
│   │   ├── whatsapp/          # WhatsApp 渠道
│   │   ├── slack/             # Slack 渠道
│   │   └── ...                # 其他渠道
│   │
│   ├── gateway/                # Gateway 服务
│   │   ├── server/            # 服务器实现
│   │   ├── routing/           # 消息路由
│   │   └── auth/              # 认证授权
│   │
│   ├── cli/                   # 命令行工具
│   │   ├── commands/          # CLI 命令
│   │   └── utils/             # CLI 工具函数
│   │
│   ├── tools/                 # 工具系统
│   │   ├── builtin/           # 内置工具
│   │   └── sdk/               # 工具开发 SDK
│   │
│   └── providers/             # AI 提供商
│       ├── anthropic/         # Anthropic Claude
│       ├── openai/            # OpenAI GPT
│       └── ...                # 其他提供商
│
├── extensions/                 # 插件目录
│   ├── msteams/               # Microsoft Teams 插件
│   ├── matrix/                # Matrix 插件
│   └── ...                    # 其他插件
│
├── packages/                   # 共享包
│   ├── core-sdk/              # 核心 SDK
│   ├── channel-sdk/           # 渠道 SDK
│   └── plugin-sdk/            # 插件 SDK
│
├── docs/                      # 文档
│   ├── zh-cn/                 # 中文文档
│   └── en/                    # 英文文档
│
├── scripts/                   # 构建和发布脚本
├── tests/                     # 测试代码
└── package.json              # 项目配置
```

### 核心模块职责

**Agents 模块**

Agents 模块是 AI 代理的核心实现，负责：对话上下文管理和历史维护；消息构造和提示词优化；工具调用决策和执行协调；模型 API 调用和响应解析。理解 Agents 模块有助于开发更智能的代理行为。

**Channels 模块**

Channels 模块实现了与各个消息平台的连接：协议适配——将通用消息格式转换为各平台特定格式；连接管理——维护 WebSocket 连接和事件监听；消息队列——处理消息的发送和接收；错误恢复——处理连接断开和重连。

**Gateway 模块**

Gateway 模块提供核心的网关服务：WebSocket 服务器实现；消息路由和分发；认证和授权；状态管理和监控。

**Tools 模块**

Tools 模块实现了工具调用系统：工具定义和注册；执行环境隔离；结果收集和返回；错误处理和重试。

---

## 🚀 快速开始

### 环境要求

**必需软件**：

| 软件 | 最低版本 | 推荐版本 | 说明 |
|------|----------|----------|------|
| **Node.js** | 22.0.0 | 22.x LTS | JavaScript 运行时 |
| **pnpm** | 9.0.0 | 最新版本 | 包管理器 |
| **Git** | 2.30.0 | 最新版本 | 版本控制 |
| **TypeScript** | 5.0.0 | 最新版本 | 类型检查 |

**可选软件**：

| 软件 | 用途 |
|------|------|
| **Docker** | 容器化开发和测试 |
| **PostgreSQL** | 数据库功能测试 |
| **Redis** | 缓存功能测试 |

### 安装步骤

**步骤一：克隆仓库**

```bash
# 克隆代码仓库
git clone https://github.com/openclaw/openclaw.git

# 进入项目目录
cd openclaw

# 切换到开发分支
git checkout develop
```

**步骤二：安装依赖**

```bash
# 使用 pnpm 安装依赖
pnpm install

# 安装 UI 相关依赖
pnpm ui:install

# 安装 git hooks
pnpm prepare
```

**步骤三：配置开发环境**

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑环境变量
nano .env

# 关键配置
ANTHROPIC_API_KEY=your-api-key
OPENAI_API_KEY=your-api-key
DATABASE_URL=postgresql://user:pass@localhost:5432/openclaw
```

**步骤四：构建项目**

```bash
# 构建所有包
pnpm build

# 或者增量构建
pnpm build --filter @openclaw/core

# 监听模式构建（开发时使用）
pnpm build:watch
```

**步骤五：运行开发版本**

```bash
# 运行 Gateway（自动重新加载）
pnpm gateway:watch

# 在另一个终端运行 CLI
pnpm openclaw --help
```

### 开发工具配置

**VS Code 设置**：

```json
// .vscode/settings.json
{
  "typescript.preferences.importModuleSpecifier": "non-relative",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": "explicit"
  },
  "files.associations": {
    "*.mdx": "markdown"
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.turbo": true
  }
}
```

**推荐的 VS Code 扩展**：

- ESLint
- Prettier
- TypeScript Hero
- GitLens
- Error Lens

---

## 📁 项目结构详解

### 核心代码结构

**Agents 模块**：

```
src/agents/
├── index.ts                     # 模块入口
├── Agent.ts                     # Agent 基类
├── AgentRuntime.ts             # 运行时环境
├── Context.ts                  # 上下文管理
├── Memory.ts                   # 记忆系统
├── ToolExecutor.ts             # 工具执行器
├── prompts/                    # 提示词模板
│   ├── system.md
│   └── default.md
└── types.ts                    # 类型定义
```

**Channels 模块**：

```
src/channels/
├── index.ts                     # 模块入口
├── Channel.ts                   # Channel 基类
├── ChannelManager.ts           # 渠道管理器
├── router.ts                   # 消息路由
├── types.ts                    # 类型定义
│
├── telegram/
│   ├── index.ts               # Telegram 适配器
│   ├── TelegramChannel.ts     # Telegram 实现
│   ├── handlers/              # 消息处理器
│   └── types.ts               # 类型定义
│
├── whatsapp/
│   └── ...
└── slack/
    └── ...
```

**Gateway 模块**：

```
src/gateway/
├── index.ts                     # 模块入口
├── Gateway.ts                   # Gateway 主类
├── WebSocketServer.ts           # WebSocket 服务器
├── Router.ts                   # HTTP 路由
├── Auth.ts                     # 认证授权
├── Middleware.ts               # 中间件
└── types.ts                    # 类型定义
```

### 插件开发结构

```
extensions/
└── my-extension/
    ├── openclaw.plugin.json    # 插件元数据
    ├── package.json            # 包配置
    ├── tsconfig.json           # TypeScript 配置
    │
    ├── src/
    │   └── index.ts           # 插件入口
    │
    ├── channels/              # 自定义渠道
    │   └── mychannel/
    │       ├── index.ts
    │       ├── MyChannel.ts
    │       └── types.ts
    │
    ├── tools/                 # 自定义工具
    │   └── mytool/
    │       ├── index.ts
    │       ├── MyTool.ts
    │       └── schema.ts
    │
    └── hooks/                 # 自定义钩子
        └── myhook/
            ├── index.ts
            └── MyHook.ts
```

---

## 🔧 开发流程

### 代码开发流程

**步骤一：创建功能分支**

```bash
# 确保从最新 develop 分支创建
git checkout develop
git pull origin develop

# 创建功能分支
git checkout -b feature/my-new-feature

# 或创建 bugfix 分支
git checkout -b fix/issue-description
```

**步骤二：开发功能**

```bash
# 运行开发服务器
pnpm gateway:watch

# 在另一个终端测试
pnpm openclaw message send --to "test" --message "test"
```

**步骤三：编写测试**

```bash
# 运行测试
pnpm test

# 运行特定测试文件
pnpm test src/agents/Agent.test.ts

# 运行测试并查看覆盖率
pnpm test:coverage
```

**步骤四：提交代码**

```bash
# 查看修改
git status

# 添加修改的文件
git add src/agents/Agent.ts

# 提交（使用标准化的提交信息）
pnpm commit "feat(agents): add new memory management feature"
```

### 提交规范

遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

| 类型 | 描述 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档更新 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构（不影响功能） |
| `perf` | 性能优化 |
| `test` | 测试相关 |
| `chore` | 构建或辅助工具变更 |

**提交信息格式**：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**示例**：

```
feat(channels): add support for voice messages

- Implement voice message recording
- Add audio codec conversion
- Integrate with speech-to-text API

Closes #123
```

---

## 📚 开发文档

### 插件开发

**创建新插件**：

```bash
# 使用模板创建插件
pnpm create extension my-extension
```

**插件元数据**：

```json
// openclaw.plugin.json
{
  "id": "my-extension",
  "name": "我的扩展",
  "version": "1.0.0",
  "description": "我的自定义扩展功能",
  "author": {
    "name": "开发者名称",
    "email": "dev@example.com"
  },
  "engine": {
    "openclaw": ">=1.0.0"
  },
  "permissions": [
    "channels",
    "tools"
  ]
}
```

**实现插件**：

```typescript
// src/index.ts
import type { Plugin, PluginAPI } from "openclaw/plugin-sdk";

export default function MyPlugin(api: PluginAPI): Plugin {
  return {
    // 插件元数据
    meta: {
      id: "my-extension",
      version: "1.0.0"
    },
    
    // 初始化
    async init() {
      // 注册渠道
      api.registerChannel(MyChannel);
      
      // 注册工具
      api.registerTool(MyTool);
      
      // 注册钩子
      api.registerHook(MyHook);
    },
    
    // 销毁
    async destroy() {
      // 清理资源
    }
  };
}
```

### 渠道开发

**实现自定义渠道**：

```typescript
// channels/mychannel/index.ts
import type { Channel, ChannelConfig, ChannelMessage } from "openclaw/channel-sdk";

export class MyChannel implements Channel {
  readonly id = "mychannel";
  readonly name = "My Channel";
  
  constructor(private config: ChannelConfig) {}
  
  async connect(): Promise<void> {
    // 建立连接
  }
  
  async disconnect(): Promise<void> {
    // 断开连接
  }
  
  async send(message: ChannelMessage): Promise<void> {
    // 发送消息
  }
  
  onMessage(callback: (message: ChannelMessage) => void): void {
    // 消息监听
  }
  
  async healthCheck(): Promise<boolean> {
    // 健康检查
    return true;
  }
}
```

### 工具开发

**实现自定义工具**：

```typescript
// tools/mytool/index.ts
import type { Tool, ToolResult, ToolContext } from "openclaw/tool-sdk";

export interface MyToolParams {
  input: string;
  option?: boolean;
}

export class MyTool implements Tool {
  readonly name = "myTool";
  readonly description = "我的自定义工具";
  
  readonly parameters = {
    type: "object",
    properties: {
      input: {
        type: "string",
        description: "输入参数"
      },
      option: {
        type: "boolean",
        description: "可选参数",
        default: false
      }
    },
    required: ["input"]
  };
  
  async execute(
    params: MyToolParams,
    context: ToolContext
  ): Promise<ToolResult> {
    try {
      // 执行工具逻辑
      const result = await doSomething(params.input);
      
      return {
        success: true,
        data: result
      };
    } catch (error) {
      return {
        success: false,
        error: {
          message: error.message,
          code: "EXECUTION_ERROR"
        }
      };
    }
  }
}
```

---

## 🧪 测试指南

### 测试框架

OpenClaw 使用 **Vitest** 作为测试框架：

```bash
# 运行所有测试
pnpm test

# 运行指定测试文件
pnpm test src/agents/Agent.test.ts

# 监听模式运行测试
pnpm test:watch

# 运行测试并生成覆盖率报告
pnpm test:coverage
```

### 测试结构

```
tests/
├── unit/                        # 单元测试
│   ├── agents/
│   │   └── Agent.test.ts
│   ├── channels/
│   │   └── ...
│   └── utils/
│       └── ...
│
├── integration/                 # 集成测试
│   ├── channels/
│   │   └── telegram.test.ts
│   └── gateway/
│       └── routing.test.ts
│
├── e2e/                         # 端到端测试
│   └── ...
│
├── fixtures/                    # 测试数据
│   └── ...
│
└── utils/                       # 测试工具
    └── setup.ts
```

### 编写测试

**单元测试示例**：

```typescript
// tests/unit/agents/Agent.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { Agent } from "../../src/agents/Agent";

describe("Agent", () => {
  let agent: Agent;
  
  beforeEach(() => {
    agent = new Agent({
      model: "anthropic/claude-sonnet-4",
      systemPrompt: "You are a helpful assistant."
    });
  });
  
  it("should create agent with correct configuration", () => {
    expect(agent.model).toBe("anthropic/claude-sonnet-4");
    expect(agent.systemPrompt).toBe("You are a helpful assistant.");
  });
  
  it("should process message and return response", async () => {
    const response = await agent.process("Hello!");
    
    expect(response).toBeDefined();
    expect(response.content).toBeDefined();
    expect(typeof response.content).toBe("string");
  });
});
```

---

## 📦 构建和发布

### 构建命令

```bash
# 构建所有包
pnpm build

# 构建特定包
pnpm build --filter @openclaw/core

# 清理构建产物
pnpm clean

# 类型检查
pnpm typecheck

# 代码检查
pnpm lint

# 代码格式化
pnpm format
```

### 发布包

```bash
# 登录 npm
npm login

# 发布到 npm
pnpm publish --access public

# 发布到特定标签
pnpm publish --tag beta
```

---

## 🛠️ 开发支持

### 常用命令速查

| 命令 | 用途 |
|------|------|
| `pnpm install` | 安装依赖 |
| `pnpm build` | 构建项目 |
| `pnpm gateway:watch` | 开发模式运行 |
| `pnpm test` | 运行测试 |
| `pnpm lint` | 代码检查 |
| `pnpm format` | 代码格式化 |
| `pnpm commit` | 提交代码 |

### 获取帮助

- **GitHub Issues**：报告 bug 和提出功能建议
- **Discord 社区**：实时交流和帮助
- **讨论区**：提出问题和分享想法

---

## 📖 相关文档

- [项目结构](/developer/project-structure)——代码结构详解
- [插件开发](/developer/plugin-development)——开发扩展插件
- [测试指南](/developer/testing)——测试最佳实践
- [贡献指南](/developer/contributing)——代码贡献规范
- [API 文档](/reference)——API 参考
