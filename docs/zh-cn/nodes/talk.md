---
summary: "Talk 模式 - 与 ElevenLabs TTS 的连续语音对话"
read_when:
  - 在 macOS/iOS/Android 上实现 Talk 模式
  - 修改语音/TTS/中断行为
  - 配置语音合成
title: "语音对话模式"
---

# 🎙️ Talk 模式

本文档详细介绍 OpenClaw 的 **Talk 模式**，一种与 AI 助手进行连续语音对话的方式。

---

## 🎯 什么是 Talk 模式？

Talk 模式是一个**连续的语音对话循环**：

```
1. 🎤 监听语音
2. 📝 发送转录到模型
3. 🤖 等待响应
4. 🔊 通过 ElevenLabs 播放
```

### 对话流程图

```
🎤 用户说话
    │
    ▼
┌─────────────┐
│  语音转文字  │ ←─ 麦克风输入
└─────────────┘
    │
    ▼
┌─────────────┐
│  发送给 AI  │ ←─ main 会话
└─────────────┘
    │
    ▼
┌─────────────┐
│  AI 响应    │
└─────────────┘
    │
    ▼
┌─────────────┐
│  TTS 播放   │ ←─ ElevenLabs
└─────────────┘
    │
    ├─→ 用户中断 → 停止播放，重新开始
    │
    ▼
    返回步骤 1
```

---

## 🍎 macOS 行为

### 界面特性

- **常驻覆盖层**：启用 Talk 模式时显示
- **状态转换**：监听 → 思考 → 说话
- **短暂停触发**：检测到静音窗口时发送当前转录
- **回复写入**：回复同时写入 WebChat
- **语音中断**（默认开启）：用户开始说话时停止播放

### 状态指示

| 状态 | 视觉效果 |
|------|----------|
| **Listening** | 云朵随麦克风音量脉冲 |
| **Thinking** | 下沉动画 |
| **Speaking** | 辐射环动画 |

### 交互操作

- 点击云朵：停止说话
- 点击 X：退出 Talk 模式

---

## 🎛️ 语音指令

AI 回复可以通过**单行 JSON** 前缀控制语音：

```json
{ "voice": "<voice-id>", "once": true }
```

### 指令规则

| 规则 | 说明 |
|------|------|
| 生效位置 | 仅第一行非空行 |
| 未知键 | 忽略 |
| `once: true` | 仅当前回复生效 |
| 无 `once` | 成为新的默认语音 |

### 支持的参数

| 参数 | 别名 | 说明 |
|------|------|------|
| `voice` | `voice_id`, `voiceId` | 语音 ID |
| `model` | `model_id`, `modelId` | 模型 ID |
| `speed` | `rate` | 语速（WPM） |
| `stability` | - | 稳定性（0-1） |
| `similarity` | - | 相似度（0-1） |
| `style` | - | 风格化 |
| `speakerBoost` | - | 语音增强 |
| `seed` | - | 随机种子 |
| `normalize` | - | 音量归一化 |
| `lang` | - | 语言 |
| `output_format` | - | 输出格式 |
| `latency_tier` | - | 延迟层级（0-4） |
| `once` | - | 单次使用 |

### 使用示例

```json
{ "voice": "Adam", "once": true }
{ "voice": "Bella", "model": "eleven_v3", "speed": 150 }
```

---

## ⚙️ 配置

### 配置文件位置

`~/.openclaw/openclaw.json`

### 基础配置

```json5
{
  talk: {
    voiceId: "your-elevenlabs-voice-id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "your-elevenlabs-api-key",
    interruptOnSpeech: true
  }
}
```

### 默认值

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `interruptOnSpeech` | true | 语音中断 |
| `voiceId` | 自动检测 | 回退到环境变量 |
| `modelId` | `eleven_v3` | 默认模型 |
| `apiKey` | `ELEVENLABS_API_KEY` | API Key |
| `outputFormat` | 平台相关 | 见下表 |

### 输出格式（按平台）

| 平台 | 默认格式 | 可选格式 |
|------|----------|----------|
| macOS/iOS | `pcm_44100` | `mp3_*` |
| Android | `pcm_24000` | `pcm_16000`, `pcm_22050`, `pcm_44100` |

### 环境变量

```bash
# ElevenLabs API Key
export ELEVENLABS_API_KEY="your-api-key"

# 备用语音 ID
export ELEVENLABS_VOICE_ID="voice-id"
export SAG_VOICE_ID="voice-id"
```

---

## 🎛️ macOS UI

### 菜单栏

- **Talk** 切换按钮

### 设置界面

```
设置 → Talk 模式
├── 语音 ID 输入框
├── 中断开关
└── 测试播放按钮
```

---

## 📋 详细配置选项

### 语音参数

```json5
{
  "talk": {
    "voiceId": "Adam",
    "modelId": "eleven_v3",
    "stability": 0.5,
    "similarity": 0.75,
    "style": 0,
    "speakerBoost": true,
    "speed": 150,
    "seed": -1,
    "normalize": true,
    "lang": "zh",
    "outputFormat": "pcm_44100",
    "latencyTier": 2
  }
}
```

### 参数验证

| 参数 | 有效值 |
|------|--------|
| `stability`（eleven_v3） | `0.0`, `0.5`, `1.0` |
| `stability`（其他模型） | `0` - `1` |
| `latency_tier` | `0` - `4` |

---

## 🔐 权限要求

| 权限 | 用途 | 平台 |
|------|------|------|
| **语音识别** | 监听用户语音 | macOS/iOS/Android |
| **麦克风** | 捕获音频输入 | macOS/iOS/Android |

---

## 🐛 故障排除

### 无法启动 Talk 模式

```bash
# 检查权限
openclaw health | grep permissions

# 检查 API Key
echo $ELEVENLABS_API_KEY

# 查看详细日志
openclaw logs --verbose | grep talk
```

### 语音播放问题

```bash
# 测试语音合成
curl -X POST "https://api.elevenlabs.io/v1/text-to-speech/{voice_id}" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello", "model_id": "eleven_v3"}'
```

### 中断功能不工作

```bash
# 检查中断设置
openclaw config get talk.interruptOnSpeech

# 启用中断
openclaw config set talk.interruptOnSpeech true
```

---

## 📊 平台支持

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| **macOS** | ✅ 完全支持 | 完整 UI 和功能 |
| **iOS** | ✅ 完全支持 | 移动端语音对话 |
| **Android** | ✅ 完全支持 | 低延迟 AudioTrack |
| **Linux** | ❌ 不支持 | 无本地 UI |
| **Windows** | ❌ 不支持 | 无本地 UI |

---

## 📈 性能优化

### 低延迟配置

```json5
{
  "talk": {
    "outputFormat": "pcm_16000",
    "latencyTier": 0,
    "interruptOnSpeech": true
  }
}
```

### 音质优先配置

```json5
{
  "talk": {
    "outputFormat": "mp3_44100_320",
    "stability": 0.5,
    "similarity": 0.9,
    "latencyTier": 4
  }
}
```

---

## 📚 相关文档

- [节点系统](/zh-CN/nodes) - 节点概述
- [语音唤醒](/zh-CN/nodes/voicewake) - 免提唤醒
- [音频处理](/zh-CN/nodes/audio) - 音频转录
- [ElevenLabs 文档](https://elevenlabs.io/docs) - 官方 API 文档

---

**Talk 模式让与 AI 对话变得像打电话一样自然！** 🦞
