---
summary: "节点系统 - 配对、能力、权限和 CLI 命令详解"
read_when:
  - 配对 iOS/Android 节点到网关
  - 使用节点相机/画布功能
  - 添加新的节点命令
title: "节点系统"
---

# 📱 节点系统

本文档详细介绍 OpenClaw 的节点系统，包括配对流程、功能特性和 CLI 命令。

## 🎯 什么是节点？

**节点（Node）** 是连接到网关的配套设备，提供额外的硬件能力：

```
网关 ←—— WebSocket ——→ 节点设备
          │
          ├── 相机控制
          ├── 屏幕截图
          ├── 位置获取
          └── 系统命令
```

### 节点类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **iOS 节点** | iPhone/iPad | 相机、麦克风、位置 |
| **Android 节点** | Android 手机 | 相机、麦克风、位置、SMS |
| **macOS 节点** | Mac 电脑 | 画布、相机、系统命令 |
| **无头节点** | Linux/Windows 服务器 | 系统命令执行 |

### 节点与网关的区别

| 特性 | 网关 | 节点 |
|------|------|------|
| 角色 | 服务端 | 客户端 |
| 运行服务 | ✅ | ❌ |
| 消息处理 | ✅ | ❌ |
| 硬件访问 | ❌ | ✅ |
| WebSocket 连接 | 监听 | 建立 |

---

## 🔗 配对与状态

### 快速配对

```bash
# 查看待配对设备
openclaw devices list

# 审批设备
openclaw devices approve <requestId>

# 拒绝设备
openclaw devices reject <requestId>

# 查看节点状态
openclaw nodes status

# 查看节点详情
openclaw nodes describe --node <idOrNameOrIp>
```

### 配对流程

```
1. 节点设备安装 OpenClaw 应用
2. 打开应用，选择"作为节点"
3. 输入网关地址和端口
4. 网关端收到配对请求
5. 管理员审批配对请求
6. 配对成功，节点上线
```

### 节点状态

| 状态 | 说明 |
|------|------|
| **pending** | 待审批 |
| **paired** | 已配对 |
| **online** | 在线 |
| **offline** | 离线 |

---

## 🎨 画布功能（Canvas）

### 屏幕截图

如果节点正在显示画布（WebView），可以获取截图：

```bash
# 获取 PNG 截图
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png

# 获取 JPG 截图（压缩）
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### 画布控制

```bash
# 显示网页
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com

# 隐藏画布
openclaw nodes canvas hide --node <idOrNameOrIp>

# 导航到 URL
openclaw nodes canvas navigate --node <idOrNameOrIp> https://example.com

# 执行 JavaScript
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

### A2UI 画布

```bash
# 推送文本
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"

# 推送 JSONL 数据
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl

# 重置画布
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

---

## 📷 相机功能

### 拍照

```bash
# 列出可用相机
openclaw nodes camera list --node <idOrNameOrIp>

# 拍照（默认使用前后摄像头）
openclaw nodes camera snap --node <idOrNameOrIp>

# 指定前置摄像头
openclaw nodes camera snap --node <idOrNameOrIp> --facing front

# 指定后置摄像头
openclaw nodes camera snap --node <idOrNameOrIp> --facing back
```

### 录像

```bash
# 录制 10 秒视频
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s

# 录制 3 秒视频（毫秒）
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000

# 禁用音频录制
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s --no-audio
```

> **注意**：节点应用必须在前台运行，`canvas.*` 和 `camera.*` 调用才有效。

---

## 🖥️ 屏幕录制

```bash
# 录制屏幕（10秒，10fps）
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10

# 禁用音频
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio

# 指定屏幕（多显示器时）
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --screen 0
```

> **注意**：屏幕录制最长 60 秒，Android 会显示系统录屏提示。

---

## 📍 位置功能

```bash
# 获取位置
openclaw nodes location get --node <idOrNameOrIp>

# 精确位置
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise

# 自定义参数
openclaw nodes location get --node <idOrNameOrIp> \
  --max-age 15000 \
  --location-timeout 10000
```

> **注意**：位置功能默认关闭，需要在设置中启用。

---

## 📱 SMS 功能（Android）

Android 节点支持发送短信：

```bash
# 发送 SMS（需要 SMS 权限）
openclaw nodes invoke --node <idOrNameOrIp> \
  --command sms.send \
  --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

---

## 🖥️ 系统命令（macOS/无头节点）

### macOS 节点

```bash
# 执行命令
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"

# 发送通知
openclaw nodes notify --node <idOrNameOrIp> \
  --title "Ping" \
  --body "Gateway ready"
```

### 无头节点主机

```bash
# 启动无头节点
openclaw node run --host <gateway-host> --port 18789

# 作为服务安装
openclaw node install --host <gateway-host> --port 18789
openclaw node restart
```

---

## 🔐 执行审批

### 添加白名单

```bash
# 添加命令到白名单
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

### 查看审批状态

```bash
# 查看节点审批配置
openclaw approvals status --node <idOrNameOrIp>
```

---

## ⚙️ 节点配置

### 绑定执行节点

当有多个节点时，可以绑定执行到特定节点：

```bash
# 全局默认
openclaw config set tools.exec.node "node-id-or-name"

# 按代理配置
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"

# 允许任何节点
openclaw config unset tools.exec.node
```

### 重命名节点

```bash
openclaw nodes rename --node <id|name|ip> --name "Build Node"
```

---

## 📊 权限状态

查看节点权限状态：

```bash
openclaw nodes permissions --node <idOrNameOrIp>
```

| 权限 | 说明 |
|------|------|
| `screenRecording` | 屏幕录制 |
| `camera` | 相机 |
| `microphone` | 麦克风 |
| `location` | 位置 |
| `accessibility` | 辅助功能 |

---

## 🐛 故障排除

### 节点无法连接

```bash
# 检查节点状态
openclaw nodes status

# 查看节点详情
openclaw nodes describe --node <idOrNameOrIp>

# 测试 WebSocket 连接
curl ws://<gateway-host>:18789
```

### 权限被拒绝

```bash
# 检查权限状态
openclaw nodes permissions --node <idOrNameOrIp>

# 重启节点应用
openclaw nodes restart --node <idOrNameOrIp>
```

### 相机无法使用

```bash
# 检查相机列表
openclaw nodes camera list --node <idOrNameOrIp>

# 确保节点应用在前台
```

---

## 📝 最佳实践

### 移动节点使用

```json5
{
  "nodes": {
    "autoConnect": true,
    "permissions": {
      "camera": true,
      "microphone": true,
      "location": "always"
    }
  }
}
```

### 服务器节点配置

```json5
{
  "tools": {
    "exec": {
      "host": "node",
      "security": "allowlist",
      "node": "build-server"
    }
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw nodes list` | 列出节点 |
| `openclaw nodes status` | 节点状态 |
| `openclaw nodes describe` | 节点详情 |
| `openclaw nodes approve` | 审批节点 |
| `openclaw nodes canvas` | 画布控制 |
| `openclaw nodes camera` | 相机控制 |
| `openclaw nodes screen` | 屏幕录制 |
| `openclaw nodes location` | 位置获取 |
| `openclaw node run` | 启动节点 |
| `openclaw devices` | 设备管理 |

---

## 📚 相关文档

- [移动节点概述](/zh-CN/nodes) - 节点简介
- [相机节点](/zh-CN/nodes/camera) - 相机详细使用
- [音频节点](/zh-CN/nodes/audio) - 音频功能
- [位置命令](/zh-CN/nodes/location-command) - 位置服务
- [语音对话](/zh-CN/nodes/talk) - 语音交互
- [语音唤醒](/zh-CN/nodes/voicewake) - 免提唤醒
- [配置参考](/zh-CN/config/reference) - 完整配置

---

**节点系统让 AI 助手拥有移动设备的强大能力！** 🦞
