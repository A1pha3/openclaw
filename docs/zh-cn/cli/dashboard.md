---
summary: "`openclaw dashboard` 命令参考（打开控制面板 UI）"
read_when:
  - 想要使用当前认证打开控制面板 UI
  - 想要打印 URL 而不启动浏览器
title: "dashboard"
---

# `openclaw dashboard`

使用当前认证打开控制面板 UI。

## 为什么需要这个命令

控制面板提供了一个可视化的 Web 界面来管理 OpenClaw：

- **图形化管理**：比命令行更直观的操作体验
- **实时监控**：查看会话、渠道、节点状态
- **配置调整**：通过 UI 修改设置
- **远程访问**：通过浏览器访问远程网关

## 基本用法

```bash
# 打开控制面板（自动启动浏览器）
openclaw dashboard

# 只打印 URL，不启动浏览器
openclaw dashboard --no-open
```

## 选项

| 选项 | 说明 |
|------|------|
| `--no-open` | 打印 URL 而不启动浏览器 |
| `--url <url>` | 指定网关 URL |
| `--token <token>` | 指定认证 token |

## 使用场景

### 本地访问

```bash
# 默认连接本地网关
openclaw dashboard
```

会在浏览器中打开类似 `http://127.0.0.1:18789/` 的地址。

### 远程访问

```bash
# 连接远程网关
openclaw dashboard --url https://gateway-host:18789 --token <your-token>
```

### 脚本中获取 URL

```bash
# 获取 URL 用于脚本或分享
URL=$(openclaw dashboard --no-open)
echo "Dashboard URL: $URL"
```

## 控制面板功能

打开控制面板后，你可以：

- **会话管理**：查看活动会话、历史记录
- **渠道状态**：监控各渠道连接状态
- **节点管理**：查看已连接的节点设备
- **配置编辑**：调整网关设置
- **日志查看**：实时查看系统日志
- **WebChat**：直接与代理对话

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 无法打开浏览器 | 无图形环境 | 使用 `--no-open` 获取 URL |
| 认证失败 | Token 无效 | 检查 token 或重新登录 |
| 连接失败 | 网关未运行 | 运行 `openclaw gateway run` |

## 相关链接

- 控制面板 UI：[Control UI](/zh-cn/web/control-ui)
- WebChat：[WebChat](/zh-cn/web/webchat)
