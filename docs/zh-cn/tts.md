---
summary: "OpenClaw 文本转语音（TTS）完全指南：多提供商配置、telephony 优化和最佳实践"
read_when:
  - 你需要配置 TTS 用于语音输出
  - 你需要 telephony 优化的 TTS 用于电话场景
  - 你想了解 TTS 提供商的选择和配置
  - 你在调试语音质量问题
title: "文本转语音（TTS）"
---

# 🗣️ OpenClaw 文本转语音完全指南

> **学习目标**：完成本章节学习后，你将能够理解 TTS 的工作原理和提供商特点，掌握从基础配置到 telephony 优化的完整知识体系，能够根据不同场景选择合适的 TTS 配置，并能够诊断和解决常见的语音问题。

---

## 为什么要学习 TTS 配置

在深入技术细节之前，我们需要先理解**TTS 在 OpenClaw 系统中的定位和价值**。文本转语音（Text-to-Speech）不仅仅是「让机器说话」，它是多模态交互的关键组成部分。

### TTS 的应用场景

```
┌─────────────────────────────────────────────────────────────────┐
│                    TTS 应用场景                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. 语音助手响应                                                │
│      - 将 AI 回复转换为语音                                      │
│      - 提供更自然的人机交互                                       │
│                                                                  │
│   2. 无障碍访问                                                 │
│      - 视觉障碍用户的语音反馈                                     │
│      - 多感官交互体验                                            │
│                                                                  │
│   3. 电话集成                                                   │
│      -  telephony 场景的语音输出                                  │
│      - 电话机器人和 IVR 系统                                     │
│                                                                  │
│   4. 多语言支持                                                 │
│      - 自然的多语言语音输出                                       │
│      - 跨语言沟通桥梁                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### TTS 提供商对比概览

| 提供商 | 质量 | 延迟 | 成本 | 特点 |
|--------|------|------|------|------|
| **OpenAI TTS** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 中等 | 稳定可靠、集成简单 |
| **ElevenLabs** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 较高 | 最高质量、自然语音 |
| **Edge TTS** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 免费 | 微软技术、多语言 |
| **telephony TTS** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 低 | 专为电话优化 |

---

## 认知负荷管理：学习路径设计

```
TTS 技能金字塔：

                    ┌─────────────────────────┐
                    │   专家级：telephony 优化 │  PCM 格式、低延迟优化
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：高级配置       │  SSML、语音定制
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：多提供商       │  提供商切换、成本优化
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：基础配置       │  基本 TTS 配置
                    └─────────────────────────┘
```

**建议学习路径**：
- 新手：从 OpenAI TTS 开始，掌握基础配置
- 开发者：学习多提供商切换和成本优化
- 高级用户：深入 telephony 优化和高级特性

---

## 第一部分：TTS 基础配置（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 TTS 配置的基本结构
- [ ] 配置 OpenAI TTS 提供商
- [ ] 选择合适的语音和格式
- [ ] 理解 TTS 的基本参数

### 1.1 TTS 配置结构

**配置路径**：

TTS 配置位于 `messages.tts` 下：

```json5
{
  messages: {
    tts: {
      // TTS 提供商选择
      provider: "openai",
      
      // 语音选择
      voice: "alloy",
      
      // 模型选择（部分提供商）
      model: "tts-1",
      
      // 语速调整
      speed: 1.0,
      
      // 音频格式
      format: "mp3"
    }
  }
}
```

**配置参数说明**：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `provider` | 字符串 | "openai" | TTS 提供商 |
| `voice` | 字符串 | 取决于提供商 | 语音选择 |
| `model` | 字符串 | 取决于提供商 | TTS 模型 |
| `speed` | 浮点数 | 1.0 | 语速（0.25-4.0） |
| `format` | 字符串 | "mp3" | 音频格式 |

### 1.2 OpenAI TTS 配置

**配置示例**：

```json5
{
  messages: {
    tts: {
      provider: "openai",
      voice: "alloy",
      model: "tts-1",
      speed: 1.0,
      format: "mp3"
    }
  }
}
```

**可用语音**：

OpenAI 提供 6 种预设语音：

| 语音名称 | 特点 | 适用场景 |
|----------|------|----------|
| `alloy` | 中性、清晰 | 通用场景 |
| `echo` | 回声感 | 有声读物 |
| `fable` | 故事感 | 叙事内容 |
| `onyx` | 低沉、权威 | 正式场合 |
| `nova` | 明亮、活泼 | 客服场景 |
| `shimmer` | 柔和、温暖 | 安慰性内容 |

**可用模型**：

| 模型 | 特点 | 延迟 | 质量 |
|------|------|------|------|
| `tts-1` | 快速 | 低 | 标准 |
| `tts-1-hd` | 高质量 | 较高 | 高清 |

**音频格式**：

| 格式 | 采样率 | 适用场景 |
|------|---------|----------|
| `mp3` | 24000Hz | 通用场景 |
| `opus` | 24000Hz | 低带宽 |
| `aac` | 24000Hz | iOS/Mac |
| `flac` | 24000Hz | 无损质量 |

### 1.3 ElevenLabs TTS 配置

**配置示例**：

```json5
{
  messages: {
    tts: {
      provider: "elevenlabs",
      voice: "21m00tcm4tvlSoVr12Ng",  // 语音 ID
      model: "eleven_turbo_v2_5",
      stability: 0.5,
      similarityBoost: 0.75
    }
  }
}
```

**配置说明**：

| 参数 | 范围 | 说明 |
|------|------|------|
| `voice` | 语音 ID | ElevenLabs 语音标识符 |
| `model` | 模型名称 | ElevenLabs TTS 模型 |
| `stability` | 0-1 | 语音稳定性 |
| `similarityBoost` | 0-1 | 与原始语音的相似度 |

**API 密钥配置**：

```bash
# 通过环境变量设置
export ELEVENLABS_API_KEY="your-api-key"
```

**语音获取**：

```bash
# 查看可用语音
openclaw tts voices --provider elevenlabs
```

### 1.4 Edge TTS 配置

**配置示例**：

```json5
{
  messages: {
    tts: {
      provider: "edge-tts",
      voice: "zh-CN-XiaoxiaoNeural",
      rate: "+0%",
      pitch: "+0Hz"
    }
  }
}
```

**中文语音列表**：

| 语音 ID | 性别 | 风格 |
|----------|------|------|
| `zh-CN-XiaoxiaoNeural` | 女 | 标准 |
| `zh-CN-YunxiNeural` | 男 | 叙事 |
| `zh-CN-YunyangNeural` | 男 | 新闻 |
| `zh-CN-XiaoshuangNeural` | 女 | 儿童 |

**参数调整**：

| 参数 | 格式 | 说明 |
|------|------|------|
| `rate` | "+N%" 或 "-N%" | 语速调整 |
| `pitch` | "+NHz" 或 "-NHz" | 音高调整 |

---

## 第二部分：高级配置（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 配置多个 TTS 提供商
- [ ] 实现提供商故障转移
- [ ] 优化 TTS 成本
- [ ] 理解 SSML 标记

### 2.1 多提供商配置

**配置结构**：

```json5
{
  messages: {
    tts: {
      // 默认提供商
      provider: "openai",
      
      // 提供商特定配置
      providers: {
        "openai": {
          voice: "alloy",
          model: "tts-1"
        },
        "elevenlabs": {
          voice: "21m00tcm4tvlSoVr12Ng",
          stability: 0.5
        }
      }
    }
  }
}
```

**提供商切换**：

```bash
# 使用指定的提供商
openclaw tts --provider openai --text "Hello"

# 切换提供商
openclaw tts --provider elevenlabs --text "Hello"
```

### 2.2 故障转移策略

**配置故障转移**：

```json5
{
  messages: {
    tts: {
      provider: "openai",
      
      // 故障转移配置
      failover: {
        enabled: true,
        providers: ["openai", "edge-tts", "elevenlabs"],
        timeout: 5000  // 超时时间（毫秒）
      }
    }
  }
}
```

**故障转移流程**：

```
┌─────────────────────────────────────────────────────────────┐
│                  TTS 故障转移流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. 尝试主要提供商                                          │
│   ┌─────────────────────────────────────────────────────┐ │
│   │  OpenAI TTS 请求                                      │ │
│   │  超时：5 秒                                          │ │
│   └─────────────────────────────────────────────────────┘ │
│                           │                                  │
│              ┌───────────┴───────────┐                     │
│              ▼                       ▼                     │
│         成功？                    失败？                     │
│           │                         │                       │
│           ▼                         ▼                       │
│      返回结果              尝试备用提供商                     │
│                           ┌─────────────────────────────┐ │
│                           │  edge-tts（免费）            │ │
│                           │  或                          │ │
│                           │  ElevenLabs（高质量）        │ │
│                           └─────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 成本优化策略

**成本对比**：

| 提供商 | 价格/1M 字符 | 质量 | 适用场景 |
|--------|--------------|------|----------|
| OpenAI TTS | ~$15 | ⭐⭐⭐⭐ | 通用场景 |
| ElevenLabs | ~$30 | ⭐⭐⭐⭐⭐ | 高质量需求 |
| Edge TTS | 免费 | ⭐⭐⭐ | 测试/开发 |

**优化策略**：

```json5
{
  messages: {
    tts: {
      // 成本感知配置
      costOptimization: {
        enabled: true,
        
        // 默认使用低成本提供商
        defaultProvider: "edge-tts",
        
        // 高质量场景切换
        premiumProvider: "elevenlabs",
        
        // 基于用户等级的提供商选择
        userTiers: {
          free: "edge-tts",
          basic: "openai",
          premium: "elevenlabs"
        }
      }
    }
  }
}
```

### 2.4 SSML 标记支持

**SSML 基本语法**：

```xml
<speak>
  你好，这是
  <break time="500ms"/>
  <emphasis level="moderate">重要</emphasis>
  的消息。
</speak>
```

**支持的 SSML 标签**：

| 标签 | 说明 | 示例 |
|------|------|------|
| `<speak>` | SSML 根元素 | `<speak>内容</speak>` |
| `<break>` | 停顿 | `<break time="500ms"/>` |
| `<emphasis>` | 强调 | `<emphasis level="strong">加粗</emphasis>` |
| `<prosody>` | 语速音高 | `<prosody rate="fast" pitch="+2st">` |
| `<say-as>` | 特殊读法 | `<say-as interpret-as="digits">12345</say-as>` |

---

## 第三部分：Telephony 优化（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 telephony TTS 的特殊要求
- [ ] 配置低延迟音频输出
- [ ] 处理 PCM 格式数据
- [ ] 优化电话场景的语音质量

### 3.1 Telephony TTS 的特殊性

**核心挑战**：

```
┌─────────────────────────────────────────────────────────────┐
│              Telephony TTS 的技术挑战                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. 带宽限制                                               │
│      - 电话网络带宽有限（通常 < 64 kbps）                     │
│      - 需要窄带音频（8kHz 采样率）                           │
│                                                              │
│   2. 延迟要求                                               │
│      - 实时通话需要低延迟                                    │
│      - TTS 响应应 < 500ms                                   │
│                                                              │
│   3. 格式兼容                                               │
│      - 传统电话网络仅支持 PCM 格式                           │
│      - 需要兼容性处理                                        │
│                                                              │
│   4. 语音清晰度                                             │
│      - 电话音质有限，需要优化语音清晰度                       │
│      - 避免低频和高频的丢失                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Telephony TTS 配置

**配置示例**：

```json5
{
  messages: {
    tts: {
      provider: "openai",
      voice: "alloy",
      format: "pcm",
      sampleRate: 8000,
      
      // telephony 特定优化
      telephony: {
        optimize: true,
        equalization: "phone-line",
        noiseReduction: true
      }
    }
  }
}
```

**音频格式对比**：

| 格式 | 采样率 | 位深度 | 用途 |
|------|---------|--------|------|
| `mp3` | 24000Hz | 16-bit | 通用场景 |
| `opus` | 24000Hz | 16-bit | 低带宽 |
| `pcm` | 8000Hz | 16-bit | 电话场景 |
| `pcm` | 16000Hz | 16-bit | 宽带电话 |

### 3.3 PCM 数据处理

**接口定义**：

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

**PCM 数据处理流程**：

```
┌─────────────────────────────────────────────────────────────┐
│                  PCM 数据处理流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. TTS 生成                                               │
│   ┌─────────────────────────────────────────────────────┐ │
│   │  输入文本 → TTS 引擎 → PCM 音频数据                  │ │
│   │  采样率：8000Hz                                      │ │
│   │  位深度：16-bit                                       │ │
│   │  通道：单声道                                        │ │
│   └─────────────────────────────────────────────────────┘ │
│                           │                                  │
│                           ▼                                  │
│   2. 音频编码（如果需要）                                   │
│   ┌─────────────────────────────────────────────────────┐ │
│   │  PCM → G.711 (A-law/μ-law)                          │ │
│   │  目的：兼容传统电话网络                               │ │
│   └─────────────────────────────────────────────────────┘ │
│                           │                                  │
│                           ▼                                  │
│   3. 传输                                                   │
│   ┌─────────────────────────────────────────────────────┐ │
│   │  RTP 包传输                                           │ │
│   │  延迟目标：< 100ms                                   │ │
│   └─────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第四部分：使用 TTS 发送语音消息

### 学习目标

完成本节学习后，你将能够：
- [ ] 在消息中使用 TTS
- [ ] 使用工具调用 TTS
- [ ] 在插件中集成 TTS
- [ ] 实现高级 TTS 功能

### 4.1 消息中使用 TTS

**语法格式**：

```markdown
[[voice:TTS:Hello, this is a voice message]]
```

**完整示例**：

```markdown
你好，这是语音消息：

[[voice:TTS:今天天气真好，适合外出散步。]]

再见！
```

### 4.2 工具调用

**命令行调用**：

```bash
# 基本用法
openclaw tts --text "Hello, world!"

# 指定提供商
openclaw tts --provider openai --voice alloy --text "Hello"

# 指定输出文件
openclaw tts --text "Hello" --output /tmp/hello.mp3
```

**脚本集成**：

```typescript
// 使用 bash 工具调用 TTS
await tool("bash", {
  command: `echo 'Hello world' | openclaw tts --provider openai --voice alloy --output /tmp/hello.mp3`,
});
```

### 4.3 插件中使用 TTS

**基础用法**：

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

**高级用法**：

```typescript
import { textToSpeech, textToSpeechTelephony } from "openclaw/tts";

// 标准 TTS
const audio = await textToSpeech({
  text: "您的订单已经发货",
  provider: "openai",
  voice: "alloy",
  format: "mp3",
});

// Telephony TTS
const telephonyAudio = await textToSpeechTelephony({
  text: "您好，欢迎致电",
  provider: "openai",
  format: "pcm",
  sampleRate: 8000,
});
```

---

## 第五部分：故障排查（⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 诊断常见的 TTS 问题
- [ ] 解决音频质量问题
- [ ] 优化 TTS 性能
- [ ] 配置替代方案

### 5.1 常见问题与解决方案

**问题一：音频质量问题**

**症状**：语音不清晰、有杂音

**排查步骤**：

```bash
# 1. 检查网络连接
curl -I https://api.openai.com

# 2. 测试不同语音
openclaw tts --voice alloy --text "Test"
openclaw tts --voice onyx --text "Test"

# 3. 检查音频格式
file /tmp/hello.mp3
```

**解决方案**：

| 问题 | 解决方案 |
|------|----------|
| 语音不清晰 | 切换到高清模型 `tts-1-hd` |
| 杂音/失真 | 检查网络稳定性 |
| 语音不自然 | 调整语速 `speed: 0.9` |

**问题二：延迟问题**

**症状**：TTS 响应慢

**排查步骤**：

```bash
# 1. 测量 API 延迟
curl -X POST https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{"model":"tts-1","input":"test","voice":"alloy"}' \
  --max-time 30 -o /dev/null -w "%{time_total}\n"

# 2. 检查提供商状态
openclaw tts --provider openai --text "status"
```

**优化方案**：

| 优化方法 | 效果 |
|----------|------|
| 使用 `tts-1` 而非 `tts-1-hd` | 降低延迟 |
| 预加载常用语音 | 减少首次请求 |
| 使用缓存 | 重复内容直接使用 |

**问题三：语音识别问题**

**症状**：TTS 读错字、多音字问题

**解决方案**：

| 问题 | 解决方案 | 示例 |
|------|----------|------|
| 多音字 | 使用拼音或同义词 | 「银行」→ 「yin2xing2」 |
| 数字 | 使用数字读法 | 「123」→ 「一二三」 |
| 缩写 | 全称展开 | 「API」→ 「应用程序接口」 |

### 5.2 调试技巧

**获取调试信息**：

```bash
# 启用详细输出
openclaw tts --verbose --text "Hello"

# 保存中间结果
openclaw tts --save-intermediate --text "Hello"

# 查看提供商响应
openclaw tts --debug-provider --text "Hello"
```

---

## 第六部分：成本与选择指南

### 6.1 提供商选择决策树

```
┌─────────────────────────────────────────────────────────────┐
│                  TTS 提供商选择决策                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Q1: 预算有限？                                             │
│   ├── 是 → Q2: 需要多语言？                                 │
│   │         ├── 是 → Edge TTS                               │
│   │         └── 否 → OpenAI TTS (低成本模式)                │
│   │                                                             │
│   └── 否 → Q3: 需要最高质量？                               │
│             ├── 是 → ElevenLabs                              │
│             └── 否 → Q4: 需要最低延迟？                       │
│                       ├── 是 → OpenAI TTS (`tts-1`)         │
│                       └── 否 → OpenAI TTS (`tts-1-hd`)       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 成本效益分析

| 场景 | 推荐提供商 | 月成本估算 | 理由 |
|------|-----------|------------|------|
| 个人使用 | OpenAI TTS | $5-10 | 够用且便宜 |
| 小团队 | OpenAI TTS | $20-50 | 平衡质量与成本 |
| 企业应用 | ElevenLabs | $100+ | 高质量品牌形象 |
| 呼叫中心 | telephony TTS | $10-30 | 低成本、通话优化 |

---

## 练习与自检

### 基础技能自检

- [ ] 能够配置基本的 TTS
- [ ] 理解不同提供商的特点
- [ ] 能够发送语音消息
- [ ] 理解基本参数含义

### 进阶技能自检

- [ ] 能够配置多提供商故障转移
- [ ] 理解成本优化策略
- [ ] 能够处理 telephony TTS
- [ ] 掌握 SSML 基本用法

### 专家技能自检

- [ ] 能够设计 TTS 架构
- [ ] 理解音频编码原理
- [ ] 能够优化大规模部署
- [ ] 掌握性能调优方法

---

## 参考资料

### 核心文档

- [消息系统](/zh-CN/concepts/messages) - 消息处理详解
- [语音对话](/zh-CN/nodes/talk) - 实时语音对话
- [插件开发](/zh-CN/developer/plugins) - 开发 OpenClaw 插件

### 相关命令

```bash
# TTS 基本命令
openclaw tts --text "Hello"           # 基础 TTS
openclaw tts --provider elevenlabs    # 指定提供商
openclaw tts --voice alloy            # 指定语音
openclaw tts --output file.mp3        # 保存文件

# TTS 管理
openclaw tts voices                   # 列出语音
openclaw tts providers                # 列出提供商
```

### 扩展阅读

- [OpenAI TTS 文档](https://platform.openai.com/docs/guides/text-to-speech)
- [ElevenLabs 文档](https://elevenlabs.io/docs)
- [SSML W3C 规范](https://www.w3.org/TR/speech-synthesis/)
- [ telephony 音频标准](https://en.wikipedia.org/wiki/G.711)
