---
summary: 介绍 OpenClaw 项目结构，涵盖目录概览、核心源代码、原生应用、插件扩展、文档、脚本、配置文件、关键模块和开发工作流
read_when:
  - 需要理解项目代码组织结构时
  - 查找特定模块位置时
  - 添加新功能需要了解目录结构时
title: 项目结构
---

# 项目结构

本文档详细介绍 OpenClaw 代码库的组织结构，帮助开发者快速理解项目布局、定位代码文件以及掌握添加新功能的最佳实践。在深入代码之前，建议您先阅读[开发者手册](/zh-CN/developer/index)了解整体开发流程。本文档是代码导航的参考指南，按模块功能组织，而非按字母顺序列出所有文件。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 理解 OpenClaw 项目目录的组织原则
- 快速定位特定功能的代码位置
- 掌握各模块的职责边界和依赖关系
- 了解添加新渠道、插件和 CLI 命令的标准流程
- 理解构建输出和版本管理的最佳实践

### 适用场景

| 场景 | 推荐阅读章节 | 期望结果 |
|------|-------------|----------|
| 首次浏览代码 | 目录概览、核心模块 | 建立整体认知 |
| 修改渠道代码 | src/channels/、渠道模块说明 | 准确定位 |
| 开发新插件 | extensions/、插件结构 | 遵循规范 |
| 添加 CLI 命令 | src/cli/、CLI 模块说明 | 正确集成 |
| 构建打包 | 构建输出、版本位置 | 理解构建流程 |

## 目录概览

### 顶层目录结构

```
openclaw/
├── src/                              # 核心源代码（TypeScript）
├── extensions/                       # 官方插件（独立包）
├── apps/                             # 原生应用（iOS/Android/macOS）
├── docs/                             # 项目文档
├── scripts/                          # 构建和工具脚本
├── test/                             # 测试辅助文件
├── ui/                               # Web UI 组件
├── skills/                           # 技能定义文件
├── dist/                             # 构建输出（自动生成）
├── node_modules/                     # 依赖包（自动生成）
├── package.json                      # 主项目配置
├── pnpm-workspace.yaml               # pnpm 工作区配置
├── tsconfig.json                     # TypeScript 配置
└── 配置文件...
```

### 组织原则

OpenClaw 的目录结构设计遵循以下核心原则，理解这些原则有助于推断未知功能的位置：

**按功能划分**：源代码按功能模块组织，而非按技术类型。CLI 相关代码集中在 `src/cli/`，网关服务在 `src/gateway/`，渠道实现在 `src/channels/`。这种组织方式使得按功能导航代码变得直观。

**层次清晰**：从高层的应用命令到底层的基础设施，层次分明。`src/` 包含所有核心功能，`extensions/` 存放可选的扩展功能，`apps/` 存放平台特定的原生应用。

**关注点分离**：每个目录有明确的职责边界。例如，`src/cli/` 只负责命令行界面，`src/channels/` 只负责消息渠道抽象。这种分离使得各模块可以独立演进和测试。

## src/ 核心源代码

### CLI 模块 (src/cli/)

CLI 模块是用户与 OpenClaw 交互的主要入口，提供所有命令行功能：

```
src/cli/
├── index.ts                         # CLI 主入口，命令注册
├── progress.ts                      # 进度条/spinner 工具 (@clack/prompts)
├── commands/                        # CLI 命令实现
│   ├── gateway.ts                   # gateway 命令
│   ├── agent.ts                     # agent 命令
│   ├── channels.ts                  # channels 命令
│   ├── sessions.ts                  # sessions 命令
│   ├── pairing.ts                   # pairing 命令
│   ├── message.ts                   # message 命令
│   ├── config.ts                    # config 命令
│   ├── doctor.ts                    # doctor 命令（诊断）
│   ├── health.ts                    # health 命令
│   ├── update.ts                    # update 命令
│   ├── service.ts                   # service 命令
│   ├── plugins.ts                   # plugins 命令
│   ├── onboarding.ts                # onboarding 命令
│   ├── logs.ts                      # logs 命令
│   └── ...
└── types.ts                         # CLI 类型定义
```

**CLI 模块职责**：

| 功能 | 文件 | 说明 |
|------|------|------|
| 命令注册 | `index.ts` | 使用 commander 注册所有命令 |
| 进度显示 | `progress.ts` | 使用 @clack/prompts 提供 UI 反馈 |
| 网关命令 | `gateway.ts` | 启动/停止 Gateway 服务 |
| 渠道管理 | `channels.ts` | 查看/配置渠道连接 |
| 消息发送 | `message.ts` | 发送消息到各渠道 |
| 系统诊断 | `doctor.ts` | 自动检测和修复问题 |
| 配置管理 | `config.ts` | 读取/修改配置 |

**CLI 入口实现示例**：

```typescript
// src/cli/index.ts
import { Command } from 'commander'
import { version } from '../utils/version'

const program = new Command()
  .name('openclaw')
  .description('Personal AI Assistant')
  .version(version)

// 注册命令
program.addCommand(gatewayCommand)
program.addCommand(agentCommand)
program.addCommand(channelsCommand)
program.addCommand(sessionsCommand)
program.addCommand(messageCommand)
program.addCommand(configCommand)
program.addCommand(doctorCommand)
// ...

export default program
```

### 渠道模块 (src/channels/)

渠道模块实现消息的统一抽象，支持多种即时通讯平台：

```
src/channels/
├── types.ts                         # 渠道接口定义
├── manager.ts                       # 渠道管理器
├── factory.ts                       # 渠道工厂
├── router.ts                        # 消息路由（与 routing/ 相关）
├── queue.ts                        # 消息队列
│
├── telegram/                        # Telegram 渠道
│   ├── index.ts                    # 入口
│   ├── provider.ts                 # TelegramProvider 实现
│   ├── types.ts                    # 类型定义
│   └── events.ts                   # 事件处理
│
├── whatsapp/                       # WhatsApp 渠道
│   ├── index.ts
│   ├── provider.ts                 # WhatsAppProvider 实现
│   ├── types.ts
│   └── baileys/                    # Baileys 底层封装
│
├── discord/                         # Discord 渠道
│   ├── index.ts
│   ├── provider.ts                 # DiscordProvider 实现
│   ├── types.ts
│   └── events.ts
│
├── slack/                          # Slack 渠道
│   └── ...
│
├── signal/                         # Signal 渠道
│   └── ...
│
├── imessage/                       # iMessage 渠道
│   └── ...
│
└── web/                            # Web 渠道（WebChat）
    ├── index.ts
    └── client.ts                   # WebSocket 客户端
```

**渠道接口定义**：

```typescript
// src/channels/types.ts

/**
 * 渠道提供商接口
 * 所有渠道必须实现此接口
 */
export interface ChannelProvider {
  /** 渠道唯一标识 */
  id: string

  /** 渠道显示名称 */
  name: string

  /** 渠道配置 */
  config: ChannelConfig

  // 生命周期方法
  start(): Promise<void>
  stop(): Promise<void>
  restart(): Promise<void>

  // 连接状态
  isConnected(): boolean
  getConnectionStatus(): ConnectionStatus

  // 消息处理
  send(message: OutboundMessage): Promise<void>
  onMessage(handler: MessageHandler): void

  // 健康检查
  healthCheck(): Promise<HealthCheckResult>

  // 凭证管理
  login(): Promise<void>
  logout(): Promise<void>
  isLoggedIn(): boolean
}

/**
 * 标准化消息格式
 * 所有渠道都转换为统一格式
 */
export interface OutboundMessage {
  /** 目标 */
  target: {
    type: 'user' | 'group' | 'channel'
    id: string
    name?: string
  }

  /** 消息内容 */
  content: {
    type: 'text' | 'image' | 'audio' | 'video' | 'file' | 'mixed'
    text?: string
    attachments?: Attachment[]
  }

  /** 发送选项 */
  options?: {
    replyTo?: string
    mentions?: string[]
    formatting?: MessageFormatting
  }
}
```

### 网关模块 (src/gateway/)

网关模块是 OpenClaw 的控制平面，通过 WebSocket 提供所有核心功能：

```
src/gateway/
├── server.ts                        # WebSocket 服务器
├── protocol.ts                      # 协议定义
│
├── handlers/                        # 消息处理器
│   ├── index.ts                    # 处理器注册
│   ├── rpc-handler.ts              # RPC 方法处理
│   ├── message-handler.ts           # 消息处理
│   ├── session-handler.ts           # 会话管理
│   ├── channel-handler.ts           # 渠道管理
│   └── ...
│
├── middleware/                      # 中间件
│   ├── auth.ts                      # 认证中间件
│   ├── logging.ts                   # 日志中间件
│   ├── rate-limit.ts               # 限流中间件
│   └── validation.ts                # 参数验证
│
├── clients/                         # 客户端管理
│   ├── client.ts                   # 客户端连接
│   ├── session.ts                  # 客户端会话
│   └── manager.ts                  # 客户端管理器
│
└── types.ts                        # Gateway 类型定义
```

**Gateway 服务器核心结构**：

```typescript
// src/gateway/server.ts

export class GatewayServer {
  /** WebSocket 服务器实例 */
  private server: WebSocketServer

  /** 客户端连接管理 */
  private clients: Map<string, Client>

  /** 消息路由器 */
  private router: MessageRouter

  /** RPC 方法注册表 */
  private rpcMethods: Map<string, RPCHandler>

  /** 渠道管理器 */
  private channelManager: ChannelManager

  constructor(options: GatewayOptions) {
    this.server = new WebSocketServer({ port: options.port })
    this.clients = new Map()
    this.router = new MessageRouter()
    this.rpcMethods = new Map()
    this.channelManager = new ChannelManager()
  }

  // 核心方法
  start(): Promise<void>
  stop(): Promise<void>

  // 客户端处理
  handleConnection(ws: WebSocket): void
  handleMessage(client: Client, message: Message): void
  handleDisconnect(client: Client): void

  // RPC 注册
  registerRPCHandler(method: string, handler: RPCHandler): void
  unregisterRPCHandler(method: string): void
}
```

### 代理模块 (src/agents/)

代理模块实现 AI 助手的核心运行时：

```
src/agents/
├── runtime.ts                       # 代理运行时核心
├── session.ts                       # 会话管理
├── context.ts                       # 上下文管理
│
├── tools/                          # 工具系统
│   ├── registry.ts                 # 工具注册表
│   ├── manager.ts                  # 工具管理器
│   ├── browser.ts                 # 浏览器工具
│   ├── bash.ts                    # Shell 执行工具
│   ├── file.ts                    # 文件操作工具
│   ├── http.ts                    # HTTP 请求工具
│   ├── cron.ts                    # 定时任务工具
│   ├── canvas.ts                  # 画布工具
│   ├── nodes.ts                   # 节点工具
│   └── ...
│
├── loop.ts                         # Agent Loop 实现
├── history.ts                      # 历史记录管理
├── parser.ts                       # 消息解析
└── types.ts                        # Agent 类型定义
```

### 路由模块 (src/routing/)

路由模块处理消息的路由决策：

```
src/routing/
├── router.ts                       # 路由逻辑
├── bindings.ts                     # 路由绑定
├── matcher.ts                      # 消息匹配器
├── filters.ts                      # 路由过滤器
│
├── rules/                         # 路由规则
│   ├── channel-rule.ts            # 渠道路由规则
│   ├── session-rule.ts            # 会话路由规则
│   ├── group-rule.ts              # 群组路由规则
│   └── mention-rule.ts             # 提及路由规则
│
└── resolvers/                      # 路由解析器
    ├── channel-resolver.ts         # 渠道解析
    └── session-resolver.ts         # 会话解析
```

### 基础设施模块 (src/infra/)

基础设施模块提供公共功能支持：

```
src/infra/
├── config/                         # 配置管理
│   ├── index.ts                   # 配置入口
│   ├── loader.ts                  # 配置加载
│   ├── schema.ts                  # JSON Schema
│   ├── validation.ts               # 配置验证
│   └── migrations.ts               # 配置迁移
│
├── logging/                        # 日志系统
│   ├── index.ts                   # 日志入口
│   ├── logger.ts                  # Logger 类
│   ├── formatters.ts              # 日志格式化
│   └── transports.ts              # 日志传输（文件/控制台）
│
├── storage/                        # 数据存储
│   ├── index.ts                   # 存储入口
│   ├── file.ts                    # 文件存储
│   ├── keyvalue.ts                # KV 存储
│   └── encrypted.ts                # 加密存储
│
├── crypto/                         # 加密工具
│   ├── index.ts
│   ├── encrypt.ts                 # 加密/解密
│   ├── hash.ts                   # 哈希函数
│   └── random.ts                  # 随机数生成
│
└── util/                           # 通用工具
    ├── async.ts                   # 异步工具
    ├── errors.ts                  # 错误定义
    └── helpers.ts                 # 辅助函数
```

### 媒体处理模块 (src/media/)

媒体处理模块处理图片、音频、视频等多媒体：

```
src/media/
├── pipeline.ts                     # 媒体处理管道
├── types.ts                        # 媒体类型定义
│
├── image.ts                        # 图像处理
│   ├── processor.ts               # 图像处理器
│   ├── formats.ts                 # 支持的格式
│   └── thumbnail.ts               # 缩略图生成
│
├── audio/                          # 音频处理
│   ├── processor.ts               # 音频处理器
│   ├── formats.ts                 # 音频格式
│   └── voice.ts                   # 语音合成
│
├── transcription.ts                # 语音转文字
│   ├── stt.ts                     # Speech-to-Text
│   └── providers/                 # STT 提供商
│
└── video.ts                        # 视频处理
```

### 其他重要模块

| 模块 | 路径 | 职责 |
|------|------|------|
| CLI 命令 | `src/commands/` | 聊天内命令实现（/status、/reset 等） |
| 终端 UI | `src/terminal/` | 表格输出、颜色主题 |
| Web 提供者 | `src/provider-web.ts` | Web 界面提供者 |
| 类型定义 | `src/types/` | 全局 TypeScript 类型 |

## apps/ 原生应用

### iOS 应用

```
apps/ios/
├── Sources/                        # Swift 源代码
│   └── OpenClaw/
│       ├── App/                    # 应用入口
│       ├── Views/                  # SwiftUI 视图
│       ├── Services/               # 服务层
│       ├── Models/                 # 数据模型
│       ├── Nodes/                  # 节点功能
│       └── Resources/              # 资源文件
│
├── Tests/                          # 测试
│
├── Project.swift                   # Tuist 项目配置
│
└── Info.plist                     # 应用配置
```

### Android 应用

```
apps/android/
├── app/
│   └── src/
│       ├── main/
│       │   ├── java/              # Kotlin 代码
│       │   │   └── com/openclaw/
│       │   │       ├── App/       # 应用入口
│       │   │       ├── Views/      # 界面
│       │   │       ├── Services/   # 服务
│       │   │       └── Nodes/      # 节点
│       │   └── res/               # 资源文件
│       ├── test/                   # 单元测试
│       └── androidTest/            # 集成测试
│
├── build.gradle.kts               # Gradle 构建配置
└── settings.gradle.kts
```

### macOS 应用

```
apps/macos/
├── Sources/
│   └── OpenClaw/
│       ├── App/                    # 应用主体
│       │   ├── OpenClawApp.swift
│       │   ├── MenuBarController.swift
│       │   └── AppDelegate.swift
│       ├── Views/                  # SwiftUI 视图
│       │   ├── MenuBarView.swift
│       │   ├── SettingsView.swift
│       │   └── ...
│       ├── Services/               # 服务层
│       │   ├── GatewayService.swift
│       │   ├── VoiceWakeService.swift
│       │   └── ...
│       ├── Nodes/                  # 节点功能
│       │   ├── CameraNode.swift
│       │   └── ScreenRecordNode.swift
│       └── Resources/              # 资源
│           ├── Assets.xcassets
│           └── Info.plist
│
├── Tests/
│
└── OpenClaw.xcodeproj            # Xcode 项目
```

## extensions/ 插件扩展

### 插件目录结构

插件是 OpenClaw 扩展功能的主要方式，每个插件是一个独立的包：

```
extensions/
├── matrix/                         # Matrix 渠道插件
│   ├── src/
│   │   ├── index.ts               # 插件入口
│   │   ├── provider.ts            # MatrixProvider 实现
│   │   ├── types.ts               # 类型定义
│   │   └── events.ts              # 事件处理
│   ├── package.json              # 插件依赖（不使用 workspace:*）
│   ├── tsconfig.json
│   ├── README.md
│   └── .npmrc
│
├── msteams/                       # Microsoft Teams 插件
│   └── ...
│
├── zalo/                          # Zalo 插件
│   └── ...
│
├── zalouser/                      # Zalo Personal 插件
│   └── ...
│
├── voice-call/                    # 语音通话插件
│   └── ...
│
└── ...                            # 其他插件
```

### 插件结构规范

每个插件必须遵循以下结构：

```
extensions/<plugin-name>/
├── src/
│   ├── index.ts                   # 必须：插件入口，导出 PluginManifest
│   ├── provider.ts               # 可选：渠道提供者实现
│   ├── types.ts                  # 可选：类型定义
│   └── ...                       # 其他模块
├── package.json                  # 必须：独立依赖管理
├── tsconfig.json                 # 必须：TypeScript 配置
├── README.md                     # 必须：插件文档
└── .npmrc                        # 推荐：禁用 workspace 协议
```

**插件入口示例**：

```typescript
// extensions/matrix/src/index.ts
import { PluginManifest, PluginContext } from '@openclaw/plugin-sdk'

export const manifest: PluginManifest = {
  name: '@openclaw/matrix',
  version: '1.0.0',
  description: 'Matrix 渠道支持',
  channels: [
    {
      id: 'matrix',
      name: 'Matrix',
      provider: './provider.js'
    }
  ],
  dependencies: {
    '@openclaw/plugin-sdk': '^2026.1.0'
  }
}

export async function init(context: PluginContext): Promise<void> {
  // 初始化插件
  const provider = await context.channels.register(
    manifest.channels[0],
    './provider.js'
  )
}

export async function destroy(): Promise<void> {
  // 清理资源
}
```

### 插件依赖管理

**重要规则**：插件必须独立管理依赖，不使用 `workspace:*` 协议：

```json
{
  "name": "@openclaw/matrix",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {
    "@openclaw/plugin-sdk": "^2026.1.0",
    "matrix-js-sdk": "^25.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

## docs/ 文档目录

```
docs/
├── index.md                        # 文档首页
├── start/                         # 入门指南
│   ├── getting-started.md
│   ├── quick-start.md
│   └── ...
├── concepts/                      # 概念说明
│   ├── architecture.md
│   ├── agent.md
│   └── ...
├── channels/                      # 渠道文档
│   ├── whatsapp.md
│   ├── telegram.md
│   └── ...
├── gateway/                       # 网关文档
│   ├── configuration.md
│   ├── security.md
│   └── ...
├── tools/                         # 工具文档
│   ├── browser.md
│   └── ...
├── platforms/                     # 平台指南
│   ├── mac/
│   ├── ios/
│   ├── android/
│   └── ...
├── reference/                    # 参考文档
├── automation/                   # 自动化
│   ├── webhooks.md
│   └── cron-jobs.md
│
└── zh-cn/                         # 中文文档
    ├── start/
    ├── concepts/
    ├── channels/
    ├── gateway/
    └── ...                        # 本文档所在位置
```

## scripts/ 构建脚本

```
scripts/
├── build/                         # 构建脚本
│   ├── build-all.sh              # 构建所有模块
│   ├── bundle-a2ui.sh            # A2UI 打包
│   └── ...
│
├── package-mac-app.sh             # macOS 应用打包脚本
├── restart-mac.sh                 # macOS 重启脚本
├── committer                      # 提交辅助脚本
├── update-clawtributors.ts        # 更新贡献者列表
├── clawlog.sh                     # 日志查询工具
└── ...
```

## 配置文件

| 文件 | 用途 |
|------|------|
| `package.json` | 主项目配置、脚本、依赖 |
| `pnpm-workspace.yaml` | pnpm 工作区配置 |
| `pnpm-lock.yaml` | 依赖锁定文件（自动生成） |
| `tsconfig.json` | TypeScript 编译配置 |
| `vitest.config.ts` | Vitest 测试配置 |
| `oxlint.json` | Oxlint 代码检查配置 |
| `.github/workflows/` | GitHub Actions CI/CD 配置 |
| `.env.example` | 环境变量示例 |
| `AGENTS.md` | AI 代理指南 |

## 关键模块详解

### CLI 模块深度解析

CLI 是用户与 OpenClaw 交互的主要入口：

```
用户输入 → CLI 解析 → 命令执行 → 结果输出
   │            │            │
   ▼            ▼            ▼
program    commander     具体命令
progress   输入验证       业务逻辑
colors     参数处理       数据获取
```

### 渠道模块深度解析

渠道抽象是 OpenClaw 的核心特性：

```
┌─────────────────────────────────────────────────────────┐
│                   渠道抽象层                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │   Telegram  │  │  WhatsApp   │  │  Discord    │   │
│  │  Provider   │  │  Provider   │  │  Provider   │   │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘   │
│         │                │                │          │
│         └────────────────┼────────────────┘          │
│                          ▼                             │
│              ┌─────────────────────┐                  │
│              │   ChannelManager   │                  │
│              │   (统一管理)         │                  │
│              └──────────┬──────────┘                  │
│                         │                             │
│         ┌───────────────┼───────────────┐             │
│         ▼               ▼               ▼             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐     │
│  │   Message   │ │  Presence   │ │   Events    │     │
│  │  Adapter   │ │  Adapter    │ │  Adapter    │     │
│  └─────────────┘ └─────────────┘ └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

### 网关模块深度解析

Gateway 是 OpenClaw 的控制平面：

```
┌─────────────────────────────────────────────────────────────┐
│                    Gateway WebSocket 服务                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    WebSocket Server                      │ │
│  │  ws://127.0.0.1:18789                                   │ │
│  └─────────────────────────┬───────────────────────────────┘ │
│                            │                                 │
│          ┌─────────────────┼─────────────────┐              │
│          ▼                 ▼                 ▼              │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐       │
│  │   CLI      │   │  Web UI    │   │   Apps     │       │
│  │  Client    │   │  Client    │   │  Client    │       │
│  └─────────────┘   └─────────────┘   └─────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### 代理模块深度解析

Agent Runtime 管理 AI 助手会话：

```
┌─────────────────────────────────────────────────────────────┐
│                     Agent Runtime                            │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                  Session Manager                       │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│  │  │ Session  │ │ Session  │ │ Session  │ │ ...      │ │ │
│  │  │  (main)  │ │  (wa)    │ │  (tg)    │ │          │ │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ │ │
│  └─────────────────────────┬───────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────┼───────────────────────────────┐ │
│  │                 Tool Registry                         │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│  │  │  bash   │ │ browser │ │  cron   │ │  file   │ │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ │ │
│  └─────────────────────────┬───────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────┼───────────────────────────────┐ │
│  │              LLM Provider (Anthropic/OpenAI)            │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## 开发工作流

### 添加新渠道

```
新渠道开发流程：

1. 创建渠道目录
   └── src/channels/<channel-name>/

2. 实现 ChannelProvider 接口
   └── provider.ts 继承 ChannelProvider
   └── 定义消息类型映射
   └── 实现生命周期方法

3. 注册渠道
   └── 在 channels/manager.ts 中注册
   └── 添加配置模式到 infra/config/schema.ts

4. 添加测试
   └── <channel-name>.test.ts

5. 编写文档
   └── docs/channels/<channel-name>.md
```

### 添加新插件

```
插件开发流程：

1. 创建插件目录
   └── extensions/<plugin-name>/

2. 初始化 package.json
   └── 使用独立依赖管理
   └── 不使用 workspace:* 协议

3. 实现插件功能
   └── 实现 PluginManifest
   └── 实现 provider（如果需要）

4. 注册到项目
   └── 在 .github/labeler.yml 添加标签

5. 编写文档
   └── extensions/<plugin-name>/README.md
```

### 添加新 CLI 命令

```
CLI 命令开发流程：

1. 创建命令文件
   └── src/cli/commands/<command-name>.ts

2. 实现命令逻辑
   └── 使用 commander.js API
   └── 添加参数解析
   └── 实现命令处理

3. 注册命令
   └── 在 src/cli/index.ts 中添加

4. 添加帮助文本
   └── 使用 description 和 addHelpText

5. 编写文档
   └── docs/cli/<command-name>.md
```

## 构建输出

### dist/ 目录结构

```
dist/                                  # 构建输出（自动生成）
├── cli/                               # CLI 编译输出
│   ├── index.js                     # CLI 入口
│   ├── index.d.ts                   # 类型声明
│   └── commands/                     # 命令编译输出
│
├── gateway/                          # Gateway 编译输出
│   ├── server.js
│   ├── server.d.ts
│   ├── handlers/
│   └── ...
│
├── channels/                         # 渠道编译输出
│   ├── index.js
│   ├── index.d.ts
│   ├── telegram/
│   ├── whatsapp/
│   └── ...
│
├── agents/                          # Agent 编译输出
│   ├── runtime.js
│   ├── runtime.d.ts
│   └── tools/
│
└── types/                           # 共享类型
    └── index.d.ts
```

## 版本位置

OpenClaw 的版本号分布在多个位置，需要同步更新：

| 文件 | 版本字段 | 用途 |
|------|---------|------|
| `package.json` | `version` | CLI/npm 包版本 |
| `apps/android/app/build.gradle.kts` | `versionName`/`versionCode` | Android 应用版本 |
| `apps/ios/Sources/Info.plist` | `CFBundleShortVersionString`/`CFBundleVersion` | iOS 应用版本 |
| `apps/macos/Sources/OpenClaw/Resources/Info.plist` | 同上 | macOS 应用版本 |
| `docs/install/updating.md` | 固定版本号 | 文档中的版本参考 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [开发者手册](/zh-CN/developer) | 开发环境搭建和规范 |
| [测试指南](/zh-CN/developer/testing) | 测试最佳实践 |
| [贡献指南](/zh-CN/developer/contributing) | 贡献代码指南 |
| [插件开发](/zh-CN/developer/plugin-development) | 插件开发详细指南 |
| [架构文档](/zh-CN/concepts/architecture) | 系统架构设计 |
| [API 参考](/zh-CN/reference) | API 详细文档 |
