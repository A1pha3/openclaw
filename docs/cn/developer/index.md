---
summary: 介绍 OpenClaw 开发者手册，涵盖环境搭建、项目结构、开发规范、测试、构建系统、调试、提交规范、PR 指南和发布流程
read_when:
  - 新贡献者开始开发前
  - 需要了解项目整体架构时
  - 设置开发环境时
title: 开发者手册
---

# 开发者手册

欢迎来到 OpenClaw 开发者手册。本文档旨在帮助开发者了解如何为 OpenClaw 贡献代码、开发插件或在其基础上构建应用。在开始开发工作之前，建议您先阅读[架构文档](/zh-CN/concepts/architecture)了解 OpenClaw 的整体设计理念和技术架构。OpenClaw 是一个开源的个人 AI 助手项目，我们欢迎各种形式的贡献，无论是代码改进、文档完善还是问题报告。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 搭建完整的 OpenClaw 开发环境
- 理解项目架构和各模块的职责划分
- 遵循项目编码规范进行开发
- 编写符合质量标准的测试用例
- 使用正确的 Git 工作流提交代码
- 理解发布流程并参与版本发布

### 适用场景

| 场景 | 目标 | 推荐阅读章节 |
|------|------|-------------|
| 首次贡献 | 环境搭建 + 第一个 PR | 环境搭建、提交规范、PR 指南 |
| 插件开发 | 开发自定义插件 | 项目结构、插件系统 |
| 核心开发 | 参与核心功能开发 | 项目结构、测试规范、构建系统 |
| 文档贡献 | 改进项目文档 | 提交规范（docs 类型） |

## 开发环境搭建

### 系统要求详解

在开始搭建开发环境之前，需要确保您的开发机器满足以下要求。这些要求基于 OpenClaw 的技术栈和实际开发需求制定：

| 组件 | 版本要求 | 说明 | 检查命令 |
|------|---------|------|----------|
| Node.js | >= 22.12.0 | 严格版本要求，不支持 21.x | `node --version` |
| pnpm | >= 10.x | 项目使用 pnpm 作为包管理器 | `pnpm --version` |
| Git | 最新稳定版 | 版本控制必需 | `git --version` |
| TypeScript | 项目内置 | 严格模式开发 | - |
| Docker | 可选 | 用于 Docker 测试和镜像构建 | `docker --version` |

**推荐开发环境**：

| 操作系统 | 推荐程度 | 说明 |
|---------|---------|------|
| macOS | ⭐⭐⭐⭐⭐ | 官方开发平台，完整支持所有功能 |
| Ubuntu/Debian | ⭐⭐⭐⭐⭐ | Linux 开发体验良好 |
| Windows + WSL2 | ⭐⭐⭐⭐ | 推荐使用 WSL2 而非原生 Windows |
| 其他 Linux | ⭐⭐⭐⭐ | 需要确保支持 systemd 和常用开发工具 |

### 克隆仓库

```bash
# 使用 SSH（推荐，需要先配置 SSH key）
git clone git@github.com:openclaw/openclaw.git
cd openclaw

# 或使用 HTTPS
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 安装依赖

```bash
# 安装项目依赖
pnpm install

# 如果遇到依赖问题，尝试
pnpm install --no-frozen-lockfile

# 安装 Git hooks（代码检查、格式化等）
pnpm prek install
```

### 构建项目

```bash
# 完整构建（编译 TypeScript）
pnpm build

# 构建 UI 组件
pnpm ui:build

# 构建特定模块
pnpm build --filter @openclaw/cli
pnpm build --filter @openclaw/gateway
```

### 运行开发模式

```bash
# 运行 CLI（TypeScript 直接执行）
pnpm openclaw --help

# 开发模式运行网关（监听文件变化）
pnpm gateway:dev

# 监听模式（自动重新加载）
pnpm gateway:watch

# 运行测试
pnpm test

# 代码检查
pnpm lint
pnpm format
```

## 项目结构概览

### 架构设计原则

OpenClaw 的项目结构设计遵循以下核心原则，这些原则影响了目录的组织方式：

**关注点分离**：不同功能的代码放置在不同的目录中，每个模块有清晰的职责边界。CLI 相关代码集中在 `src/cli/`，渠道实现集中在 `src/channels/`，网关服务在 `src/gateway/`。这种组织方式使得代码易于导航和理解。

**可扩展性**：渠道和插件系统采用插件化架构，新渠道或插件只需实现标准接口即可集成到系统中。`extensions/` 目录用于存放官方插件，`src/` 中的渠道实现遵循统一的接口规范。

**层次清晰**：从顶层到底层依次为应用层（CLI 命令）、网关层（WebSocket 服务）、渠道层（消息收发）和基础设施层（配置、日志、存储）。数据流向清晰，各层之间的依赖关系明确。

### 目录结构总览

```
openclaw/
├── src/                              # 核心源代码
│   ├── agents/                       # AI 代理系统
│   │   ├── runtime.ts               # 代理运行时核心
│   │   ├── session.ts               # 会话管理
│   │   ├── tools/                   # 工具系统
│   │   └── ...
│   ├── channels/                    # 渠道抽象层
│   │   ├── types.ts                 # 渠道接口定义
│   │   ├── manager.ts               # 渠道管理器
│   │   ├── telegram/                # Telegram 渠道实现
│   │   ├── whatsapp/                # WhatsApp 渠道实现
│   │   ├── discord/                 # Discord 渠道实现
│   │   └── ...
│   ├── cli/                         # CLI 命令框架
│   │   ├── index.ts                # CLI 主入口
│   │   ├── commands/                # CLI 命令实现
│   │   └── progress.ts             # 进度条工具
│   ├── commands/                    # 聊天命令实现
│   │   ├── status.ts               # /status 命令
│   │   ├── reset.ts                # /reset 命令
│   │   └── ...
│   ├── gateway/                     # 网关服务
│   │   ├── server.ts               # WebSocket 服务器
│   │   ├── protocol.ts             # 协议定义
│   │   ├── handlers/               # 消息处理器
│   │   └── ...
│   ├── routing/                     # 消息路由
│   │   ├── router.ts              # 路由逻辑
│   │   └── bindings.ts            # 路由绑定
│   ├── infra/                       # 基础设施
│   │   ├── config/                 # 配置管理
│   │   ├── logging/                # 日志系统
│   │   ├── storage/               # 数据存储
│   │   └── crypto/                 # 加密工具
│   ├── media/                       # 媒体处理
│   │   ├── pipeline.ts            # 处理管道
│   │   ├── transcription.ts       # 语音转文字
│   │   └── image.ts               # 图像处理
│   └── ...
├── extensions/                      # 官方插件
│   ├── mattermost/                 # Mattermost 插件
│   ├── matrix/                     # Matrix 插件
│   ├── msteams/                    # Microsoft Teams 插件
│   └── ...
├── apps/                            # 原生应用
│   ├── macos/                      # macOS 菜单栏应用
│   ├── ios/                        # iOS 应用
│   └── android/                    # Android 应用
├── docs/                            # 文档
├── scripts/                         # 构建脚本
├── test/                            # 测试辅助
├── ui/                              # Web UI
├── skills/                          # 技能定义
└── 配置文件...
```

### 核心模块说明

| 模块 | 路径 | 职责 | 关键类/接口 |
|------|------|------|-------------|
| CLI | `src/cli/` | 命令行界面入口 | `program`, `Command` |
| 渠道 | `src/channels/` | 消息收发抽象 | `ChannelProvider`, `Message` |
| 网关 | `src/gateway/` | WebSocket 服务 | `GatewayServer`, `WSHandler` |
| 代理 | `src/agents/` | AI 运行时 | `AgentRuntime`, `Session` |
| 路由 | `src/routing/` | 消息路由 | `Router`, `Bindings` |
| 媒体 | `src/media/` | 媒体处理 | `Pipeline`, `Transcriber` |
| 基础设施 | `src/infra/` | 配置、日志、存储 | `Config`, `Logger` |

## 开发规范

### 代码风格

OpenClaw 使用 Oxlint 进行代码检查，Oxfmt 进行代码格式化。保持代码风格一致是每个贡献者的责任：

```bash
# 检查代码风格
pnpm lint

# 自动修复格式问题
pnpm format:fix

# 完整检查并修复
pnpm lint:fix

# 检查并修复特定文件
pnpm lint:fix src/cli/index.ts
```

### TypeScript 规范

**必须遵循的规则**：

| 规则 | 说明 | 违反示例 |
|------|------|----------|
| ESM 模块 | 使用 ESM 导入导出 | `require()` |
| 严格模式 | 启用 TypeScript strict | `any` 类型 |
| 类型定义 | 避免使用 `any` | 隐式 `any` |
| 注释 | 为复杂逻辑添加注释 | 无注释的算法 |
| 命名 | 遵循项目命名约定 | 错误的命名风格 |

**命名约定速查**：

| 类型 | 规范 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `message-router.ts` |
| 类名 | PascalCase | `MessageRouter` |
| 函数名 | camelCase | `routeMessage()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 接口/类型 | PascalCase | `MessagePayload` |
| 私有属性 | 以 `_` 开头 | `_privateField` |

### 文件大小管理

**指导原则**：

- **目标**：保持文件在 500 行以内
- **警告线**：超过 700 行应考虑拆分
- **检查工具**：`pnpm check:loc` 检查文件行数

**何时拆分文件**：

| 信号 | 建议操作 |
|------|----------|
| 文件超过 700 行 | 评估是否可以按功能拆分 |
| 单一文件包含多个不相关功能 | 按功能拆分为多个文件 |
| 类/函数过于庞大 | 提取为独立模块 |
| 测试文件过长 | 拆分测试用例 |

### 代码注释规范

```typescript
/**
 * 函数简要说明
 *
 * 更详细的说明，解释函数的目的、工作原理和重要细节
 *
 * @param paramName - 参数说明
 * @returns 返回值说明
 *
 * @example
 * ```typescript
 * // 使用示例
 * const result = myFunction('input')
 * ```
 */
function myFunction(paramName: string): Promise<Result> {
  // 复杂逻辑需要详细注释
  // 解释为什么这样做，而非只说明做了什么

  // TODO: 标记需要改进的地方
  // FIXME: 标记已知问题

  return result
}
```

## 测试规范

### 测试框架

OpenClaw 使用 Vitest 作为测试框架：

```bash
# 运行所有测试
pnpm test

# 监听模式（开发时使用）
pnpm test:watch

# 生成覆盖率报告
pnpm test:coverage

# 运行 E2E 测试
pnpm test:e2e

# 运行特定测试文件
pnpm test src/cli/index.test.ts
```

### 测试文件组织

| 测试类型 | 文件命名模式 | 位置 |
|---------|-------------|------|
| 单元测试 | `*.test.ts` | 与源文件同目录 |
| 集成测试 | `*.integration.test.ts` | `test/` 目录 |
| E2E 测试 | `*.e2e.test.ts` | `test/` 目录 |

### 覆盖率要求

| 指标 | 阈值 | 说明 |
|------|------|------|
| 行覆盖率 | 70% | 代码行数覆盖 |
| 分支覆盖率 | 70% | 条件分支覆盖 |
| 函数覆盖率 | 70% | 函数/方法覆盖 |
| 语句覆盖率 | 70% | 语句覆盖 |

### 编写测试

```typescript
// src/utils/calculator.test.ts
import { describe, it, expect } from 'vitest'
import { Calculator } from './calculator'

describe('Calculator', () => {
  describe('add', () => {
    it('should add two numbers correctly', () => {
      const calculator = new Calculator()
      expect(calculator.add(2, 3)).toBe(5)
    })

    it('should handle negative numbers', () => {
      const calculator = new Calculator()
      expect(calculator.add(-1, 1)).toBe(0)
    })

    it('should throw on overflow', () => {
      const calculator = new Calculator({ maxValue: 100 })
      expect(() => calculator.add(100, 1)).toThrow('Overflow')
    })
  })
})
```

### Docker 测试

```bash
# 在线模型测试（需要 API 密钥）
pnpm test:docker:live-models

# 网关测试
pnpm test:docker:live-gateway

# 所有 Docker 测试
pnpm test:docker:all

# 运行单个 Docker 测试
docker run --rm -it \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  ghcr.io/openclaw/openclaw:test \
  pnpm test
```

## 构建系统

### 主要构建命令

| 命令 | 用途 | 输出 |
|------|------|------|
| `pnpm build` | 完整构建 | `dist/` 目录 |
| `pnpm prepack` | 打包前准备 | - |
| `pnpm protocol:gen` | 生成协议定义 | `src/gateway/protocol/` |
| `pnpm protocol:check` | 检查协议变更 | - |

### 平台特定构建

```bash
# macOS 应用
pnpm mac:package

# iOS 应用
pnpm ios:build

# Android 应用
pnpm android:assemble

# 构建所有应用
pnpm build:apps
```

### 发布构建

```bash
# 发布前完整构建
pnpm build
pnpm test
pnpm lint

# 生成发布包
pnpm package
```

## 调试技巧

### 日志调试

```bash
# 详细日志模式
openclaw gateway --verbose

# 查看日志
openclaw logs --tail 100

# 按级别过滤
openclaw logs --level debug
```

### 配置调试日志级别

```json5
{
  "logging": {
    "level": "debug",           // debug | info | warn | error
    "consoleLevel": "debug",    // 控制台输出级别
    "consoleStyle": "pretty",   // pretty | json | compact
    "file": "debug.log"         // 调试日志文件
  }
}
```

### VSCode 调试配置

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Gateway",
      "program": "${workspaceFolder}/dist/cli/index.js",
      "args": ["gateway", "--verbose"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "sourceMaps": true,
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug CLI",
      "program": "${workspaceFolder}/dist/cli/index.js",
      "args": ["--help"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "sourceMaps": true
    }
  ]
}
```

### 常见调试场景

```
调试场景速查：

1. Gateway 启动失败
   └── pnpm build && node dist/cli/index.js gateway --verbose

2. 渠道连接问题
   └── openclaw logs --level debug --channel <channel>

3. 消息路由问题
   └── openclaw logs --level debug | grep routing

4. 性能问题
   └── openclaw status --deep
   └── top -p $(pgrep -f openclaw)

5. 内存泄漏
   └── node --inspect dist/cli/index.js gateway
   └── 使用 Chrome DevTools 分析堆快照
```

## 提交规范

### 约定式提交

OpenClaw 使用[约定式提交](https://www.conventionalcommits.org/)规范：

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**提交类型**：

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(telegram): add custom command support` |
| `fix` | 修复 bug | `fix(gateway): resolve memory leak in session manager` |
| `docs` | 文档变更 | `docs(readme): update installation instructions` |
| `style` | 代码格式（不影响逻辑） | `style(cli): format code with oxfmt` |
| `refactor` | 重构 | `refactor(channels): simplify message parsing` |
| `test` | 测试相关 | `test(utils): add unit tests for calculator` |
| `chore` | 构建/工具变更 | `chore: update dependencies` |
| `perf` | 性能优化 | `perf(agent): improve message processing speed` |
| `ci` | CI 配置变更 | `ci: add parallel test execution` |

**范围（Scope）参考**：

| 范围 | 说明 |
|------|------|
| `cli` | CLI 命令相关 |
| `gateway` | Gateway 服务相关 |
| `channel` | 渠道抽象层 |
| `telegram` / `whatsapp` / `discord` | 特定渠道 |
| `agent` | AI 代理相关 |
| `docs` | 文档相关 |
| `build` | 构建系统 |
| `test` | 测试相关 |

**提交示例**：

```
feat(telegram): add custom command support

- Add customCommands config option
- Support /command_name format
- Add tests for command parsing

Closes #123
```

```
fix(gateway): resolve WebSocket connection leak

The gateway was not properly cleaning up WebSocket connections
when clients disconnected abruptly. This led to resource leaks
and degraded performance under high load.

Fixes #456
```

```
docs(readme): update installation instructions

- Add Node.js version requirement (>= 22.12.0)
- Fix outdated pnpm installation command
- Add troubleshooting section for common issues
```

### 使用提交脚本

OpenClaw 提供了提交辅助脚本来确保提交信息格式正确：

```bash
# 使用提交脚本（推荐）
scripts/committer "feat(cli): add verbose flag" src/cli/verbose.ts

# 手动提交（需自行确保格式正确）
git add .
git commit -m "feat(channel): add new feature"
```

## 拉取请求指南

### PR 检查清单

提交 PR 前请确认以下项目：

- [ ] 代码通过所有测试 `pnpm test`
- [ ] 代码通过检查 `pnpm lint`
- [ ] 构建成功 `pnpm build`
- [ ] 添加必要的测试
- [ ] 更新相关文档
- [ ] 遵循提交规范
- [ ] PR 描述清晰完整

### PR 流程

```
PR 提交流程：

1. Fork 仓库（首次贡献者）
   └── GitHub 页面点击 Fork 按钮

2. 创建分支
   └── git checkout -b feature/my-feature

3. 实现功能/修复
   └── 编写代码
   └── 添加测试
   └── 运行检查

4. 提交更改
   └── 使用 commit script
   └── 遵循提交规范

5. 推送到远程
   └── git push origin feature/my-feature

6. 创建 PR
   └── GitHub 页面创建 PR
   └── 填写 PR 模板
   └── 链接相关 Issue

7. 响应审查
   └── 回复审查意见
   └── 必要时修改代码
   └── 通过 CI 检查

8. 合并 PR
   └── 等待维护者合并
   └── 删除分支
```

### PR 描述模板

```markdown
## 概述
[简要描述此次 PR 的内容和目的]

## 变更内容
- [ ] 新增功能
- [ ] 修复 bug
- [ ] 性能优化
- [ ] 代码重构
- [ ] 文档更新
- [ ] 测试更新

## 测试说明
[描述如何测试这些更改]

## 检查清单
- [ ] 代码通过 lint
- [ ] 代码通过测试
- [ ] 构建成功
- [ ] 新增测试覆盖变更
- [ ] 更新相关文档

## 截图（如果适用）
[添加界面截图]

## 相关 Issue
[链接相关 Issue]

## 注意事项
[任何需要审查者注意的事项]
```

### 审查反馈处理

```
审查反馈处理流程：

1. 理解反馈
   └── 仔细阅读每条评论
   └── 如有疑问，请求澄清

2. 处理反馈
   └── 直接在代码中修改
   └── 或在评论中回复解释

3. 更新 PR
   └── 添加新提交
   └── 通知审查者

4. 继续审查
   └── 等待再次审查
   └── 重复直到通过
```

## 发布流程

### 版本号规范

OpenClaw 使用时间基版本号：

```
格式：YYYY.M.D[-beta.N]

示例：
- 2026.1.27    - 稳定版本
- 2026.1.27-beta.1  - 第一个测试版本
- 2026.1.27-beta.2  - 第二个测试版本
```

### 发布前检查

```bash
# 发布前完整检查
pnpm release:check

# 同步插件版本
pnpm plugins:sync

# 运行完整测试
pnpm test
pnpm test:coverage

# 代码检查
pnpm lint

# 构建
pnpm build
```

### 发布步骤

```bash
# 1. 更新版本号
# 编辑 package.json 中的 version 字段

# 2. 更新 CHANGELOG
# 参考 CHANGELOG.md 格式添加条目

# 3. 提交更改
scripts/committer "chore: release v2026.1.28" CHANGELOG.md package.json

# 4. 创建标签
git tag v2026.1.28
git push origin v2026.1.28

# 5. GitHub Actions 自动发布
# CI 会自动构建并发布到 npm
```

### 发布渠道

| 渠道 | 说明 | npm 标签 |
|------|------|----------|
| stable | 稳定版本 | `latest` |
| beta | 测试版本 | `beta` |
| dev | 开发版本 | `dev` |

## 开发者资源

### 常用命令速查

| 命令 | 用途 |
|------|------|
| `pnpm install` | 安装依赖 |
| `pnpm build` | 构建项目 |
| `pnpm test` | 运行测试 |
| `pnpm lint` | 代码检查 |
| `pnpm format` | 代码格式化 |
| `pnpm openclaw --help` | CLI 帮助 |
| `pnpm gateway:dev` | 开发模式运行网关 |

### 开发工作流建议

```
日常开发工作流：

1. 早晨同步
   └── git pull origin main
   └── pnpm install（如有更新）

2. 开始工作
   └── git checkout -b feature/xxx
   └── 开发实现
   └── 编写测试
   └── 本地验证

3. 提交
   └── 运行测试
   └── 运行 lint
   └── 使用 commit script 提交

4. 定期同步
   └── git fetch upstream
   └── git rebase upstream/main
```

### 常见问题

```
Q: 构建失败怎么办？
A: 
   1. 确保 Node.js 版本 >= 22.12.0
   2. 删除 node_modules 重新安装
   3. 检查是否有未提交的 lock 文件变更

Q: 测试失败怎么办？
A:
   1. 查看测试输出了解失败原因
   2. 检查是否修改了会影响测试的代码
   3. 本地修复后提交

Q: lint 报错怎么办？
A:
   1. 运行 pnpm format:fix 自动修复格式问题
   2. 检查特定问题并手动修复
   3. 理解规则，避免再次出现

Q: 如何贡献新渠道？
A:
   1. 参考现有渠道实现（如 src/channels/telegram/）
   2. 实现 ChannelProvider 接口
   3. 在 channels/manager.ts 中注册
   4. 添加配置模式
   5. 编写测试和文档
```

## 下一步

| 文档 | 说明 |
|------|------|
| [项目结构](/zh-CN/developer/project-structure) | 详细代码组织结构 |
| [插件开发](/zh-CN/developer/plugin-development) | 创建自定义插件 |
| [测试指南](/zh-CN/developer/testing) | 测试最佳实践 |
| [贡献指南](/zh-CN/developer/contributing) | 贡献指南详细说明 |
| [架构文档](/zh-CN/concepts/architecture) | 系统架构设计 |
