---
summary: "移动节点 - iOS、Android 设备作为 AI 助手节点"
read_when:
  - 使用移动设备作为节点
  - 配置 iOS/Android
  - 理解节点系统
title: "移动节点"
---

# 📱 移动节点

将 iOS 和 Android 设备作为 OpenClaw 的**移动节点**，提供额外的硬件能力。

---

## 🎯 什么是节点？

节点是**特殊客户端**，提供：

- 📷 **相机** - 拍照、图像识别
- 🎤 **音频** - 语音输入
- 📍 **位置** - GPS 信息
- 🔊 **语音** - 语音输出

---

## 🚀 配置节点

### 1. 安装节点应用

- **iOS**: 使用 OpenClaw iOS 应用
- **Android**: 使用 OpenClaw Android 应用

### 2. 配对节点

```bash
openclaw pairing list
openclaw pairing approve <device-id>
```

### 3. 使用节点功能

```
用户：拍张照片
代理：[使用相机节点] 已拍照...
```

---

## 🔧 节点功能

| 功能 | iOS | Android |
|------|-----|---------|
| 相机 | ✅ | ✅ |
| 麦克风 | ✅ | ✅ |
| 位置 | ✅ | ✅ |
| 语音合成 | ✅ | ✅ |

---

## 📖 相关文档

- [部署指南](/zh-CN/operations/deployment)
- [配置参考](/zh-CN/config/reference)
