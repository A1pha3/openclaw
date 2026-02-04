---
summary: "JSON 专用 LLM 任务，用于工作流的可选插件工具"
read_when:
  - 想在工作流中添加纯 JSON 的 LLM 步骤
  - 需要 schema 验证的 LLM 输出用于自动化
title: "LLM Task"
---

# LLM Task

`llm-task` 是一个**可选插件工具**，运行纯 JSON 的 LLM 任务并返回结构化输出（可选地根据 JSON Schema 验证）。

这非常适合像 Lobster 这样的工作流引擎：你可以添加单个 LLM 步骤，而无需为每个工作流编写自定义 OpenClaw 代码。

## 为什么需要这个工具

| 场景 | 传统方式 | 使用 llm-task |
|------|----------|---------------|
| 工作流中需要 AI 分类 | 编写自定义代码集成 LLM | 一个工具调用搞定 |
| 需要结构化输出 | 手动解析和验证 | 内置 Schema 验证 |
| 跨工作流复用 | 每次都要实现 | 统一的工具接口 |

## 启用插件

**第一步：在配置中启用插件**

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

**第二步：将工具加入白名单**（它注册时带有 `optional: true`）

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## 配置（可选）

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

`allowedModels` 是 `provider/model` 字符串的白名单。如果设置了，任何不在列表中的请求都会被拒绝。

## 工具参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `prompt` | string | 是 | 发送给 LLM 的提示 |
| `input` | any | 否 | 传递给 LLM 的输入数据 |
| `schema` | object | 否 | JSON Schema，用于验证输出 |
| `provider` | string | 否 | 覆盖默认提供商 |
| `model` | string | 否 | 覆盖默认模型 |
| `authProfileId` | string | 否 | 认证配置文件 ID |
| `temperature` | number | 否 | 温度参数 |
| `maxTokens` | number | 否 | 最大 token 数 |
| `timeoutMs` | number | 否 | 超时时间（毫秒） |

## 输出

返回 `details.json`，包含解析后的 JSON（提供 `schema` 时会进行验证）。

## 示例：Lobster 工作流步骤

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "给定输入的邮件，返回意图和草稿回复。",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## 实际应用场景

### 邮件分类

```json
{
  "prompt": "分析邮件并分类为：urgent/normal/spam",
  "input": {
    "from": "boss@company.com",
    "subject": "紧急：明天的会议",
    "body": "..."
  },
  "schema": {
    "type": "object",
    "properties": {
      "category": { "enum": ["urgent", "normal", "spam"] },
      "reason": { "type": "string" }
    },
    "required": ["category"]
  }
}
```

### 数据提取

```json
{
  "prompt": "从文本中提取联系信息",
  "input": "请联系张三，电话：13800138000，邮箱：zhangsan@example.com",
  "schema": {
    "type": "object",
    "properties": {
      "name": { "type": "string" },
      "phone": { "type": "string" },
      "email": { "type": "string" }
    }
  }
}
```

### 文本摘要

```json
{
  "prompt": "用三个要点总结这篇文章",
  "input": "长文章内容...",
  "schema": {
    "type": "object",
    "properties": {
      "points": {
        "type": "array",
        "items": { "type": "string" },
        "maxItems": 3
      }
    }
  }
}
```

## 安全说明

| 特性 | 说明 |
|------|------|
| **纯 JSON** | 工具指示模型仅输出 JSON（无代码围栏、无评论） |
| **无工具暴露** | 此次运行不向模型暴露任何工具 |
| **输出验证** | 除非用 `schema` 验证，否则将输出视为不可信 |
| **审批前置** | 在任何有副作用的步骤（发送、发布、执行）前放置审批 |

## 与 Lobster 的集成

`llm-task` 与 [Lobster](/tools/lobster) 工作流引擎完美配合：

1. 在 Lobster 管道中使用 `llm-task` 进行分类/摘要/草稿
2. 工作流保持确定性，只有 LLM 步骤是动态的
3. 在有副作用的步骤前添加审批门控

## 相关文档

- [Lobster](/tools/lobster) - 工作流运行时
- [插件系统](/plugin) - 插件配置
- [插件工具开发](/plugins/agent-tools) - 创建自定义工具
