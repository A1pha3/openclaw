---
summary: "相机捕获 - iOS/Android/macOS 节点拍照和录像功能"
read_when:
  - 添加或修改 iOS/Android/macOS 节点相机捕获
  - 扩展代理可访问的媒体临时文件工作流
title: "相机捕获"
---

# 📷 相机捕获

本文档详细介绍 OpenClaw 在各平台上支持相机捕获功能，包括拍照和录像。

---

## 🎯 支持的平台与功能

| 平台 | 拍照 | 录像 | 音频 | 权限控制 |
|------|------|------|------|----------|
| **iOS 节点** | ✅ | ✅ | ✅ | 用户设置 |
| **Android 节点** | ✅ | ✅ | ✅ | 运行时权限 |
| **macOS 应用** | ✅ | ✅ | ✅ | 系统偏好设置 |

---

## 🍎 iOS 节点

### 用户设置（默认开启）

在 iOS 设置中：
- **设置 → 相机 → 允许相机** (`camera.enabled`)
- 默认：**开启**（缺少配置视为启用）
- 关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`

### 命令（通过 Gateway `node.invoke`）

#### 列出相机设备

```bash
# 返回设备列表
{
  devices: [
    { id: "front", name: "前置摄像头", position: "front", deviceType: "camera" },
    { id: "back", name: "后置摄像头", position: "back", deviceType: "camera" }
  ]
}
```

#### 拍照

```bash
# 默认（前后摄像头各拍一张）
openclaw nodes camera snap --node <id>

# 指定前置摄像头
openclaw nodes camera snap --node <id> --facing front

# 指定后置摄像头
openclaw nodes camera snap --node <id> --facing back

# 自定义参数
openclaw nodes camera snap --node <id> \
  --max-width 1600 \
  --quality 0.9 \
  --delay-ms 1000
```

**参数说明：**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `facing` | front | 摄像头方向 |
| `maxWidth` | 1600 | 最大宽度 |
| `quality` | 0.9 | JPEG 质量（0-1） |
| `format` | jpg | 格式（仅支持 jpg） |
| `delayMs` | 0 | 延迟拍摄（毫秒） |
| `deviceId` | - | 设备 ID（来自 list） |

**返回值：**
```json
{
  "format": "jpg",
  "base64": "<base64编码的图片>",
  "width": 1600,
  "height": 1200
}
```

> **注意**：照片会被重新压缩，保持 base64 payload 在 5MB 以下。

#### 录像

```bash
# 默认（3秒，前置摄像头，有音频）
openclaw nodes camera clip --node <id> --duration 3000

# 后置摄像头
openclaw nodes camera clip --node <id> --facing back --duration 5000

# 无音频
openclaw nodes camera clip --node <id> --no-audio --duration 10s
```

**参数说明：**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `facing` | front | 摄像头方向 |
| `durationMs` | 3000 | 时长（毫秒，最大 60000） |
| `includeAudio` | true | 是否包含音频 |
| `format` | mp4 | 格式（仅支持 mp4） |
| `deviceId` | - | 设备 ID |

**返回值：**
```json
{
  "format": "mp4",
  "base64": "<base64编码的视频>",
  "durationMs": 3000,
  "hasAudio": true
}
```

### 前置要求

与 `canvas.*` 类似，iOS 节点只允许在前台执行 `camera.*` 命令。后台调用返回 `NODE_BACKGROUND_UNAVAILABLE`。

---

## 🤖 Android 节点

### 用户设置（默认开启）

在 Android 设置中：
- **设置 → 相机 → 允许相机** (`camera.enabled`)
- 默认：**开启**

### 权限要求

Android 需要运行时权限：

| 权限 | 用途 |
|------|------|
| `CAMERA` | 拍照和录像 |
| `RECORD_AUDIO` | 录像时录制音频 |

如果权限缺失，应用会在可能时提示用户；拒绝后 `camera.*` 请求会失败并返回 `*_PERMISSION_REQUIRED` 错误。

### 使用方式

与 iOS 节点相同，使用 `openclaw nodes camera` 命令。

---

## 🖥️ macOS 应用

### 用户设置（默认关闭）

macOS 配套应用提供复选框：
- **设置 → 通用 → 允许相机** (`openclaw.cameraEnabled`)
- 默认：**关闭**
- 关闭时：相机请求返回"相机已被用户禁用"

### 使用示例

```bash
# 列出相机
openclaw nodes camera list --node <id>

# 拍照
openclaw nodes camera snap --node <id>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000

# 录像
openclaw nodes camera clip --node <id> --duration 10s
openclaw nodes camera clip --node <id> --duration-ms 3000
openclaw nodes camera clip --node <id> --no-audio
```

> **注意**：macOS 上 `camera.snap` 在拍摄前会等待 `delayMs`（默认 2000ms）以完成预热和曝光调整。

---

## 🔒 安全与实际限制

### 系统权限

相机和麦克风访问会触发常规的操作系统权限提示，需要在 Info.plist 中配置使用说明字符串。

### 视频限制

- 视频剪辑最长 **60 秒**
- 防止过大的节点 payload（base64 开销 + 消息限制）

### 隐私考虑

| 平台 | 权限类型 |
|------|----------|
| iOS | 隐私设置开关 |
| Android | 运行时权限 |
| macOS | TCC 权限 |

---

## 📸 macOS 屏幕录制

对于**屏幕**视频（不是相机），使用 macOS 配套应用：

```bash
# 屏幕录制（10秒，15fps）
openclaw nodes screen record --node <id> --duration 10s --fps 15
```

> **注意**：需要 macOS **屏幕录制** 权限（TCC）。

---

## 🐛 故障排除

### 相机不可用

```bash
# 检查相机列表
openclaw nodes camera list --node <id>

# 查看节点状态
openclaw nodes status

# 检查权限设置
openclaw config get
```

### 拍照失败

```bash
# 确保节点应用在前台
# 检查相机权限
# 查看日志
openclaw logs --lines 50 | grep camera
```

### 视频过大

```bash
# 减少时长
openclaw nodes camera clip --node <id> --duration 5s

# 降低帧率
openclaw nodes screen record --node <id> --fps 10
```

---

## 📊 最佳实践

### 移动节点配置

```json5
{
  "nodes": {
    "camera": {
      "enabled": true,
      "defaultFacing": "back",
      "maxWidth": 1600,
      "quality": 0.9,
      "maxDurationMs": 30000
    }
  }
}
```

### 生产环境配置

```json5
{
  "tools": {
    "media": {
      "image": {
        "enabled": true,
        "maxBytes": 5242880  // 5MB
      }
    }
  }
}
```

---

## 📚 相关文档

- [节点系统](/zh-CN/nodes) - 节点概述
- [音频处理](/zh-CN/nodes/audio) - 音频转录
- [屏幕录制](/zh-CN/nodes/screen) - 屏幕视频捕获
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

**相机捕获让 AI 助手能够"看见"世界！** 🦞
