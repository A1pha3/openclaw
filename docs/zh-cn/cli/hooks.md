---
summary: "`openclaw hooks` 命令参考（代理钩子）"
read_when:
  - 想要管理代理钩子
  - 想要安装或更新钩子
title: "hooks"
---

# `openclaw hooks`

管理代理钩子（事件驱动的自动化，用于 `/new`、`/reset` 等命令和网关启动）。

## 为什么需要钩子

钩子让你能够：

- **自动保存上下文**：会话重置时保存对话摘要到记忆
- **审计日志**：记录所有命令事件
- **自定义行为**：在特定事件发生时执行自定义逻辑
- **启动脚本**：网关启动时自动运行初始化任务

## 相关链接

- 钩子概念：[Hooks](/zh-cn/hooks)
- 插件钩子：[Plugins](/zh-cn/plugin#plugin-hooks)

## 列出所有钩子

```bash
openclaw hooks list
```

列出从工作区、托管和内置目录发现的所有钩子。

**选项**：

| 选项 | 说明 |
|------|------|
| `--eligible` | 仅显示符合条件的钩子（需求已满足） |
| `--json` | JSON 输出 |
| `-v, --verbose` | 显示详细信息，包括缺失的需求 |

**示例输出**：

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - 网关启动时运行 BOOT.md
  📝 command-logger ✓ - 将所有命令事件记录到集中审计文件
  💾 session-memory ✓ - 发出 /new 命令时将会话上下文保存到记忆
  😈 soul-evil ✓ - 在清除窗口或随机时机替换注入的 SOUL 内容
```

## 获取钩子信息

```bash
openclaw hooks info <name>
```

显示特定钩子的详细信息。

```bash
openclaw hooks info session-memory
```

**输出**：

```
💾 session-memory ✓ Ready

发出 /new 命令时将会话上下文保存到记忆

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## 检查钩子资格

```bash
openclaw hooks check
```

显示钩子资格状态摘要（多少已就绪 vs 未就绪）。

**输出**：

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## 启用钩子

```bash
openclaw hooks enable <name>
```

启用特定钩子，将其添加到配置（`~/.openclaw/config.json`）。

```bash
openclaw hooks enable session-memory
```

**输出**：

```
✓ Enabled hook: 💾 session-memory
```

**说明**：

- 检查钩子是否存在且符合条件
- 更新 `hooks.internal.entries.<name>.enabled = true`
- 保存配置到磁盘

**注意**：由插件管理的钩子在 `openclaw hooks list` 中显示 `plugin:<id>`，不能在这里启用/禁用。请启用/禁用插件本身。

**启用后**：重启网关以重新加载钩子。

## 禁用钩子

```bash
openclaw hooks disable <name>
```

禁用特定钩子。

```bash
openclaw hooks disable command-logger
```

**输出**：

```
⏸ Disabled hook: 📝 command-logger
```

**禁用后**：重启网关以重新加载钩子。

## 安装钩子

```bash
openclaw hooks install <path-or-spec>
```

从本地文件夹/归档或 npm 安装钩子包。

**功能**：

- 将钩子包复制到 `~/.openclaw/hooks/<id>`
- 在 `hooks.internal.entries.*` 中启用已安装的钩子
- 在 `hooks.internal.installs` 下记录安装

**选项**：

- `-l, --link`：链接本地目录而不是复制（添加到 `hooks.internal.load.extraDirs`）

**支持的归档格式**：`.zip`、`.tgz`、`.tar.gz`、`.tar`

```bash
# 本地目录
openclaw hooks install ./my-hook-pack

# 本地归档
openclaw hooks install ./my-hook-pack.zip

# NPM 包
openclaw hooks install @openclaw/my-hook-pack

# 链接本地目录（不复制）
openclaw hooks install -l ./my-hook-pack
```

## 更新钩子

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

更新已安装的钩子包（仅 npm 安装）。

**选项**：

- `--all`：更新所有跟踪的钩子包
- `--dry-run`：显示将要更改的内容而不实际写入

## 内置钩子

### session-memory

发出 `/new` 命令时将会话上下文保存到记忆。

```bash
openclaw hooks enable session-memory
```

**输出路径**：`~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**文档**：[session-memory](/zh-cn/hooks#session-memory)

### command-logger

将所有命令事件记录到集中审计文件。

```bash
openclaw hooks enable command-logger
```

**输出路径**：`~/.openclaw/logs/commands.log`

**查看日志**：

```bash
# 最近命令
tail -n 20 ~/.openclaw/logs/commands.log

# 格式化输出
cat ~/.openclaw/logs/commands.log | jq .

# 按操作过滤
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**文档**：[command-logger](/zh-cn/hooks#command-logger)

### soul-evil

在清除窗口或随机时机将注入的 `SOUL.md` 内容替换为 `SOUL_EVIL.md`。

```bash
openclaw hooks enable soul-evil
```

**文档**：[SOUL Evil Hook](/zh-cn/hooks/soul-evil)

### boot-md

网关启动时（渠道启动后）运行 `BOOT.md`。

**事件**：`gateway:startup`

```bash
openclaw hooks enable boot-md
```

**文档**：[boot-md](/zh-cn/hooks#boot-md)

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 钩子不运行 | 未启用 | `openclaw hooks enable <name>` |
| 需求未满足 | 缺少配置 | `openclaw hooks info <name>` 查看需求 |
| 安装失败 | 归档格式不支持 | 使用支持的格式 |
| 更新失败 | 非 npm 安装 | 手动更新或重新安装 |
