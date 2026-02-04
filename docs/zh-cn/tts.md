---
summary: "OpenClaw 文本转语音（TTS）配置，支持多种 TTS 提供商和 telephony 优化"
read_when:
  - 你需要配置 TTS 用于语音输出
  - 你需要 telephony 优化的 TTS 用于电话场景
  - 你想了解 TTS 提供商的选择和配置
title: "文本转语音（TTS）"
---

# 文本转语音（TTS）

OpenClaw 支持**文本转语音（Text-to-Speech）**功能，可以将文本内容转换为语音输出。本文档介绍 TTS 的配置和使用方法。

## TTS 配置

TTS 配置位于 `messages.tts` 下：

```json5
{
  messages: {
    tts: {
      provider: "openai", // 或 "elevenlabs", "edge-tts"
      voice: "alloy",
      model: "tts-1",
      speed: 1.0,
      format: "mp3",
    },
  },
}
```

## TTS 提供商

### OpenAI TTS

```json5
{
  messages: {
    tts: {
      provider: "openai",
      voice: "alloy",      // alloy, echo, fable, onyx, nova, shimmer
      model: "tts-1",      // tts-1 或 tts-1-hd
      speed: 1.0,          // 0.25 到 4.0
      format: "mp3",       // mp3, opus, aac, flac
    },
  },
}
```

### ElevenLabs TTS

```json5
{
  messages: {
    tts: {
      provider: "elevenlabs",
      voice: "21m00tcm4tvlSoVr12Ng",  // 语音 ID
      model: "eleven_turbo_v2_5",
      stability: 0.5,
      similarityBoost: 0.75,
    },
  },
}
```

配置 ElevenLabs API 密钥：

```bash
export ELEVENLABS_API_KEY="your-api-key"
```

### Edge TTS（微软）

```json5
{
  messages: {
    tts: {
      provider: "edge-tts",
      voice: "zh-CN-XiaoxiaoNeural",
      rate: "+0%",   // 语速调整
      pitch: "+0Hz", // 音高调整
    },
  },
}
```

## Telephony 优化 TTS

 telephony 场景（如电话呼叫）需要特殊优化的 TTS 输出：

### telephony TTS 特点

- **低延迟**：快速响应，适合实时通话
- **窄带音频**：8kHz 采样率，适合电话网络
- **清晰发音**：优化语音清晰度
- ** PCM 格式**：返回原始音频数据

### telephony TTS 配置

```json5
{
  messages: {
    tts: {
      provider: "openai",  // 支持 telephony 优化
      voice: "alloy",
      format: "pcm",       // telephony 使用 PCM 格式
      sampleRate: 8000,    // 电话采样率
    },
  },
}
```

### 返回格式

| format | sampleRate | 用途 |
|--------|------------|------|
| `mp3` | 24000 | 默认，通用场景 |
| `opus` | 24000 | 低带宽场景 |
| `aac` | 24000 | iOS/Mac 优化 |
| `flac` | 24000 | 无损压缩 |
| `pcm` | 8000/16000 | telephony 场景 |

### telephony 音频处理

 telephony TTS 返回 PCM 数据，插件需要处理：

```typescript
interface TelephonyTTSResult {
  audio: Buffer;           // PCM 音频数据
  sampleRate: number;      // 采样率（8000 或 16000）
  channels: number;        // 通道数（通常为 1）
  duration: number;        // 音频时长（毫秒）
}

async function handleTelephonyTTS(text: string): Promise<TelephonyTTSResult> {
  const result = await api.runtime.tts.textToSpeechTelephony({
    text,
    cfg: api.config,
  });
  
  // result.audio 是 PCM 缓冲区
  // result.sampleRate 是采样率
  
  return result;
}
```

## 使用 TTS 发送语音消息

### 在消息中使用 TTS

```markdown
[[voice:TTS:Hello, this is a voice message]]
```

### 工具调用

```typescript
// 使用 bash 工具调用 TTS
await tool("bash", {
  command: "echo 'Hello world' | openclaw tts --provider openai --voice alloy",
});
```

### 插件中使用

```typescript
import { textToSpeech } from "openclaw/tts";

const audio = await textToSpeech({
  text: "Hello, world!",
  provider: "openai",
  voice: "alloy",
  format: "mp3",
});

// 发送音频
await sendAudio(audio);
```

## TTS 故障排除

### 音频质量问题

1. **检查网络连接**：TTS 需要访问外部 API
2. **选择合适的语音**：不同语音质量差异较大
3. **调整语速**：过快的语速可能影响清晰度

### 延迟问题

1. **使用缓存**：重复内容使用缓存的音频
2. **选择低延迟模型**：如 OpenAI TTS-1 比 TTS-1-HD 延迟更低
3. **预加载**：提前生成常用消息的音频

### 语音识别问题

1. **避免多音字**：使用拼音或同义词替代
2. **添加停顿**：在适当位置添加标点符号
3. **调整发音**：使用 SSML 标记调整发音

## 成本考虑

| 提供商 | 价格 | 特点 |
|--------|------|------|
| OpenAI TTS | ~$15/1M 字符 | 稳定可靠 |
| ElevenLabs | ~$30/1M 字符 | 高质量自然语音 |
| Edge TTS | 免费 | 多语言支持 |

## 相关文档

- [消息系统](/concepts/messages) - 消息处理详解
- [语音对话](/nodes/talk) - 实时语音对话
- [插件开发](/developer/plugins) - 开发 OpenClaw 插件
- [providers](/providers) - AI 提供商配置
