---
summary: "OpenClaw 调试工具：监控模式、原始模型流和推理泄漏追踪"
read_when:
  - 你需要检查原始模型输出中的推理泄漏
  - 你想在迭代时以监控模式运行 Gateway
  - 你需要可重复的调试工作流程
title: "调试"
---

# 调试

本文档介绍流式输出的调试辅助工具，特别是当提供商将推理混合到普通文本中时。

## 运行时调试覆盖

在聊天中使用 `/debug` 设置**仅运行时**配置覆盖（内存中，不写入磁盘）。

`/debug` 默认禁用；通过 `commands.debug: true` 启用。这很方便，当你需要切换 obscure 设置而无需编辑 `openclaw.json` 时。

示例：

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` 清除所有覆盖并返回磁盘配置。

## Gateway 监控模式

为实现快速迭代，在文件监控器下运行 Gateway：

```bash
pnpm gateway:watch --force
```

这映射到：

```bash
tsx watch src/entry.ts gateway --force
```

在每次重启后添加任何 Gateway CLI 标志，它们会被传递。

## 开发配置文件 + 开发 Gateway（--dev）

使用开发配置文件隔离状态，并启动一个安全的、可丢弃的设置用于调试。有**两个** `--dev` 标志：

- **全局 `--dev`（配置文件）：** 在 `~/.openclaw-dev` 下隔离状态，并将 Gateway 端口默认为 `19001`（派生端口随之移动）
- **`gateway --dev`：告诉 Gateway 在缺少时自动创建默认配置 + 工作区**（并跳过 BOOTSTRAP.md）

推荐流程（开发配置文件 + 开发引导）：

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

如果你还没有全局安装，通过 `pnpm openclaw ...` 运行 CLI。

这会做什么：

1. **配置文件隔离**（全局 `--dev`）
   - `OPENCLAW_PROFILE=dev`
   - `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   - `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   - `OPENCLAW_GATEWAY_PORT=19001`（浏览器/画布随之移动）

2. **开发引导**（`gateway --dev`）
   - 如果缺少则写入最小配置（`gateway.mode=local`，绑定 loopback）
   - 将 `agent.workspace` 设置为开发工作区
   - 设置 `agent.skipBootstrap=true`（无 BOOTSTRAP.md）
   - 如果缺少则植入工作区文件：`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`
   - 默认身份：**C3‑PO**（协议机器人）
   - 在开发模式下跳过渠道提供商（`OPENCLAW_SKIP_CHANNELS=1`）

重置流程（全新开始）：

```bash
pnpm gateway:dev:reset
```

注意：`--dev` 是一个**全局**配置文件标志，会被一些运行程序消耗。如果需要明确说明，使用环境变量形式：

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` 擦除配置、凭证、sessions 和开发工作区（使用 `trash`，不是 `rm`），然后重新创建默认开发设置。

提示：如果非开发 Gateway 已在运行（launchd/systemd），先停止它：

```bash
openclaw gateway stop
```

## 原始流日志（OpenClaw）

OpenClaw 可以在解析/格式化之前记录**原始助手流**。这是查看推理是否作为纯文本增量到达（或作为单独的 thinking 块）的最佳方式。

通过 CLI 启用：

```bash
pnpm gateway:watch --force --raw-stream
```

可选路径覆盖：

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

等效环境变量：

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

默认文件：

`~/.openclaw/logs/raw-stream.jsonl`

## 原始块日志（pi-mono）

为捕获**原始 OpenAI 兼容块**（在解析成块之前），pi-mono 暴露了单独的日志记录器：

```bash
PI_RAW_STREAM=1
```

可选路径：

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

默认文件：

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注意：这仅由使用 pi-mono 的 `openai-compat` 提供商的进程发出。

## 安全说明

- 原始流日志可能包含完整提示、工具输出和用户数据
- 将日志保留在本地，并在调试后删除
- 如果共享日志，先清除敏感信息和 PII

## 故障排除

### Gateway 不可达？

运行 `openclaw doctor`。

### 日志为空？

检查 Gateway 是否正在运行并写入 `logging.file` 中的文件路径。

### 需要更多细节？

将 `logging.level` 设置为 `debug` 或 `trace` 然后重试。

## 相关文档

- [日志](/logging) - 日志记录详解
- [诊断标志](/diagnostics/flags) - 定向调试日志
- [开发模式](/developer) - 开发配置
