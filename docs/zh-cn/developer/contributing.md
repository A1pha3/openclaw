# 贡献指南

感谢您对 OpenClaw 的贡献兴趣！本文档介绍如何为项目做出贡献。

## 开始之前

### 环境要求

- **Node.js**: 22+
- **包管理器**: pnpm（推荐）或 Bun
- **Git**: 最新稳定版

### 克隆仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### 安装依赖

```bash
pnpm install
```

### 设置 pre-commit hooks

```bash
prek install
```

这会安装与 CI 相同的检查钩子。

## 开发工作流

### 1. 创建分支

从 `main` 分支创建功能分支：

```bash
git checkout main
git pull
git checkout -b feature/your-feature-name
```

分支命名约定：

| 类型 | 格式 | 示例 |
|------|------|------|
| 功能 | `feature/description` | `feature/add-matrix-support` |
| 修复 | `fix/description` | `fix/telegram-reconnect` |
| 文档 | `docs/description` | `docs/zh-cn-channels` |
| 重构 | `refactor/description` | `refactor/router-cleanup` |

### 2. 开发

```bash
# 开发模式（自动重载）
pnpm dev

# 或使用网关监视模式
pnpm gateway:watch

# 运行 CLI
pnpm openclaw <command>
```

### 3. 检查代码

```bash
# 类型检查和构建
pnpm build

# 代码检查
pnpm lint

# 格式化
pnpm format

# 运行测试
pnpm test
```

### 4. 提交更改

使用项目提供的提交脚本：

```bash
scripts/committer "提交信息" file1.ts file2.ts
```

或手动提交（确保遵循提交消息规范）：

```bash
git add <files>
git commit -m "类型: 简洁的描述"
```

### 5. 创建 Pull Request

```bash
git push -u origin feature/your-feature-name
```

然后在 GitHub 上创建 PR。

## 提交消息规范

遵循简洁、面向动作的提交消息格式：

```
<类型>: <描述>
```

### 类型

| 类型 | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档更新 |
| `refactor` | 代码重构（无功能变化） |
| `test` | 测试相关 |
| `chore` | 构建/工具/依赖更新 |
| `style` | 代码格式化（无功能变化） |
| `perf` | 性能优化 |

### 示例

```
feat: add Matrix channel support
fix: resolve Telegram reconnection issue
docs: add Chinese translation for channels
refactor: simplify message routing logic
test: add unit tests for router
chore: update dependencies
```

### 提交消息指南

- 使用英文
- 第一行不超过 72 个字符
- 使用现在时态（"add feature" 而非 "added feature"）
- 不要以句号结尾
- 相关更改分组提交，避免混合不相关的重构

## Pull Request 指南

### PR 标题

与提交消息格式相同：

```
feat: add Matrix channel support
```

### PR 描述模板

```markdown
## 概述
简要描述此 PR 的目的和范围。

## 变更内容
- 变更 1
- 变更 2

## 测试
描述如何测试这些变更。

## 相关 Issue
Fixes #123
```

### PR 检查清单

- [ ] 代码通过 lint 检查 (`pnpm lint`)
- [ ] 代码通过构建 (`pnpm build`)
- [ ] 测试通过 (`pnpm test`)
- [ ] 添加了必要的测试
- [ ] 更新了相关文档
- [ ] 更新了 CHANGELOG（如果是用户可见的变更）

## 代码风格

### TypeScript

- 使用严格类型，避免 `any`
- 遵循 ESM 模块规范
- 使用 Oxlint 和 Oxfmt 进行检查和格式化

```typescript
// 好的做法
function processMessage(message: Message): Result {
  // ...
}

// 避免
function processMessage(message: any): any {
  // ...
}
```

### 禁止的模式

| 禁止 | 原因 |
|------|------|
| `as any` | 类型安全 |
| `@ts-ignore` | 类型安全 |
| `@ts-expect-error` | 类型安全 |
| 空的 `catch(e) {}` | 错误处理 |
| 删除失败的测试 | 测试完整性 |

### 文件组织

- 保持文件简洁，目标 ~500-700 行
- 相关代码放在一起
- 测试文件与源文件共置（`foo.ts` 旁边放 `foo.test.ts`）

### 注释

为复杂或不明显的逻辑添加简短注释：

```typescript
// 使用指数退避重试，最多 3 次
async function fetchWithRetry(url: string): Promise<Response> {
  // ...
}
```

## 文档贡献

### 文档位置

- 英文文档: `docs/`
- 中文文档: `docs/zh-CN/`

### 文档格式

- 使用 Mintlify 格式
- 内部链接使用根相对路径，不带 `.md` 扩展名
- 避免在标题中使用 em-dash 和撇号

```markdown
# 正确的链接
[配置参考](/zh-CN/config/reference)
[网关配置](/zh-CN/concepts/gateway#配置)

# 错误的链接
[配置参考](./reference.md)
[配置参考](reference)
```

### 代码示例

- 代码块保持英文（配置键、命令）
- 代码注释可以使用中文

```json5
{
  // 代理配置
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

## 添加新功能

### 添加新渠道

1. 在 `src/` 创建渠道目录
2. 实现 `ChannelProvider` 接口
3. 注册到渠道管理器
4. 添加配置模式
5. 编写测试
6. 编写文档
7. 更新 `.github/labeler.yml`

### 添加新插件

1. 在 `extensions/` 创建插件目录
2. 初始化 `package.json`
   - 不使用 `workspace:*` 依赖
   - `openclaw` 放在 `devDependencies` 或 `peerDependencies`
3. 实现插件入口
4. 编写文档
5. 更新标签规则

### 添加新 CLI 命令

1. 在 `src/cli/commands/` 创建命令文件
2. 实现命令逻辑
3. 注册到 CLI 主入口
4. 添加帮助文本
5. 编写测试和文档

## Changelog 更新

对于用户可见的变更，需要更新 Changelog：

```markdown
## [版本号] - 日期

### Added
- 新增 Matrix 渠道支持 (#123) - 感谢 @contributor

### Fixed
- 修复 Telegram 重连问题 (#456)

### Changed
- 改进消息路由性能
```

注意事项：

- 保持最新发布版本在顶部
- 包含 PR 号和贡献者致谢
- 纯测试更改通常不需要 Changelog 条目

## 获取帮助

- **Discord**: [加入社区](https://discord.gg/clawd)
- **GitHub Issues**: 报告 bug 或功能请求
- **GitHub Discussions**: 一般问题讨论

## 贡献者许可协议

通过提交 PR，您同意您的贡献将在 MIT 许可证下发布。

## 致谢

感谢所有贡献者！AI/vibe-coded PRs 欢迎！

## 相关文档

- [开发指南](/zh-CN/developer)
- [项目结构](/zh-CN/developer/project-structure)
- [测试指南](/zh-CN/developer/testing)
- [插件开发](/zh-CN/developer/plugin-development)
