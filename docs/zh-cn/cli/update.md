---
summary: "CLI 参考 - `openclaw update` (安全的源码更新 + 网关自动重启)"
read_when:
  - 想要安全地更新源码检出
  - 需要了解 `--update` 简写行为
title: "update"
---

# `openclaw update`

安全更新 OpenClaw 并在 stable/beta/dev 渠道之间切换。

## 安装方式说明

如果你通过 **npm/pnpm** 安装（全局安装，无 git 元数据），更新通过包管理器流程进行，参见 [更新指南](/zh-cn/install/updating)。

## 用法

```bash
# 更新到当前渠道的最新版本
openclaw update

# 查看更新状态
openclaw update status

# 交互式更新向导
openclaw update wizard

# 切换到 beta 渠道
openclaw update --channel beta

# 切换到 dev 渠道
openclaw update --channel dev

# 指定特定版本标签
openclaw update --tag beta

# 更新后不重启网关
openclaw update --no-restart

# JSON 输出
openclaw update --json

# 简写形式
openclaw --update
```

## 选项

| 选项 | 说明 |
|------|------|
| `--no-restart` | 更新成功后跳过重启网关服务 |
| `--channel <stable\|beta\|dev>` | 设置更新渠道（git + npm；保存到配置） |
| `--tag <dist-tag\|version>` | 仅此次更新覆盖 npm dist-tag 或版本 |
| `--json` | 打印机器可读的 `UpdateRunResult` JSON |
| `--timeout <seconds>` | 每步超时（默认 1200 秒） |

**注意**：降级需要确认，因为旧版本可能破坏配置。

## `update status`

显示当前更新渠道 + git 标签/分支/SHA（源码检出），以及更新可用性。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

选项：

| 选项 | 说明 |
|------|------|
| `--json` | 打印机器可读的状态 JSON |
| `--timeout <seconds>` | 检查超时（默认 3 秒） |

## `update wizard`

交互式流程，选择更新渠道并确认更新后是否重启网关（默认重启）。如果选择 `dev` 但没有 git 检出，会提供创建选项。

## 工作原理

当你显式切换渠道（`--channel ...`）时，OpenClaw 会同步安装方式：

| 渠道 | 行为 |
|------|------|
| `dev` | 确保 git 检出存在（默认 `~/openclaw`，可用 `OPENCLAW_GIT_DIR` 覆盖），更新它，并从该检出安装全局 CLI |
| `stable`/`beta` | 使用匹配的 dist-tag 从 npm 安装 |

## Git 检出流程

### 渠道行为

| 渠道 | 操作 |
|------|------|
| `stable` | 检出最新的非 beta 标签，然后构建 + doctor |
| `beta` | 检出最新的 `-beta` 标签，然后构建 + doctor |
| `dev` | 检出 `main`，然后 fetch + rebase |

### 高级流程

1. 需要干净的工作树（无未提交更改）
2. 切换到选定的渠道（标签或分支）
3. 获取上游（仅 dev）
4. 仅 dev：在临时工作树中预检 lint + TypeScript 构建；如果 tip 失败，回退最多 10 个提交找到最新的干净构建
5. Rebase 到选定的提交（仅 dev）
6. 安装依赖（优先 pnpm；npm 回退）
7. 构建 + 构建控制 UI
8. 运行 `openclaw doctor` 作为最终的"安全更新"检查
9. 同步插件到活动渠道（dev 使用内置扩展；stable/beta 使用 npm）并更新 npm 安装的插件

## `--update` 简写

`openclaw --update` 重写为 `openclaw update`（便于 shell 和启动脚本使用）。

## 使用场景

### 日常更新

```bash
# 检查更新
openclaw update status

# 执行更新
openclaw update
```

### 测试 Beta 功能

```bash
# 切换到 beta
openclaw update --channel beta

# 测试完成后切回 stable
openclaw update --channel stable
```

### 开发模式

```bash
# 切换到 dev 渠道
openclaw update --channel dev

# 之后保持更新
openclaw update
```

## 相关文档

- `openclaw doctor`（在 git 检出上提供先运行更新的选项）
- [开发渠道](/zh-cn/install/development-channels)
- [更新指南](/zh-cn/install/updating)
- [CLI 参考](/zh-cn/cli)
