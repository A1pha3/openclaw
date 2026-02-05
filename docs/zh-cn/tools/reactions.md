---
summary: "表情回应工具完整指南——跨渠道语义统一与操作详解"
read_when:
  - 在任何渠道处理表情回应功能
  - 理解各消息平台的表情API差异
  - 实现跨渠道一致的表情操作
title: "表情回应工具"
---

# 表情回应工具

本指南全面介绍OpenClaw的表情回应（Reaction）工具，帮助您理解如何通过统一的接口在多个消息渠道中添加、移除和查询表情回应。完成本章节学习后，您将能够跨渠道一致地使用表情回应工具，理解各平台的差异和限制，并掌握常见问题的处理方法。

## 学习目标

完成本章节学习后，您将能够：

### 基础目标（必掌握）

- 理解表情回应的核心概念和参数
- 掌握添加和移除表情的基本操作
- 理解各主要渠道的行为差异
- 实现跨渠道一致的表情操作

### 进阶目标（建议掌握）

- 根据渠道特性调整表情操作策略
- 处理跨渠道的表情同步问题
- 实现复杂的表情工作流程
- 优化表情操作的错误处理

### 专家目标（挑战）

- 设计跨渠道的表情管理策略
- 实现企业级的表情审计和合规
- 处理边缘情况和兼容性问题

---

## 第一部分：核心概念解析

### 为什么需要统一的表情回应工具

在深入技术细节之前，我们需要理解表情回应工具的设计目的和价值。

**核心问题**：不同消息渠道对表情回应的支持方式不同。

| 渠道 | API方式 | 限制 | 特殊行为 |
|------|---------|------|----------|
| Discord | 反应Emoji | 无 | 支持自定义emoji |
| Slack | 反应Emoji | 无 | 支持多emoji |
| Telegram | Inline Bot | 有限 | 机器人限制 |
| WhatsApp | 特殊API | 限制 | 特殊映射 |
| Google Chat | 表情符号 | 标准 | 统一支持 |
| Signal | 通知系统 | 通知仅 | 无发送API |

**统一工具的解决方案**：

```
问题：如何让AI助手在不同渠道上以一致的方式使用表情？

解决方案：统一的reactions工具

核心思想：
├── 提供统一的API接口
├── 屏蔽渠道差异
├── 规范化返回值
└── 处理渠道特定行为

优势：
├── 一次编写，到处使用
├── 错误处理一致
├── 易于维护和扩展
└── 清晰的渠道行为文档
```

### 核心参数

reactions工具的核心参数设计简洁明确：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| messageId | string | 是 | 目标消息的ID |
| emoji | string | 是 | 要添加的表情（移除时也需要） |
| remove | boolean | 否 | 是否移除模式（默认false） |

**参数说明**：

```
emoji参数：
├── 添加回应时：必须是有效的emoji字符
├── 移除回应时：指定要移除的emoji
└── 空字符串：特殊语义（见各渠道行为）

remove参数：
├── false（默认）：添加emoji回应
├── true：移除指定emoji（需同时提供emoji）
└── 注意：false时移除所有，true时移除指定
```

---

## 第二部分：核心操作

### 2.1 添加表情回应

**基本调用**：

```typescript
await reaction({
  messageId: "abc123",
  emoji: "👍"
});
```

**返回结果**：

```typescript
{
  success: true,
  channel: "discord",
  messageId: "abc123",
  emoji: "👍",
  addedAt: "2024-02-05T10:30:00Z"
}
```

### 2.2 移除表情回应

**移除指定表情**：

```typescript
await reaction({
  messageId: "abc123",
  emoji: "👍",
  remove: true
});
```

**移除所有回应（部分渠道）**：

```typescript
// Discord、Slack、Google Chat
await reaction({
  messageId: "abc123",
  emoji: ""
});
```

### 2.3 完整参数示例

```typescript
await reaction({
  messageId: "msg-456",
  emoji: "🎉",
  remove: false
});
```

---

## 第三部分：渠道行为差异

### 3.1 行为差异总览

理解各渠道的差异是正确使用工具的关键：

| 渠道 | 空emoji行为 | remove:true行为 | 特殊限制 |
|------|------------|-----------------|----------|
| **Discord** | 移除该消息上机器人的所有回应 | 仅移除指定emoji | 支持自定义emoji |
| **Slack** | 移除该消息上机器人的所有回应 | 仅移除指定emoji | 支持多工作区 |
| **Google Chat** | 移除应用的回应 | 仅移除指定emoji | 统一emoji支持 |
| **Telegram** | 移除机器人的回应 | 同样移除回应 | 需非空emoji验证 |
| **WhatsApp** | 移除机器人回应 | 移除机器人回应 | 映射为空emoji |
| **Signal** | N/A | N/A | 仅通知，无发送 |

### 3.2 Discord行为详解

**添加回应**：

```typescript
await reaction({
  messageId: "123456789",
  emoji: "👍"  // 标准Unicode emoji
});
// 或
await reaction({
  messageId: "123456789",
  emoji: "<:custom:123456789>"  // 自定义emoji
});
```

**移除回应**：

```typescript
// 移除指定emoji
await reaction({
  messageId: "123456789",
  emoji: "👍",
  remove: true
});

// 移除所有回应（使用空emoji）
await reaction({
  messageId: "123456789",
  emoji: ""
});
```

**特点**：
- 完全支持Unicode emoji
- 支持服务器自定义emoji
- API行为一致

### 3.3 Slack行为详解

**添加回应**：

```typescript
await reaction({
  messageId: "msg-id",
  emoji: "white_check_mark"  // Slack名称格式
});
// 或
await reaction({
  messageId: "msg-id",
  emoji: "✅"  // Unicode格式
});
```

**移除回应**：

```typescript
// 移除指定emoji
await reaction({
  messageId: "msg-id",
  emoji: "white_check_mark",
  remove: true
});

// 移除所有回应
await reaction({
  messageId: "msg-id",
  emoji: ""
});
```

**注意事项**：
- Slack使用emoji名称（非Unicode）更可靠
- 支持跨工作区一致操作
- Enterprise Grid支持企业emoji

### 3.4 Google Chat行为详解

**添加回应**：

```typescript
await reaction({
  messageId: "spaces/xxx/messages/yyy",
  emoji: "👍"  // 标准Unicode emoji
});
```

**移除回应**：

```typescript
// 移除指定emoji
await reaction({
  messageId: "spaces/xxx/messages/yyy",
  emoji: "👍",
  remove: true
});

// 移除所有回应
await reaction({
  messageId: "spaces/xxx/messages/yyy",
  emoji: ""
});
```

**特点**：
- 统一的表情API
- 消息ID格式特殊（spaces/xxx/messages/yyy）
- 企业环境支持良好

### 3.5 Telegram行为详解

**添加回应**：

```typescript
await reaction({
  messageId: "12345",
  emoji: "👍"
});
```

**移除回应**：

```typescript
// 移除机器人的所有回应
await reaction({
  messageId: "12345",
  emoji: ""
});

// 移除指定emoji
await reaction({
  messageId: "12345",
  emoji: "👍",
  remove: true
});
```

**特殊限制**：
- Telegram对机器人的表情有限制
- 移除时工具验证仍要求非空emoji
- 某些emoji可能不被支持

### 3.6 WhatsApp行为详解

**添加回应**：

```typescript
await reaction({
  messageId: "message-id",
  emoji: "👍"
});
```

**移除回应**：

```typescript
// 移除机器人回应（映射为空emoji）
await reaction({
  messageId: "message-id",
  emoji: ""
});

// 移除指定emoji
await reaction({
  messageId: "message-id",
  emoji: "👍",
  remove: true
});
```

**特殊限制**：
- WhatsApp的表情API有特殊限制
- 某些高级表情可能不支持
- 建议使用标准Unicode emoji

### 3.7 Signal行为详解

**特点**：

```
Signal与其他渠道不同：
├── 仅支持入站通知
├── 无发送API
└── 需要配置启用通知

配置：
{
  "channels": {
    "signal": {
      "reactionNotifications": true
    }
  }
}
```

**使用方式**：

```typescript
// 入站时触发系统事件
// 无法主动发送表情回应
```

---

## 第四部分：高级用法

### 4.1 批量操作

对于需要多个表情的场景：

```typescript
// 依次添加多个表情
const emojis = ["👍", "❤️", "🎉"];

for (const emoji of emojis) {
  await reaction({
    messageId: "msg-id",
    emoji
  });
}
```

**最佳实践**：
- 批量操作时添加适当延迟
- 处理单个失败的继续其他
- 记录操作结果

### 4.2 条件性操作

根据消息状态决定操作：

```typescript
async function smartReaction(messageId: string) {
  // 查询当前回应
  const currentReactions = await getReactions(messageId);
  
  // 条件判断
  if (!currentReactions.includes("👀")) {
    await reaction({
      messageId,
      emoji: "👀"
    });
  }
}
```

### 4.3 工作流集成

表情回应可以集成到自动化工作流中：

```
场景：自动标记重要消息

步骤：
1. 检测消息内容
2. 判断是否重要
3. 添加对应表情
4. 记录到任务系统
```

**示例**：

```typescript
async function markImportant(messageId: string, reason: string) {
  // 添加重要标记
  await reaction({
    messageId,
    emoji: "⭐"
  });
  
  // 记录到日志
  log(`标记重要: ${messageId}, 原因: ${reason}`);
}
```

---

## 第五部分：使用场景

### 5.1 消息确认

**场景**：确认收到用户消息

**实现**：

```typescript
async function acknowledgeMessage(messageId: string, channel: string) {
  // 根据渠道选择合适的确认emoji
  const emoji = getAckEmoji(channel);
  
  await reaction({
    messageId,
    emoji
  });
}

function getAckEmoji(channel: string): string {
  const emojiMap = {
    discord: "✅",
    slack: "white_check_mark",
    telegram: "👍",
    whatsapp: "👍",
    googlechat: "👍"
  };
  
  return emojiMap[channel] || "✅";
}
```

### 5.2 状态标记

**场景**：标记消息状态

**常见状态emoji**：

| 状态 | emoji | 说明 |
|------|-------|------|
| 待处理 | "⏳" | 等待处理 |
| 进行中 | "🔄" | 正在处理 |
| 已完成 | "✅" | 处理完成 |
| 问题 | "❓" | 需要澄清 |
| 紧急 | "🚨" | 需要立即关注 |

**实现**：

```typescript
async function updateStatus(messageId: string, status: string) {
  const statusEmojis: Record<string, string> = {
    pending: "⏳",
    inProgress: "🔄",
    completed: "✅",
    question: "❓",
    urgent: "🚨"
  };
  
  // 移除旧状态（如有）
  const oldEmoji = getCurrentStatusEmoji(messageId);
  if (oldEmoji) {
    await reaction({
      messageId,
      emoji: oldEmoji,
      remove: true
    });
  }
  
  // 添加新状态
  await reaction({
    messageId,
    emoji: statusEmojis[status]
  });
}
```

### 5.3 投票系统

**场景**：简单的emoji投票

**实现**：

```typescript
async function startPoll(messageId: string, options: string[]) {
  // 为每个选项添加投票emoji
  for (let i = 0; i < options.length; i++) {
    await reaction({
      messageId,
      emoji: numberToEmoji(i + 1)
    });
  }
}

function numberToEmoji(n: number): string {
  const emojis = ["1️⃣", "2️⃣", "3️⃣", "4️⃣", "5️⃣", "6️⃣", "7️⃣", "8️⃣", "9️⃣", "🔟"];
  return emojis[n - 1] || String(n);
}
```

### 5.4 消息归档

**场景**：归档已处理的消息

**实现**：

```typescript
async function archiveMessage(messageId: string) {
  // 添加归档表情
  await reaction({
    messageId,
    emoji: "📁"
  });
  
  // 可选：记录归档日志
  log(`消息归档: ${messageId}`);
}
```

---

## 第六部分：故障排除

### 6.1 常见错误

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| CHANNEL_UNSUPPORTED | 渠道不支持表情 | 检查渠道配置 |
| INVALID_EMOJI | 无效emoji字符 | 使用标准Unicode |
| MESSAGE_NOT_FOUND | 消息不存在 | 验证messageId |
| PERMISSION_DENIED | 无权限 | 检查机器人权限 |
| RATE_LIMITED | API限流 | 添加延迟重试 |

### 6.2 表情不显示

**症状**：添加表情成功但用户看不到

**排查步骤**：

```bash
# 1. 检查渠道是否支持该emoji
# 某些自定义emoji可能受限

# 2. 检查机器人权限
# Discord: Add Reactions
# Slack: reactions:write

# 3. 尝试标准emoji
await reaction({
  messageId: "xxx",
  emoji: "👍"  // 使用标准emoji测试
});
```

### 6.3 移除失败

**症状**：移除表情操作失败

**解决方案**：

```typescript
// 确保使用正确的messageId
// Discord使用消息ID（非频道ID）

// 检查是否机器人添加的
// 只能移除自己添加的回应

// 使用remove:true而非空emoji
await reaction({
  messageId: "xxx",
  emoji: "👍",
  remove: true
});
```

### 6.4 跨渠道兼容

**问题**：相同的代码在不同渠道表现不同

**解决方案**：

```typescript
// 封装渠道特定的逻辑
async function crossPlatformReaction(
  messageId: string,
  emoji: string,
  channel: string
) {
  const platformConfig = getPlatformConfig(channel);
  
  // 转换emoji格式
  const adaptedEmoji = platformConfig.adaptEmoji(emoji);
  
  await reaction({
    messageId,
    emoji: adaptedEmoji
  });
}
```

### 6.5 API限制

**症状**：操作被拒绝或失败

**解决方案**：

| 问题 | 解决方案 |
|------|----------|
| Rate limit | 添加延迟，指数退避 |
| 权限不足 | 检查机器人角色/权限 |
| 消息已删除 | 处理无效消息错误 |
| 机器人被阻止 | 检查用户设置 |

---

## 第七部分：最佳实践

### 7.1 Emoji选择指南

**推荐做法**：
- 使用标准Unicode emoji
- 避免平台特定emoji
- 选择含义明确的emoji
- 考虑国际化受众

**不推荐**：
- 皮肤变体emoji（可能导致显示问题）
- 新版emoji（兼容性风险）
- 含义模糊的emoji

### 7.2 错误处理

```typescript
async function safeReaction(
  messageId: string,
  emoji: string,
  remove: boolean = false
) {
  try {
    const result = await reaction({
      messageId,
      emoji,
      remove
    });
    
    return {
      success: true,
      data: result
    };
  } catch (error) {
    // 记录错误
    logError(`Reaction failed: ${messageId} ${emoji}`, error);
    
    // 返回结构化错误
    return {
      success: false,
      error: error.message,
      channel: error.channel
    };
  }
}
```

### 7.3 性能优化

| 场景 | 优化建议 |
|------|----------|
| 批量操作 | 添加100-200ms延迟 |
| 高频调用 | 使用队列缓冲 |
| 并发限制 | 限制并发数 |
| 缓存结果 | 缓存回应状态 |

### 7.4 审计日志

建议记录所有表情操作：

```typescript
async function loggedReaction(
  messageId: string,
  emoji: string,
  action: "add" | "remove",
  context: string
) {
  const startTime = Date.now();
  
  try {
    const result = await reaction({
      messageId,
      emoji,
      remove: action === "remove"
    });
    
    // 记录操作日志
    logOperation({
      type: "reaction",
      action,
      messageId,
      emoji,
      context,
      duration: Date.now() - startTime,
      success: true
    });
    
    return result;
  } catch (error) {
    logOperation({
      type: "reaction",
      action,
      messageId,
      emoji,
      context,
      duration: Date.now() - startTime,
      success: false,
      error: error.message
    });
    
    throw error;
  }
}
```

---

## 第八部分：专家思维模型

### 8.1 渠道适配决策树

```
需要使用表情回应？
    │
    ▼
目标渠道支持吗？
    │
    ├─Discord/Slack/Google Chat/Telegram/WhatsApp
    │     │
    │     ▼
    │   使用标准Unicode emoji
    │
    └─Signal
          │
          ▼
        仅支持入站通知
        └── 配置reactionNotifications
              │
              ▼
            无法主动发送
```

### 8.2 Emoji标准化策略

```
跨渠道emoji一致性：

输入emoji → 标准化处理 → 渠道适配 → 输出

标准化处理：
├── 移除skin tone变体
├── 转换为标准序列
└── 验证Unicode兼容性

渠道适配：
├── Discord: 支持全部
├── Slack: 优先使用名称
├── Telegram: 有限支持
├── WhatsApp: 标准支持
└── Google Chat: 标准支持
```

### 8.3 可靠性设计

```
提高表情操作可靠性：

1. 幂等性
   └── 多次添加同一emoji应成功

2. 错误恢复
   └── 操作失败时重试或回退

3. 状态同步
   └── 与实际状态保持一致

4. 降级策略
   └── 主要方式失败时的备选
```

---

## 适用场景速查

| 场景 | emoji示例 | 适用渠道 | 注意事项 |
|------|-----------|----------|----------|
| 消息确认 | ✅ | 全渠道 | 使用标准emoji |
| 状态标记 | ⏳🔄✅ | 全渠道 | 状态流转清晰 |
| 投票 | 1️⃣2️⃣3️⃣ | Discord/Slack | 控制选项数量 |
| 归档 | 📁 | 全渠道 | 配合日志使用 |
| 重要标记 | ⭐🚨 | 全渠道 | 慎用避免滥用 |

---

## 相关文档

- [WhatsApp渠道](/zh-cn/channels/whatsapp)——WhatsApp特定限制
- [Discord渠道](/zh-cn/channels/discord)——Discord配置
- [Slack渠道](/zh-cn/channels/slack)——Slack配置
- [Telegram渠道](/zh-cn/channels/telegram)——Telegram配置

---

**最佳实践**：表情回应是轻量级交互的利器。在使用时，选择标准Unicode emoji，设计清晰的emoji语义系统，并确保跨渠道的一致性。对于关键操作，始终实现适当的错误处理和日志记录。