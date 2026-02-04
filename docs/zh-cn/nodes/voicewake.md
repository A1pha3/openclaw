---
summary: "全局语音唤醒词 - Gateway 拥有的唤醒词及其跨节点同步机制"
read_when:
  - 修改语音唤醒词行为或默认值
  - 添加需要唤醒词同步的新节点平台
title: "语音唤醒"
---

# 🎤 语音唤醒（全局唤醒词）

本文档详细介绍 OpenClaw 的全局语音唤醒词系统，包括存储、同步和配置。

---

## 🎯 核心概念

**唤醒词**是全局列表，由 **Gateway** 统一管理：

| 特性 | 说明 |
|------|------|
| **全局列表** | 所有设备共享同一个唤醒词列表 |
| **Gateway 拥有** | 唤醒词存储在 Gateway 主机上 |
| **任何客户端可编辑** | 更改持久化并广播到所有设备 |
| **本地开关** | 每个设备保留自己的启用/禁用开关 |

### 为什么全局管理？

```
┌─────────────────────────────────────────┐
│           Gateway（中央管理）            │
│                                         │
│  ~/.openclaw/settings/voicewake.json    │
│  {                                      │
│    "triggers": ["openclaw", "claude"],  │
│    "updatedAtMs": 1730000000000         │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
     ┌───────────┼───────────┐
     │           │           │
     ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ macOS   │ │  iOS    │ │Android  │
│  App    │ │  Node   │ │  Node   │
└─────────┘ └─────────┘ └─────────┘
```

---

## 💾 存储位置

### Gateway 主机

唤醒词存储在网关机器上：

```
~/.openclaw/settings/voicewake.json
```

### 数据格式

```json5
{
  "triggers": ["openclaw", "claude", "computer"],
  "updatedAtMs": 1730000000000
}
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `triggers` | 唤醒词列表 |
| `updatedAtMs` | 最后更新时间（毫秒） |

---

## 📡 协议

### RPC 方法

| 方法 | 输入 | 输出 |
|------|------|------|
| `voicewake.get` | 无 | `{ triggers: string[] }` |
| `voicewake.set` | `{ triggers: string[] }` | `{ triggers: string[] }` |

### 事件

| 事件 | Payload | 接收者 |
|------|---------|--------|
| `voicewake.changed` | `{ triggers: string[] }` | 所有 WebSocket 客户端和节点 |

### 规则

- **规范化**：去除首尾空白，空值丢弃
- **空列表**：回退到默认唤醒词
- **安全限制**：强制执行数量和长度限制

---

## 📱 客户端行为

### macOS 应用

- 使用全局列表控制 `VoiceWakeRuntime` 触发
- 在语音唤醒设置中编辑"触发词"调用 `voicewake.set`，然后依赖广播保持其他客户端同步

### iOS 节点

- 使用全局列表进行 `VoiceWakeManager` 触发检测
- 在设置中编辑唤醒词调用 `voicewake.set`（通过 Gateway WS），同时保持本地唤醒词检测响应

### Android 节点

- 在设置中暴露唤醒词编辑器
- 调用 `voicewake.set`（通过 Gateway WS），使编辑同步到所有设备

---

## ⚙️ 使用方式

### 查看唤醒词

```bash
# 通过 CLI
openclaw voicewake get

# 返回示例
{
  "triggers": ["openclaw", "claude", "computer"],
  "updatedAtMs": 1730000000000
}
```

### 设置唤醒词

```bash
# 设置唤醒词
openclaw voicewake set --triggers "openclaw,hey claude,computer"

# 仅启用
openclaw voicewake enable

# 禁用
openclaw voicewake disable
```

### 节点唤醒词管理

```bash
# iOS 节点
openclaw nodes voicewake --node <id>

# Android 节点
openclaw nodes voicewake --node <id>
```

---

## 🎛️ 配置示例

### Gateway 配置

```json5
{
  "voicewake": {
    "enabled": true,
    "triggers": ["openclaw", "claude", "computer"],
    "minLength": 3,
    "maxLength": 20,
    "maxTriggers": 10
  }
}
```

### 默认唤醒词

| 唤醒词 | 说明 |
|--------|------|
| `openclaw` | OpenClaw 默认 |
| `claude` | Claude 引用 |
| `computer` | 经典科幻风格 |

### 约束

| 约束 | 最小值 | 最大值 |
|------|--------|--------|
| 单个唤醒词长度 | 3 字符 | 20 字符 |
| 唤醒词数量 | 1 | 10 |

---

## 🔄 同步流程

```
1. 用户在设备 A 编辑唤醒词
          │
          ▼
2. 设备 A 调用 voicewake.set
          │
          ▼
3. Gateway 更新存储
          │
          ▼
4. Gateway 广播 voicewake.changed
          │
          ▼
5. 所有连接的客户端和节点收到更新
          │
          ├──→ macOS app 更新 VoiceWakeRuntime
          ├──→ iOS 节点更新 VoiceWakeManager
          └──→ Android 节点更新唤醒词编辑器
```

---

## 🔐 权限与安全

### 本地控制

每个设备保留独立的启用/禁用开关：

| 设备 | 控制项 |
|------|--------|
| macOS | 系统偏好设置 → 隐私 → 麦克风 |
| iOS | 设置 → OpenClaw → 麦克风 |
| Android | 设置 → 应用 → OpenClaw → 权限 |

### 安全考虑

| 措施 | 说明 |
|------|------|
| **长度限制** | 防止过长唤醒词 |
| **数量限制** | 防止过多唤醒词 |
| **规范化** | 防止特殊字符 |
| **广播加密** | WebSocket 使用 TLS |

---

## 🐛 故障排除

### 唤醒词不工作

```bash
# 检查全局设置
openclaw voicewake get

# 检查设备状态
openclaw nodes status

# 验证同步
openclaw logs --lines 50 | grep voicewake
```

### 同步问题

```bash
# 手动触发同步
openclaw voicewake set --triggers "openclaw"

# 检查 Gateway 连接
openclaw status

# 查看同步事件
openclaw logs --verbose | grep "voicewake.changed"
```

### 麦克风权限

```bash
# macOS - 检查权限
tccutil prompt Microphone com.openclaw.openclaw

# iOS - 重置权限
设置 → 通用 → iPhone 存储 → OpenClaw → 删除 App
重新安装

# Android - 检查权限
设置 → 应用 → OpenClaw → 权限
```

---

## 📊 最佳实践

### 个人使用

```json5
{
  "voicewake": {
    "enabled": true,
    "triggers": ["openclaw"]
  }
}
```

### 团队使用

```json5
{
  "voicewake": {
    "enabled": true,
    "triggers": ["openclaw", "hey assistant"]
  }
}
```

### 多设备环境

```json5
{
  "voicewake": {
    "enabled": true,
    "triggers": ["openclaw", "computer"],
    "deviceSpecific": false  // 全局同步
  }
}
```

---

## 📚 相关文档

- [节点系统](/zh-CN/nodes) - 节点概述
- [语音对话模式](/zh-CN/nodes/talk) - Talk 模式
- [音频处理](/zh-CN/nodes/audio) - 音频转录
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

**语音唤醒让免提与 AI 对话成为可能！** 🦞
