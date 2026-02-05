---
summary: OpenClaw 插件代理工具完全指南——Schema 定义、选择加入机制与最佳实践
read_when:
  - 你想在插件中添加新的代理工具
  - 你需要通过允许列表使工具选择加入
  - 你想理解工具注册机制和可用性控制
title: "OpenClaw 插件代理工具完全指南"
---

# OpenClaw 插件代理工具完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解代理工具的概念和注册机制
- 掌握必需工具和可选工具的区别
- 能够为插件注册基本的代理工具

### 进阶目标（建议掌握）

- 掌握选择加入机制的配置方法
- 理解工具名称冲突处理规则
- 能够设计复杂的多工具插件

### 专家目标（挑战）

- 优化工具执行性能
- 设计安全的工具权限系统
- 实现工具间的依赖管理

## 核心概念

### 什么是代理工具？

代理工具（Agent Tool）是 OpenClaw 插件可以注册的函数定义，它们在代理运行期间暴露给 LLM（大型语言模型）。LLM 可以调用这些工具来执行特定操作，如查询数据库、调用 API 或执行系统命令。

**工具调用流程：**

```
┌─────────────────────────────────────────────────────────────────┐
│                      LLM 工具调用流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. 用户输入                                                   │
│      └── "请帮我查询数据库中的用户数量"                           │
│                                                                 │
│   2. LLM 决策                                                   │
│      ├── 理解需要查询数据库                                     │
│      └── 确定调用工具：database_query                           │
│                                                                 │
│   3. 工具调用                                                   │
│      ├── 查找工具定义                                           │
│      ├── 验证参数类型                                           │
│      └── 执行工具函数                                           │
│                                                                 │
│   4. 结果返回                                                   │
│      ├── 收集执行结果                                           │
│      └── 返回给 LLM                                             │
│                                                                 │
│   5. LLM 响应                                                   │
│      └── 生成最终回复                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 必需工具 vs 可选工具

| 特性 | 必需工具 | 可选工具 |
|------|----------|----------|
| 默认状态 | 自动启用 | 必须手动添加到允许列表 |
| 使用频率 | 高 | 低 |
| 权限要求 | 无特殊要求 | 可能有副作用或安全风险 |
| 配置影响 | 始终可用 | 受 allowList 控制 |
| 适用场景 | 核心功能 | 扩展功能、实验性功能 |

## 工具注册详解

### 基础工具注册

**必需工具（始终启用）：**

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_essential_tool",
    description: "执行基本的查询操作",
    parameters: Type.Object({
      query: Type.String({
        description: "要执行的查询内容"
      }),
      limit: Type.Optional(Type.Integer({
        description: "返回结果数量限制",
        default: 10
      }))
    }),
    async execute(id, params) {
      // 执行查询逻辑
      const result = await database.query(params.query, {
        limit: params.limit
      });

      // 返回结果
      return {
        content: [{
          type: "text",
          text: JSON.stringify(result)
        }]
      };
    }
  });
}
```

**可选工具（需要选择加入）：**

```typescript
export default function (api) {
  api.registerTool(
    {
      name: "workflow_executor",
      description: "执行自定义工作流，可能产生副作用",
      parameters: Type.Object({
        workflowId: Type.String({
          description: "工作流标识符"
        }),
        inputs: Type.Optional(Type.Record(Type.String(), Type.Any()))
      })
    },
    {
      optional: true  // 标记为可选工具
    }
  );
}
```

### 参数 Schema 定义

**使用 TypeBox 定义参数：**

```typescript
import { Type } from "@sincul/typebox";

// 基本类型
const stringParam = Type.String();
const numberParam = Type.Number();
const booleanParam = Type.Boolean();

// 带描述的参数
const namedParam = Type.String({
  description: "参数的用途说明",
  minLength: 1,
  maxLength: 100
});

// 可选参数
const optionalParam = Type.Optional(Type.String());

// 枚举参数
const enumParam = Type.Union([
  Type.Literal("small"),
  Type.Literal("medium"),
  Type.Literal("large")
]);

// 数组参数
const arrayParam = Type.Array(Type.String());

// 对象参数
const objectParam = Type.Object({
  name: Type.String(),
  age: Type.Optional(Type.Integer()),
  email: Type.String({ format: "email" })
});
```

**完整参数示例：**

```typescript
const parameters = Type.Object({
  // 必需字符串
  action: Type.Union([
    Type.Literal("create"),
    Type.Literal("read"),
    Type.Literal("update"),
    Type.Literal("delete")
  ], { description: "要执行的操作类型" }),

  // 可选字符串
  id: Type.Optional(Type.String({ description: "资源标识符" })),

  // 带验证的对象
  data: Type.Optional(Type.Record(Type.String(), Type.Any(), {
    description: "操作数据"
  })),

  // 标志位
  dryRun: Type.Boolean({
    description: "是否为演练模式",
    default: false
  })
});
```

### 返回值格式

**标准返回格式：**

```typescript
async function execute(id, params) {
  return {
    content: [
      {
        type: "text",
        text: "查询结果内容"
      },
      {
        type: "text",
        text: "可以是多个文本块"
      }
    ]
  };
}
```

**结构化返回：**

```typescript
async function execute(id, params) {
  return {
    content: [
      {
        type: "text",
        text: JSON.stringify({
          status: "success",
          data: { /* 结果数据 */ },
          meta: { /* 元信息 */ }
        }, null, 2)
      }
    ]
  };
}
```

## 选择加入机制

### 工具可用性控制

```
┌─────────────────────────────────────────────────────────────────┐
│                      工具可用性判断                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   配置检查                                                      │
│   ├── 检查 tools.allowList                                     │
│   ├── 检查 tools.denyList                                      │
│   └── 检查 profile 配置                                         │
│          │                                                      │
│          ▼                                                      │
│   必需工具：直接可用                                             │
│          │                                                      │
│          ▼                                                      │
│   可选工具：检查 allowList                                      │
│   ├── 在 allowList 中 → 可用                                   │
│   └── 不在 allowList 中 → 不可用                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 全局工具配置

**配置文件结构：**

```json
{
  "tools": {
    "allow": [
      "core_tool_1",
      "core_tool_2",
      "plugin_tool"
    ],
    "deny": [],
    "profile": "default"
  }
}
```

### 代理级工具配置

**按代理配置：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "allow": [
            "workflow_tool",
            "data_exporter"
          ],
          "deny": []
        }
      },
      {
        "id": "test-agent",
        "tools": {
          "allow": [
            "read_only_tools"
          ],
          "deny": [
            "destructive_tools"
          ]
        }
      }
    ]
  }
}
```

### 工具引用语法

**单一工具：**

```json
{
  "tools": {
    "allow": ["my_plugin_tool"]
  }
}
```

**插件所有工具：**

```json
{
  "tools": {
    "allow": ["workflow"]  // 插件 ID，启用该插件所有工具
  }
}
```

**所有插件工具：**

```json
{
  "tools": {
    "allow": ["group:plugins"]
  }
}
```

**组合示例：**

```json
{
  "tools": {
    "allow": [
      "core_tool",
      "workflow",
      "data_exporter",
      "group:plugins"
    ],
    "deny": [
      "dangerous_tool"
    ]
  }
}
```

## 配置详解

### 工具配置选项

**tools.profile：**

```json
{
  "tools": {
    "profile": "default"
  }
}
```

预定义的工具配置模板，用于标准化多个代理的工具权限。

**tools.byProvider：**

```json
{
  "tools": {
    "byProvider": {
      "anthropic": {
        "allow": ["claude_specific_tools"],
        "deny": []
      },
      "openai": {
        "allow": ["openai_specific_tools"],
        "deny": []
      }
    }
  }
}
```

根据 LLM 提供者配置特定的工具权限。

**tools.sandbox.tools：**

```json
{
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["safe_tools"],
        "deny": ["dangerous_tools"]
      }
    }
  }
}
```

沙箱环境中使用的工具权限策略。

### 工具名称冲突处理

**命名空间前缀：**

```
推荐：<插件前缀>_<工具名>
示例：workflow_execute, data_export, api_query

避免：与核心工具同名
示例：read → ❌（与 core.read 冲突）
      my_read → ✅
```

**冲突检测：**

```typescript
api.registerTool({
  name: "my_tool",
  description: "我的工具"
}, { optional: true });

// 如果与核心工具冲突
// 日志输出：
// [warn] Tool 'my_tool' conflicts with core tool
// [info] Skipping registration of conflicting tool
```

## 最佳实践

### 工具设计指南

**原则一：单一职责**

```
每个工具应该只做一件事，并把它做好

✅ 好的设计：
- database_query
- file_read
- api_call

❌ 不好的设计：
- database_query_and_file_read
- do_everything
```

**原则二：清晰的描述**

```typescript
// ❌ 模糊
{
  "name": "process",
  "description": "处理数据"
}

// ✅ 清晰
{
  "name": "process_user_data",
  "description": "根据用户 ID 处理用户数据，包括验证、转换和存储。输入参数：userId (必需)，transformations (可选)"
}
```

**原则三：参数验证**

```typescript
async execute(id, params) {
  // 1. 验证必需参数
  if (!params.requiredParam) {
    return {
      content: [{
        type: "text",
        text: "错误：缺少必需参数 requiredParam"
      }],
      isError: true
    };
  }

  // 2. 验证参数格式
  if (params.numberParam < 0) {
    return {
      content: [{
        type: "text",
        text: "错误：numberParam 必须大于等于 0"
      }],
      isError: true
    };
  }

  // 3. 执行实际逻辑
  const result = await doWork(params);

  return {
    content: [{
      type: "text",
      text: JSON.stringify(result)
    }]
  };
}
```

### 安全性考虑

**高风险工具标记：**

```typescript
api.registerTool({
  name: "delete_user",
  description: "删除用户账户（高风险操作）",
  parameters: Type.Object({
    userId: Type.String(),
    confirm: Type.Boolean({ description: "确认删除操作" })
  }),
  // 使用 optional: true 强制用户通过 allowList 启用
}, { optional: true });
```

**敏感信息处理：**

```typescript
async execute(id, params) {
  // 不要在日志中记录敏感信息
  console.log("Executing with params:", {
    userId: params.userId,
    // 移除敏感字段
    // apiKey: [REDACTED]
    action: params.action
  });

  // 在返回结果中脱敏
  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        status: "success",
        // 只返回必要信息
        userId: result.userId,
        deleted: true
      })
    }]
  };
}
```

## 故障排查

### 工具不可用

**问题：工具未注册**

```bash
# 检查已注册的工具列表
openclaw tools list

# 查找问题插件
openclaw doctor | rg "tool"
```

**可能原因：**

- 插件未加载
- 工具名称冲突
- 可选工具未在 allowList 中

### 工具执行失败

**调试步骤：**

```bash
# 1. 启用工具调试日志
openclaw logs | rg "tool"

# 2. 检查参数传递
# 查看完整的工具调用日志

# 3. 验证返回值格式
```

### 参数类型错误

**常见问题：**

```
错误：Parameter type mismatch

原因：
├── LLM 传递了错误类型的参数
└── Schema 定义与实际接受类型不一致

解决：
├── 检查 Schema 中的类型定义
├── 添加参数验证逻辑
└── 提供清晰的错误消息
```

## 相关文档

### 核心文档

- [插件系统概述](/plugins)
- [插件清单规范](/plugins/manifest)
- [RPC 适配器](/reference/rpc)

### 配置参考

- [网关配置](/gateway/configuration)
- [代理配置](/concepts/agent)
- [工具策略](/tools/skills)

### 外部资源

- [TypeBox 文档](https://github.com/sinclairzx81/typebox)
- [JSON Schema 规范](https://json-schema.org/)
