---
summary: "`openclaw tui` 命令参考（连接网关的终端 UI）"
read_when:
  - 想要使用网关的终端 UI（支持远程）
  - 想要从脚本传递 url/token/session
title: "tui"
---

# `openclaw tui`

打开连接到网关的终端 UI。

## 为什么需要 TUI

终端 UI 提供了一种轻量级的交互方式：

- **SSH 友好**：在远程服务器上使用
- **无需图形界面**：纯终端环境下工作
- **脚本集成**：可以传递参数自动化启动
- **会话管理**：直接指定会话和投递选项

## 相关链接

- TUI 指南：[TUI](/zh-cn/tui)

## 基本用法

```bash
# 连接本地网关
openclaw tui

# 连接远程网关
openclaw tui --url ws://127.0.0.1:18789 --token <token>

# 指定会话并启用投递
openclaw tui --session main --deliver
```

## 选项

| 选项 | 说明 |
|------|------|
| `--url <url>` | 网关 WebSocket URL |
| `--token <token>` | 认证 token |
| `--password <password>` | 认证密码 |
| `--session <id>` | 会话 ID 或键 |
| `--deliver` | 启用消息投递到渠道 |
| `--agent <id>` | 指定代理 |

## 使用场景

### 本地开发

```bash
# 简单启动
openclaw tui
```

### 远程访问

```bash
# 通过 SSH 访问远程网关的 TUI
ssh user@server
openclaw tui --url ws://127.0.0.1:18789
```

### 指定会话

```bash
# 使用特定会话
openclaw tui --session agent:main:support

# 创建或附加到命名会话
openclaw tui --session-label "调试会话"
```

### 启用投递

```bash
# 对话回复投递到配置的渠道
openclaw tui --deliver
```

## TUI 功能

在 TUI 中你可以：

- **对话**：直接与代理交互
- **查看历史**：滚动查看会话历史
- **切换会话**：使用快捷键切换
- **发送命令**：使用 `/` 前缀发送命令

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送消息 |
| `Ctrl+C` | 退出 |
| `Ctrl+L` | 清屏 |
| `↑/↓` | 浏览历史 |
| `Page Up/Down` | 滚动输出 |

## 命令

在 TUI 中可以使用以下命令：

| 命令 | 功能 |
|------|------|
| `/new` | 重置会话 |
| `/status` | 查看状态 |
| `/model <name>` | 切换模型 |
| `/think <level>` | 设置思考级别 |
| `/help` | 显示帮助 |

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 连接失败 | 网关未运行 | 启动网关 |
| 认证失败 | Token 无效 | 检查认证配置 |
| 显示异常 | 终端不兼容 | 尝试其他终端 |
| 输入无响应 | 网络延迟 | 检查网络连接 |

## 与其他 UI 对比

| 特性 | TUI | WebChat | Dashboard |
|------|-----|---------|-----------|
| 需要浏览器 | ❌ | ✅ | ✅ |
| SSH 可用 | ✅ | ❌ | ❌ |
| 功能丰富度 | 基础 | 中等 | 完整 |
| 资源占用 | 最低 | 低 | 中等 |
