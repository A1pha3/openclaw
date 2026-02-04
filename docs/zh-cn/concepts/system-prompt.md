---
summary: "OpenClaw 系统提示包含什么以及如何组装"
read_when:
  - 编辑系统提示文本、工具列表或时间/心跳部分
  - 更改工作区引导或技能注入行为
title: "系统提示"
---

# 系统提示

OpenClaw 为每个代理运行构建自定义系统提示。提示是 **OpenClaw 拥有的**，不使用 p-coding-agent 默认提示。

提示由 OpenClaw 组装并注入到每个代理运行中。

## 结构

提示故意紧凑并使用固定部分：

- **工具**：当前工具列表 + 简短描述。
- **安全**：简短护栏提醒，避免权力寻求行为或绕过监督。
- **技能**（当可用）：告诉模型如何按需加载技能说明。
- **OpenClaw 自我更新**：如何运行 `config.apply` 和 `update.run`。
- **工作区**：工作目录（`agents.defaults.workspace`）。
- **文档**：OpenClaw 文档的本地路径（仓库或 npm 包）以及何时阅读它们。
- **工作区文件（注入）**：指示引导文件包含在下方。
- **沙箱**（当启用时）：指示沙箱化运行时、沙箱路径，以及是否提供提权执行。
- **当前日期和时间**：用户本地时区、时区和时间格式。
- **回复标签**：支持提供程序的可选回复标签语法。
- **心跳**：心跳提示和确认行为。
- **运行时**：主机、操作系统、节点、模型、仓库根目录（当检测到时）、思考级别（一行）。
- **推理**：当前可见性级别 + /reasoning 切换提示。

系统提示中的安全护栏是建议性的。它们指导模型行为但不强制执行策略。使用工具策略、执行批准、沙箱和频道允许列表进行硬强制；操作员可以通过设计禁用这些。

## 提示模式

OpenClaw 可以为子代理渲染更小的系统提示。运行时为每个运行设置 `promptMode`（不是面向用户的配置）：

- `full`（默认）：包含上述所有部分。
- `minimal`：用于子代理；省略 **技能**、**内存回忆**、**OpenClaw 自我更新**、**模型别名**、**用户身份**、**回复标签**、**消息**、**静默回复**和**心跳**。工具、**安全**、工作区、沙箱、当前日期和时间（当已知时）、运行时和注入的上下文保持可用。
- `none`：仅返回基础身份行。

当 `promptMode=minimal` 时，额外的注入提示被标记为 **Subagent Context** 而不是 **Group Chat Context**。

## 工作区引导注入

引导文件被修整并附加在 **Project Context** 下，以便模型看到身份和配置文件上下文，而不需要显式读取：

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md`（仅限全新工作区）

大型文件带有标记被截断。每个文件的最大大小由 `agents.defaults.bootstrapMaxChars` 控制（默认：20000）。缺失文件注入一个简短的缺失文件标记。

内部钩子可以通过 `agent:bootstrap` 拦截此步骤以更改或替换注入的引导文件（例如，将 `SOUL.md` 交换为替代人格）。

要检查每个注入文件贡献了多少（原始 vs 注入、截断，加上工具 schema 开销），使用 `/context list` 或 `/context detail`。参见 [Context](/concepts/context)。

## 时间处理

当用户时区已知时，系统提示包括专用的 **Current Date & Time** 部分。为了保持提示缓存稳定，它现在只包括**时区**（没有动态时钟或时间格式）。

当代理需要当前时间时使用 `session_status`；状态卡包括时间戳行。

配置：

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat`（`auto` | `12` | `24`）

参见 [Date & Time](/date-time) 了解完整行为详情。

## 技能

当存在符合条件的技能时，OpenClaw 注入一个紧凑的**可用技能列表**（`formatSkillsForPrompt`），其中包括每个技能的**文件路径**。提示指示模型使用 `read` 在列出的位置（工作区、托管或捆绑）加载 SKILL.md。如果没有符合条件的技能，省略技能部分。

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

这保持了基础提示小，同时仍启用有针对性的技能使用。

## 文档

当可用时，系统提示包括一个 **Documentation** 部分，指向本地 OpenClaw 文档目录（工作区中的 `docs/` 或捆绑的 npm 包 docs），还指出公共镜像、源仓库、社区 Discord 和 ClawHub (https://clawhub.com) 用于技能发现。提示指示模型首先咨询本地文档以了解 OpenClaw 行为、命令、配置或架构，并在可能时自行运行 `openclaw status`（仅当它缺乏访问权限时才询问用户）。
