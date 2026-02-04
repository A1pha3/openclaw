---
summary: "CLI 参考 - `openclaw skills` (列出/查看/检查技能及其资格)"
read_when:
  - 查看哪些技能可用并准备就绪
  - 调试技能缺失的二进制文件/环境变量/配置
title: "skills"
---

# `openclaw skills`

检查技能（内置 + 工作区 + 托管覆盖）并查看哪些符合运行条件、哪些缺少依赖。

## 为什么需要这个命令

技能是 OpenClaw 的能力扩展：

- **查看可用技能**：了解代理有哪些能力
- **检查资格**：哪些技能满足运行条件
- **调试问题**：找出缺失的依赖项

## 相关文档

- 技能系统：[技能](/zh-cn/tools/skills)
- 技能配置：[技能配置](/zh-cn/tools/skills-config)
- ClawHub 安装：[ClawHub](/zh-cn/tools/clawhub)

## 命令

```bash
# 列出所有技能
openclaw skills list

# 只显示符合条件的技能
openclaw skills list --eligible

# 查看特定技能详情
openclaw skills info <name>

# 检查所有技能的依赖状态
openclaw skills check
```

## 子命令详解

### `list` - 列出技能

```bash
# 所有技能
openclaw skills list

# 仅符合条件的技能
openclaw skills list --eligible

# JSON 格式
openclaw skills list --json
```

### `info` - 技能详情

```bash
# 查看特定技能的完整信息
openclaw skills info web
openclaw skills info github
```

输出包括：

- 技能名称和描述
- 来源（内置/工作区/托管）
- 依赖要求
- 当前状态（是否可用）

### `check` - 检查依赖

```bash
# 检查所有技能的依赖状态
openclaw skills check
```

显示每个技能缺失的：

- 二进制文件
- 环境变量
- 配置项

## 技能类型

| 类型 | 位置 | 说明 |
|------|------|------|
| **内置** | OpenClaw 包内 | 随 OpenClaw 一起安装 |
| **工作区** | `~/.openclaw/workspace/skills/` | 用户自定义技能 |
| **托管** | ClawHub | 通过 ClawHub 安装的技能 |

## 使用场景

### 查看代理能力

```bash
# 查看当前可用的所有能力
openclaw skills list --eligible
```

### 调试技能问题

```bash
# 某技能不工作？检查依赖
openclaw skills check

# 查看特定技能详情
openclaw skills info <problematic-skill>
```

### 安装新技能

```bash
# 从 ClawHub 安装技能
openclaw skills install web

# 验证安装
openclaw skills info web
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 技能显示为不可用 | 运行 `openclaw skills check` 查看缺失依赖 |
| 缺少二进制文件 | 安装所需的命令行工具 |
| 缺少环境变量 | 在 `~/.profile` 或 shell 配置中设置 |
| 缺少配置项 | 使用 `openclaw config set` 添加所需配置 |
