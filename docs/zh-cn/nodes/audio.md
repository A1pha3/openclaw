---
summary: "音频与语音消息处理 - 下载、转录和注入回复"
read_when:
  - 修改音频转录或媒体处理
  - 配置语音识别服务
  - 了解音频处理流程
title: "音频与语音消息"
---

# 🎤 音频与语音消息处理

本文档详细介绍 OpenClaw 如何处理入站音频和语音消息，包括下载、转录和注入回复。

---

## ✅ 支持的功能

### 媒体理解（音频）

如果启用了音频理解功能（自动检测或手动配置），OpenClaw 会：

1. **定位附件**：找到第一个音频附件（本地路径或 URL），必要时下载
2. **大小限制**：在发送到模型前强制执行 `maxBytes` 限制
3. **模型调用**：按顺序运行第一个符合条件的模型条目（提供商或 CLI）
4. **故障转移**：如果失败或跳过（大小/超时），尝试下一个条目
5. **结果注入**：成功后，用 `[Audio]` 块替换 `Body`，并设置 `{{Transcript}}`

### 命令解析

转录成功后，`CommandBody` 和 `RawBody` 会被设置为转录文本，确保斜杠命令仍然可用。

### 详细日志

在 `--verbose` 模式下，会记录转录运行和内容替换的详细信息。

---

## 🔄 自动检测（默认）

如果**未配置模型**且 `tools.media.audio.enabled` 未设置为 `false`，OpenClaw 会按以下顺序自动检测：

| 优先级 | 方式 | 说明 |
|--------|------|------|
| 1 | **本地 CLI** | sherpa-onnx-offline、whisper-cli、whisper |
| 2 | **Gemini CLI** | 使用 `read_many_files` |
| 3 | **提供商密钥** | OpenAI → Groq → Deepgram → Google |

### 本地 CLI 要求

| CLI | 要求 |
|-----|------|
| `sherpa-onnx-offline` | 需要 `SHERPA_ONNX_MODEL_DIR` 环境变量，包含 encoder、decoder、joiner、tokens |
| `whisper-cli` | 使用 `WHISPER_CPP_MODEL` 或内置的 tiny 模型 |
| `whisper` | Python CLI，自动下载模型 |

### 禁用自动检测

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```

---

## ⚙️ 配置示例

### 方式 1：提供商 + CLI 回退

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,  // 20MB
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45
          }
        ]
      }
    }
  }
}
```

### 方式 2：仅提供商 + 范围限制

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [
            { action: "deny", match: { chatType: "group" } }
          ]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" }
        ]
      }
    }
  }
}
```

### 方式 3：仅提供商（Deepgram）

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3" }
        ]
      }
    }
  }
}
```

---

## 📋 详细配置选项

### 基础配置

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,              // 是否启用
        maxBytes: 20971520,         // 最大文件大小（20MB）
        maxChars: null,             // 最大转录字符数（不限制）
        models: [...]               // 模型配置
      }
    }
  }
}
```

### 范围限制

```json5
{
  scope: {
    default: "allow",              // 默认行为
    rules: [
      { action: "deny", match: { chatType: "group" } },  // 群组禁用
      { action: "allow", match: { channel: "telegram" } }  // Telegram 启用
    ]
  }
}
```

### 附件处理

```json5
{
  attachments: {
    mode: "all",                   // 处理方式
    maxAttachments: 5              // 最大附件数
  }
}
```

---

## 🔐 认证配置

### 提供商认证顺序

1. Auth 配置文件
2. 环境变量
3. `models.providers.*.apiKey`

### Deepgram 配置

```bash
# 设置 API Key
export DEEPGRAM_API_KEY="your-api-key"
```

### OpenAI 配置

```json5
{
  models: {
    providers: {
      openai: {
        apiKey: "${OPENAI_API_KEY}"
      }
    }
  }
}
```

---

## ⚠️ 注意事项与限制

| 项目 | 说明 |
|------|------|
| **大小限制** | 默认 20MB，超大音频会被跳过 |
| **超时时间** | CLI 默认 60 秒，需合理设置 |
| **输出限制** | CLI stdout 限制 5MB |
| **转录变量** | 模板中可用 `{{Transcript}}` |
| **多附件** | 使用 `attachments` 配置处理多条语音 |

### 模型选择

| 模型 | 特点 |
|------|------|
| `gpt-4o-mini-transcribe` | 快速、低成本 |
| `gpt-4o-transcribe` | 高精度 |

### 输出格式

| 参数 | 说明 |
|------|------|
| `baseUrl` | 自定义 API 地址 |
| `headers` | 自定义请求头 |
| `providerOptions` | 提供商特定选项 |

---

## 🐛 常见问题

### 转录失败

```bash
# 检查音频文件
file audio.mp3

# 检查 CLI 是否安装
which whisper
whisper --version

# 查看详细日志
openclaw logs --verbose | grep audio
```

### 权限问题

```bash
# 检查 API Key
echo $OPENAI_API_KEY

# 验证配置文件
openclaw config get tools.media.audio
```

### 大文件处理

```bash
# 调整大小限制
openclaw config set tools.media.audio.maxBytes 52428800  # 50MB
```

---

## 📊 支持的平台

| 平台 | 本地 CLI | 提供商 | 备注 |
|------|----------|--------|------|
| macOS | ✅ | ✅ | 完整支持 |
| Linux | ✅ | ✅ | 完整支持 |
| Windows | ✅ | ✅ | 完整支持 |
| iOS 节点 | ❌ | ✅ | 依赖节点传输 |
| Android 节点 | ❌ | ✅ | 依赖节点传输 |

---

## 📚 相关文档

- [媒体理解](/zh-CN/nodes/media-understanding) - 媒体处理概述
- [相机节点](/zh-CN/nodes/camera) - 图像捕获
- [配置参考](/zh-CN/config/reference) - 完整配置选项
- [CLI 参考](/zh-CN/cli) - 命令行工具

---

**音频处理让语音消息也能被 AI 理解和回应！** 🦞
