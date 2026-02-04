---
summary: "~/.openclaw/openclaw.json 的所有配置选项及示例"
read_when:
  - "添加或修改配置字段"
title: "配置"
---

# 配置 🔧

OpenClaw 从 `~/.openclaw/openclaw.json` 读取可选的 **JSON5** 配置（允许注释和尾随逗号）。

如果文件缺失，OpenClaw 使用安全的默认值（嵌入式 Pi 代理 + 每个发送者会话 + 工作区 `~/.openclaw/workspace`）。你通常只需要配置来：

- 限制谁可以触发机器人（`channels.whatsapp.allowFrom`、`channels.telegram.allowFrom` 等）
- 控制群组允许列表 + 提及行为（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.discord.guilds`、`agents.list[].groupChat`）
- 自定义消息前缀（`messages`）
- 设置代理的工作区（`agents.defaults.workspace` 或 `agents.list[].workspace`）
- 调整嵌入式代理默认值（`agents.defaults`）和会话行为（`session`）
- 设置每个代理的身份（`agents.list[].identity`）

> **配置新手？** 查看[配置示例](/gateway/configuration-examples)指南，获取带有详细解释的完整示例！

## 严格配置验证

OpenClaw 只接受完全匹配模式的配置。未知键、格式错误的类型或无效值会导致网关**拒绝启动**以确保安全。

当验证失败时：

- 网关不会启动。
- 只允许诊断命令（例如：`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`、`openclaw service`、`openclaw help`）。
- 运行 `openclaw doctor` 查看确切问题。
- 运行 `openclaw doctor --fix`（或 `--yes`）以应用迁移/修复。

Doctor 除非你明确选择 `--fix`/`--yes`，否则永远不会写入更改。

## 模式 + UI 提示

网关通过 `config.schema` 暴露配置的 JSON Schema 表示以供 UI 编辑器使用。控制 UI 从此模式渲染表单，并提供 **Raw JSON** 编辑器作为逃生出口。

通道插件和扩展可以为其配置注册模式 + UI 提示，因此通道设置保持跨应用的模式驱动，无需硬编码表单。

提示（标签、分组、敏感字段）与模式一起提供，以便客户端可以渲染更好的表单，而无需硬编码配置知识。

## 应用 + 重启（RPC）

使用 `config.apply` 验证 + 写入完整配置并一步重启网关。它写入重启标记，并在网关恢复后 ping 最后活动的会话。

警告：`config.apply` 替换**整个配置**。如果你只想更改几个键，使用 `config.patch` 或 `openclaw config set`。保留 `~/.openclaw/openclaw.json` 的备份。

参数：

- `raw`（字符串）— 整个配置的 JSON5 负载
- `baseHash`（可选）— 来自 `config.get` 的配置哈希（当配置已存在时需要）
- `sessionKey`（可选）— 最后活动会话键，用于唤醒 ping
- `note`（可选）— 包含在重启标记中的注释
- `restartDelayMs`（可选）— 重启前的延迟（默认 2000）

示例（通过 `gateway call`）：

```bash
openclaw gateway call config.get --params '{}' # 捕获 payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## 部分更新（RPC）

使用 `config.patch` 将部分更新合并到现有配置中，而不会覆盖无关键。它应用 JSON 合并补丁语义：

- 对象递归合并
- `null` 删除键
- 数组替换
  像 `config.apply` 一样，它验证、写入配置、存储重启标记，并安排网关重启（当提供 `sessionKey` 时，可选唤醒）。

参数：

- `raw`（字符串）— 仅包含要更改的键的 JSON5 负载
- `baseHash`（必填）— 来自 `config.get` 的配置哈希
- `sessionKey`（可选）— 最后活动会话键，用于唤醒 ping
- `note`（可选）— 包含在重启标记中的注释
- `restartDelayMs`（可选）— 重启前的延迟（默认 2000）

示例：

```bash
openclaw gateway call config.get --params '{}' # 捕获 payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## 最小配置（推荐的起点）

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

使用以下命令构建默认镜像一次：

```bash
scripts/sandbox-setup.sh
```

## 自聊天模式（推荐用于群组控制）

要防止机器人响应群组中的 WhatsApp @-提及（仅响应特定文本触发器）：

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // 允许列表仅适用于 DMs；包含你自己的号码启用自聊天模式。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## 配置包含（`$include`）

使用 `$include` 指令将配置拆分为多个文件。这对于以下情况很有用：

- 组织大型配置（例如，每个客户端的代理定义）
- 跨环境共享常见设置
- 保持敏感配置分开

### 基本用法

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // 包含单个文件（替换键的值）
  agents: { $include: "./agents.json5" },

  // 包含多个文件（按顺序深度合并）
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### 合并行为

- **单个文件**：替换包含 `$include` 的对象
- **文件数组**：按顺序深度合并文件（后面的文件覆盖前面的文件）
- **同级键**：同级键在包含之后合并（覆盖包含的值）
- **同级键 + 数组/原始类型**：不支持（包含的内容必须是对象）

```json5
// 同级键覆盖包含的值
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // 结果: { a: 1, b: 99 }
}
```

### 嵌套包含

包含的文件本身可以包含 `$include` 指令（最多 10 层深度）：

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### 路径解析

- **相对路径**：相对于包含文件解析
- **绝对路径**：原样使用
- **父目录**：`../` 引用按预期工作

```json5
{ "$include": "./sub/config.json5" }      // 相对
{ "$include": "/etc/openclaw/base.json5" } // 绝对
{ "$include": "../shared/common.json5" }   // 父目录
```

### 错误处理

- **缺少文件**：显示解析路径的清晰错误
- **解析错误**：显示哪个包含的文件失败
- **循环包含**：检测并报告包含链

### 示例：多客户端法律设置

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // 常见的代理默认值
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // 从所有客户端合并代理列表
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // 合并广播配置
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## 常见选项

### 环境变量 + `.env`

OpenClaw 从父进程读取环境变量（shell、launchd/systemd、CI 等）。

此外，它加载：

- 当前工作目录中的 `.env`（如果存在）
- 全局回退 `.env` 来自 `~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`）

两个 `.env` 文件都不会覆盖现有的环境变量。

你也可以在配置中提供内联环境变量。这些仅在进程环境缺少键时应用（相同的非覆盖规则）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

有关完整优先级和来源，请参阅 [/environment](/environment)。

### `env.shellEnv`（可选）

选择性便利：如果启用且尚未设置任何预期的键，OpenClaw 运行你的登录 shell 并仅导入缺失的预期键（从不覆盖）。这有效地获取了你的 shell 配置。

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

环境变量等效项：

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### 配置中的环境变量替换

你可以使用 `${VAR_NAME}` 语法在任何配置字符串值中直接引用环境变量。变量在配置加载时、验证之前替换。

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**规则：**

- 仅匹配大写环境变量名称：`[A-Z_][A-Z0-9_]*`
- 缺失或为空的环境变量在配置加载时抛出错误
- 用 `$${VAR}` 转义以输出字面的 `${VAR}`
- 适用于 `$include`（包含的文件也会得到替换）

**内联替换：**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### 认证存储（OAuth + API 密钥）

OpenClaw 存储**每个代理**的认证配置文件（OAuth + API 密钥）在：

- `<agentDir>/auth-profiles.json`（默认：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`）

另请参阅：[/concepts/oauth](/concepts/oauth)

旧版 OAuth 导入：

- `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）

嵌入式 Pi 代理在以下位置维护运行时缓存：

- `<agentDir>/auth.json`（自动管理；不要手动编辑）

旧版代理目录（多代理之前）：

- `~/.openclaw/agent/*`（由 `openclaw doctor` 迁移到 `~/.openclaw/agents/<defaultAgentId>/agent/*`）

覆盖：

- OAuth 目录（旧版仅导入）：`OPENCLAW_OAUTH_DIR`
- 代理目录（默认代理根目录覆盖）：`OPENCLAW_AGENT_DIR`（首选）、`PI_CODING_AGENT_DIR`（旧版）

首次使用时，OpenClaw 将 `oauth.json` 条目导入 `auth-profiles.json`。

### `auth`

认证配置文件可选元数据。这**不**存储机密；它将配置文件 ID 映射到提供商 + 模式（以及可选的电子邮件）并定义用于故障转移的提供商轮换顺序。

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

用于默认和用户体验的可选每个代理身份。这由 macOS 入职助手编写。

如果设置，OpenClaw 派生默认值（仅当你没有显式设置它们时）：

- `messages.ackReaction` 来自**活动代理**的 `identity.emoji`（回退到 👀）
- `agents.list[].groupChat.mentionPatterns` 来自代理的 `identity.name`/`identity.emoji`（所以 "@Samantha" 在 Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp 的群组中工作）
- `identity.avatar` 接受工作区相对图像路径或远程 URL/数据 URL。本地文件必须位于代理工作区内。

`identity.avatar` 接受：

- 工作区相对路径（必须保持在代理工作区内）
- `http(s)` URL
- `data:` URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

由 CLI 向导（`onboard`、`configure`、`doctor`）编写的元数据。

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- 默认日志文件：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- 如果你想要稳定路径，设置 `logging.file` 为 `/tmp/openclaw/openclaw.log`。
- 控制台输出可以通过以下方式单独调整：
  - `logging.consoleLevel`（默认为 `info`，当 `--verbose` 时提升到 `debug`）
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- 工具摘要可以编辑以避免泄露机密：
  - `logging.redactSensitive` (`off` | `tools`，默认：`tools`）
  - `logging.redactPatterns`（正则表达式字符串数组；覆盖默认值）

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // 示例：用你自己的规则覆盖默认值。
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

控制如何处理 WhatsApp 直接聊天（DM）：

- `"pairing"`（默认）：未知发送者获得配对代码；所有者必须批准
- `"allowlist"`：仅允许 `channels.whatsapp.allowFrom` 中的发送者（或配对允许存储）
- `"open"`：允许所有入站 DM（**要求** `channels.whatsapp.allowFrom` 包含 `"*"`）
- `"disabled"`：忽略所有入站 DM

配对代码在 1 小时后过期；机器人仅在新请求创建时发送配对代码。挂起的 DM 配对请求默认上限为**每个通道 3 个**。

配对审批：

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

可能触发 WhatsApp 自动回复的 E.164 电话号码允许列表（**仅 DM**）。如果为空且 `channels.whatsapp.dmPolicy="pairing"`，未知发送者将收到配对代码。对于群组，使用 `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // 可选的出站块大小（字符）
      chunkMode: "length", // 可选的块模式（length | newline）
      mediaMaxMb: 50, // 可选的入站媒体上限（MB）
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

控制入站 WhatsApp 消息是否标记为已读（蓝勾）。默认：`true`。

自聊天模式总是跳过已读回执，即使已启用。每个账户覆盖：`channels.whatsapp.accounts.<id>.sendReadReceipts`。

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts`（多账户）

在一个网关中运行多个 WhatsApp 账户：

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // 可选；保持默认 ID 稳定
        personal: {},
        biz: {
          // 可选覆盖。默认：~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

注意：

- 出站命令默认使用账户 `default`（如果存在）；否则使用第一个配置的账户 ID（排序）。
- 旧版单账户 Baileys 认证目录由 `openclaw doctor` 迁移到 `whatsapp/default`。

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

每个通道运行多个账户（每个账户有自己的 `accountId` 和可选的 `name`）：

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

注意：

- `default` 在省略 `accountId` 时使用（CLI + 路由）。
- 环境令牌仅适用于**默认**账户。
- 基础通道设置（群组策略、提及门控等）适用于所有账户，除非每个账户覆盖。
- 使用 `bindings[].match.accountId` 将每个账户路由到不同的 agents.defaults。

### 群聊提及门控（`agents.list[].groupChat` + `messages.groupChat`）

群组消息默认**要求提及**（元数据提及或正则表达式模式）。适用于 WhatsApp、Telegram、Discord、Google Chat 和 iMessage 群聊。

**提及类型：**

- **元数据提及**：原生平台 @-提及（例如，WhatsApp 点击提及）。在 WhatsApp 自聊天模式中忽略（见 `channels.whatsapp.allowFrom`）。
- **文本模式**：在 `agents.list[].groupChat.mentionPatterns` 中定义的正则表达式模式。无论自聊天模式如何，始终检查。
- 提及门控仅在提及检测可能时强制执行（原生提及或至少一个 `mentionPattern`）。

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` 设置群组历史上下文的全局默认值。通道可以使用 `channels.<channel>.historyLimit`（或 `channels.<channel>.accounts.*.historyLimit` 用于多账户）覆盖。设置 `0` 禁用历史包装。

#### DM 历史限制

DM 对话使用代理管理的基于会话的历史记录。你可以限制每个 DM 会话保留的用户回合数：

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // 将 DM 会话限制为 30 个用户回合
      dms: {
        "123456789": { historyLimit: 50 }, // 每个用户的覆盖（用户 ID）
      },
    },
  },
}
```

解析顺序：

1. 每个 DM 覆盖：`channels.<provider>.dms[userId].historyLimit`
2. 提供商默认值：`channels.<provider>.dmHistoryLimit`
3. 无限制（保留所有历史）

支持的提供商：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

每个代理覆盖（设置时优先，即使 `[]`）：

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

提及门控默认值位于每个通道（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups`、`channels.discord.guilds`）。当设置 `*.groups` 时，它也充当群组允许列表；包含 `"*"` 以允许所有群组。

要**仅**响应特定文本触发器（忽略原生 @-提及）：

```json5
{
  channels: {
    whatsapp: {
      // 包含你自己的号码以启用自聊天模式（忽略原生 @-提及）。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // 仅这些文本模式会触发响应
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### 群组策略（每个通道）

使用 `channels.*.groupPolicy` 控制是否完全接受群组/房间消息：

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

注意：

- `"open"`：群组绕过允许列表；提及门控仍然适用。
- `"disabled"`：阻止所有群组/房间消息。
- `"allowlist"`：仅允许匹配配置允许列表的群组/房间。
- `channels.defaults.groupPolicy` 在提供商的 `groupPolicy` 未设置时设置默认值。
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams 使用 `groupAllowFrom`（回退：显式 `allowFrom`）。
- Discord/Slack 使用通道允许列表（`channels.discord.guilds.*.channels`、`channels.slack.channels`）。
- 群组 DM（Discord/Slack）仍由 `dm.groupEnabled` + `dm.groupChannels` 控制。
- 默认是 `groupPolicy: "allowlist"`（除非被 `channels.defaults.groupPolicy` 覆盖）；如果未配置允许列表，群组消息会被阻止。

### 多代理路由（`agents.list` + `bindings`）

在一个网关内运行多个隔离的代理（单独的工作区、`agentDir`、会话）。入站消息通过绑定路由到代理。

- `agents.list[]`：每个代理覆盖。
  - `id`：稳定的代理 ID（必填）。
  - `default`：可选；当设置多个时，第一个获胜并记录警告。
    如果都没有设置，**列表中的第一个条目**是默认代理。
  - `name`：代理的显示名称。
  - `workspace`：默认 `~/.openclaw/workspace-<agentId>`（对于 `main`，回退到 `agents.defaults.workspace`）。
  - `agentDir`：默认 `~/.openclaw/agents/<agentId>/agent`。
  - `model`：每个代理默认模型，覆盖该代理的 `agents.defaults.model`。
    - 字符串形式：`"provider/model"`，仅覆盖 `agents.defaults.model.primary`
    - 对象形式：`{ primary, fallbacks }`（回退覆盖 `agents.defaults.model.fallbacks`；`[]` 禁用该代理的全局回退）
  - `identity`：每个代理名称/主题/表情符号（用于提及模式 + 确认反应）。
  - `groupChat`：每个代理提及门控（`mentionPatterns`）。
  - `sandbox`：每个代理沙箱配置（覆盖 `agents.defaults.sandbox`）。
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: 自定义沙箱工作区根目录
    - `docker`: 每个代理 docker 覆盖（例如 `image`、`network`、`env`、`setupCommand`、限制；当 `scope: "shared"` 时忽略）
    - `browser`: 每个代理沙箱浏览器覆盖（当 `scope: "shared"` 时忽略）
    - `prune`: 每个代理沙箱清理覆盖（当 `scope: "shared"` 时忽略）
  - `subagents`: 每个代理子代理默认值。
    - `allowAgents`: 允许从该代理使用 `sessions_spawn` 的代理 ID 列表（`["*"]` = 允许任何；默认：仅相同代理）
  - `tools`: 每个代理工具限制（在沙箱工具策略之前应用）。
    - `profile`: 基础工具配置文件（应用在允许/拒绝之前）
    - `allow`: 允许的工具名称数组
    - `deny`: 拒绝的工具名称数组（拒绝优先）
- `agents.defaults`: 共享代理默认值（模型、工作区、沙箱等）。
- `bindings[]`: 将入站消息路由到 `agentId`。
  - `match.channel`（必填）
  - `match.accountId`（可选；`*` = 任何账户；省略 = 默认账户）
  - `match.peer`（可选；`{ kind: dm|group|channel, id }`）
  - `match.guildId` / `match.teamId`（可选；通道特定）

确定性匹配顺序：

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（精确，无 peer/guild/team）
5. `match.accountId: "*"`（通道范围，无 peer/guild/team）
6. 默认代理（`agents.list[].default`，否则列表第一个条目，否则 `"main"`）

在每个匹配层级中，`bindings` 中第一个匹配的条目获胜。

#### 每个代理访问配置文件（多代理）

每个代理可以携带自己的沙箱 + 工具策略。使用这个在一个网关上混合访问级别：

- **完全访问**（个人代理）
- **只读**工具 + 工作区
- **无文件系统访问**（仅消息/会话工具）

有关优先级和其他示例，请参阅[多代理沙箱和工具](/multi-agent-sandbox-tools)。

完全访问（无沙箱）：

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

只读工具 + 只读工作区：

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

无文件系统访问（启用消息/会话工具）：

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

示例：两个 WhatsApp 账户 → 两个代理：

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent`（可选）

代理到代理消息是选择加入的：

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `messages.queue`

控制入站消息在代理运行已活动时的行为。

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

对来自**同一发送者**的快速入站消息进行去抖动，以便多个连续消息变成单个代理回合。去抖动按每个通道 + 对话范围，并使用最新的消息进行回复线程/ID。

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 禁用
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

注意：

- 去抖动批量处理**纯文本**消息；媒体/附件立即刷新。
- 控制命令（例如 `/queue`、`/new`）绕过去抖动，因此它们保持独立。

### `commands`（聊天命令处理）

控制聊天命令如何在连接器之间启用。

```json5
{
  commands: {
    native: "auto", // 在支持时注册原生命令（自动）
    text: true, // 在聊天消息中解析斜杠命令
    bash: false, // 允许 ! (别名: /bash)（仅主机；需要 tools.elevated 允许列表）
    bashForegroundMs: 2000, // bash 前台窗口（0 立即进入后台）
    config: false, // 允许 /config（写入磁盘）
    debug: false, // 允许 /debug（运行时仅覆盖）
    restart: false, // 允许 /restart + 网关工具重启操作
    useAccessGroups: true, // 对命令强制执行访问组允许列表/策略
  },
}
```

注意：

- 文本命令必须作为**独立**消息发送，并使用前导 `/`（无纯文本别名）。
- `commands.text: false` 禁用解析聊天消息中的命令。
- `commands.native: "auto"`（默认）为 Discord/Telegram 开启原生命令，并让 Slack 保持关闭；不支持的通道保持纯文本。
- 设置 `commands.native: true|false` 以强制全部，或使用 `channels.discord.commands.native`、`channels.telegram.commands.native`、`channels.slack.commands.native`（布尔值或 `"auto"`）按通道覆盖。`false` 在启动时清除 Discord/Telegram 上先前注册的命令；Slack 命令在 Slack 应用中管理。
- `channels.telegram.customCommands` 添加额外的 Telegram 机器人菜单条目。名称被标准化；与原生命令的冲突被忽略。
- `commands.bash: true` 启用 `! <cmd>` 来运行主机 shell 命令（`/bash <cmd>` 也可以作为别名）。需要 `tools.elevated.enabled` 并在 `tools.elevated.allowFrom.<channel>` 中允许发送者。
- `commands.bashForegroundMs` 控制 bash 在进入后台之前等待多长时间。当 bash 作业运行时，新的 `! <cmd>` 请求被拒绝（一次一个）。
- `commands.config: true` 启用 `/config`（读取/写入 `openclaw.json`）。
- `channels.<provider>.configWrites` 阻止该通道发起的配置更改（包括 `/config set|unset` 以及提供商特定的自动迁移，如 Telegram 超级群组 ID 更改、Slack 通道 ID 更改）。
- `commands.debug: true` 启用 `/debug`（运行时仅覆盖）。
- `commands.restart: true` 启用 `/restart` 和网关工具重启操作。
- `commands.useAccessGroups: false` 允许命令绕过访问组允许列表/策略。
- 斜杠命令和指令仅对**授权发送者**生效。授权来自通道允许列表/配对加上 `commands.useAccessGroups`。

### `web`（WhatsApp 网页通道运行时）

WhatsApp 通过网关的网页通道（Baileys Web）运行。当链接的会话存在时，它会自动启动。设置 `web.enabled: false` 以默认保持关闭。

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram`（机器人传输）

仅当存在 `channels.telegram` 配置部分时，OpenClaw 才会启动 Telegram。机器人令牌从 `channels.telegram.botToken`（或 `channels.telegram.tokenFile`）解析，`TELEGRAM_BOT_TOKEN` 作为默认账户的回退。设置 `channels.telegram.enabled: false` 以禁用自动启动。多账户支持位于 `channels.telegram.accounts`（见上文多账户部分）。环境令牌仅适用于默认账户。设置 `channels.telegram.configWrites: false` 以阻止 Telegram 发起的配置写入（包括超级群组 ID 迁移和 `/config set|unset`）。

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // 可选；"open" 需要 ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // 作为上下文包含最后 N 条群组消息（0 禁用）
      replyToMode: "first", // off | first | all
      linkPreview: true, // 切换出站链接预览
      streamMode: "partial", // off | partial | block（草稿流式传输；与块流式传输分开）
      draftChunk: {
        // 可选；仅用于 streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // 工具操作门控（false 禁用）
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // 出站重试策略
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // 传输覆盖
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

草稿流式传输说明：

- 使用 Telegram `sendMessageDraft`（草稿气泡，不是真实消息）。
- 需要**私聊话题**（DM 中的 message_thread_id；机器人已启用话题）。
- `/reasoning stream` 将推理流式传输到草稿，然后发送最终答案。
  重试策略默认和行为在[重试策略](/concepts/retry)中记录。

### `channels.discord`（机器人传输）

通过设置机器人令牌和可选门控来配置 Discord 机器人。多账户支持位于 `channels.discord.accounts`（见上文多账户部分）。环境令牌仅适用于默认账户。

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // 钳制入站媒体大小
      allowBots: false, // 允许机器人作者的消息
      actions: {
        // 工具操作门控（false 禁用）
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dm: {
        enabled: true, // false 时禁用所有 DM
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // 可选 DM 允许列表（"open" 需要 ["*"]）
        groupEnabled: false, // 启用群组 DM
        groupChannels: ["openclaw-dm"], // 可选群组 DM 允许列表
      },
      guilds: {
        "123456789012345678": {
          // 公会 ID（首选）或 slug
          slug: "friends-of-openclaw",
          requireMention: false, // 每个公会默认
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // 可选每个公会用户允许列表
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // 作为上下文包含最后 N 条公会消息
      textChunkLimit: 2000, // 可选的出站文本块大小（字符）
      chunkMode: "length", // 可选的块模式（length | newline）
      maxLinesPerMessage: 17, // 每条消息的软最大行数（Discord UI 裁剪）
      retry: {
        // 出站重试策略
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

仅当存在 `channels.discord` 配置部分时，OpenClaw 才会启动 Discord。令牌从 `channels.discord.token` 解析，`DISCORD_BOT_TOKEN` 作为默认账户的回退（除非 `channels.discord.enabled` 为 `false`）。在为 cron/CLI 命令指定传递目标时使用 `user:<id>`（DM）或 `channel:<id>`（公会通道）；裸数字 ID 是模糊的并被拒绝。公会 slug 是小写的，空格替换为 `-`；通道键使用 slug 化的通道名称（无前导 `#`）。优先使用公会 ID 作为键以避免重命名模糊性。默认忽略机器人作者的消息。使用 `channels.discord.allowBots` 启用（自己的消息仍会被过滤以防止自回复循环）。反应通知模式：

- `off`：无反应事件。
- `own`：机器人自己消息上的反应（默认）。
- `all`：所有消息上的所有反应。
- `allowlist`：所有消息上来自 `guilds.<id>.users` 的反应（空列表禁用）。
  出站文本按 `channels.discord.textChunkLimit`（默认 2000）分块。设置 `channels.discord.chunkMode="newline"` 在长度分块之前按空行（段落边界）分割。Discord 客户端可以裁剪非常高的消息，因此 `channels.discord.maxLinesPerMessage`（默认 17）即使在 2000 字符以下也会分割长的多行回复。
  重试策略默认和行为在[重试策略](/concepts/retry)中记录。

### `channels.googlechat`（Chat API webhook）

Google Chat 通过带有应用级认证（服务账户）的 HTTP Webhook 运行。多账户支持位于 `channels.googlechat.accounts`（见上文多账户部分）。环境变量仅适用于默认账户。

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选；改善提及检测
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // 可选；"open" 需要 ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

注意：

- 服务账户 JSON 可以是内联的（`serviceAccount`）或基于文件的（`serviceAccountFile`）。
- 默认账户的环境回退：`GOOGLE_CHAT_SERVICE_ACCOUNT` 或 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
- `audienceType` + `audience` 必须匹配 Chat 应用的 webhook 认证配置。
- 设置传递目标时使用 `spaces/<spaceId>` 或 `users/<userId|email>`。

### `channels.slack`（socket 模式）

Slack 在 Socket 模式下运行，需要机器人令牌和应用令牌：

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // 可选；"open" 需要 ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50, // 作为上下文包含最后 N 条通道/群组消息（0 禁用）
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

多账户支持位于 `channels.slack.accounts`（见上文多账户部分）。环境令牌仅适用于默认账户。

当提供商启用且两个令牌都设置时（通过配置或 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`），OpenClaw 启动 Slack。在为 cron/CLI 命令指定传递目标时使用 `user:<id>`（DM）或 `channel:<id>`。设置 `channels.slack.configWrites: false` 以阻止 Slack 发起的配置写入（包括通道 ID 迁移和 `/config set|unset`）。

默认忽略机器人作者的消息。使用 `channels.slack.allowBots` 或 `channels.slack.channels.<id>.allowBots` 启用。

反应通知模式：

- `off`：无反应事件。
- `own`：机器人自己消息上的反应（默认）。
- `all`：所有消息上的所有反应。
- `allowlist`：所有消息上来自 `channels.slack.reactionAllowlist` 的反应（空列表禁用）。

线程会话隔离：

- `channels.slack.thread.historyScope` 控制线程历史是每个线程（`thread`，默认）还是在通道中共享（`channel`）。
- `channels.slack.thread.inheritParent` 控制新的线程会话是否继承父通道的记录（默认：false）。

Slack 操作组（门控 `slack` 工具操作）：

| 操作组 | 默认 | 备注 |
| --- | --- | --- |
| reactions | 启用 | 反应 + 列出反应 |
| messages | 启用 | 读取/发送/编辑/删除 |
| pins | 启用 | 固定/取消固定/列出 |
| memberInfo | 启用 | 成员信息 |
| emojiList | 启用 | 自定义表情符号列表 |

### `channels.mattermost`（机器人令牌）

Mattermost 作为插件提供，不随附核心安装。首先安装它：`openclaw plugins install @openclaw/mattermost`（或来自 git 检出的 `./extensions/mattermost`）。

Mattermost 需要机器人令牌加上服务器的 base URL：

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

当账户配置好（机器人令牌 + base URL）并启用时，OpenClaw 启动 Mattermost。令牌 + base URL 从 `channels.mattermost.botToken` + `channels.mattermost.baseUrl` 或默认账户的 `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` 解析（除非 `channels.mattermost.enabled` 为 `false`）。

聊天模式：

- `oncall`（默认）：仅在被 @ 提及时响应通道消息。
- `onmessage`：响应每条通道消息。
- `onchar`：当消息以触发前缀开头时响应（`channels.mattermost.oncharPrefixes`，默认 `[" >", "!"]`）。

访问控制：

- 默认 DM：`channels.mattermost.dmPolicy="pairing"`（未知发送者获得配对代码）。
- 公共 DM：`channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`。
- 群组：`channels.mattermost.groupPolicy="allowlist"` 默认（提及门控）。使用 `channels.mattermost.groupAllowFrom` 限制发送者。

多账户支持位于 `channels.mattermost.accounts`（见上文多账户部分）。环境变量仅适用于默认账户。在指定传递目标时使用 `channel:<id>` 或 `user:<id>`（或 `@username`）；裸 ID 被视为通道 ID。

### `channels.signal`（signal-cli）

Signal 反应可以发出系统事件（共享反应工具）：

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // 作为上下文包含最后 N 条群组消息（0 禁用）
    },
  },
}
```

反应通知模式：

- `off`：无反应事件。
- `own`：机器人自己消息上的反应（默认）。
- `all`：所有消息上的所有反应。
- `allowlist`：所有消息上来自 `channels.signal.reactionAllowlist` 的反应（空列表禁用）。

### `channels.imessage`（imsg CLI）

OpenClaw 生成 `imsg rpc`（通过 stdio 的 JSON-RPC）。不需要守护进程或端口。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // 使用 SSH 包装器时用于远程附件的 SCP
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // 作为上下文包含最后 N 条群组消息（0 禁用）
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

多账户支持位于 `channels.imessage.accounts`（见上文多账户部分）。

注意：

- 需要对消息数据库的完全磁盘访问。
- 第一次发送将提示消息自动化权限。
- 优先使用 `chat_id:<id>` 目标。使用 `imsg chats --limit 20` 列出聊天。
- `channels.imessage.cliPath` 可以指向包装脚本（例如 `ssh` 到另一台运行 `imsg rpc` 的 Mac）；使用 SSH 密钥避免密码提示。
- 对于远程 SSH 包装器，当 `includeAttachments` 启用时，设置 `channels.imessage.remoteHost` 以通过 SCP 获取附件。

示例包装脚本：

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

设置代理用于文件操作的**单个全局工作区目录**。

默认：`~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

如果启用了 `agents.defaults.sandbox`，非主会话可以使用 `agents.defaults.sandbox.workspaceRoot` 下的自己的每个作用域工作区覆盖这个。

### `agents.defaults.repoRoot`

可选的仓库根目录，显示在系统提示的 Runtime 行中。如果未设置，OpenClaw 尝试通过从工作区（和当前工作目录）向上走来检测 `.git` 目录。该路径必须存在才能使用。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

禁用自动创建工作区引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md` 和 `BOOTSTRAP.md`）。

用于你的工作区文件来自仓库的预种子部署。

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

注入系统提示之前每个工作区引导文件的最大字符数。默认：`20000`。

当文件超过此限制时，OpenClaw 记录警告并注入带有标记的截断头/尾。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

设置用户的时区用于**系统提示上下文**（不用于消息信封中的时间戳）。如果未设置，OpenClaw 在运行时使用主机时区。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

控制系统提示的当前日期和时间部分中显示的**时间格式**。默认：`auto`（操作系统偏好）。

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `messages`

控制入站/出站前缀和可选的确认反应。
请参阅[消息](/concepts/messages)了解队列、会话和流式传输上下文。

```json5
{
  messages: {
    responsePrefix: "🦞", // 或 "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` 应用于**所有出站回复**（工具摘要、块流式传输、最终回复）跨通道，除非已存在。

如果 `messages.responsePrefix` 未设置，默认不应用前缀。WhatsApp 自聊天回复是例外：当设置时默认为 `[{identity.name}]`，否则为 `[openclaw]`，以便同手机对话保持可读。将其设置为 `"auto"` 以派生路由代理的 `[{identity.name}]`（当设置时）。

#### 模板变量

`responsePrefix` 字符串可以包括动态解析的模板变量：

| 变量 | 描述 | 示例 |
| --- | --- | --- |
| `{model}` | 短模型名称 | `claude-opus-4-5`, `gpt-4o` |
| `{modelFull}` | 完整模型标识符 | `anthropic/claude-opus-4-5` |
| `{provider}` | 提供商名称 | `anthropic`, `openai` |
| `{thinkingLevel}` | 当前思考级别 | `high`, `low`, `off` |
| `{identity.name}` | 代理身份名称 | （与 `"auto"` 模式相同） |

变量不区分大小写（`{MODEL}` = `{model}`）。`{think}` 是 `{thinkingLevel}` 的别名。未解析的变量保持为字面文本。

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

示例输出：`[claude-opus-4-5 | think:high] Here's my response...`

WhatsApp 入站前缀通过 `channels.whatsapp.messagePrefix` 配置（旧版：`messages.messagePrefix`）。默认保持**不变**：当 `channels.whatsapp.allowFrom` 为空时为 `"[openclaw]"`，否则为 `""`（无前缀）。使用 `"[openclaw]"` 时，当路由的代理设置了 `identity.name`，OpenClaw 将改用 `[{identity.name}]`。

`ackReaction` 发送尽力而为的表情符号反应以确认入站消息，在支持反应的通道上（Slack/Discord/Telegram/Google Chat）。默认设置为活动代理的 `identity.emoji`（如果设置），否则为 `"👀"`。设置为 `""` 以禁用。

`ackReactionScope` 控制反应触发的时间：

- `group-mentions`（默认）：仅当群组/房间要求提及**并且**机器人被提及时
- `group-all`：所有群组/房间消息
- `direct`：仅直接消息
- `all`：所有消息

`removeAckAfterReply` 在发送回复后移除机器人的确认反应

...
