---
summary: "CLI 参考 - `openclaw logs` (通过 RPC 查看网关日志)"
read_when:
  - 需要远程查看网关日志（无需 SSH）
  - 需要 JSON 格式日志用于工具处理
title: "logs"
---

# `openclaw logs`

通过 RPC 查看网关文件日志（支持远程模式）。

## 为什么需要这个命令

日志是调试的生命线：

- **远程查看**：无需 SSH 即可查看远程网关日志
- **实时跟踪**：`--follow` 模式实时显示新日志
- **工具友好**：`--json` 输出便于脚本处理

## 相关文档

- 日志概述：[日志](/zh-cn/logging)

## 示例

```bash
# 查看最近日志
openclaw logs

# 实时跟踪（类似 tail -f）
openclaw logs --follow

# JSON 格式输出（便于处理）
openclaw logs --json

# 查看更多历史记录
openclaw logs --limit 500
```

## 选项

| 选项 | 说明 |
|------|------|
| `--follow` / `-f` | 实时跟踪新日志 |
| `--json` | JSON 格式输出 |
| `--limit <n>` | 显示最近 n 行（默认 100） |

## 使用场景

### 调试问题

```bash
# 实时查看日志定位问题
openclaw logs -f
```

### 导出分析

```bash
# 导出 JSON 日志便于分析
openclaw logs --json --limit 1000 > debug.json
```

### 远程诊断

```bash
# 远程查看网关日志（需已配置远程连接）
openclaw logs --follow
```

## 与其他命令配合

```bash
# 查看状态后跟踪日志
openclaw status && openclaw logs -f

# 重启后查看启动日志
openclaw gateway restart && openclaw logs -f
```
