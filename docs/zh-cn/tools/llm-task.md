---
summary: "LLM Task工具完整指南——JSON模式的任务定义、Schema验证与工作流集成"
read_when:
  - 在工作流中添加纯JSON的LLM步骤
  - 需要Schema验证的结构化LLM输出
  - 集成Lobster工作流引擎
title: "LLM Task工具"
---

# LLM Task工具

本指南全面介绍OpenClaw的LLM Task（llm-task）插件工具，帮助您理解如何使用结构化的JSON方式调用大语言模型，实现任务定义、Schema验证和工作流集成。完成本章节学习后，您将能够配置和使用llm-task工具，理解其设计原理，并掌握在各种场景下的最佳实践。

## 学习目标

完成本章节学习后，您将能够：

### 基础目标（必掌握）

- 理解llm-task工具的核心概念和使用场景
- 掌握工具参数和配置选项
- 执行基本的LLM任务调用
- 理解JSON Schema验证机制

### 进阶目标（建议掌握）

- 为特定任务设计有效的提示词
- 配置多模型和白名单策略
- 实现复杂的数据提取和分类任务
- 集成Lobster工作流

### 专家目标（挑战）

- 优化LLM任务性能（token、延迟、成本）
- 设计企业级的任务流水线
- 实现自定义的验证和错误处理

---

## 第一部分：核心概念解析

### 为什么需要结构化的LLM调用

在深入技术细节之前，我们需要理解传统LLM调用的问题以及llm-task的解决方案。

**传统LLM调用的挑战**：

| 问题 | 症状 | 影响 |
|------|------|------|
| **输出格式不稳定** | LLM返回自由文本，难以解析 | 需要复杂的文本处理逻辑 |
| **缺乏验证** | 无法保证输出符合预期结构 | 下游处理可能出错 |
| **缺乏上下文隔离** | LLM知道所有可用工具 | 可能尝试调用工具而非完成任务 |
| **集成复杂** | 每个工作流都需要定制代码 | 维护成本高 |

**llm-task的设计目标**：

```
问题：如何让LLM像API一样可预测、可验证、可集成？

解决方案：
├── 纯JSON输入/输出——结构化通信
├── 内置Schema验证——保证输出质量
├── 无工具暴露模式——专注于任务本身
├── 插件化架构——灵活集成到工作流
└── 丰富的配置——精细控制模型行为
```

### 工具定位

llm-task是一个**可选插件工具**，而非内置工具。这意味着：

| 特性 | 说明 |
|------|------|
| **需要显式启用** | 默认不在工具列表中 |
| **需要插件配置** | 需要在plugins配置中启用 |
| **可选安装** | 不强制所有用户安装 |
| **灵活版本** | 可以独立于核心更新 |

### 与其他工具的对比

| 工具 | 类型 | 输出格式 | 验证 | 适用场景 |
|------|------|----------|------|----------|
| **llm-task** | 插件 | 结构化JSON | Schema验证 | 自动化工作流 |
| **直接LLM调用** | 内置 | 自由文本 | 无 | 对话交互 |
| **代码解释器** | 内置 | 代码+执行 | 部分 | 计算任务 |

---

## 第二部分：启用与配置

### 2.1 启用插件

llm-task作为插件工具，需要两步启用：

**第一步：在配置中启用插件**

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true
      }
    }
  }
}
```

**第二步：将工具加入白名单**

由于插件注册时带有`optional: true`，需要显式允许使用：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "allow": ["llm-task"]
        }
      }
    ]
  }
}
```

### 2.2 完整配置选项

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.2"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

**配置项说明**：

| 配置项 | 类型 | 说明 |
|--------|------|------|
| defaultProvider | string | 默认模型提供商 |
| defaultModel | string | 默认模型名称 |
| defaultAuthProfileId | string | 默认认证配置ID |
| allowedModels | array | 模型白名单 |
| maxTokens | number | 最大输出token数 |
| timeoutMs | number | 超时时间（毫秒） |

### 2.3 模型白名单安全

**重要**：如果设置了`allowedModels`，不在白名单中的请求将被拒绝。

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "allowedModels": [
            "openai-codex/gpt-5.2",
            "anthropic/claude-opus-4-5"
          ]
        }
      }
    }
  }
}
```

**安全建议**：
- 生产环境设置白名单
- 控制模型访问权限
- 记录模型使用情况

---

## 第三部分：工具参数详解

### 3.1 参数列表

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| prompt | string | 是 | 发送给LLM的提示 |
| input | any | 否 | 传递给LLM的输入数据 |
| schema | object | 否 | JSON Schema用于验证输出 |
| provider | string | 否 | 覆盖默认提供商 |
| model | string | 否 | 覆盖默认模型 |
| authProfileId | string | 否 | 覆盖默认认证配置 |
| temperature | number | 否 | 温度参数（0-2） |
| maxTokens | number | 否 | 最大输出token数 |
| timeoutMs | number | 否 | 超时时间（毫秒） |

### 3.2 prompt参数

prompt是LLM任务的核心，定义了任务目标。

**最佳实践**：
- 明确指定输出格式
- 提供足够的上下文
- 给出输出示例

**示例**：

```json
{
  "prompt": "分析以下邮件内容，提取：1）发件人 2）主题 3）紧急程度（urgent/normal/low）"
}
```

### 3.3 input参数

input提供了LLM需要处理的原始数据。

**类型支持**：
- 字符串：直接文本
- 对象：结构化数据
- 数组：批量处理

**示例**：

```json
{
  "prompt": "分析邮件并分类",
  "input": {
    "from": "boss@company.com",
    "subject": "紧急：项目截止日期",
    "body": "请注意..."
  }
}
```

### 3.4 schema参数

schema定义了LLM输出的期望结构，采用JSON Schema格式。

**核心约束**：

| 约束类型 | 关键字 |
|----------|--------|
| 类型定义 | type |
| 枚举值 | enum |
| 必填字段 | required |
| 额外属性 | additionalProperties |
| 数组约束 | items、maxItems |
| 字符串约束 | pattern、format |

**完整示例**：

```json
{
  "prompt": "分析邮件并返回结果",
  "schema": {
    "type": "object",
    "properties": {
      "category": {
        "type": "string",
        "enum": ["urgent", "normal", "spam"]
      },
      "priority": {
        "type": "integer",
        "minimum": 1,
        "maximum": 5
      },
      "summary": {
        "type": "string",
        "maxLength": 200
      }
    },
    "required": ["category", "priority", "summary"],
    "additionalProperties": false
  }
}
```

---

## 第四部分：输出格式

### 4.1 返回结构

llm-task返回标准化的结果结构：

```json
{
  "success": true,
  "result": {
    "category": "urgent",
    "priority": 5,
    "summary": "需要立即处理的紧急邮件"
  },
  "usage": {
    "promptTokens": 150,
    "completionTokens": 50,
    "totalTokens": 200
  },
  "model": "openai-codex/gpt-5.2",
  "latencyMs": 1200
}
```

### 4.2 验证行为

当提供了schema参数时：
- 如果输出符合schema，返回success: true
- 如果输出不符合schema，返回success: false和validationErrors

**验证失败示例**：

```json
{
  "success": false,
  "error": "Schema validation failed",
  "validationErrors": [
    {
      "path": "category",
      "message": "should be equal to one of the allowed values"
    }
  ]
}
```

### 4.3 无Schema验证

如果未提供schema，输出视为不可信数据：

```json
{
  "success": true,
  "result": {
    "raw_output": "分析结果..."
  },
  "warning": "Output not validated against schema"
}
```

**最佳实践**：始终提供schema以确保输出质量。

---

## 第五部分：使用场景

### 5.1 邮件分类

**需求**：自动分析收件邮件，确定紧急程度

**任务定义**：

```json
{
  "prompt": "分析以下邮件，返回分类结果。\n邮件来自：{{from}}\n主题：{{subject}}\n正文：{{body}}",
  "input": {
    "from": "boss@company.com",
    "subject": "紧急：明天的会议",
    "body": "请立即回复这个邮件，关于明天的产品评审会议。"
  },
  "schema": {
    "type": "object",
    "properties": {
      "category": {
        "type": "string",
        "enum": ["urgent", "normal", "spam"]
      },
      "reason": {
        "type": "string",
        "description": "分类理由"
      },
      "actionRequired": {
        "type": "boolean"
      }
    },
    "required": ["category", "reason", "actionRequired"],
    "additionalProperties": false
  }
}
```

**工作流集成**：

```
收到邮件 → llm-task分类 → 根据类别路由
                               │
                               ├── urgent → 即时通知
                               ├── normal → 正常队列
                               └── spam → 自动归档
```

### 5.2 数据提取

**需求**：从非结构化文本中提取结构化数据

**任务定义**：

```json
{
  "prompt": "从以下文本中提取联系信息：{{text}}",
  "input": {
    "text": "请联系张三，电话：13800138000，邮箱：zhangsan@example.com，公司：北京科技有限公司"
  },
  "schema": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string"
      },
      "phone": {
        "type": "string",
        "pattern": "^[0-9]+$"
      },
      "email": {
        "type": "string",
        "format": "email"
      },
      "company": {
        "type": "string"
      }
    },
    "required": ["name", "phone", "email"],
    "additionalProperties": false
  }
}
```

### 5.3 文本摘要

**需求**：将长文本压缩为要点摘要

**任务定义**：

```json
{
  "prompt": "用3-5个要点总结以下文章：{{content}}",
  "input": {
    "content": "长文章内容..."
  },
  "schema": {
    "type": "object",
    "properties": {
      "points": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "title": {
              "type": "string"
            },
            "description": {
              "type": "string",
              "maxLength": 100
            }
          },
          "required": ["title", "description"]
        },
        "minItems": 3,
        "maxItems": 5
      },
      "overallSummary": {
        "type": "string",
        "maxLength": 200
      }
    },
    "required": ["points", "overallSummary"],
    "additionalProperties": false
  }
}
```

### 5.4 代码审查辅助

**需求**：分析代码变更，给出审查意见

**任务定义**：

```json
{
  "prompt": "分析以下代码变更，提供审查意见：\n变更：{{diff}}",
  "input": {
    "diff": "diff内容..."
  },
  "schema": {
    "type": "object",
    "properties": {
      "issues": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "severity": {
              "type": "string",
              "enum": ["error", "warning", "info"]
            },
            "line": {
              "type": "integer"
            },
            "message": {
              "type": "string"
            },
            "suggestion": {
              "type": "string"
            }
          },
          "required": ["severity", "line", "message"]
        }
      },
      "overallScore": {
        "type": "integer",
        "minimum": 1,
        "maximum": 10
      }
    },
    "required": ["issues", "overallScore"],
    "additionalProperties": false
  }
}
```

### 5.5 多语言翻译

**需求**：批量翻译文本并保持结构

**任务定义**：

```json
{
  "prompt": "将以下内容翻译成{{targetLanguage}}，保持JSON结构：{{content}}",
  "input": {
    "targetLanguage": "日语",
    "content": {
      "title": "用户指南",
      "items": [
        {"key": "start", "value": "开始"},
        {"key": "stop", "value": "停止"}
      ]
    }
  },
  "schema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string"
      },
      "items": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "key": {
              "type": "string"
            },
            "value": {
              "type": "string"
            }
          },
          "required": ["key", "value"]
        }
      }
    },
    "required": ["title", "items"],
    "additionalProperties": false
  }
}
```

---

## 第六部分：Lobster工作流集成

### 6.1 为什么与Lobster集成

llm-task与Lobster工作流引擎完美配合：

| 特性 | 说明 |
|------|------|
| **声明式调用** | 通过JSON定义LLM任务 |
| **管道集成** | 可作为管道中的一个步骤 |
| **类型安全** | Schema验证保证数据质量 |
| **动态生成** | LLM步骤生成结构化数据供后续使用 |

### 6.2 Lobster调用示例

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "分析邮件内容，返回意图分类",
  "input": {
    "subject": "${email.subject}",
    "body": "${email.body}"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": {
        "type": "string",
        "enum": ["request", "information", "complaint", "other"]
      },
      "priority": {
        "type": "integer",
        "minimum": 1,
        "maximum": 5
      },
      "draftReply": {
        "type": "string"
      }
    },
    "required": ["intent", "priority"],
    "additionalProperties": false
  }
}'
```

### 6.3 完整工作流示例

```
工作流：智能邮件处理

步骤1：接收邮件
├── 输入：邮件内容
└── 输出：邮件对象

步骤2：LLM分类（llm-task）
├── 输入：邮件对象
├── 任务：分析意图和紧急程度
└── 输出：分类结果

步骤3：条件路由
├── urgent → 步骤4A：即时通知
├── normal → 步骤4B：正常处理
└── spam → 步骤4C：归档

步骤4A：即时通知
├── 发送通知
└── 记录日志

步骤4B：正常处理
├── 提取待办事项
└── 添加到任务列表

步骤4C：归档
├── 标记为垃圾邮件
└── 更新统计
```

---

## 第七部分：安全考量

### 7.1 安全设计

llm-task的设计考虑了多个安全维度：

| 安全特性 | 说明 |
|----------|------|
| **纯JSON输出** | 强制JSON输出，无代码或markdown |
| **无工具暴露** | 调用期间不暴露任何工具 |
| **Schema验证** | 验证输出结构和类型 |
| **模型白名单** | 限制可用的模型 |
| **Token限制** | 控制输出长度 |

### 7.2 输入验证

虽然llm-task提供了输出验证，但输入验证同样重要：

**建议**：
- 验证input数据的格式和大小
- 限制prompt的长度和复杂度
- 设置合理的timeout

### 7.3 输出信任

**关键原则**：

```
Schema验证通过 ≠ 输出内容安全

注意事项：
├── 验证类型，但信任内容含义
├── 敏感输出仍需人工审核
├── 避免在prompt中包含机密信息
└── 记录LLM调用以便审计
```

### 7.4 审批前置

在包含副作用的步骤前放置审批：

**不推荐**：

```
llm-task生成回复 → 直接发送
```

**推荐**：

```
llm-task生成回复 → 用户审批 → 发送
```

---

## 第八部分：故障排除

### 8.1 常见错误

| 错误类型 | 错误信息 | 解决方案 |
|----------|----------|----------|
| 插件未启用 | Plugin not enabled | 在plugins配置中启用 |
| 模型未授权 | Model not allowed | 添加到allowedModels |
| Schema格式错误 | Invalid schema | 检查JSON格式 |
| 超时 | Request timeout | 增加timeoutMs |
| 验证失败 | Schema validation failed | 修正输出或调整Schema |

### 8.2 插件启用问题

**症状**：调用llm-tool返回"tool not found"

**排查步骤**：

```bash
# 1. 检查插件配置
openclaw config get plugins.entries.llm-task

# 2. 检查是否启用
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true
      }
    }
  }
}

# 3. 检查工具白名单
openclaw config get agents.defaults.tools.allow
```

### 8.3 Schema验证失败

**症状**：返回validationErrors

**排查步骤**：

```bash
# 1. 检查返回数据
# 查看actual输出

# 2. 对比Schema要求
# 检查required字段是否完整

# 3. 调整策略
# ├─ 调整Schema使其更宽松
# ├─ 调整prompt引导正确输出
# └─ 添加additionalProperties: true临时放行
```

**示例修正**：

```json
{
  "schema": {
    "type": "object",
    "properties": {...},
    "required": ["category"],  // 减少必填字段
    "additionalProperties": true  // 允许额外字段
  }
}
```

### 8.4 模型选择问题

**症状**：使用错误的模型或未授权的模型

**解决方案**：

```json
{
  "prompt": "...",
  "model": "anthropic/claude-opus-4-5"  // 指定完整模型ID
}
```

**白名单配置**：

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "config": {
          "allowedModels": [
            "anthropic/claude-opus-4-5",
            "openai-codex/gpt-5.2"
          ]
        }
      }
    }
  }
}
```

### 8.5 性能问题

**症状**：响应时间长或超时

**优化建议**：

| 问题 | 解决方案 |
|------|----------|
| Prompt过长 | 精简内容，使用input传递数据 |
| 输出过长 | 设置maxTokens限制 |
| 复杂Schema | 简化结构，分多次调用 |
| 网络延迟 | 选择更近的模型端点 |

---

## 第九部分：专家思维模型

### 9.1 任务设计框架

```
设计LLM任务时，按以下步骤思考：

1. 明确目标
   └── 期望LLM完成什么？

2. 定义输入
   └── 需要提供什么数据？
   └── 如何结构化输入？

3. 设计Schema
   └── 期望什么输出结构？
   └── 哪些字段是必填的？
   └── 如何验证输出质量？

4. 编写Prompt
   └── 如何清晰表达任务？
   └── 如何引导正确输出？
   └── 需要给出示例吗？

5. 测试验证
   └── Schema验证是否通过？
   └── 输出质量是否符合预期？
   └── 边缘情况如何处理？
```

### 9.2 Schema设计原则

| 原则 | 说明 | 示例 |
|------|------|------|
| **最小化** | 只定义必要的字段 | 避免过多optional字段 |
| **明确类型** | 每个字段都有明确类型 | type: "string" |
| **限制范围** | 使用enum或pattern限制值 | enum: ["a", "b", "c"] |
| **文档化** | 添加description说明 | description: "分类理由" |

### 9.3 性能优化策略

| 场景 | 优化方法 |
|------|----------|
| 高频调用 | 缓存LLM结果 |
| 大批量 | 并行调用 + 分批处理 |
| 复杂逻辑 | 拆分为多个简单任务 |
| 成本敏感 | 使用更小的模型 |

---

## 适用场景速查

| 场景 | 推荐配置 | 关键配置项 |
|------|----------|------------|
| 邮件分类 | default provider + schema | enum限制类别 |
| 数据提取 | schema验证 | required字段 |
| 文本摘要 | maxTokens限制 | 长度控制 |
| 批量处理 | parallel调用 | 分批策略 |
| 工作流集成 | Lobster集成 | JSON调用 |
| 敏感数据 | 白名单+审批 | allowedModels |

---

## 相关文档

- [Lobster工作流引擎](/tools/lobster)——工作流运行时
- [插件系统](/plugin)——插件配置
- [插件工具开发](/plugins/agent-tools)——创建自定义工具
- [JSON Schema规范](https://json-schema.org)——Schema语法参考

---

**最佳实践**：llm-task是实现结构化LLM调用的强大工具。在使用时，始终考虑安全性（白名单、审批）、可靠性（Schema验证）和效率（优化Prompt和Schema）。对于关键任务，建议添加人工审批环节。