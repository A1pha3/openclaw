---
summary: OpenClaw 插件清单完整指南——JSON Schema 要求、验证行为与最佳实践
read_when:
  - 你正在构建 OpenClaw 插件
  - 你需要提供插件配置 Schema 或调试插件验证错误
  - 你想理解插件清单的设计哲学和验证机制
title: "OpenClaw 插件清单完全指南"
---

# OpenClaw 插件清单完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解插件清单文件的作用和必要性
- 掌握必需字段和可选字段的完整规范
- 能够编写符合要求的 JSON Schema 配置

### 进阶目标（建议掌握）

- 理解验证行为和错误处理机制
- 掌握高级 Schema 设计技巧
- 能够设计用户友好的配置界面

### 专家目标（挑战）

- 为复杂插件设计类型安全的配置 Schema
- 优化验证性能
- 贡献开源插件的最佳实践

## 核心概念

### 什么是插件清单？

插件清单（`openclaw.plugin.json`）是 OpenClaw 插件的声明式配置文件，它定义了插件的元数据、能力声明和配置验证规则。

**设计哲学：**

> 「先验证，后执行」——OpenClaw 在加载任何插件代码之前，先通过清单文件验证配置的有效性。这种前置验证确保了系统的稳定性和可预测性。

### 为什么需要清单文件？

**1. 安全性**

- 阻止未授权的配置访问
- 防止配置注入攻击
- 确保插件只能访问其声明的能力

**2. 可预测性**

- 配置错误在启动时发现，而非运行时
- 提供清晰的错误信息和修复建议
- 避免部分加载导致的不一致状态

**3. 可发现性**

- 自动发现和注册插件能力
- 生成配置 UI 和文档
- 支持插件市场的索引和搜索

## 完整规范

### 必需字段

每个插件必须提供以下字段：

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

**字段说明：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | 字符串 | 是 | 规范化的插件唯一标识符 |
| `configSchema` | 对象 | 是 | 插件配置的 JSON Schema |

### id 字段规范

**命名规则：**

```
[id] ::= [a-z][a-z0-9]*(-[a-z0-9]+)*
```

**示例：**

| 有效 ID | 无效 ID | 原因 |
|---------|---------|------|
| `voice-call` | VoiceCall | 必须使用小写 |
| `matrix` | matrix_channel | 必须使用连字符 |
| `zalo-user` | 123plugin | 必须以字母开头 |

### configSchema 字段规范

**最小 Schema：**

```json
{
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

**完整 Schema 结构：**

```json
{
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "enabled": { "type": "boolean" },
      "endpoint": { "type": "string", "format": "uri" },
      "timeout": { "type": "integer", "minimum": 1, "maximum": 300 }
    },
    "required": ["endpoint"]
  }
}
```

### 可选字段

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `kind` | 字符串 | 无 | 插件类型（memory、channel 等） |
| `channels` | 数组 | 空 | 注册的通道 ID 列表 |
| `providers` | 数组 | 空 | 注册的提供者 ID 列表 |
| `skills` | 数组 | 空 | 要加载的技能目录 |
| `name` | 字符串 | `id` 值 | 插件显示名称 |
| `description` | 字符串 | 空 | 插件简短描述 |
| `uiHints` | 对象 | 空 | UI 渲染提示 |
| `version` | 字符串 | 无 | 插件版本（信息性） |

### channels 字段

**示例：**

```json
{
  "channels": ["matrix", "msteams"]
}
```

**验证规则：**

- 数组元素必须是字符串
- 通道 ID 必须唯一
- 必须与 `channels.*` 配置键对应

### providers 字段

**示例：**

```json
{
  "providers": ["anthropic", "openai"]
}
```

### uiHints 字段

**结构：**

```json
{
  "uiHints": {
    "fields": {
      "endpoint": {
        "label": "API 端点",
        "placeholder": "https://api.example.com",
        "sensitive": false,
        "group": "连接设置"
      },
      "apiKey": {
        "label": "API 密钥",
        "placeholder": "输入你的 API 密钥",
        "sensitive": true,
        "group": "认证"
      }
    }
  }
}
```

**字段属性：**

| 属性 | 类型 | 说明 |
|------|------|------|
| `label` | 字符串 | 显示标签 |
| `placeholder` | 字符串 | 占位符文本 |
| `sensitive` | 布尔值 | 是否敏感（脱敏显示） |
| `group` | 字符串 | 字段分组 |
| `help` | 字符串 | 帮助文本 |
| `validation` | 对象 | 自定义验证规则 |

## JSON Schema 深度指南

### Schema 验证时机

```
┌─────────────────────────────────────────────────────────────────┐
│                    配置生命周期                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. 配置读取                                                   │
│      └── 从 openclaw.json 读取用户配置                          │
│                                                                 │
│   2. Schema 验证  ←───  在此处验证                               │
│      ├── 检查必需字段                                           │
│      ├── 检查类型正确性                                         │
│      └── 检查约束条件                                           │
│                                                                 │
│   3. 插件加载                                                   │
│      └── 仅在验证通过后加载插件代码                             │
│                                                                 │
│   4. 运行时访问                                                 │
│      └── 插件访问已验证的配置                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 常用 Schema 类型

**对象类型：**

```json
{
  "configSchema": {
    "type": "object",
    "properties": {
      "server": { "type": "string" },
      "port": { "type": "integer" }
    },
    "required": ["server", "port"],
    "additionalProperties": false
  }
}
```

**字符串类型：**

```json
{
  "url": {
    "type": "string",
    "format": "uri",
    "pattern": "^https://"
  }
}
```

**数组类型：**

```json
{
  "allowedUsers": {
    "type": "array",
    "items": { "type": "string" },
    "minItems": 1,
    "uniqueItems": true
  }
}
```

**枚举类型：**

```json
{
  "logLevel": {
    "type": "string",
    "enum": ["debug", "info", "warn", "error"]
  }
}
```

### 复杂 Schema 示例

**带条件验证的配置：**

```json
{
  "configSchema": {
    "type": "object",
    "properties": {
      "mode": {
        "type": "string",
        "enum": ["auto", "manual"]
      },
      "autoConfig": {
        "type": "object",
        "properties": {
          "interval": { "type": "integer" }
        }
      },
      "manualConfig": {
        "type": "object",
        "properties": {
          "value": { "type": "string" }
        }
      }
    },
    "allOf": [
      {
        "if": { "properties": { "mode": { "const": "auto" } } },
        "then": { "required": ["autoConfig"] }
      },
      {
        "if": { "properties": { "mode": { "const": "manual" } } },
        "then": { "required": ["manualConfig"] }
      }
    ]
  }
}
```

## 验证行为详解

### 验证规则矩阵

| 场景 | 配置状态 | 验证结果 | 处理方式 |
|------|----------|----------|----------|
| 通道已声明，配置正确 | 有效 | 通过 | 加载插件 |
| 通道已声明，配置错误 | 无效 | 失败 | 报告错误 |
| 通道未声明，使用配置 | 无效 | 失败 | 报告错误 |
| 插件已禁用，配置存在 | 有效 | 警告 | 保留配置 |

### 错误处理

**未知通道错误：**

```
配置：
{
  "channels": {
    "matrix": { "enabled": true },
    "unknown-channel": { "enabled": true }
  }
}

错误输出：
[error] Unknown channel key 'unknown-channel'.
        Channel must be declared in plugin manifest.
```

**引用验证错误：**

```
配置：
{
  "plugins": {
    "entries": {
      "matrix": { "enabled": true }
    },
    "allow": ["non-existent-plugin"]
  }
}

错误输出：
[error] Plugin reference 'non-existent-plugin' not found.
        Available plugins: matrix, voice-call
```

### Doctor 集成

**检测清单损坏：**

```bash
$ openclaw doctor

Plugin Status:
├── matrix: ERROR (manifest.json missing)
├── voice-call: WARNING (configuration exists but disabled)
└── zalo-user: OK
```

**自动修复建议：**

```
[error] Plugin 'matrix' has invalid manifest.
        Run 'openclaw doctor --fix' to reset configuration.
```

## 专家思维模型

### Schema 设计原则

**原则一：最小权限**

```
只请求必要的配置项，不收集无关信息

示例：
├── 必需的连接信息 ✓
├── 可选的性能调优 ✓
└── 敏感的用户数据 ✗（应使用环境变量）
```

**原则二：渐进式披露**

```
先显示必需字段，展开显示高级选项

UI 展示：
┌────────────────────────────────────┐
│ 基本设置                            │
│ ├─ 端点地址  [_____________]       │
│ └─ 超时时间  [____] ms             │
│                                    │
│ ▼ 高级设置（点击展开）              │
│   ├─ 重试次数  [___]               │
│   └─ 缓存大小  [____] KB           │
└────────────────────────────────────┘
```

**原则三：智能默认值**

```
提供合理的默认值，减少用户配置负担

策略：
├── 使用安全的默认值
├── 记录默认值来源
└── 允许覆盖
```

### 性能优化

**Schema 编译缓存：**

```typescript
// 首次使用时编译
let cachedSchema:compiled | null = null;

function getCompiledSchema(raw: object): compiled {
  if (!cachedSchema) {
    cachedSchema = compile(raw);
  }
  return cachedSchema;
}
```

**验证结果缓存：**

```
同一个配置只需验证一次，后续访问直接使用缓存结果

流程：
配置变更 → 清除缓存 → 重新验证 → 缓存结果
```

## 最佳实践

### 清单文件模板

**最小插件：**

```json
{
  "id": "minimal-plugin",
  "name": "最小插件",
  "description": "一个不做任何事情的最小插件",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

**完整插件：**

```json
{
  "id": "my-plugin",
  "name": "我的插件",
  "description": "提供自定义功能的插件",
  "kind": "provider",
  "providers": ["my-provider"],
  "version": "1.0.0",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "endpoint": {
        "type": "string",
        "format": "uri"
      },
      "apiKey": {
        "type": "string"
      },
      "timeout": {
        "type": "integer",
        "minimum": 1,
        "maximum": 600,
        "default": 30
      }
    },
    "required": ["endpoint"]
  },
  "uiHints": {
    "fields": {
      "endpoint": {
        "label": "API 端点",
        "placeholder": "https://api.example.com"
      },
      "apiKey": {
        "label": "API 密钥",
        "sensitive": true
      }
    }
  }
}
```

### 常见错误

**错误一：忘记 additionalProperties**

```json
// ❌ 错误
{
  "configSchema": {
    "type": "object",
    "properties": {
      "enabled": { "type": "boolean" }
    }
  }
}

// ✅ 正确
{
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "enabled": { "type": "boolean" }
    }
  }
}
```

**错误二：required 与 optional 混淆**

```json
// ❌ 错误：required 中包含可选字段
{
  "configSchema": {
    "type": "object",
    "properties": {
      "optionalField": { "type": "string" }
    },
    "required": ["optionalField"]
  }
}

// ✅ 正确：只包含真正必需的字段
{
  "configSchema": {
    "type": "object",
    "properties": {
      "optionalField": { "type": "string" }
    },
    "required": []
  }
}
```

## 故障排查

### 调试清单问题

**步骤一：验证 JSON 语法**

```bash
# 使用 jq 解析
cat openclaw.plugin.json | jq . > /dev/null && echo "Valid JSON" || echo "Invalid JSON"
```

**步骤二：验证 Schema 结构**

```bash
# 使用 OpenClaw Doctor
openclaw doctor --verbose
```

**步骤三：检查清单路径**

```
正确的目录结构：
plugins/
└── my-plugin/
    ├── openclaw.plugin.json  ← 必须在插件根目录
    └── src/
        └── index.ts
```

### 常见错误消息

| 错误消息 | 原因 | 解决方案 |
|----------|------|----------|
| `manifest.json is missing` | 文件不存在 | 创建清单文件 |
| `Invalid JSON syntax` | JSON 格式错误 | 修复语法错误 |
| `Missing required field: id` | 缺少必需字段 | 添加 id 字段 |
| `Invalid configSchema type` | Schema 类型错误 | 使用正确的类型 |
| `Unknown channel referenced` | 引用未声明的通道 | 在 channels 中声明 |

## 相关文档

### 核心文档

- [插件系统概述](/plugins)
- [插件代理工具](/plugins/agent-tools)
- [RPC 适配器](/reference/rpc)

### 配置参考

- [网关配置](/gateway/configuration)
- [日志配置](/logging)
- [通道配置](/channels)

### 开发资源

- [JSON Schema 规范](https://json-schema.org/)
- [OpenClaw SDK](https://www.npmjs.com/package/openclaw/plugin-sdk)
- [插件开发模板](https://github.com/openclaw/plugin-template)
