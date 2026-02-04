---
summary: "图片与媒体支持 - 发送、网关和代理回复的媒体处理规则"
read_when:
  - 修改媒体管道或附件
  - 配置媒体处理流程
  - 了解图片/音频/视频处理
title: "图片与媒体支持"
---

# 🖼️ 图片与媒体支持

本文档详细介绍 OpenClaw 的媒体处理规则，包括发送、网关处理和代理回复。

---

## 🎯 设计目标

| 目标 | 说明 |
|------|------|
| **媒体发送** | 通过 `openclaw message send --media` 发送带可选标题的媒体 |
| **自动回复** | 支持在网页收件箱回复中包含媒体和文本 |
| **合理限制** | 每种类型有可预测的大小限制 |

---

## 💻 CLI 命令

### 发送媒体

```bash
# 发送图片
openclaw message send --media /path/to/image.jpg --message "这是一张图片"

# 发送音频
openclaw message send --media /path/to/audio.mp3

# 发送视频
openclaw message send --media /path/to/video.mp4 --message "看看这个视频"

# 发送文档
openclaw message send --media /path/to/document.pdf --message "附件文档"

# 仅发送媒体（无标题）
openclaw message send --media /path/to/image.jpg
```

### 常用参数

| 参数 | 说明 |
|------|------|
| `--media` | 媒体文件路径或 URL |
| `--message` | 标题文字 |
| `--dry-run` | 打印解析的 payload |
| `--json` | 输出 JSON 格式 `{ channel, to, messageId, mediaUrl, caption }` |
| `--gif-playback` | 发送 GIF 循环播放（MP4） |

---

## 📱 WhatsApp Web 行为

### 输入处理

支持**本地文件路径**或 **HTTP(S) URL**。

### 处理流程

```
本地文件/URL
    │
    ▼
┌──────────────┐
│  加载到 Buffer│
└──────────────┘
    │
    ▼
┌──────────────┐
│  媒体类型检测 │ ←─ 优先 magic bytes，其次 headers，最后文件扩展名
└──────────────┘
    │
    ├─→ 图片 → 调整大小 & 重新压缩 → JPEG（最大边 2048px）
    │
    ├─→ 音频/语音/视频 → 直传（最大 16MB）
    │
    └─→ 文档 → 其他类型（最大 100MB）
```

### 媒体类型处理

| 类型 | 处理方式 | 大小限制 |
|------|----------|----------|
| **图片** | 调整大小、重新压缩为 JPEG | ~6MB |
| **音频/语音** | 直传，语音发送为语音便签 (`ptt: true`) | 16MB |
| **视频** | 直传 | 16MB |
| **文档** | 直传，保留文件名 | 100MB |

### GIF 播放

发送 MP4 时添加 `gifPlayback: true`（CLI：`--gif-playback`），移动客户端会循环播放。

---

## 🔄 自动回复管道

### 配置返回格式

```json5
{
  text?: "回复文本",
  mediaUrl?: "媒体URL",
  mediaUrls?: ["媒体URL1", "媒体URL2"]
}
```

### 处理流程

1. `getReplyFromConfig` 返回上述格式
2. 存在媒体时，使用与 `openclaw message send` 相同的管道解析本地路径或 URL
3. 多个媒体条目按顺序发送

---

## 📥 入站媒体处理（Pi）

### 临时文件变量

入站网页消息包含媒体时，OpenClaw 会下载到临时文件并暴露模板变量：

| 变量 | 说明 |
|------|------|
| `{{MediaUrl}}` | 入站媒体的伪 URL |
| `{{MediaPath}}` | 运行命令前写入的本地临时路径 |

### 沙箱中的媒体

启用按会话 Docker 沙箱时：
1. 入站媒体复制到沙箱工作区
2. `MediaPath`/`MediaUrl` 重写为相对路径如 `media/inbound/<filename>`

### 媒体理解

如果配置了 `tools.media.*`，媒体理解在模板化之前运行：

| 媒体类型 | 插入内容 |
|----------|----------|
| 图片 | `[Image]` 块 |
| 音频 | `[Audio]` 块，设置 `{{Transcript}}` |
| 视频 | `[Video]` 块 |

### 特点

- 默认只处理第一个匹配的附件
- 设置 `tools.media.<cap>.attachments` 可处理多个附件
- 音频转录后用于命令解析，确保斜杠命令可用

---

## ⚠️ 限制与错误

### 出站发送限制（WhatsApp）

| 类型 | 限制 |
|------|------|
| 图片 | ~6MB（重新压缩后） |
| 音频/语音/视频 | 16MB |
| 文档 | 100MB |

### 媒体理解限制

| 类型 | 默认限制 | 配置项 |
|------|----------|--------|
| 图片 | 10MB | `tools.media.image.maxBytes` |
| 音频 | 20MB | `tools.media.audio.maxBytes` |
| 视频 | 50MB | `tools.media.video.maxBytes` |

### 错误处理

超大或不可读的媒体会在日志中显示清晰错误，并跳过回复。

---

## 📊 配置示例

### 基础配置

```json5
{
  "agents": {
    "defaults": {
      "mediaMaxMb": 5
    }
  }
}
```

### 媒体理解配置

```json5
{
  "tools": {
    "media": {
      "image": {
        "enabled": true,
        "maxBytes": 10485760,
        "models": [
          { "provider": "anthropic", "model": "claude-sonnet-4-20250514" }
        ]
      },
      "audio": {
        "enabled": true,
        "maxBytes": 20971520
      },
      "video": {
        "enabled": true,
        "maxBytes": 52428800
      }
    }
  }
}
```

### 多附件处理

```json5
{
  "tools": {
    "media": {
      "image": {
        "attachments": {
          "mode": "all",
          "maxAttachments": 5
        }
      }
    }
  }
}
```

---

## 🐛 故障排除

### 媒体发送失败

```bash
# 检查文件是否存在
ls -la /path/to/media.jpg

# 检查文件大小
du -h /path/to/media.jpg

# 验证 MIME 类型
file --mime-type /path/to/media.jpg

# 查看详细日志
openclaw logs --verbose | grep media
```

### 图片过大

```bash
# 压缩图片
convert input.jpg -resize 2048x2048\> output.jpg

# 或调整质量
convert input.jpg -quality 80 output.jpg
```

### 视频无法发送

```bash
# 检查视频格式
ffprobe -v quiet -print_format json -show_streams video.mp4

# 转换格式
ffmpeg -i input.mov -vcodec h264 -acodec aac output.mp4
```

---

## 📚 相关文档

- [音频处理](/zh-CN/nodes/audio) - 音频转录
- [相机捕获](/zh-CN/nodes/camera) - 拍照和录像
- [媒体理解](/zh-CN/nodes/media-understanding) - AI 媒体分析
- [CLI 参考](/zh-CN/cli) - 命令行工具

---

**完善的媒体支持让 AI 助手能够处理各种类型的文件！** 🦞
