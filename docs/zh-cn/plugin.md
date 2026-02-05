---
summary: "OpenClaw 插件系统完全指南：发现机制、配置管理、安全策略与自定义开发"
read_when:
  - 需要添加或修改插件时
  - 了解插件的发现机制和加载规则时
  - 开发自定义插件时
  - 理解插件安全模型时
title: "插件系统（Plugins）"
---

# OpenClaw 插件系统完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 **插件（Plugin）** 的核心概念和在 OpenClaw 架构中的定位
- [ ] 掌握插件的发现机制和加载优先级规则
- [ ] 能够安装、配置和管理官方插件
- [ ] 理解插件的安全模型和信任边界

### 进阶目标（建议掌握）

- [ ] 理解插件 API 的核心功能和使用场景
- [ ] 能够开发简单的自定义插件
- [ ] 掌握插件槽位（Plugin Slots）的概念和配置方法
- [ ] 理解控制 UI 与插件配置的集成机制

### 专家目标（挑战）

- [ ] 能够开发复杂的消息渠道插件
- [ ] 理解插件系统的架构设计和扩展点
- [ ] 掌握插件测试和发布的最佳实践

---

## 第一部分：插件系统概述

### 1.1 什么是 OpenClaw 插件？

**核心定义**：插件是扩展 OpenClaw 功能的**小型代码模块**，可以添加以下功能组件：

| 组件类型 | 功能描述 | 集成深度 |
|----------|----------|----------|
| Gateway RPC 方法 | 扩展 Gateway 的远程过程调用能力 | 进程内 |
| Gateway HTTP 处理器 | 处理 HTTP 请求和响应 | 进程内 |
| 代理工具 | 为 AI 助手提供新工具 | 工具层 |
| CLI 命令 | 添加新的命令行指令 | 命令层 |
| 后台服务 | 运行持续性服务任务 | 服务层 |
| 技能 | 提供完整的工作流和能力 | 技能层 |
| 自动回复命令 | 无需 AI 介入的快捷命令 | 命令层 |

**专家思维模型 — 插件即扩展点**：

OpenClaw 的插件系统不是简单的"功能开关"，而是一个精心设计的**扩展架构**。理解这个架构需要从以下角度思考：

```
OpenClaw 架构层次图：

┌─────────────────────────────────────────────────────────┐
│                    用户交互层                            │
│  ├─ Web UI / TUI / CLI / 消息渠道                       │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    代理运行时层                          │
│  ├─ 代理（Agent）                                      │
│  ├─ 会话（Session）                                    │
│  └─ 工具（Tools）← 插件可以扩展这里                     │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    Gateway 控制层                       │
│  ├─ RPC 方法 ← 插件可以扩展这里                          │
│  ├─ HTTP 处理器 ← 插件可以扩展这里                      │
│  └─ 后台服务 ← 插件可以扩展这里                          │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    渠道接入层                            │
│  └─ 消息渠道 ← 插件可以添加新渠道                         │
└─────────────────────────────────────────────────────────┘

关键洞察：插件可以在多个层次扩展系统，
但所有插件都在 Gateway 进程内运行
```

### 1.2 为什么需要插件系统？

**核心价值：平衡核心与扩展**

OpenClaw 的设计哲学是**保持核心精简，提供扩展能力**。插件系统解决了以下问题：

| 问题 | 解决方案 |
|------|----------|
| 核心臃肿 | 只包含必备功能，其他作为插件 |
| 需求多样性 | 用户选择需要的插件 |
| 迭代速度 | 插件可以独立于核心发布 |
| 生态建设 | 允许社区贡献插件 |
| 定制化 | 高手可以深度定制 |

**对比分析：内置 vs 插件**

| 维度 | 内置功能 | 插件功能 |
|------|----------|----------|
| 发布节奏 | 与核心同步 | 独立发布 |
| 用户选择 | 必须接受 | 可选安装 |
| 开发门槛 | 核心团队开发 | 开放社区 |
| 稳定性 | 高（核心测试） | 取决于作者 |
| 定制性 | 有限 | 高度可定制 |

### 1.3 插件类型分类

```
插件分类矩阵：

                    ┌─────────────────┐
                    │   渠道插件        │
                    │  添加新消息渠道   │
                    │  WhatsApp等      │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
       ┌───────────┐  ┌───────────┐  ┌───────────┐
       │ 功能插件   │  │ 提供商插件 │  │ 工具插件  │
       │ 语音通话   │  │ OAuth认证  │  │ 自定义工具│
       └───────────┘  └───────────┘  └───────────┘
```

---

## 第二部分：快速入门

### 2.1 查看已加载的插件

**操作步骤**：

```bash
# 列出所有已加载的插件
openclaw plugins list
```

**输出示例**：

```
已加载的插件：

✓ voice-call        (v1.2.0)   官方    语音通话功能
✓ matrix            (v2.0.1)   官方    Matrix 消息渠道
✓ zalo              (v1.0.0)   官方    Zalo 消息渠道
- msteams           (v0.9.0)   官方    Microsoft Teams（已禁用）
✗ suspicious-plugin (v1.0.0)   第三方  未授权插件
```

### 2.2 安装官方插件

**以语音通话插件为例**：

```bash
# 安装官方语音通话插件
openclaw plugins install @openclaw/voice-call

# 安装后需要重启 Gateway 才能生效
openclaw gateway restart
```

**安装流程图**：

```
安装过程：

    ┌─────────────────────────────────────┐
    │  执行安装命令                        │
    │  openclaw plugins install <plugin>   │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  下载插件包                          │
    │  从 npm 仓库获取插件包                │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  解压到扩展目录                       │
    │  ~/.openclaw/extensions/<plugin-id>/  │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  更新配置文件                         │
    │  自动添加到 plugins.entries          │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  需要重启 Gateway                    │
    │  配置更改才生效                       │
    └─────────────────────────────────────┘
```

### 2.3 配置插件

**插件配置位置**：插件配置位于 `plugins.entries.<id>.config` 下：

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
          twilio: {
            accountSid: "ACxxxxxxxxxxxxx",
            authToken: "${OPENCLAW_TWILIO_AUTH_TOKEN}",
            from: "+1234567890"
          }
        }
      }
    }
  }
}
```

**配置验证**：插件配置使用插件清单中嵌入的 JSON Schema 进行验证。

---

## 第三部分：官方插件列表

### 3.1 内置插件

| 插件 | 类型 | 说明 | 默认状态 |
|------|------|------|----------|
| Memory（Core） | 功能插件 | 捆绑的内存搜索插件 | 默认启用 |
| Memory（LanceDB） | 功能插件 | 长期内存插件 | 需配置启用 |
| Google Antigravity OAuth | 提供商插件 | Google OAuth 认证 | 默认禁用 |
| Gemini CLI OAuth | 提供商插件 | Gemini CLI 认证 | 默认禁用 |
| Qwen OAuth | 提供商插件 | Qwen Portal 认证 | 默认禁用 |
| Copilot Proxy | 功能插件 | VS Code Copilot 桥接 | 默认禁用 |

### 3.2 渠道插件

| 插件 | NPM 包 | 优先级 | 说明 |
|------|--------|--------|------|
| WhatsApp | 内置 | 🔴 高 | WhatsApp 消息渠道 |
| Telegram | 内置 | 🔴 高 | Telegram 消息渠道 |
| Discord | 内置 | 🔴 高 | Discord 消息渠道 |
| Slack | 内置 | 🔴 高 | Slack 消息渠道 |
| Signal | 内置 | 🔴 高 | Signal 消息渠道 |
| iMessage | 内置 | 🔴 高 | iMessage 消息渠道 |
| Matrix | `@openclaw/matrix` | 🟠 中 | Matrix 消息渠道 |
| Microsoft Teams | `@openclaw/msteams` | 🟠 中 | Teams 消息渠道 |
| Zalo | `@openclaw/zalo` | 🟡 低 | Zalo 消息渠道 |
| Zalo Personal | `@openclaw/zalouser` | 🟡 低 | Zalo 个人消息 |
| Nostr | `@openclaw/nostr` | 🟡 低 | Nostr 消息渠道 |

### 3.3 功能插件

| 插件 | NPM 包 | 说明 |
|------|--------|------|
| 语音通话 | `@openclaw/voice-call` | 电话呼叫功能 |
| BlueBubbles | `@openclaw/bluebubbles` | BlueBubbles 集成 |

---

## 第四部分：发现机制与优先级

### 4.1 扫描顺序

OpenClaw 按以下顺序扫描插件：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | `plugins.load.paths` | 配置文件中显式指定的路径 |
| 2 | `<workspace>/.openclaw/extensions/*.ts` | 工作区单个文件扩展 |
| 3 | `<workspace>/.openclaw/extensions/*/index.ts` | 工作区目录扩展 |
| 4 | `~/.openclaw/extensions/*.ts` | 用户全局单个文件扩展 |
| 5 | `~/.openclaw/extensions/*/index.ts` | 用户全局目录扩展 |
| 6 | `<openclaw>/extensions/*` | 捆绑扩展目录 |

**优先级冲突处理**：如果多个插件解析为相同 ID，**顺序中第一个匹配获胜**，后续被忽略。

### 4.2 插件包（Plugin Packages）

插件目录可以包含带有 `openclaw.extensions` 字段的 `package.json`：

```json
{
  "name": "my-plugin-pack",
  "version": "1.0.0",
  "openclaw": {
    "extensions": [
      "./src/safety.ts",
      "./src/tools.ts",
      "./src/commands.ts"
    ]
  }
}
```

**多扩展命名规则**：如果包列出多个扩展，插件 ID 变为 `name/<fileBase>`。

### 4.3 外部渠道目录

OpenClaw 可以合并外部渠道目录（支持 MPM 注册表导出）：

```
外部渠道配置位置：

┌─────────────────────────────────────────────────────────┐
│  用户本地目录                                             │
│  ├─ ~/.openclaw/mpm/plugins.json                        │
│  ├─ ~/.openclaw/mpm/catalog.json                       │
│  └─ ~/.openclaw/plugins/catalog.json                    │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  环境变量（可选）                                         │
│  OPENCLAW_PLUGIN_CATALOG_PATHS                          │
│  或                                                      │
│  OPENCLAW_MPM_CATALOG_PATHS                             │
└─────────────────────────────────────────────────────────┘
```

**目录文件格式**：

```json
{
  "entries": [
    {
      "name": "@scope/external-channel",
      "openclaw": {
        "channel": {
          "id": "external-channel",
          "label": "External Channel",
          "docsPath": "/channels/external-channel"
        },
        "install": {
          "npmSpec": "@scope/external-channel",
          "localPath": "extensions/external-channel"
        }
      }
    }
  ]
}
```

---

## 第五部分：插件配置详解

### 5.1 配置结构

```json5
{
  plugins: {
    // 主开关
    enabled: true,

    // 白名单（可选）
    allow: ["voice-call", "matrix"],

    // 黑名单（可选）
    deny: ["untrusted-plugin"],

    // 额外扫描路径
    load: {
      paths: ["~/Projects/oss/my-extension"]
    },

    // 每个插件的配置
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      },
      "matrix": {
        enabled: true,
        config: {
          homeserver: "https://matrix.example.com"
        }
      }
    },

    // 插件槽位（独占类别）
    slots: {
      memory: "memory-core"
    }
  }
}
```

### 5.2 配置验证规则

**严格模式**：

| 规则 | 说明 | 处理方式 |
|------|------|----------|
| 未知插件 ID | `entries`、`allow`、`deny`、`slots` 中未声明的插件 | 报错 |
| 未知渠道 ID | `channels.<id>` 未被任何插件声明 | 报错 |
| Schema 验证 | 插件配置不符合 JSON Schema | 报错 |
| 禁用插件 | 插件被禁用但配置存在 | 警告（保留配置） |

### 5.3 插件槽位（Plugin Slots）

某些插件类别是**独占的**（同时只有一个活动）：

```
槽位类型：

┌─────────────────────────────────────────────────────────┐
│                    内存槽位                              │
│  同一时间只加载一个内存插件                               │
│  配置：plugins.slots.memory                             │
│  选项："memory-core" / "memory-lancedb" / "none"       │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    其他槽位（未来扩展）                   │
│  ...                                                     │
└─────────────────────────────────────────────────────────┘
```

**配置示例**：

```json5
{
  plugins: {
    slots: {
      memory: "memory-core"  // 使用内置内存插件
      // memory: "memory-lancedb"  // 切换到 Lancedb 插件
      // memory: "none"  // 禁用内存插件
    }
  }
}
```

---

## 第六部分：插件 API 详解

### 6.1 插件导出格式

插件可以以两种方式导出：

**函数格式**：

```typescript
export default function (api: OpenClawPluginAPI) {
  // 注册插件功能
  api.registerGatewayMethod(...);
  api.registerCli(...);
}
```

**对象格式**：

```typescript
export default {
  id: "my-plugin",
  name: "我的插件",
  configSchema: {
    type: "object",
    properties: {
      apiKey: { type: "string" }
    }
  },
  register(api: OpenClawPluginAPI) {
    // 注册插件功能
  }
};
```

### 6.2 注册 Gateway RPC 方法

```typescript
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { status: "running", version: "1.0.0" });
  });

  api.registerGatewayMethod("myplugin.action", async ({ params, respond }) => {
    try {
      const result = await doSomething(params);
      respond(true, result);
    } catch (error) {
      respond(false, { error: error.message });
    }
  });
}
```

### 6.3 注册 CLI 命令

```typescript
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program
        .command("myplugin:hello")
        .description("Say hello from my plugin")
        .option("-n, --name <name>", "Your name")
        .action((options) => {
          console.log(`Hello, ${options.name || "World"}!`);
        });
    },
    { commands: ["myplugin:hello"] }
  );
}
```

### 6.4 注册自动回复命令

无需 AI 介入的快捷命令：

```typescript
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => {
      return {
        text: `Plugin is running! Channel: ${ctx.channel}`
      };
    }
  });
}
```

**命令上下文属性**：

| 属性 | 类型 | 说明 |
|------|------|------|
| `senderId` | string | 发送者的 ID |
| `channel` | string | 命令发送到的渠道 |
| `isAuthorizedSender` | boolean | 发送者是否授权 |
| `args` | object | 命令参数 |
| `commandBody` | string | 完整命令文本 |
| `config` | object | 当前 OpenClaw 配置 |

### 6.5 注册后台服务

```typescript
export default function (api) {
  api.registerService({
    id: "my-background-service",
    start: () => {
      api.logger.info("Background service started");
      // 启动服务逻辑
    },
    stop: () => {
      api.logger.info("Background service stopped");
      // 清理资源
    }
  });
}
```

### 6.6 运行时辅助函数

插件可以通过 `api.runtime` 访问核心功能：

```typescript
// 电话 TTS 示例
export default function (api) {
  api.registerCommand({
    name: "call-me",
    handler: async (ctx) => {
      const result = await api.runtime.tts.textToSpeechTelephony({
        text: "Hello from OpenClaw",
        cfg: api.config,
      });

      // result 包含 PCM 音频缓冲区和采样率
      // 插件需要为提供商重新采样/编码

      return {
        text: `Calling with ${result.duration} seconds of audio`
      };
    }
  });
}
```

---

## 第七部分：消息渠道插件开发

### 7.1 渠道插件概述

插件可以注册**消息渠道**，行为类似于内置渠道（WhatsApp、Telegram 等）。

```
渠道插件架构图：

┌─────────────────────────────────────────────────────────┐
│                   渠道适配器层                           │
│  ┌─────────────────────────────────────────────────┐  │
│  │                  渠道插件                         │  │
│  │  ├─ 元数据（meta）                               │  │
│  │  ├─ 配置适配器（config）                          │  │
│  │  ├─ 出站适配器（outbound）                        │  │
│  │  └─ 入站适配器（inbound）← 可选                  │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                   Gateway 路由层                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │  统一的消息入口和分发机制                         │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 7.2 渠道元数据

```typescript
const channelMeta = {
  id: "acmechat",
  label: "AcmeChat",
  selectionLabel: "AcmeChat (API)",
  docsPath: "/channels/acmechat",
  blurb: "demo channel plugin.",
  aliases: ["acme", "acme-chat"],
  detailLabel: "AcmeChat Messaging",
  systemImage: "chat-bubble"
};
```

### 7.3 配置适配器

```typescript
const channelConfig = {
  listAccountIds: (cfg) =>
    Object.keys(cfg.channels?.acmechat?.accounts ?? {}),

  resolveAccount: (cfg, accountId) =>
    cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
      accountId,
      enabled: true
    }
};
```

### 7.4 出站适配器

```typescript
const outboundAdapter = {
  deliveryMode: "direct",  // 或 "batch"
  sendText: async ({ text, account, recipients }) => {
    // 调用渠道 API 发送消息
    return { ok: true, messageId: "xxx" };
  },

  // 可选：支持媒体
  sendMedia: async ({ media, account, recipients }) => {
    // 处理媒体消息发送
    return { ok: true, mediaId: "xxx" };
  }
};
```

### 7.5 完整渠道插件示例

```typescript
const acmeChannelPlugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: {
    chatTypes: ["direct", "group"],
    mediaTypes: ["image", "video", "audio", "file"],
    threading: true
  },
  config: {
    listAccountIds: (cfg) =>
      Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId
      }
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text, account, recipients }) => {
      const response = await fetch(`https://api.acmechat.com/send`, {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${account.token}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          to: recipients[0],
          message: text
        })
      });

      if (!response.ok) {
        throw new Error(`Failed to send: ${response.statusText}`);
      }

      const result = await response.json();
      return { ok: true, messageId: result.id };
    }
  }
};

export default function (api) {
  api.registerChannel({ plugin: acmeChannelPlugin });
}
```

### 7.6 渠道配置示例

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: {
          token: "${OPENCLAW_ACMECHAT_TOKEN}",
          enabled: true
        },
        work: {
          token: "${OPENCLAW_ACMECHAT_WORK_TOKEN}",
          enabled: true
        }
      }
    }
  }
}
```

---

## 第八部分：提供商认证插件

### 8.1 提供商注册

插件可以注册**模型提供商认证**流程：

```typescript
export default function (api) {
  api.registerProvider({
    id: "acme-ai",
    label: "AcmeAI",
    auth: [
      {
        id: "oauth",
        label: "OAuth",
        kind: "oauth",
        run: async (ctx) => {
          // 运行 OAuth 流程
          const credentials = await ctx.oauth.createVpsAwareHelpers({
            clientId: "your-client-id",
            clientSecret: "your-client-secret",
            authUrl: "https://acme.ai/oauth/authorize",
            tokenUrl: "https://acme.ai/oauth/token",
            scopes: ["model.read", "model.write"]
          });

          return {
            profiles: [
              {
                profileId: "acme-ai:default",
                credential: credentials,
                expires: Date.now() + 3600 * 1000
              }
            ],
            defaultModel: "acme-ai/opus-1"
          };
        }
      },
      {
        id: "api-key",
        label: "API Key",
        kind: "api-key",
        run: async (ctx) => {
          const apiKey = await ctx.prompter.input("Enter your API Key:", {
            type: "password"
          });

          return {
            profiles: [
              {
                profileId: "acme-ai:apikey",
                credential: {
                  type: "api-key",
                  value: apiKey
                }
              }
            ],
            defaultModel: "acme-ai/opus-1"
          };
        }
      }
    ]
  });
}
```

### 8.2 使用提供商认证

```bash
# 触发 OAuth 认证流程
openclaw models auth login --provider acme-ai --method oauth

# 触发 API Key 认证流程
openclaw models auth login --provider acme-ai --method api-key
```

---

## 第九部分：插件钩子

### 9.1 钩子系统概述

插件可以在运行时附带钩子（Hooks），实现事件驱动的自动化：

```typescript
// 插件内注册钩子目录
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

### 9.2 钩子结构

```
hooks/
├── message-received/
│   ├── HOOK.md          # 钩子说明文档
│   └── handler.ts       # 钩子处理函数
├── agent-response/
│   ├── HOOK.md
│   └── handler.ts
└── cron-hourly/
    ├── HOOK.md
    └── handler.ts
```

**钩子处理函数示例**：

```typescript
import { HookHandler } from "openclaw/plugin-sdk";

export const handler: HookHandler = async (ctx) => {
  const { event, config } = ctx;

  // 检查事件类型
  if (event.type !== "message.received") {
    return { continue: true };
  }

  // 处理消息
  const { message, channel } = event.data;

  // 简单关键词过滤
  const blockedWords = ["spam", "scam"];
  const hasBlocked = blockedWords.some(word =>
    message.text.toLowerCase().includes(word)
  );

  if (hasBlocked) {
    return {
      continue: false,
      result: {
        action: "block",
        reason: "Message contains blocked content"
      }
    };
  }

  // 继续处理
  return { continue: true };
};

export const metadata = {
  event: "message.received",
  os: ["darwin", "linux"],
  binary: null,
  env: null,
  config: null
};
```

### 9.3 钩子管理

```bash
# 列出所有钩子（包括插件管理的）
openclaw hooks list

# 钩子命名格式
# plugin:<plugin-id>/<hook-name>
```

**注意**：插件管理的钩子不能通过 `openclaw hooks` 命令启用/禁用，需要启用/禁用整个插件。

---

## 第十部分：命名约定

### 10.1 标识符命名

| 类型 | 格式 | 示例 |
|------|------|------|
| Gateway 方法 | `pluginId.action` | `voicecall.status` |
| 工具 | `snake_case` | `voice_call` |
| CLI 命令 | `kebab-case` | `voice-call` |
| 插件 ID | `kebab-case` | `voice-call` |

### 10.2 保留名称

以下名称不能被插件覆盖：

- 内置命令：`help`、`status`、`reset`、`new` 等
- 系统保留方法：Gateway 核心 RPC 方法

---

## 第十一部分：技能

### 11.1 技能与插件的关系

```
┌─────────────────────────────────────────────────────────┐
│                    插件                                 │
│  ┌─────────────────────────────────────────────────┐  │
│  │  技能：skills/<skill-name>/SKILL.md             │  │
│  │  ┌───────────────────────────────────────────┐  │  │
│  │  │               技能核心                      │  │  │
│  │  │  ├─ SKILL.md（技能文档）                   │  │  │
│  │  │  ├─ 工具定义                               │  │  │
│  │  │  └─ 系统提示                               │  │  │
│  │  └───────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 11.2 技能启用

技能通过以下方式启用：

```json5
{
  plugins: {
    entries: {
      "my-plugin": {
        enabled: true,
        config: {
          // 技能相关配置
        }
      }
    }
  }
}
```

---

## 第十二部分：分发与发布

### 12.1 npm 包结构

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.0.0",
  "description": "My custom plugin for OpenClaw",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest"
  }
}
```

### 12.2 发布流程

```
发布流程：

    ┌─────────────────────────────────────┐
    │  1. 准备代码                        │
    │  ├─ 代码完成并测试通过               │
    │  ├─ 更新版本号                      │
    │  └─ 编写 CHANGELOG                  │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  2. 构建发布包                       │
    │  pnpm build                         │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  3. 发布到 npm                       │
    │  npm publish --access public         │
    └─────────────────┬───────────────────┘
                      ▼
    ┌─────────────────────────────────────┐
    │  4. 更新文档                         │
    │  ├─ 更新插件文档                     │
    │  └─ 更新插件列表                     │
    └─────────────────────────────────────┘
```

### 12.3 安装方式

| 方式 | 命令 | 说明 |
|------|------|------|
| npm 包 | `openclaw plugins install @openclaw/voice-call` | 从 npm 安装 |
| 本地文件 | `openclaw plugins install ./voice-call` | 从本地目录安装 |
| 本地压缩包 | `openclaw plugins install ./plugin.tgz` | 从 tarball 安装 |
| 符号链接 | `openclaw plugins install -l ./dev-extension` | 开发时使用（不复制） |

---

## 第十三部分：安全

### 13.1 安全模型

**关键事实**：插件与 Gateway **进程内运行**，因此插件被视为**可信代码**。

```
安全边界图：

┌─────────────────────────────────────────────────────────┐
│                   Gateway 进程                           │
│  ┌─────────────────────────────────────────────────┐  │
│  │                   插件代码                         │  │
│  │  ├─ 完全访问 Gateway API                          │  │
│  │  ├─ 访问配置                                      │  │
│  │  ├─ 访问文件系统                                  │  │
│  │  └─ 执行任意代码                                  │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘

│                                                    │
│                    信任边界                          │
│    只安装你完全信任的插件（如同安装 npm 包）          │
```

### 13.2 安全建议

| 建议 | 说明 |
|------|------|
| 只安装可信插件 | 像信任 npm 包一样信任插件 |
| 使用白名单 | 通过 `plugins.allow` 限制可用插件 |
| 及时更新 | 保持插件版本最新 |
| 隔离运行 | 考虑使用沙箱环境运行不信任的插件 |

### 13.3 审计清单

```markdown
插件安全审计清单：

安装前：
□ 插件来源是否可信？
□ 插件代码是否开源？
□ 插件是否有活跃维护？
□ 插件权限是否合理？

安装后：
□ 插件是否请求了不必要的权限？
□ 插件是否访问了敏感数据？
□ 插件网络请求是否正常？
□ 插件是否定期更新？

定期检查：
□ 是否需要更新插件版本？
□ 插件行为是否异常？
□ 是否有安全漏洞报告？
```

---

## 第十四部分：测试

### 14.1 测试策略

| 测试类型 | 位置 | 说明 |
|----------|------|------|
| 单元测试 | `src/**/*.test.ts` | 测试单个函数/模块 |
| 集成测试 | `test/**/*.{test,spec}.ts` | 测试插件功能 |
| 手动测试 | 测试脚本 | 端到端功能验证 |

### 14.2 测试示例

```typescript
// src/plugins/my-plugin.test.ts
import { describe, it, expect } from "vitest";
import MyPlugin from "./my-plugin";

describe("My Plugin", () => {
  it("should register gateway method", () => {
    const mockApi = {
      registerGatewayMethod: vi.fn(),
      registerCommand: };

    const vi.fn()
    plugin = MyPlugin.default || MyPlugin;
    plugin(mockApi);

    expect(mockApi.registerGatewayMethod).toHaveBeenCalledWith(
      "my-plugin.test",
      expect.any(Function)
    );
  });

  it("should handle command", async () => {
    const mockApi = {};
    const plugin = MyPlugin.default || MyPlugin;

    let capturedCtx = null;
    mockApi.registerCommand = ({ handler }) => {
      capturedCtx = {
        channel: "test",
        args: {}
      };
    };

    plugin(mockApi);

    expect(capturedCtx).not.toBeNull();
  });
});
```

### 14.3 CI/CD 要求

```yaml
# .github/workflows/plugin-ci.yml
name: Plugin CI

on:
  push:
    paths:
      - "extensions/my-plugin/**"

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm build
      - run: pnpm test
```

---

## 第十五部分：故障排查

### 15.1 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 插件不加载 | 路径配置错误 | 检查 `plugins.load.paths` |
| 配置不生效 | 未重启 Gateway | 重启 Gateway |
| RPC 方法不存在 | 注册顺序问题 | 检查插件加载顺序 |
| 权限错误 | 插件被禁用 | 检查 `plugins.entries.<id>.enabled` |

### 15.2 调试命令

```bash
# 列出所有插件
openclaw plugins list

# 查看插件详情
openclaw plugins info <plugin-id>

# 检查插件配置
openclaw config get plugins.entries

# 诊断插件问题
openclaw plugins doctor

# 查看插件日志
openclaw logs --follow | grep plugin
```

---

## 总结与进阶

### 知识点回顾

本章节的核心知识点包括：

1. **插件系统架构**：理解插件在 OpenClaw 中的定位和类型
2. **发现机制**：掌握插件扫描顺序和优先级规则
3. **配置管理**：理解插件配置的层次和验证规则
4. **API 使用**：掌握注册 RPC、CLI、命令、服务的方法
5. **渠道开发**：理解消息渠道插件的开发流程
6. **安全模型**：理解插件的安全边界和最佳实践

### 学习成果检验

完成本章节学习后，你应该能够：

- [ ] 说出插件系统的核心价值和使用场景
- [ ] 描述插件发现机制的工作原理
- [ ] 解释插件配置验证的规则
- [ ] 开发简单的自定义插件
- [ ] 理解消息渠道插件的架构设计

### 下一步学习

- **实践项目**：开发一个简单的自定义插件
- **深度阅读**：[插件开发指南](/developer/plugins) — 详细开发文档
- **参考资源**：[插件 SDK](https://github.com/openclaw/plugin-sdk) — 官方 SDK

---

## 附录

### CLI 命令速查

| 命令 | 说明 |
|------|------|
| `openclaw plugins list` | 列出所有插件 |
| `openclaw plugins info <id>` | 查看插件详情 |
| `openclaw plugins install <path>` | 安装插件 |
| `openclaw plugins update <id>` | 更新插件 |
| `openclaw plugins enable <id>` | 启用插件 |
| `openclaw plugins disable <id>` | 禁用插件 |
| `openclaw plugins doctor` | 诊断插件问题 |

### 相关文档链接

- [插件开发](/developer/plugins) — 开发 OpenClaw 插件
- [插件代理工具](/plugins/agent-tools) — 编写插件工具
- [技能系统](/tools/skills) — 技能机制
- [钩子系统](/hooks) — 事件驱动自动化
- [安全指南](/gateway/security) — 安全最佳实践

---

_文档版本：v2.0_
_最后更新：2026-02-05_
