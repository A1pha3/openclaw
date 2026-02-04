---
summary: "代理工作区：位置、布局和备份策略"
read_when:
  - 你需要解释代理工作区或其文件布局
  - 你想要备份或迁移代理工作区
title: "代理工作区"
---

# 代理工作区

工作区是代理的家。它是文件工具和工作区上下文使用的唯一工作目录。保持私有并将其视为记忆。

这与 `~/.openclaw/` 分开，后者存储配置、凭证和会话。

**重要：** 工作区是**默认 cwd**，而不是硬沙箱。工具相对于工作区解析相对路径，但绝对路径仍然可以到达主机上的其他位置，除非启用了沙箱。如果你需要隔离，使用 [`agents.defaults.sandbox`](/gateway/sandboxing)（和/或每个代理沙箱配置）。当启用沙箱且 `workspaceAccess` 不是 `"rw"` 时，工具在 `~/.openclaw/sandboxes` 下的沙箱工作区中操作，而不是你的主机工作区。

## 默认位置

- 默认：`~/.openclaw/workspace`
- 如果设置了 `OPENCLAW_PROFILE` 且不是 `"default"`，默认值变为 `~/.openclaw/workspace-<profile>`。
- 在 `~/.openclaw/openclaw.json` 中覆盖：

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`、`openclaw configure` 或 `openclaw setup` 将创建工作区并在缺失时植入引导文件。

如果你已经自己管理工作区文件，可以禁用引导文件创建：

```json5
{ agent: { skipBootstrap: true } }
```

## 额外的工作区文件夹

旧版本安装可能创建了 `~/openclaw`。保留多个工作区目录可能会导致混乱的认证或状态漂移，因为一次只有一个工作区是活动的。

**建议：** 保持单个活动工作区。如果你不再使用额外文件夹，将其存档或移动到废纸篓（例如 `trash ~/openclaw`）。如果你故意保留多个工作区，请确保 `agents.defaults.workspace` 指向活动的工作区。

`openclaw doctor` 在检测到额外工作区目录时发出警告。

## 工作区文件映射（每个文件的含义）

这些是 OpenClaw 在工作区内期望的标准文件：

- `AGENTS.md`
  - 代理的操作说明以及如何使用记忆。
  - 在每个会话开始时加载。
  - 放置规则、优先级和"如何行为"细节的好地方。

- `SOUL.md`
  - 人格、语气和边界。
  - 每个会话加载。

- `USER.md`
  - 用户是谁以及如何称呼他们。
  - 每个会话加载。

- `IDENTITY.md`
  - 代理的名称、氛围和表情符号。
  - 在引导仪式期间创建/更新。

- `TOOLS.md`
  - 关于本地工具和约定的注释。
  - 不控制工具可用性；它只是指导。

- `HEARTBEAT.md`
  - 可选的心跳运行小检查表。
  - 保持简短以避免令牌消耗。

- `BOOT.md`
  - 可选的启动检查表，在启用内部钩子时在网关重启时执行。
  - 保持简短；使用消息工具进行出站发送。

- `BOOTSTRAP.md`
  - 一次性首次运行仪式。
  - 仅针对全新工作区创建。
  - 仪式完成后删除它。

- `memory/YYYY-MM-DD.md`
  - 每日记忆日志（每天一个文件）。
  - 建议在会话开始时阅读今天和昨天的内容。

- `MEMORY.md`（可选）
  - 精选的长期记忆。
  - 仅在主要、私人会话中加载（不共享/组上下文）。

参见[Memory](/concepts/memory)了解工作流程和自动内存刷新。

- `skills/`（可选）
  - 工作区特定技能。
  - 当名称冲突时覆盖托管/捆绑技能。

- `canvas/`（可选）
  - 节点显示的 Canvas UI 文件（例如 `canvas/index.html`）。

如果任何引导文件缺失，OpenClaw 会向会话注入一个"缺失文件"标记并继续。大型引导文件在注入时被截断；使用 `agents.defaults.bootstrapMaxChars` 调整限制（默认：20000）。
`openclaw setup` 可以在不覆盖现有文件的情况下重新创建缺失的默认值。

## 工作区中没有什么

这些位于 `~/.openclaw/` 下，不应提交到工作区仓库：

- `~/.openclaw/openclaw.json`（配置）
- `~/.openclaw/credentials/`（OAuth 令牌、API 密钥）
- `~/.openclaw/agents/<agentId>/sessions/`（会话记录 + 元数据）
- `~/.openclaw/skills/`（托管技能）

如果你需要迁移会话或配置，请单独复制它们并将它们保持在版本控制之外。

## Git 备份（推荐，私有）

将工作区视为私有记忆。将其放入**私有** git 仓库，以便备份和可恢复。

在运行网关的机器上执行这些步骤（那是工作区所在的位置）。

### 1）初始化仓库

如果安装了 git，全新的工作区会自动初始化。如果这个工作区还不是一个仓库，运行：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2）添加私有远程（初学者友好选项）

选项 A：GitHub Web UI

1. 在 GitHub 上创建一个新的**私有**仓库。
2. 不要使用 README 初始化（避免合并冲突）。
3. 复制 HTTPS 远程 URL。
4. 添加远程并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

选项 B：GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

选项 C：GitLab Web UI

1. 在 GitLab 上创建一个新的**私有**仓库。
2. 不要使用 README 初始化（避免合并冲突）。
3. 复制 HTTPS 远程 URL。
4. 添加远程并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3）持续更新

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## 不要提交机密

即使在私有仓库中，也要避免在工作区中存储机密：

- API 密钥、OAuth 令牌、密码或私人凭证。
- `~/.openclaw/` 下的任何内容。
- 聊天或敏感附件的原始转储。

如果你必须存储敏感引用，使用占位符并将真实机密保存在其他地方（密码管理器、环境变量或 `~/.openclaw/`）。

建议的 `.gitignore` 启动器：

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## 将工作区移动到新机器

1. 将仓库克隆到所需路径（默认 `~/.openclaw/workspace`）。
2. 在 `~/.openclaw/openclaw.json` 中将 `agents.defaults.workspace` 设置为该路径。
3. 运行 `openclaw setup --workspace <path>` 以植入任何缺失的文件。
4. 如果你需要会话，请从旧机器单独复制 `~/.openclaw/agents/<agentId>/sessions/`。

## 高级说明

- 多代理路由可以为每个代理使用不同的工作区。参见
  [频道路由](/concepts/channel-routing)了解路由配置。
- 如果启用了 `agents.defaults.sandbox`，非主要会话可以在 `agents.defaults.sandbox.workspaceRoot` 下使用每个会话的沙箱工作区。
