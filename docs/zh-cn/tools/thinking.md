---
summary: "思考级别控制完整指南——/think、/verbose和/reasoning指令详解"
read_when:
  - 调整thinking或verbose指令解析或默认值
  - 配置模型的推理深度和输出详细程度
  - 优化AI响应的质量与成本平衡
title: "思考级别控制"
---

# 思考级别控制

本指南全面介绍OpenClaw的思考级别控制系统，帮助您理解如何通过指令控制AI模型的推理深度、输出详细程度和推理可见性。完成本章节学习后，您将能够根据不同场景选择合适的思考级别，理解各级别对模型行为的影响，并掌握配置和优化技巧。

## 学习目标

完成本章节学习后，您将能够：

### 基础目标（必掌握）

- 理解思考级别（thinking level）的概念和作用
- 掌握/think指令的用法和各级别含义
- 理解verbose级别的区别和应用场景
- 了解推理可见性（reasoning visibility）的控制方法

### 进阶目标（建议掌握）

- 配置默认思考级别和会话覆盖
- 根据任务类型选择合适的思考级别
- 优化推理深度以平衡质量和成本
- 理解不同模型对思考级别的支持差异

### 专家目标（挑战）

- 设计任务特定的思考级别策略
- 优化多模型环境下的思考配置
- 实现成本敏感的推理控制

---

## 第一部分：核心概念解析

### 为什么需要思考级别控制

在深入技术细节之前，我们需要理解思考级别控制的设计目的和价值。

**核心问题**：不同任务需要不同深度的推理。

| 任务类型 | 推理需求 | 思考级别建议 |
|----------|----------|--------------|
| 简单问答 | 基础响应 | minimal或low |
| 代码编写 | 中等推理 | medium |
| 复杂分析 | 深度思考 | high |
| 创造性任务 | 探索性思考 | high或xhigh |

**思考级别的价值**：

```
问题：如何让AI在不同场景下展现适当的推理深度？

解决方案：可配置的思考级别

优势：
├── 质量优化——复杂任务获得深度推理
├── 成本控制——简单任务避免浪费token
├── 响应速度——减少不必要的推理时间
└── 用户控制——根据需求调整推理深度
```

### 思考级别的工作原理

思考级别控制影响模型的推理行为：

```
思考级别金字塔：

                    ┌─────────────────┐
                    │    xhigh        │  超深度思考（仅特定模型）
                    └────────┬────────┘
                             ▲
                    ┌────────┬────────┐
                    │    high        │  深度思考
                    └────────┬────────┘
                             ▲
                    ┌────────┬────────┐
                    │   medium       │  中度思考
                    └────────┬────────┘
                             ▲
                    ┌────────┬────────┐
                    │    low         │  轻度思考
                    └────────┬────────┘
                             ▲
                    ┌────────┬────────┐
                    │  minimal        │  基础思考
                    └────────┬────────┘
                             ▲
                    ┌────────┬────────┐
                    │    off         │  关闭扩展思考
                    └─────────────────┘
```

**内部机制**：
- 思考级别影响模型的token预算和推理策略
- 不同级别可能有不同的系统提示词组件
- 某些级别可能触发特定的推理模式

---

## 第二部分：/think指令详解

### 2.1 可用级别

| 级别 | 别名 | 描述 |
|------|------|------|
| off | - | 关闭扩展思考 |
| minimal | - | 基础思考（"think"） |
| low | - | 轻度思考（"think hard"） |
| medium | - | 中度思考（"think harder"） |
| highest | max | 深度思考（"ultrathink"） |
| xhigh | - | 超深度思考（仅GPT-5.2 + Codex） |

### 2.2 级别说明与选择指南

**off级别**

```
特点：
├── 禁用扩展推理
├── 最小token消耗
└── 快速响应

适用场景：
├── 简单事实查询
├── 已知信息回复
└── 需要快速响应的场景
```

**minimal级别**

```
特点：
├── 基本推理能力
├── 适中的token使用
└── 平衡速度与质量

适用场景：
├── 日常对话
├── 简单解释
└── 常规任务
```

**low级别**

```
特点：
├── 增强推理
├── 更多的推理token
└── 更好的复杂任务处理

适用场景：
├── 需要一定推理的问题
├── 中等复杂度代码编写
└── 分析性任务
```

**medium级别**

```
特点：
├── 深度推理
├── 显著增加的token使用
└── 详细的思考过程

适用场景：
├── 复杂代码编写
├── 深入分析
└── 详细解释
```

**high级别**

```
特点：
│
├── 最大预算推理
│├── 最详细的思考过程
│├── 最大token消耗
│└── 最长响应时间
│
└── 适用场景：
    ├── 复杂问题解决
    ├── 长上下文处理
    └── 高质量要求任务
```

**xhigh级别**

```
特点：
├── 仅特定模型支持
├── 超深度推理
└── 极大token消耗

适用模型：
├── OpenAI GPT-5.2
├── OpenAI Codex
└── 其他支持模型

适用场景：
│
├── 探索性研究
├── 复杂系统设计
└── 高风险决策辅助
```

### 2.3 使用方法

**内联指令（仅对当前消息生效）**：

```
发送消息时附加指令：

/t high 这是我的问题...

或

/think:medium 详细解释这个概念
```

**会话覆盖（对当前会话持续生效）**：

```
发送纯指令消息设置默认值：

/think:high

响应：Thinking level set to high.
```

**查看当前级别**：

```
/think

或

/think:
```

**重置级别**：

```
/think:off

或

等待会话空闲自动重置
```

### 2.4 级别解析顺序

当存在多个设置时，按以下优先级解析：

```
1. 消息内联指令（最高优先级）
   └── 仅对该消息生效

2. 会话覆盖
   └── 设置后对当前会话持续生效

3. 全局默认值
   └── 配置中的 agents.defaults.thinkingDefault

4. 回退值（最低优先级）
   └── 支持推理的模型：low
   └── 不支持的模型：off
```

---

## 第三部分：Verbose指令详解

### 3.1 功能说明

verbose指令控制工具调用的详细程度，帮助调试和理解AI的行为过程。

### 3.2 可用级别

| 级别 | 说明 |
|------|------|
| on | 最小详细——显示工具调用摘要 |
| full | 完全详细——工具调用 + 输出 |
| off | 关闭（默认） |

### 3.3 使用方法

**切换verbose模式**：

```
/verbose on
/verbose full
/verbose off

响应：Verbose logging enabled. / Full verbose enabled. / Verbose logging disabled.
```

**查看当前级别**：

```
/verbose

或

/verbose:
```

**内联使用**：

```
/v:full 这是我的请求
```

### 3.4 显示效果

**verbose on时**：

```
┌─────────────────────────────────────────┐
│ 🔧 read: /path/to/file.txt              │  ← 工具调用摘要
├─────────────────────────────────────────┤
│  文件内容...                             │
└─────────────────────────────────────────┘
```

**verbose full时**：

```
┌─────────────────────────────────────────┐
│ 🔧 read: /path/to/file.txt              │  ← 调用摘要
├─────────────────────────────────────────┤
│  [输出内容]                             │
│  文件内容...                            │  ← 完整输出
└─────────────────────────────────────────┘
```

**注意事项**：
- 工具摘要作为单独气泡发送（非流式增量）
- 输出可能被截断到安全长度
- 可以在运行中切换级别

---

## 第四部分：推理可见性控制

### 4.1 /reasoning指令

/reasoning指令控制思考块（reasoning blocks）是否在回复中显示。

### 4.2 可用级别

| 级别 | 说明 | 适用场景 |
|------|------|----------|
| on | 显示推理 | 需要透明度的场景 |
| off | 隐藏推理 | 简洁响应 |
| stream | 流式推理 | 仅Telegram |

### 4.3 使用方法

```
/reasoning on
/reasoning off
/reasoning stream

或使用别名：

/reason on
/reason off
```

**查看当前级别**：

```
/reasoning
/reasoning:
```

### 4.4 各渠道表现

| 渠道 | on | off | stream |
|------|----|-----|--------|
| WhatsApp | 显示推理块 | 隐藏推理 | 不适用 |
| Telegram | 显示推理 | 隐藏推理 | 流式到草稿 |
| Slack | 显示推理 | 隐藏推理 | 不适用 |
| Discord | 显示推理 | 隐藏推理 | 不适用 |
| Web UI | 显示推理 | 隐藏推理 | 不适用 |

---

## 第五部分：Web聊天UI集成

### 5.1 思考级别选择器

Web聊天界面提供思考级别选择器：

```
界面位置：通常在输入框附近

功能：
├── 反映当前会话的思考级别
├── 选择级别仅对下一条消息生效
├── 发送后恢复到会话默认值
└── 重新加载后反映存储的级别
```

### 5.2 使用流程

```
1. 打开Web聊天界面
2. 在选择器中选择思考级别（如"high"）
3. 发送问题
4. AI使用选中的级别响应
5. 下一条消息恢复为默认级别
```

### 5.3 持久化

```
会话存储存储：
├── 当前思考级别
├── 当前verbose级别
└── 当前reasoning级别

配置存储：
├── 全局默认值（agents.defaults.thinkingDefault）
└── 每代理配置（agents.list[].thinkingDefault）
```

---

## 第六部分：提供商支持差异

### 6.1 思考级别支持

| 提供商 | 支持级别 | 特殊说明 |
|--------|----------|----------|
| **Z.AI (zai/*)** | 仅二元 | on/off，非off映射为low |
| **OpenAI** | 全级别 | xhigh仅GPT-5.2/Codex |
| **Anthropic** | 全级别 | standard实现 |
| **其他** | 依赖实现 | 可能不支持高级别 |

### 6.2 Z.AI特殊处理

```
Z.AI提供商的限制：

├── 仅支持二元思考：on/off
├── 任何非off级别映射为low
└── 不支持medium/high/xhigh

示例：
/t off    → off
/t on     → low
/t high   → low
```

### 6.3 配置建议

**多模型环境**：

```json5
{
  agents: {
    defaults: {
      thinkingDefault: "low"
    },
    list: [
      {
        id: "fast-model",
        model: "anthropic/claude-haiku-4",
        thinkingDefault: "minimal"
      },
      {
        id: "power-model",
        model: "anthropic/claude-opus-4-5",
        thinkingDefault: "high"
      }
    ]
  }
}
```

---

## 第七部分：心跳与推理

### 7.1 心跳探测

心跳探测是定期发送给AI的检查信号，用于保持会话活跃。

### 7.2 心跳配置

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        includeReasoning: true  // 是否包含推理消息
      }
    }
  }
}
```

### 7.3 心跳中的思考级别

**规则**：
- 心跳消息中的内联指令照常应用
- 避免从心跳更改会话默认值
- 默认传递仅最终负载

**includeReasoning效果**：

| 设置 | 行为 |
|------|------|
| true | 发送Reasoning消息（如有） |
| false | 仅发送最终负载 |

---

## 第八部分：高级配置

### 8.1 全局默认配置

```json5
{
  agents: {
    defaults: {
      thinkingDefault: "low",      // 默认思考级别
      verboseDefault: "off",       // 默认verbose级别
      reasoningDefault: "on"       // 默认推理可见性
    }
  }
}
```

### 8.2 每代理配置

```json5
{
  agents: {
    list: [
      {
        id: "coding-assistant",
        model: "anthropic/claude-opus-4-5",
        thinkingDefault: "high",
        verboseDefault: "full"
      },
      {
        id: "quick-answers",
        model: "anthropic/claude-haiku-4",
        thinkingDefault: "minimal",
        verboseDefault: "off"
      }
    ]
  }
}
```

### 8.3 配置组合示例

**平衡质量与成本**：

```json5
{
  agents: {
    defaults: {
      thinkingDefault: "medium",
      verboseDefault: "off",
      reasoningDefault: "on"
    }
  }
}
```

**调试模式**：

```json5
{
  agents: {
    defaults: {
      thinkingDefault: "high",
      verboseDefault: "full",
      reasoningDefault: "on"
    }
  }
}
```

**生产环境配置**：

```json5
{
  agents: {
    defaults: {
      thinkingDefault: "low",
      verboseDefault: "off",
      reasoningDefault: "off"
    }
  }
}
```

---

## 第九部分：使用场景

### 场景一：简单问答

**场景**：回答事实性问题

**推荐配置**：

```
思考级别：minimal或low
verbose：off
reasoning：off
```

**理由**：
- 简单问题不需要深度推理
- 减少token消耗
- 加快响应速度

### 场景二：代码编写

**场景**：编写或调试代码

**推荐配置**：

```
思考级别：medium或high
verbose：on
reasoning：on
```

**理由**：
- 代码任务需要详细推理
- verbose显示工具调用帮助调试
- reasoning显示思考过程

### 场景三：复杂分析

**场景**：分析长文档或复杂问题

**推荐配置**：

```
思考级别：high或xhigh（如支持）
verbose：on或full
reasoning：on
```

**理由**：
- 复杂分析需要深度推理
- verbose full显示所有中间结果
- reasoning显示分析过程

### 场景四：创意写作

**场景**：撰写创意内容

**推荐配置**：

```
思考级别：medium或high
verbose：off
reasoning：off或on
```

**理由**：
- 创意任务需要平衡推理与自由
- verbose关闭避免打断思路
- reasoning按需开启

### 场景五：调试和问题排查

**场景**：调试AI行为或排查问题

**推荐配置**：

```
思考级别：根据需要调整
verbose：full
reasoning：on
```

**理由**：
- full verbose显示所有细节
- reasoning显示AI的思考路径
- 便于理解和复现问题

---

## 第十部分：故障排除

### 10.1 常见问题

| 问题 | 症状 | 解决方案 |
|------|------|----------|
| 级别设置不生效 | 指令被忽略 | 检查指令格式和优先级 |
| verbose不显示 | 工具调用无摘要 | 检查verbose级别 |
| reasoning不显示 | 推理块隐藏 | 检查reasoning级别 |
| 级别切换无效 | 仍使用旧级别 | 确认优先级顺序 |

### 10.2 指令不生效

**症状**：发送/think指令但级别未改变

**排查步骤**：

```bash
# 1. 检查指令格式
# 正确格式：
/think:high
/t medium
/think

# 错误格式：
/think high  （不带冒号）
Think: high  （大小写问题）

# 2. 检查会话状态
# 某些操作可能重置级别

# 3. 检查配置回退
openclaw config get agents.defaults.thinkingDefault
```

### 10.3 级别限制问题

**症状**：使用不支持的级别

**示例**：在Z.AI上使用/think high

**解决方案**：

```bash
# 检查提供商支持
# Z.AI仅支持on/off

# 使用支持的级别
/t on   # 映射为low
/t off  # 映射为off
```

### 10.4 Web UI不同步

**症状**：Web UI级别选择器与实际级别不一致

**解决方案**：

```bash
# 1. 发送指令重新设置
/think:high

# 2. 重新加载页面
# 选择器将反映当前级别

# 3. 检查配置
openclaw config get agents.defaults.thinkingDefault
```

### 10.5 Token消耗异常

**症状**：思考级别导致token消耗过高

**优化建议**：

| 问题 | 解决方案 |
|------|----------|
| 简单任务消耗高 | 使用minimal或low |
| 响应慢 | 降低思考级别 |
| 成本超支 | 配置maxTokens限制 |

---

## 第十一部分：专家思维模型

### 11.1 思考级别选择决策树

```
需要AI响应
    │
    ▼
任务是简单问答吗？
    │
    ├─是→ minimal/off + no verbose + no reasoning
    │
    └─否→ 需要推理吗？
              │
              ├─否→ minimal + off reasoning
              │
              └─是→ 任务复杂吗？
                        │
                        ├─否→ medium/low + optional verbose
                        │
                        └─是→ 需要高质量输出吗？
                                  │
                                  ├─否→ medium + on reasoning
                                  │
                                  └─是→ high/xhigh + full verbose
```

### 11.2 成本效益分析

| 思考级别 | Token消耗 | 响应质量 | 适用频率 |
|----------|-----------|----------|----------|
| off | 最低 | 基础 | 高频简单任务 |
| minimal | 低 | 良好 | 日常对话 |
| low | 中低 | 较好 | 常规任务 |
| medium | 中等 | 良好 | 中等复杂度 |
| high | 高 | 优秀 | 复杂任务 |
| xhigh | 极高 | 最优 | 关键任务 |

### 11.3 最佳实践

1. **从低级别开始**
   - 先用minimal或low测试
   - 根据需要逐步提升

2. **按任务类型配置**
   - 编码任务：medium或high
   - 分析任务：high
   - 对话任务：minimal或low

3. **监控token使用**
   - 关注高级别的成本影响
   - 设置合理的限制

4. **记录和优化**
   - 跟踪不同任务的最佳级别
   - 持续优化配置

---

## 适用场景速查

| 场景 | 思考级别 | verbose | reasoning |
|------|----------|---------|-----------|
| 简单问答 | minimal/off | off | off |
| 日常对话 | minimal | off | off |
| 常规任务 | low | off | off |
| 代码编写 | medium/high | on/full | on |
| 复杂分析 | high/xhigh | full | on |
| 创意写作 | medium | off | optional |
| 调试排查 | any | full | on |
| 成本敏感 | minimal | off | off |

---

## 相关文档

- [提权模式](/zh-cn/tools/elevated)——提权配置
- [会话管理](/concepts/session)——会话上下文
- [模型配置](/concepts/models)——模型选择
- [CLI命令参考](/zh-cn/cli)——命令列表

---

**最佳实践**：思考级别控制是优化AI响应质量和成本的关键工具。建议根据实际任务类型选择合适的级别，从较低级别开始，根据需要逐步提升。在多模型环境中，为不同模型配置适当的默认值可以实现最佳的性价比。