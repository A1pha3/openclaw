---
summary: "斜杠命令：文本 vs 本机、配置和支持的命令"
read_when:
  - 使用或配置聊天命令
  - 调试命令路由或权限
title: "斜杠命令"
---

# 斜杠命令

命令由网关处理。大多数命令必须作为以 `/` 开头的**独立**消息发送。主机-only bash 聊天命令使用 `! <cmd>`（带有 `/bash <cmd>` 作为别名）。

有两个相关系统：

- **命令**：独立的 `/...` 消息。
- **指令**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/exec`、`/model`、`/queue`。
  - 指令在模型看到消息之前从消息中剥离。
  - 在正常聊天消息中（不是仅指令），它们被视为"内联提示"并且**不**持久化会话设置。
  - 在仅指令消息中（消息只包含指令），它们持久化到会话并回复确认。
  - 指令仅对**授权发送者**（频道 allowlist/配对加上 `commands.useAccessGroups`）应用。
    未经授权的发送者看到指令被视为纯文本。

还有一些**内联快捷方式**（仅限 allowlisted/授权发送者）：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。它们立即运行，在模型看到消息之前被剥离，剩余文本继续通过正常流程。

## 配置

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true,
  },
}
```

- `commands.text`（默认 `true`）启用在聊天消息中解析 `/...`。
  - 在没有本机命令的表面（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams），即使你将其设置为 `false`，文本命令仍然工作。
- `commands.native`（默认 `"auto"`）注册本机命令。
  - 自动：Discord/Telegram 开启；Slack 关闭（直到你添加斜杠命令）；对于没有本机支持的提供程序被忽略。
  - 设置 `channels.discord.commands.native`、`channels.telegram.commands.native` 或 `channels.slack.commands.native` 以覆盖每个提供程序（布尔值或 `"auto"`）。
  - `false` 在启动时清除 Discord/Telegram 上先前注册的命令。Slack 命令在 Slack 应用中管理，不会自动删除。
- `commands.nativeSkills`（默认 `"auto"`）在支持时本机注册**技能**命令。
  - 自动：Discord/Telegram 开启；Slack 关闭（Slack 需要为每个技能创建斜杠命令）。
  - 设置 `channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills` 或 `channels.slack.commands.nativeSkills` 以覆盖每个提供程序（布尔值或 `"auto"`）。
- `commands.bash`（默认 `false`）启用 `! <cmd>` 以运行主机 shell 命令（`/bash <cmd>` 是别名；需要 `tools.elevated` allowlist）。
- `commands.bashForegroundMs`（默认 `2000`）控制 bash 在切换到后台模式之前等待多长时间（`0` 立即后台）。
- `commands.config`（默认 `false`）启用 `/config`（读取/写入 `openclaw.json`）。
- `commands.debug`（默认 `false`）启用 `/debug`（运行时-only 覆盖）。
- `commands.useAccessGroups`（默认 `true`）对命令强制执行 allowlist/策略。

## 命令列表

文本 + 本机（启用时）：

- `/help`
- `/commands`
- `/skill <name> [input]`（按名称运行技能）
- `/status`（显示当前状态；可用时包括当前模型提供程序的使用量/配额）
- `/allowlist`（列出/添加/删除 allowlist 条目）
- `/approve <id> allow-once|allow-always|deny`（解决执行批准提示）
- `/context [list|detail|json]`（解释"上下文"；`detail` 显示每个文件 + 每个工具 + 每个技能 + 系统提示大小）
- `/whoami`（显示你的发送者 id；别名：`/id`）
- `/subagents list|stop|log|info|send`（检查、停止、日志或消息当前会话的子代理运行）
- `/config show|get|set|unset`（持久化配置到磁盘，仅所有者；需要 `commands.config: true`）
- `/debug show|set|unset|reset`（运行时覆盖，仅所有者；需要 `commands.debug: true`）
- `/usage off|tokens|full|cost`（每个回复的使用量页脚或本地成本摘要）
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（控制 TTS；参见 [/tts](/tts)）
  - Discord：本机命令是 `/voice`（Discord 保留 `/tts`）；文本 `/tts` 仍然工作。
- `/stop`
- `/restart`
- `/dock-telegram`（别名：`/dock_telegram`）（切换回复到 Telegram）
- `/dock-discord`（别名：`/dock_discord`）（切换回复到 Discord）
- `/dock-slack`（别名：`/dock_slack`）（切换回复到 Slack）
- `/activation mention|always`（仅限群组）
- `/send on|off|inherit`（仅所有者）
- `/reset` 或 `/new [model]`（可选模型提示；其余部分传递通过）
- `/think <off|minimal|low|medium|high|xhigh>`（动态选择由模型/提供程序；别名：`/thinking`、`/t`）
- `/verbose on|full|off`（别名：`/v`）
- `/reasoning on|off|stream`（别名：`/reason`；开启时，发送以 `Reasoning:` 为前缀的单独消息；`stream` = 仅 Telegram 草稿）
- `/elevated on|off|ask|full`（别名：`/elev`；`full` 跳过执行批准）
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`（发送 `/exec` 以显示当前）
- `/model <name>`（别名：`/models`；或 `/<alias>` 来自 `agents.defaults.models.*.alias`）
- `/queue <mode>`（加上选项如 `debounce:2s cap:25 drop:summarize`；发送 `/queue` 以查看当前设置）
- `/bash <command>`（主机-only；`! <command>` 的别名；需要 `commands.bash: true` + `tools.elevated` allowlist）

仅文本：

- `/compact [instructions]`（参见 [/concepts/compaction](/concepts/compaction)）
- `! <command>`（主机-only；一次一个；使用 `!poll` + `!stop` 用于长时间运行作业）
- `!poll`（检查输出 / 状态；接受可选的 `sessionId`；`/bash poll` 也可以工作）
- `!stop`（停止运行的 bash 作业；接受可选的 `sessionId`；`/bash stop` 也可以工作）

注意：

- 命令接受命令和参数之间的可选 `:`（例如 `/think: high`、`/send: on`、`/help:`）。
- `/new <model>` 接受模型别名、`provider/model` 或提供程序名称（模糊匹配）；如果没有匹配，文本被视为消息体。
- 有关完整提供程序使用量分解，使用 `openclaw status --usage`。
- `/allowlist add|remove` 需要 `commands.config=true` 并遵守频道 `configWrites`。
- `/usage` 控制每个回复的使用量页脚；`/usage cost` 从 OpenClaw 会话日志打印本地成本摘要。
- `/restart` 默认禁用；设置 `commands.restart: true` 以启用它。
- `/verbose` 旨在用于调试和额外可见性；在正常使用中保持**关闭**。
- `/reasoning`（和 `/verbose`）在群组设置中有风险：它们可能暴露你不想暴露的内部推理或工具输出。优先保持关闭，尤其是在群组聊天中。
- **快速路径：** 来自 allowlisted 发送者的仅命令消息被立即处理（绕过队列 + 模型）。
- **群组提及门控：** 来自 allowlisted 发送者的仅命令消息绕过提及要求。
- **内联快捷方式（仅限 allowlisted 发送者）：** 某些命令在嵌入正常消息中时也工作，并在模型看到剩余文本之前被剥离。
  - 示例：`hey /status` 触发状态回复，剩余文本继续通过正常流程。
  - 当前：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。
- 未经授权的仅命令消息被静默忽略，内联 `/...` 令牌被视为纯文本。
- **技能命令：** `user-invocable` 技能暴露为斜杠命令。名称被清理为 `a-z0-9_`（最大 32 个字符）；冲突获得数字后缀（例如 `_2`）。
  - `/skill <name> [input]` 按名称运行技能（当本机命令限制阻止每个技能命令时有用）。
  - 默认情况下，技能命令作为正常请求转发到模型。
  - 技能可以选择性地声明 `command-dispatch: tool` 以将命令直接路由到工具（确定性，无模型）。
  - 示例：`/prose`（OpenProse 插件）——参见 [OpenProse](/prose)。
- **本机命令参数：** Discord 对动态选项使用自动完成（当你省略必需参数时使用按钮菜单）。Telegram 和 Slack 在命令支持选择且你省略参数时显示按钮菜单。

## 使用表面（什么显示在哪里）

- **提供程序使用量/配额**（示例："Claude 80% left"）在可用时显示在 `/status` 中用于当前模型提供程序。
- **每个回复令牌/成本**由 `/usage off|tokens|full` 控制（附加到正常回复）。
- `/model status` 是关于**模型/认证/端点**，不是使用量。

## 模型选择（`/model`）

`/model` 实现为指令。

示例：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

注意：

- `/model` 和 `/model list` 显示一个紧凑的、编号的选择器（模型系列 + 可用提供程序）。
- `/model <#>` 从该选择器中选择（并在可能时优先当前提供程序）。
- `/model status` 显示详细视图，包括配置的提供程序端点（`baseUrl`）和 API 模式（`api`）（当可用时）。

## 调试覆盖

`/debug` 允许你设置**运行时-only** 配置覆盖（内存，不是磁盘）。仅所有者。默认禁用；用 `commands.debug: true` 启用。

示例：

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注意：

- 覆盖立即应用于新的配置读取，但**不**写入 `openclaw.json`。
- 使用 `/debug reset` 清除所有覆盖并返回磁盘配置。

## 配置更新

`/config` 写入你的磁盘配置（`openclaw.json`）。仅所有者。默认禁用；用 `commands.config: true` 启用。

示例：

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

注意：

- 配置在写入前验证；无效更改被拒绝。
- `/config` 更新在重启后持久化。

## 表面注意

- **文本命令**在正常聊天会话中运行（DM 共享 `groups`，群组有自己的会话）。
- **本机命令**使用隔离的会话：
  - Discord：`agent:<agentId>:discord:slash:<userId>`
  - Slack：`agent:<agentId>:slack:slash:<userId>`（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）
  - Telegram：`telegram:slash:<userId>`（通过 `CommandTargetSessionKey` 定位聊天会话）
- **`/stop`** 定位活动的聊天会话以便可以中止当前运行。
- **Slack：** `channels.slack.slashCommand` 仍支持单个 `/openclaw` 风格命令。如果你启用 `commands.native`，你必须为每个内置命令创建一个 Slack 斜杠命令（与 `/help` 相同的名称）。Slack 的命令参数菜单作为临时 Block Kit 按钮传递。
