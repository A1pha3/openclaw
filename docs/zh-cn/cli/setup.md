---
summary: "`openclaw setup` 命令参考（初始化配置 + 工作区）"
read_when:
  - 首次运行设置但不使用完整的引导向导
  - 想要设置默认工作区路径
title: "setup"
---

# `openclaw setup`

初始化 `~/.openclaw/openclaw.json` 和代理工作区。

## 为什么需要这个命令

`setup` 是快速初始化 OpenClaw 的方式：

- **轻量初始化**：创建必要的配置文件和目录
- **自定义工作区**：指定工作区位置
- **可选向导**：需要时可以启动完整向导

## 相关链接

- 入门指南：[Getting started](/zh-cn/start/getting-started)
- 引导向导：[Onboarding](/zh-cn/start/onboarding)

## 基本用法

```bash
# 使用默认设置初始化
openclaw setup

# 指定工作区路径
openclaw setup --workspace ~/.openclaw/workspace

# 启动完整向导
openclaw setup --wizard
```

## 创建的内容

运行 `openclaw setup` 后会创建：

```
~/.openclaw/
├── openclaw.json      # 主配置文件
├── credentials/       # 渠道凭据
├── sessions/          # 会话数据
└── workspace/         # 代理工作区
    ├── AGENTS.md      # 代理指令
    ├── SOUL.md        # 人格定义
    └── skills/        # 技能目录
```

## 选项

| 选项 | 说明 |
|------|------|
| `--workspace <path>` | 指定工作区路径 |
| `--wizard` | 启动完整引导向导 |
| `--force` | 覆盖现有配置 |

## setup vs onboard

| 命令 | 适用场景 |
|------|----------|
| `openclaw setup` | 快速初始化，熟悉 OpenClaw 的用户 |
| `openclaw onboard` | 完整引导，新用户推荐 |

## 使用场景

### 快速开始

```bash
# 初始化并立即启动网关
openclaw setup
openclaw gateway run
```

### 自定义工作区

```bash
# 使用项目特定的工作区
openclaw setup --workspace ~/projects/my-assistant/workspace
```

### 多代理设置

```bash
# 为不同代理创建独立工作区
openclaw setup --workspace ~/.openclaw/workspace-work
openclaw setup --workspace ~/.openclaw/workspace-personal
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 配置已存在 | 之前运行过 setup | 使用 `--force` 覆盖 |
| 权限错误 | 目录权限问题 | 检查目录权限 |
| 工作区创建失败 | 路径无效 | 确保父目录存在 |
