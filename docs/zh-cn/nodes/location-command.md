---
summary: "位置命令 - 节点位置获取、权限模式和后台行为"
read_when:
  - 添加位置节点支持或权限 UI
  - 设计后台位置 + 推送流程
  - 配置位置获取参数
title: "位置命令"
---

# 📍 位置命令

本文档详细介绍 OpenClaw 节点的位置获取功能，包括权限配置和命令使用。

---

## 🎯 快速概述

| 特性 | 说明 |
|------|------|
| **命令** | `location.get`（通过 `node.invoke`） |
| **默认状态** | 关闭 |
| **权限模式** | 关闭 / 使用时 / 始终 |
| **精确位置** | 独立开关 |

---

## 🤔 为什么需要选择器？

操作系统权限是多级别的，应用可以选择但系统决定实际授予。

| 平台 | 说明 |
|------|------|
| **iOS/macOS** | 用户可以选择"使用时"或"始终" |
| **Android** | 后台位置是独立权限，通常需要设置流程 |
| **精确位置** | iOS 14+ "精确"，Android "精确" vs "粗略" |

应用 UI 中的选择器控制请求的模式，实际授权由系统设置管理。

---

## ⚙️ 设置模型

### 配置项

每个节点设备：

```json5
{
  "location": {
    "enabledMode": "off | whileUsing | always",
    "preciseEnabled": true
  }
}
```

### 权限模式

| 模式 | 说明 | UI 显示 |
|------|------|----------|
| `off` | 禁用 | "位置分享已禁用" |
| `whileUsing` | 仅使用时 | "仅在 OpenClaw 打开时" |
| `always` | 始终（包括后台） | "允许后台位置。需要系统权限。" |

### 精确位置

| 状态 | UI 显示 |
|------|----------|
| 开启 | "使用精确 GPS 位置" |
| 关闭 | "关闭以共享大致位置" |

---

## 📡 命令：location.get

### 调用方式

通过 `node.invoke` 调用：

```json5
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

### CLI 使用

```bash
# 获取位置
openclaw nodes location get --node <id>

# 精确位置
openclaw nodes location get --node <id> --accuracy precise

# 自定义参数
openclaw nodes location get --node <id> \
  --max-age 15000 \
  --location-timeout 10000
```

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `timeoutMs` | 10000 | 超时时间（毫秒） |
| `maxAgeMs` | 15000 | 位置最大年龄（毫秒） |
| `desiredAccuracy` | balanced | 期望精度 |

### 返回值

```json5
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `lat` | 纬度 |
| `lon` | 经度 |
| `accuracyMeters` | 精度（米） |
| `altitudeMeters` | 海拔（米） |
| `speedMps` | 速度（米/秒） |
| `headingDeg` | 方向（度） |
| `timestamp` | 时间戳 |
| `isPrecise` | 是否精确位置 |
| `source` | 位置来源 |

### 错误代码

| 错误代码 | 说明 |
|----------|------|
| `LOCATION_DISABLED` | 选择器已关闭 |
| `LOCATION_PERMISSION_REQUIRED` | 请求的模式缺少权限 |
| `LOCATION_BACKGROUND_UNAVAILABLE` | 应用在后台但仅允许"使用时" |
| `LOCATION_TIMEOUT` | 时间内未获取到位置 |
| `LOCATION_UNAVAILABLE` | 系统故障或无位置提供商 |

---

## 🔐 权限映射

### 节点权限

可选。macOS 节点通过权限映射报告 `location`：

```json5
{
  "permissions": {
    "location": true
  }
}
```

### iOS/Android

可能不包含此权限字段。

---

## 🔄 后台行为（未来）

### 目标

模型可以请求位置，即使节点在后台，但需要：

1. 用户选择了"始终"
2. OS 授予后台位置
3. 应用允许后台运行（iOS 后台模式 / Android 前台服务）

### 推送触发流程（未来）

```
1. 网关向节点发送推送（静默推送或 FCM 数据）
2. 节点短暂唤醒并请求设备位置
3. 节点将 payload 转发给网关
```

### 平台注意事项

| 平台 | 要求 |
|------|------|
| **iOS** | 需要"始终"权限 + 后台位置模式。静默推送可能被限流。 |
| **Android** | 后台位置可能需要前台服务，否则可能被拒绝。 |

---

## 🛠️ 工具集成

### 工具表面

`nodes` 工具添加 `location_get` 操作（需要节点）：

```json5
{
  "action": "location_get",
  "params": {
    "timeoutMs": 10000,
    "maxAgeMs": 15000
  }
}
```

### CLI 命令

```bash
openclaw nodes location get --node <id>
openclaw nodes location get --node <id> --accuracy precise
openclaw nodes location get --node <id> --max-age 15000 --location-timeout 10000
```

### 代理指南

仅在用户启用位置并理解范围时才调用。

---

## 📱 平台支持

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| **iOS** | ✅ | 完整支持 |
| **Android** | ✅ | 完整支持 |
| **macOS** | ✅ | 完整支持 |
| **无头节点** | ⚠️ | 依赖系统位置服务 |

---

## 🐛 故障排除

### 位置获取失败

```bash
# 检查位置设置
openclaw config get location

# 查看节点状态
openclaw nodes status

# 检查权限
openclaw nodes permissions --node <id>

# 查看详细日志
openclaw logs --lines 50 | grep location
```

### 精度问题

```bash
# 请求精确位置
openclaw nodes location get --node <id> --accuracy precise

# 确保精确位置开关已开启
```

### 超时问题

```bash
# 增加超时时间
openclaw nodes location get --node <id> --location-timeout 20000

# 确保设备有良好的 GPS 信号
```

---

## 📊 配置示例

### 节点配置

```json5
{
  "location": {
    "enabledMode": "whileUsing",
    "preciseEnabled": true,
    "timeoutMs": 15000,
    "maxAgeMs": 20000,
    "desiredAccuracy": "precise"
  }
}
```

### 工具配置

```json5
{
  "tools": {
    "nodes": {
      "location": {
        "enabled": true,
        "requirePrecise": false,
        "maxCacheAgeMs": 30000
      }
    }
  }
}
```

---

## 📚 相关文档

- [节点系统](/zh-CN/nodes) - 节点概述
- [相机捕获](/zh-CN/nodes/camera) - 相机功能
- [配置参考](/zh-CN/config/reference) - 完整配置选项
- [隐私与安全](/zh-CN/concepts/security) - 隐私保护

---

**位置功能让 AI 助手能够了解用户的地理位置！** 🦞
