# 开发者手册

欢迎来到 OpenClaw 开发者手册。本文档帮助您了解如何为 OpenClaw 贡献代码、开发插件或在其基础上构建应用。

## 开发环境搭建

### 系统要求

| 组件 | 版本要求 |
|------|----------|
| Node.js | ≥ 22.12.0 |
| pnpm | ≥ 10.x |
| Git | 最新稳定版 |
| TypeScript | 项目内置 |

可选（用于原生应用开发）：
- Xcode（macOS/iOS 开发）
- Android Studio（Android 开发）
- SwiftFormat、SwiftLint（Swift 代码规范）

### 克隆仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 安装依赖

```bash
pnpm install
```

### 构建项目

```bash
# 完整构建
pnpm build

# 构建 UI
pnpm ui:build
```

### 运行开发模式

```bash
# 运行 CLI
pnpm openclaw --help

# 开发模式运行网关
pnpm gateway:dev

# 监听文件变化
pnpm gateway:watch
```

## 项目结构

```
openclaw/
├── src/                    # 核心源代码
│   ├── agents/             # AI 代理系统
│   ├── channels/           # 渠道抽象层
│   ├── cli/                # CLI 命令框架
│   ├── commands/           # CLI 命令实现
│   ├── config/             # 配置系统
│   ├── gateway/            # 网关服务
│   ├── routing/            # 消息路由
│   ├── plugins/            # 插件系统
│   ├── plugin-sdk/         # 插件 SDK
│   └── ...
├── extensions/             # 官方插件
│   ├── mattermost/
│   ├── matrix/
│   ├── msteams/
│   └── ...
├── apps/                   # 原生应用
│   ├── macos/              # macOS 应用
│   ├── ios/                # iOS 应用
│   └── android/            # Android 应用
├── docs/                   # 文档
├── scripts/                # 构建脚本
├── test/                   # 测试辅助
├── ui/                     # Web UI
└── skills/                 # 技能定义
```

### 核心目录说明

| 目录 | 说明 |
|------|------|
| `src/agents/` | AI 代理生命周期、工具系统、会话管理 |
| `src/gateway/` | WebSocket 服务器、RPC 方法、消息处理 |
| `src/channels/` | 渠道抽象、消息标准化 |
| `src/config/` | 配置解析、验证、类型定义 |
| `src/cli/` | CLI 框架、参数解析、输出格式化 |
| `src/commands/` | 所有 CLI 命令实现 |

## 开发规范

### 代码风格

项目使用 Oxlint 和 Oxfmt 进行代码检查和格式化：

```bash
# 检查代码
pnpm lint

# 格式化代码
pnpm format:fix

# 自动修复
pnpm lint:fix
```

### TypeScript 规范

- 使用 ESM 模块
- 启用严格模式
- 避免使用 `any`
- 为复杂逻辑添加注释

### 命名约定

| 类型 | 规范 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `message-router.ts` |
| 类名 | PascalCase | `MessageRouter` |
| 函数名 | camelCase | `routeMessage` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 类型/接口 | PascalCase | `MessagePayload` |

### 文件大小

- 目标：保持文件在 500 行以内
- 指导线：超过 700 行考虑拆分
- 工具：`pnpm check:loc` 检查文件行数

## 测试

### 测试框架

项目使用 Vitest 进行测试：

```bash
# 运行所有测试
pnpm test

# 监听模式
pnpm test:watch

# 覆盖率报告
pnpm test:coverage

# E2E 测试
pnpm test:e2e
```

### 测试文件命名

- 单元测试：`*.test.ts`（与源文件同目录）
- E2E 测试：`*.e2e.test.ts`

### 覆盖率要求

| 指标 | 阈值 |
|------|------|
| 行覆盖率 | 70% |
| 分支覆盖率 | 70% |
| 函数覆盖率 | 70% |
| 语句覆盖率 | 70% |

### Docker 测试

```bash
# 在线模型测试
pnpm test:docker:live-models

# 网关测试
pnpm test:docker:live-gateway

# 所有 Docker 测试
pnpm test:docker:all
```

## 构建系统

### 主要构建命令

```bash
# TypeScript 编译
pnpm build

# 打包前准备
pnpm prepack

# 生成协议定义
pnpm protocol:gen

# 检查协议变更
pnpm protocol:check
```

### 平台特定构建

```bash
# macOS 应用
pnpm mac:package

# iOS 应用
pnpm ios:build

# Android 应用
pnpm android:assemble
```

## 调试

### 日志调试

```bash
# 详细日志模式
openclaw gateway --verbose

# 查看日志
openclaw logs --tail 100
```

### 配置调试

```json5
{
  logging: {
    level: "debug",
    consoleLevel: "debug",
    consoleStyle: "pretty"
  }
}
```

### VSCode 调试

创建 `.vscode/launch.json`：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Gateway",
      "program": "${workspaceFolder}/openclaw.mjs",
      "args": ["gateway", "--verbose"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    }
  ]
}
```

## 提交规范

### 提交消息格式

使用约定式提交：

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

类型：
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档变更
- `style`: 代码格式（不影响逻辑）
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具变更

示例：
```
feat(telegram): add custom command support

- Add customCommands config option
- Support /command_name format
- Add tests for command parsing
```

### 创建提交

推荐使用项目提供的提交脚本：

```bash
scripts/committer "feat(cli): add verbose flag" src/cli/verbose.ts
```

## 拉取请求

### PR 检查清单

- [ ] 通过所有测试 `pnpm test`
- [ ] 通过代码检查 `pnpm lint`
- [ ] 通过构建 `pnpm build`
- [ ] 添加必要的测试
- [ ] 更新相关文档
- [ ] 更新 CHANGELOG（如需要）

### PR 流程

1. Fork 仓库或创建分支
2. 实现功能/修复
3. 添加测试
4. 确保 CI 通过
5. 提交 PR
6. 等待代码审查
7. 合并

## 发布流程

### 版本号规范

```
YYYY.M.D[-beta.N]
```

示例：
- `2026.1.27` - 稳定版本
- `2026.1.27-beta.1` - 测试版本

### 发布检查

```bash
# 发布前检查
pnpm release:check

# 同步插件版本
pnpm plugins:sync
```

## 下一步

- [项目结构](/zh-CN/developer/project-structure) - 详细代码组织
- [插件开发](/zh-CN/developer/plugin-development) - 创建自定义插件
- [测试指南](/zh-CN/developer/testing) - 测试最佳实践
- [贡献指南](/zh-CN/developer/contributing) - 如何贡献代码
