---
summary: "CLI 参考 - `openclaw health` (通过 RPC 获取网关健康状态)"
read_when:
  - 快速检查运行中网关的健康状态
title: "health"
---

# `openclaw health`

从运行中的网关获取健康状态。

## 为什么需要这个命令

快速了解系统是否正常运行：

- **一目了然**：快速查看网关和所有渠道状态
- **详细信息**：`--verbose` 显示每个账号的响应时间
- **脚本友好**：`--json` 输出便于监控集成

## 示例

```bash
# 基础健康检查
openclaw health

# JSON 格式输出
openclaw health --json

# 详细模式（包括探测时间）
openclaw health --verbose
```

## 选项

| 选项 | 说明 |
|------|------|
| `--json` | JSON 格式输出 |
| `--verbose` | 详细模式，运行实时探测并显示每个账号的响应时间 |

## 输出说明

- **基础模式**：显示网关状态和渠道连接概览
- **详细模式**：
  - 运行实时探测
  - 显示每个账号的响应时间（多账号配置时）
  - 包含每个代理的会话存储信息（多代理配置时）

## 使用场景

### 快速检查

```bash
# 每日健康检查
openclaw health
```

### 监控集成

```bash
# 用于监控脚本
openclaw health --json | jq '.status'
```

### 性能诊断

```bash
# 查看详细响应时间
openclaw health --verbose
```

## 与 `status` 的区别

| 命令 | 用途 |
|------|------|
| `openclaw health` | 快速健康检查，专注于"是否正常" |
| `openclaw status` | 完整状态报告，包括版本、配置、会话等 |
| `openclaw status --deep` | 深度探测，包括渠道连接测试 |

推荐工作流：

```bash
# 快速检查
openclaw health

# 发现问题后查看详细状态
openclaw status --all

# 需要更多信息时深度探测
openclaw status --deep
```
