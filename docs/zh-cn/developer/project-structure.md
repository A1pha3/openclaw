# 项目结构

本文档介绍 Moltbot 代码库的组织结构，帮助开发者快速理解项目布局。

## 目录概览

```
moltbot/
├── src/                    # 核心源代码
├── apps/                   # 原生应用（iOS、Android、macOS）
├── extensions/             # 插件/扩展
├── docs/                   # 文档
├── scripts/                # 构建和工具脚本
├── tests/                  # 测试文件
├── dist/                   # 构建输出
└── 配置文件
```

## src/ - 核心源代码

```
src/
├── cli/                    # CLI 入口和命令处理
│   ├── index.ts           # CLI 主入口
│   ├── commands/          # 各个 CLI 命令
│   └── progress.ts        # 进度条/spinner 工具
│
├── commands/              # 聊天命令实现
│   ├── status.ts
│   ├── reset.ts
│   └── ...
│
├── channels/              # 渠道抽象层
│   ├── types.ts           # 渠道接口定义
│   └── manager.ts         # 渠道管理器
│
├── telegram/              # Telegram 渠道实现
├── discord/               # Discord 渠道实现
├── slack/                 # Slack 渠道实现
├── signal/                # Signal 渠道实现
├── imessage/              # iMessage 渠道实现
├── web/                   # WhatsApp Web 实现
│
├── routing/               # 消息路由
│   ├── router.ts          # 路由逻辑
│   └── bindings.ts        # 路由绑定
│
├── infra/                 # 基础设施
│   ├── config/            # 配置管理
│   ├── logging/           # 日志系统
│   ├── storage/           # 数据存储
│   └── crypto/            # 加密工具
│
├── media/                 # 媒体处理管道
│   ├── pipeline.ts        # 媒体处理流程
│   ├── transcription.ts   # 语音转文字
│   └── image.ts           # 图像处理
│
├── agent/                 # AI 代理运行时
│   ├── runtime.ts         # 代理运行时
│   ├── session.ts         # 会话管理
│   └── tools/             # 工具实现
│
├── gateway/               # WebSocket 网关
│   ├── server.ts          # WS 服务器
│   ├── protocol.ts        # 协议定义
│   └── handlers/          # 消息处理器
│
├── provider-web.ts        # Web 提供者
├── terminal/              # 终端 UI 工具
│   ├── table.ts           # 表格输出
│   └── palette.ts         # 颜色主题
│
└── types/                 # 类型定义
    └── index.ts
```

## apps/ - 原生应用

```
apps/
├── ios/                   # iOS 节点应用
│   ├── Sources/           # Swift 源代码
│   ├── Tests/             # 测试
│   └── Project.swift      # Tuist 项目配置
│
├── android/               # Android 节点应用
│   ├── app/
│   │   └── src/
│   │       ├── main/
│   │       │   ├── java/  # Kotlin/Java 代码
│   │       │   └── res/   # 资源文件
│   │       └── test/
│   └── build.gradle.kts
│
└── macos/                 # macOS 菜单栏应用
    ├── Sources/
    │   └── Moltbot/
    │       ├── App/       # 应用主体
    │       ├── Views/     # SwiftUI 视图
    │       ├── Services/  # 服务层
    │       └── Resources/ # 资源
    └── Tests/
```

## extensions/ - 插件扩展

```
extensions/
├── matrix/                # Matrix 渠道插件
│   ├── src/
│   │   └── index.ts
│   ├── package.json
│   └── tsconfig.json
│
├── msteams/               # Microsoft Teams 插件
├── zalo/                  # Zalo 插件
├── zalouser/              # Zalo 个人账号插件
├── voice-call/            # 语音通话插件
└── ...
```

### 插件结构

每个插件遵循标准结构：

```
extensions/<plugin-name>/
├── src/
│   ├── index.ts           # 插件入口
│   ├── provider.ts        # 渠道提供者实现
│   └── types.ts           # 类型定义
├── package.json           # 插件依赖（不使用 workspace:*）
├── tsconfig.json
└── README.md
```

## docs/ - 文档

```
docs/
├── index.md               # 文档首页
├── start/                 # 入门指南
├── concepts/              # 概念说明
├── channels/              # 渠道文档
├── gateway/               # 网关文档
├── tools/                 # 工具文档
├── platforms/             # 平台指南
│   ├── mac/
│   ├── ios/
│   ├── android/
│   ├── linux/
│   └── windows/
├── reference/             # 参考文档
├── zh-cn/                 # 中文文档
└── images/                # 文档图片
```

## scripts/ - 脚本

```
scripts/
├── build/                 # 构建脚本
│   ├── build-all.sh
│   └── bundle-a2ui.sh
├── package-mac-app.sh     # macOS 应用打包
├── restart-mac.sh         # 重启 macOS 应用
├── committer             # 提交辅助脚本
├── update-clawtributors.ts # 更新贡献者列表
└── clawlog.sh             # 日志查询工具
```

## 配置文件

```
moltbot/
├── package.json           # 主项目配置
├── pnpm-workspace.yaml    # pnpm 工作区配置
├── pnpm-lock.yaml         # 依赖锁定文件
├── tsconfig.json          # TypeScript 配置
├── vitest.config.ts       # Vitest 测试配置
├── oxlint.json            # Oxlint 配置
├── .github/
│   ├── workflows/         # GitHub Actions
│   └── labeler.yml        # PR 标签规则
├── .env.example           # 环境变量示例
└── AGENTS.md              # AI 代理指南
```

## 关键模块说明

### CLI 模块 (`src/cli/`)

CLI 是用户与 Moltbot 交互的主要入口：

```typescript
// src/cli/index.ts
import { Command } from 'commander'

const program = new Command()
  .name('moltbot')
  .description('Personal AI Assistant')
  .version(version)

// 注册各个命令
program.addCommand(gatewayCommand)
program.addCommand(agentCommand)
program.addCommand(channelsCommand)
// ...
```

### 渠道模块 (`src/channels/`)

渠道抽象层定义了统一的消息接口：

```typescript
// src/channels/types.ts
interface ChannelProvider {
  id: string
  name: string
  
  // 生命周期
  start(): Promise<void>
  stop(): Promise<void>
  
  // 消息处理
  send(message: OutboundMessage): Promise<void>
  onMessage(handler: MessageHandler): void
}
```

### 网关模块 (`src/gateway/`)

WebSocket 网关是控制平面的核心：

```typescript
// src/gateway/server.ts
class GatewayServer {
  // 客户端连接管理
  clients: Map<string, Client>
  
  // 消息路由
  router: MessageRouter
  
  // RPC 处理
  handleRPC(method: string, params: unknown): Promise<unknown>
}
```

### 代理模块 (`src/agent/`)

AI 代理运行时管理会话和工具调用：

```typescript
// src/agent/runtime.ts
class AgentRuntime {
  // 会话管理
  sessions: SessionManager
  
  // 工具注册
  tools: ToolRegistry
  
  // 消息处理
  process(message: InboundMessage): Promise<Response>
}
```

## 开发工作流

### 添加新渠道

1. 在 `src/` 下创建渠道目录
2. 实现 `ChannelProvider` 接口
3. 在 `src/channels/manager.ts` 注册
4. 添加配置模式到 `src/infra/config/schema.ts`
5. 编写文档和测试

### 添加新插件

1. 在 `extensions/` 下创建插件目录
2. 初始化 `package.json`（不使用 `workspace:*`）
3. 实现插件入口
4. 在 `.github/labeler.yml` 添加标签规则
5. 编写文档

### 添加新 CLI 命令

1. 在 `src/cli/commands/` 创建命令文件
2. 实现命令逻辑
3. 在 `src/cli/index.ts` 注册命令
4. 添加帮助文本和文档

## 构建输出

```
dist/
├── cli/                   # CLI 编译输出
├── gateway/               # 网关编译输出
├── channels/              # 渠道编译输出
└── types/                 # 类型声明文件
```

## 版本位置

版本号分布在多个位置：

| 文件 | 用途 |
|------|------|
| `package.json` | CLI/npm 版本 |
| `apps/android/app/build.gradle.kts` | Android versionName/versionCode |
| `apps/ios/Sources/Info.plist` | iOS CFBundleShortVersionString |
| `apps/macos/Sources/Moltbot/Resources/Info.plist` | macOS 版本 |
| `docs/install/updating.md` | 文档中的固定版本 |

## 相关文档

- [开发指南](/zh-cn/developer)
- [测试指南](/zh-cn/developer/testing)
- [贡献指南](/zh-cn/developer/contributing)
- [插件开发](/zh-cn/developer/plugin-development)
