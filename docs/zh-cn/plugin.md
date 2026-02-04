---
summary: "OpenClaw 插件系统：发现、配置、安全性和开发指南"
read_when:
  - 你需要添加或修改插件/扩展
  - 你想了解插件的发现机制和加载规则
  - 你想开发自定义插件
title: "插件（Plugins）"
---

# 插件（Plugins）

插件是扩展 OpenClaw 功能的小型代码模块，可以添加**命令、工具和 Gateway RPC**。

大多数情况下，当你需要的功能不在核心 OpenClaw 中时，可以使用插件。

## 快速入门

### 查看已加载的插件

```bash
openclaw plugins list
```

### 安装官方插件（以语音通话为例）

```bash
openclaw plugins install @openclaw/voice-call
```

### 重启 Gateway 并配置

安装后重启 Gateway，然后在 `plugins.entries.<id>.config` 下配置。

详见 [语音通话插件](/plugins/voice-call)。

## 官方插件列表

| 插件 | 说明 |
|------|------|
| Memory（Core）| 捆绑的内存搜索插件（默认通过 `plugins.slots.memory` 启用）|
| Memory（LanceDB）| 捆绑的长期内存插件（设置 `plugins.slots.memory = "memory-lancedb"`）|
| [语音通话](/plugins/voice-call) | `@openclaw/voice-call` |
| [Zalo 个人](/plugins/zalouser) | `@openclaw/zalouser` |
| [Matrix](/channels/matrix) | `@openclaw/matrix` |
| [Nostr](/channels/nostr) | `@openclaw/nostr` |
| [Zalo](/channels/zalo) | `@openclaw/zalo` |
| [Microsoft Teams](/channels/msteams) | `@openclaw/msteams` |
| Google Antigravity OAuth | 捆绑为 `google-antigravity-auth`（默认禁用）|
| Gemini CLI OAuth | 捆绑为 `google-gemini-cli-auth`（默认禁用）|
| Qwen OAuth | 捆绑为 `qwen-portal-auth`（默认禁用）|
| Copilot Proxy | 本地 VS Code Copilot Proxy 桥接（捆绑，默认禁用）|

OpenClaw 插件是**TypeScript 模块**，在运行时通过 jiti 加载。**配置验证不执行插件代码**，而是使用插件清单和 JSON Schema。

## 插件功能

插件可以注册：

- Gateway RPC 方法
- Gateway HTTP 处理器
- 代理工具
- CLI 命令
- 后台服务
- 可选配置验证
- **技能**（在插件清单中列出 `skills` 目录）
- **自动回复命令**（无需调用 AI 代理即可执行）

插件与 Gateway **进程内**运行，因此将其视为可信代码。

工具编写指南：[插件代理工具](/plugins/agent-tools)。

## 运行时辅助函数

插件可以通过 `api.runtime` 访问核心辅助函数。用于电话 TTS：

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

注意：

- 使用核心 `messages.tts` 配置（OpenAI 或 ElevenLabs）
- 返回 PCM 音频缓冲区和采样率。插件必须为提供商重新采样/编码
- Edge TTS 不支持电话场景

## 发现机制与优先级

OpenClaw 按以下顺序扫描：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | `plugins.load.paths` | 配置文件中指定的路径 |
| 2 | `<workspace>/.openclaw/extensions/*.ts` | 工作区扩展 |
| 3 | `<workspace>/.openclaw/extensions/*/index.ts` | 工作区扩展目录 |
| 4 | `~/.openclaw/extensions/*.ts` | 用户全局扩展 |
| 5 | `~/.openclaw/extensions/*/index.ts` | 用户全局扩展目录 |
| 6 | `<openclaw>/extensions/*` | 捆绑扩展（默认禁用）|

捆绑插件必须通过 `plugins.entries.<id>.enabled` 或 `openclaw plugins enable <id>` 显式启用。安装的插件默认启用，但也可以通过相同方式禁用。

每个插件必须在根目录包含 `openclaw.plugin.json` 文件。如果路径指向文件，插件根目录必须是该文件所在目录，且必须包含清单文件。

如果多个插件解析为相同 ID，顺序中第一个匹配获胜，后续被忽略。

### 包包

插件目录可以包含带有 `openclaw.extensions` 的 `package.json`：

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

每个条目成为一个插件。如果包列出多个扩展，插件 ID 变为 `name/<fileBase>`。

如果插件导入 npm 依赖，在该目录安装它们以便 `node_modules` 可用（`npm install` / `pnpm install`）。

### 渠道目录元数据

渠道插件可以通过 `openclaw.channel` 广告引导元数据，通过 `openclaw.install` 提供安装提示。这使核心目录数据保持无数据。

示例：

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Self-hosted chat via Nextcloud Talk webhook bots.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw 还可以合并**外部渠道目录**（例如 MPM 注册表导出）。将 JSON 文件放在：

- `~/.openclaw/mpm/plugins.json`
- `~/.openclaw/mpm/catalog.json`
- `~/.openclaw/plugins/catalog.json`

或通过 `OPENCLAW_PLUGIN_CATALOG_PATHS`（或 `OPENCLAW_MPM_CATALOG_PATHS`）指向一个或多个 JSON 文件（逗号/分号/`PATH` 分隔）。每个文件应包含 `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`。

## 插件 ID

默认插件 ID：

- 包包：`package.json` `name`
- 独立文件：文件基本名（`~/.../voice-call.ts` → `voice-call`）

如果插件导出 `id`，OpenClaw 会使用它，但在不匹配配置的 ID 时会发出警告。

## 配置

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

配置字段：

- `enabled`：主开关（默认：true）
- `allow`：允许列表（可选）
- `deny`：拒绝列表（可选；deny 优先）
- `load.paths`：额外插件文件/目录
- `entries.<id>`：每个插件的开关 + 配置

配置更改**需要重启 Gateway**。

验证规则（严格）：

- `entries`、`allow`、`deny` 或 `slots` 中未知的插件 ID 是**错误**
- 未知的 `channels.<id>` 键是**错误**，除非插件清单声明了渠道 ID
- 插件配置使用 `openclaw.plugin.json` 中嵌入的 JSON Schema（`configSchema`）进行验证
- 如果插件被禁用，其配置被保留并发出**警告**

## 插件槽位（独占类别）

某些插件类别是**独占的**（同时只有一个活动）。使用 `plugins.slots` 选择哪个插件拥有槽位：

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // 或 "none" 禁用内存插件
    },
  },
}
```

如果多个插件声明 `kind: "memory"`，只加载选定的插件。其他插件被禁用并发出诊断信息。

## 控制 UI（Schema + 标签）

控制 UI 使用 `config.schema`（JSON Schema + `uiHints`）呈现更好的表单。

OpenClaw 在运行时根据发现的插件增强 `uiHints`：

- 为 `plugins.entries.<id>` / `.enabled` / `.config` 添加每个插件标签
- 将可选的插件提供的配置字段提示合并到：`plugins.entries.<id>.config.<field>`

如果希望插件配置字段显示良好的标签/占位符（并将标记为敏感），请在插件清单中提供 `uiHints` 和 JSON Schema。

示例：

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Region", "placeholder": "us-east-1" }
  }
}
```

## CLI 命令

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # 将本地文件/目录复制到 ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # 相对路径也可以
openclaw plugins install ./plugin.tgz           # 从本地 tarball 安装
openclaw plugins install ./plugin.zip           # 从本地 zip 安装
openclaw plugins install -l ./extensions/voice-call # 链接（不复制）用于开发
openclaw plugins install @openclaw/voice-call # 从 npm 安装
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` 仅对 `plugins.installs` 下跟踪的 npm 安装有效。

插件还可以注册自己的顶级命令（例如 `openclaw voicecall`）。

## 插件 API 概述

插件导出：

- 函数：`(api) => { ... }`
- 对象：`{ id, name, configSchema, register(api) { ... } }`

## 插件钩子

插件可以在运行时附带钩子并注册。这允许插件捆绑事件驱动的自动化，无需单独的钩子包安装。

### 示例

```typescript
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

注意：

- 钩子目录遵循正常钩子结构（`HOOK.md` + `handler.ts`）
- 钩子资格规则仍然适用（OS/二进制/环境/配置要求）
- 插件管理的钩子显示在 `openclaw hooks list` 中，带有 `plugin:<id>` 前缀
- 你无法通过 `openclaw hooks` 启用/禁用插件管理的钩子；而是启用/禁用插件

## 提供商插件（模型认证）

插件可以注册**模型提供商认证**流程，以便用户可以在 OpenClaw 中运行 OAuth 或 API 密钥设置（无需外部脚本）。

通过 `api.registerProvider(...)` 注册提供商。每个提供商暴露一个或多个认证方法（OAuth、API 密钥、设备代码等）。这些方法支持：

- `openclaw models auth login --provider <id> [--method <id>]`

示例：

```typescript
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // 运行 OAuth 流程并返回认证配置
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

注意：

- `run` 接收带有 `prompter`、`runtime`、`openUrl` 和 `oauth.createVpsAwareHelpers` 帮助函数的 `ProviderAuthContext`
- 当需要添加默认模型或提供商配置时返回 `configPatch`
- 返回 `defaultModel` 以便 `--set-default` 可以更新代理默认值

### 注册消息渠道

插件可以注册**渠道插件**，其行为类似于内置渠道（WhatsApp、Telegram 等）。渠道配置位于 `channels.<id>` 下，由你的渠道插件代码验证。

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

注意：

- 将配置放在 `channels.<id>` 下（不是 `plugins.entries`）
- `meta.label` 用于 CLI/UI 列表中的标签
- `meta.aliases` 为规范化添加替代 ID 和 CLI 输入
- `meta.preferOver` 列出当两者都配置时要跳过的渠道 ID
- `meta.detailLabel` 和 `meta.systemImage` 让 UI 显示更丰富的渠道标签/图标

### 编写新消息渠道（逐步指南）

当你想要新的聊天界面（"消息渠道"）而不是模型提供商时使用此方法。模型提供商文档位于 `/providers/*`。

1. **选择 ID + 配置形状**
   - 所有渠道配置位于 `channels.<id>` 下
   - 多账户设置首选 `channels.<id>.accounts.<accountId>`

2. **定义渠道元数据**
   - `meta.label`、`meta.selectionLabel`、`meta.docsPath`、`meta.blurb` 控制 CLI/UI 列表
   - `meta.docsPath` 应指向类似 `/channels/<id>` 的文档页面
   - `meta.preferOver` 让插件替换另一个渠道（自动启用时首选）
   - `meta.detailLabel` 和 `meta.systemImage` 由 UI 用于详细文本/图标

3. **实现所需的适配器**
   - `config.listAccountIds` + `config.resolveAccount`
   - `capabilities`（聊天类型、媒体、线程等）
   - `outbound.deliveryMode` + `outbound.sendText`（用于基本发送）

4. **根据需要添加可选适配器**
   - `setup`（向导）、`security`（DM 策略）、`status`（健康/诊断）
   - `gateway`（启动/停止/登录）、`mentions`、`threading`、`streaming`
   - `actions`（消息操作）、`commands`（本机命令行为）

5. **在插件中注册渠道**
   - `api.registerChannel({ plugin })`

最小配置示例：

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

最小渠道插件（仅出站）：

```typescript
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // 在这里将 `text` 传递到你的渠道
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

加载插件（扩展目录或 `plugins.load.paths`），重启 Gateway，然后配置 `channels.<id>`。

### 代理工具

见专用指南：[插件代理工具](/plugins/agent-tools)。

### 注册 Gateway RPC 方法

```typescript
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### 注册 CLI 命令

```typescript
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### 注册自动回复命令

插件可以注册自定义斜杠命令，这些命令**无需调用 AI 代理**即可执行。这对于切换命令、状态检查或不需要 LLM 处理的快速操作很有用。

```typescript
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

命令处理程序上下文：

- `senderId`：发送者的 ID（如果可用）
- `channel`：命令发送到的渠道
- `isAuthorizedSender`：发送者是否是授权用户
- `args`：命令后传递的参数（如果 `acceptsArgs: true`）
- `commandBody`：完整命令文本
- `config`：当前 OpenClaw 配置

命令选项：

- `name`：命令名称（不带前导 `/`）
- `description`：命令列表中显示的帮助文本
- `acceptsArgs`：命令是否接受参数（默认：false）。如果为 false 且提供了参数，命令将不匹配，消息会传递给其他处理程序
- `requireAuth`：是否需要授权发送者（默认：true）
- `handler`：返回 `{ text: string }` 的函数（可以是 async）

带授权和参数的示例：

```typescript
api.registerCommand({
  name: "setmode",
  description: "Set plugin mode",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

注意：

- 插件命令在**内置命令和 AI 代理之前**处理
- 命令全局注册并适用于所有渠道
- 命令名称不区分大小写（`/MyStatus` 匹配 `/mystatus`）
- 命令名称必须以字母开头，并且只能包含字母、数字、连字符和下划线
- 保留的命令名称（如 `help`、`status`、`reset` 等）不能被插件覆盖
- 跨插件的重复命令注册将因诊断错误而失败

### 注册后台服务

```typescript
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

## 命名约定

- Gateway 方法：`pluginId.action`（例如 `voicecall.status`）
- 工具：`snake_case`（例如 `voice_call`）
- CLI 命令：kebab 或 camel，但避免与核心命令冲突

## 技能

插件可以在仓库中附带技能（`skills/<name>/SKILL.md`）。通过 `plugins.entries.<id>.enabled`（或其他配置门）启用，并确保它出现在你的工作区/托管技能位置。

## 分发（npm）

推荐打包方式：

- 主包：`openclaw`（此仓库）
- 插件：`@openclaw/*` 下的单独 npm 包（例如 `@openclaw/voice-call`）

发布约定：

- 插件 `package.json` 必须包含 `openclaw.extensions`，带有一个或多个入口文件
- 入口文件可以是 `.js` 或 `.ts`（jiti 在运行时加载 TS）
- `openclaw plugins install <npm-spec>` 使用 `npm pack`，提取到 `~/.openclaw/extensions/<id>/`，并在配置中启用
- 配置键稳定性：带范围的包规范化为**无范围** ID，用于 `plugins.entries.*`

## 示例插件：语音通话

此仓库包括语音通话插件（Twilio 或日志回退）：

- 源码：`extensions/voice-call`
- 技能：`skills/voice-call`
- CLI：`openclaw voicecall start|status`
- 工具：`voice_call`
- RPC：`voicecall.start`, `voicecall.status`
- 配置（twilio）：`provider: "twilio"` + `twilio.accountSid/authToken/from`（可选 `statusCallbackUrl`, `twimlUrl`）
- 配置（dev）：`provider: "log"`（无网络）

详见 [语音通话](/plugins/voice-call) 和 `extensions/voice-call/README.md`。

## 安全说明

插件与 Gateway 进程内运行。将它们视为可信代码：

- 只安装你信任的插件
- 优先使用 `plugins.allow` 允许列表
- 更改后重启 Gateway

## 测试插件

插件可以（也应该）附带测试：

- 仓库内插件可以将 Vitest 测试保留在 `src/**` 下（例如 `src/plugins/voice-call.plugin.test.ts`）
- 单独发布的插件应运行自己的 CI（lint/build/test）并验证 `openclaw.extensions` 指向构建的入口点（`dist/index.js`）

## 相关文档

- [插件开发](/developer/plugins) - 开发 OpenClaw 插件
- [插件代理工具](/plugins/agent-tools) - 编写插件工具
- [技能](/tools/skills) - 技能系统
- [钩子](/hooks) - 事件驱动自动化
