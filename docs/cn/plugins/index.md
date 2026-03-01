---
summary: OpenClaw 插件系统完全指南——扩展能力、类型与开发规范
read_when:
  - 想要了解 OpenClaw 插件系统的整体架构
  - 计划开发或集成第三方插件
  - 需要理解插件的生命周期和发现机制
title: "OpenClaw 插件系统完全指南"
---

# OpenClaw 插件系统完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 OpenClaw 插件系统的架构设计
- 掌握插件发现、加载和生命周期管理
- 能够区分不同类型的插件及其用途

### 进阶目标（建议掌握）

- 理解插件配置验证机制
- 掌握插件清单文件的编写规范
- 能够为新通道或提供者开发插件

### 专家目标（挑战）

- 设计复杂的插件交互模式
- 优化插件性能并解决冲突
- 为 OpenClaw 贡献核心插件

## 插件系统架构

### 设计理念

OpenClaw 插件系统采用「发现即验证」的设计哲学，在加载任何插件代码之前，先通过清单文件验证配置的有效性。这种设计确保了系统的稳定性和可预测性。

**核心原则：**

- **声明式配置**：插件必须声明其能力（通道、提供者、技能）
- **前置验证**：配置验证发生在代码执行之前
- **沙箱隔离**：插件运行在受控的上下文中
- **发现机制**：插件通过清单文件被自动发现

### 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenClaw 核心                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    插件发现层                          │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │   │
│   │  │ 文件扫描  │→│ 清单解析  │→│ 验证检查  │              │   │
│   │  └──────────┘  └──────────┘  └──────────┘              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    插件加载层                          │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │   │
│   │  │ 依赖解析  │→│  代码加载 │→│  初始化  │              │   │
│   │  └──────────┘  └──────────┘  └──────────┘              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    运行时层                            │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │   │
│   │  │ 通道注册  │→│ 工具注册  │→│ 事件监听  │              │   │
│   │  └──────────┘  └──────────┘  └──────────┘              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 插件类型

| 类型 | 标识符 | 说明 | 示例 |
|------|--------|------|------|
| 通道插件 | `channel` | 添加新的消息通道 | Matrix、Microsoft Teams |
| 提供者插件 | `provider` | 添加新的 LLM 提供者 | 第三方 API |
| 技能插件 | `skill` | 添加新的代理技能 | 自定义工具集 |
| 混合插件 | 任意组合 | 组合多种能力 | 完整的通道+提供者 |

## 插件发现与加载

### 发现机制

**目录扫描顺序：**

```
1. ~/.openclaw/plugins/          # 用户级插件
2. ./extensions/*/              # 工作区插件
3. node_modules/*/openclaw-plugin # 依赖插件
```

**清单文件检测：**

每个插件目录必须包含 `openclaw.plugin.json` 文件：

```json
{
  "id": "matrix",
  "name": "Matrix 通道",
  "description": "添加 Matrix 即时消息支持",
  "kind": "channel",
  "channels": ["matrix"],
  "configSchema": {
    "type": "object",
    "properties": {
      "homeserver": { "type": "string" }
    }
  }
}
```

### 加载流程

```
┌─────────────────────────────────────────────────────────────────┐
│                      插件加载流程                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   阶段 1：发现与验证                                             │
│   ├── 扫描插件目录                                               │
│   ├── 解析清单文件                                               │
│   ├── 验证配置 Schema                                            │
│   └── 验证通道/提供者声明                                        │
│          │                                                      │
│          ▼                                                      │
│   阶段 2：依赖解析                                               │
│   ├── 解析 package.json                                         │
│   ├── 安装运行时依赖                                             │
│   └── 处理依赖冲突                                               │
│          │                                                      │
│          ▼                                                      │
│   阶段 3：代码加载                                               │
│   ├── 加载插件入口模块                                           │
│   ├── 执行初始化函数                                            │
│   └── 注册能力到 OpenClaw                                       │
│          │                                                      │
│          ▼                                                      │
│   阶段 4：运行就绪                                               │
│   ├── 通道可用                                                   │
│   ├── 工具可调用                                                 │
│   └── 事件可监听                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 配置验证行为

**未知通道键检测：**

```json
{
  "channels": {
    "matrix": { "enabled": true },  // ✅ 声明了 matrix 通道
    "unknown": { "enabled": true }   // ❌ 未声明的通道
  }
}
```

**引用验证：**

```json
{
  "plugins": {
    "entries": {
      "matrix": { "enabled": true },
      "voice-call": { "enabled": true }
    },
    "allow": ["matrix", "voice-call"],  // ✅ 引用已声明的插件
    "deny": ["invalid-plugin"]           // ❌ 引用了未知插件
  }
}
```

## 插件配置

### 配置文件结构

**用户配置：**

```json
{
  "plugins": {
    "entries": {
      "matrix": {
        "enabled": true,
        "config": {
          "homeserver": "https://matrix.example.com"
        }
      }
    },
    "allow": ["matrix"],
    "deny": []
  }
}
```

**插件清单：**

```json
{
  "id": "matrix",
  "name": "Matrix",
  "description": "Matrix 消息通道",
  "kind": "channel",
  "channels": ["matrix"],
  "version": "1.0.0",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "required": ["homeserver"],
    "properties": {
      "homeserver": {
        "type": "string",
        "format": "uri"
      },
      "userId": { "type": "string" },
      "accessToken": { "type": "string" }
    }
  }
}
```

### 运行时行为

**清单损坏处理：**

```
场景：插件已安装但清单文件损坏

处理：
├── Doctor 报告插件错误
├── 配置验证失败
└── 插件不加载

日志输出：
[error] plugin(matrix): manifest.json is invalid or missing
[error] configuration validation failed for plugin matrix
```

**插件被禁用处理：**

```
场景：存在插件配置但插件被禁用

处理：
├── 配置保留
├── Doctor 显示警告
└── 日志输出警告

日志输出：
[warn] plugin(matrix): configuration exists but plugin is disabled
```

## 插件开发指南

### 开发流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    插件开发流程                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. 项目初始化                                                  │
│      └── 创建插件目录结构                                        │
│                                                                 │
│   2. 清单编写                                                    │
│      └── 定义插件元数据和 Schema                                 │
│                                                                 │
│   3. 代码实现                                                    │
│      └── 实现通道/提供者/工具逻辑                                │
│                                                                 │
│   4. 测试验证                                                    │
│      └── 单元测试 + 集成测试                                     │
│                                                                 │
│   5. 打包发布                                                    │
│      └── npm publish 或本地分发                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 目录结构

```
my-plugin/
├── openclaw.plugin.json      # 插件清单（必需）
├── package.json               # 依赖定义
├── src/
│   ├── index.ts              # 入口模块
│   ├── channel.ts            # 通道实现
│   └── tools.ts              # 工具定义
├── test/
│   └── index.test.ts
└── README.md
```

### 依赖管理

**运行时依赖：**

```json
{
  "dependencies": {
    "matrix-js-sdk": "^3.0.0"
  }
}
```

**开发依赖：**

```json
{
  "devDependencies": {
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

**重要提示：**

- 避免使用 `workspace:*` 依赖
- 将 `openclaw/plugin-sdk` 放在 `peerDependencies` 或 `devDependencies` 中

## 专家思维模型

### 插件设计模式

**模式一：最小依赖**

```
原则：减少外部依赖，降低冲突风险

策略：
├── 使用标准库替代品
├── 明确声明依赖版本
└── 避免依赖链过深
```

**模式二：渐进式降级**

```
原则：部分功能不可用时保持核心功能

策略：
├── 核心功能无依赖
├── 可选功能按需加载
└── 优雅处理依赖缺失
```

**模式三：配置驱动**

```
原则：行为可配置，不硬编码

策略：
├── 所有常量可配置
├── 提供合理的默认值
└── Schema 验证输入
```

### 性能优化

**懒加载模式：**

```typescript
// 入口模块
export default function(api) {
  api.registerChannel({
    id: 'matrix',
    onReady: async () => {
      // 延迟加载重型依赖
      const { MatrixChannel } = await import('./channel.js');
      return new MatrixChannel(api.config);
    }
  });
}
```

## 故障排查

### 常见问题

**问题一：插件无法发现**

```bash
# 检查清单文件
ls -la openclaw.plugin.json

# 验证 JSON 语法
cat openclaw.plugin.json | jq .

# 检查目录权限
ls -la plugins/
```

**问题二：配置验证失败**

```bash
# 查看详细错误
openclaw doctor --verbose

# 检查 Schema
cat openclaw.plugin.json | jq '.configSchema'
```

**问题三：依赖安装失败**

```bash
# 进入插件目录
cd plugins/my-plugin

# 清理并重新安装
rm -rf node_modules package-lock.json
npm install --omit=dev
```

### 调试技巧

**启用插件调试日志：**

```json
{
  "diagnostics": {
    "flags": ["plugins.*"]
  }
}
```

**查看加载顺序：**

```bash
openclaw doctor | rg "plugin"
```

## 相关文档

### 核心文档

- [插件清单规范](/plugins/manifest)
- [插件代理工具](/plugins/agent-tools)
- [RPC 适配器](/reference/rpc)

### 示例插件

- [Voice Call 插件](/plugins/voice-call)
- [Zalo User 插件](/plugins/zalouser)

### 外部资源

- [OpenClaw SDK](https://www.npmjs.com/package/openclaw/plugin-sdk)
- [插件模板仓库](https://github.com/openclaw/plugin-template)
