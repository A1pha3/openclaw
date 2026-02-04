---
summary: "技能系统：托管 vs 工作区技能、门控规则、配置与环境变量"
read_when:
  - 添加或修改技能
  - 调整技能门控或加载规则
title: "技能"
---

# 技能系统

OpenClaw 使用 **[AgentSkills](https://agentskills.io) 兼容**的技能文件夹来教会 agent 如何使用工具。每个技能是一个包含 `SKILL.md` 的目录，带有 YAML frontmatter 和使用说明。OpenClaw 加载**内置技能**加上可选的本地覆盖，并在加载时根据环境、配置和二进制文件存在性进行过滤。

## 技能位置与优先级

技能从**三个**位置加载：

1. **内置技能**：随安装包（npm 包或 OpenClaw.app）一起发布
2. **托管/本地技能**：`~/.openclaw/skills`
3. **工作区技能**：`<workspace>/skills`

当技能名称冲突时，优先级为：

```
工作区技能（最高） → 托管技能 → 内置技能（最低）
```

此外，可以通过 `skills.load.extraDirs` 配置额外的技能目录（优先级最低）。

## 单 Agent vs 多 Agent 场景

在**多 Agent** 场景中，每个 agent 有独立的工作区：

| 技能类型 | 位置 | 可见范围 |
|---------|------|---------|
| **Agent 专属** | `<workspace>/skills` | 仅该 agent |
| **共享技能** | `~/.openclaw/skills` | 同一机器上的**所有** agent |
| **额外目录** | `skills.load.extraDirs` 配置的路径 | 所有 agent（最低优先级） |

## 插件与技能

插件可以在 `openclaw.plugin.json` 中声明 `skills` 目录（相对于插件根目录的路径）来发布自己的技能。插件技能在插件启用时加载，并参与正常的优先级规则。

可以通过技能的 `metadata.openclaw.requires.config` 对插件配置项进行门控。详见 [插件](/zh-cn/plugin) 和 [工具](/zh-cn/tools)。

## ClawHub：安装与同步

ClawHub 是 OpenClaw 的公共技能注册表。浏览地址：https://clawhub.com

常用操作：

```bash
# 安装技能到工作区
clawhub install <skill-slug>

# 更新所有已安装技能
clawhub update --all

# 同步（扫描 + 发布更新）
clawhub sync --all
```

默认情况下，`clawhub` 安装到当前目录下的 `./skills`（或回退到配置的 OpenClaw 工作区）。OpenClaw 在下一个会话中将其识别为 `<workspace>/skills`。

完整指南：[ClawHub](/zh-cn/tools/clawhub)

## 安全注意事项

- 将第三方技能视为**受信任的代码**。启用前请先阅读其内容
- 对不受信任的输入和高风险工具，优先使用沙箱运行。详见 [沙箱](/zh-cn/gateway/sandboxing)
- `skills.entries.*.env` 和 `skills.entries.*.apiKey` 将密钥注入 agent 运行时的**宿主**进程（非沙箱）。请勿在提示词和日志中暴露密钥
- 完整的威胁模型和检查清单，详见 [安全](/zh-cn/gateway/security)

## 技能格式（AgentSkills + Pi 兼容）

`SKILL.md` 至少需要包含：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图片
---

这里是技能的使用说明...
```

### 可选 frontmatter 字段

| 字段 | 说明 |
|------|------|
| `homepage` | 网站 URL（macOS 技能 UI 显示为"网站"链接） |
| `user-invocable` | `true`/`false`（默认：`true`）。为 `true` 时作为用户斜杠命令暴露 |
| `disable-model-invocation` | `true`/`false`（默认：`false`）。为 `true` 时从模型提示词中排除（仍可通过用户调用） |
| `command-dispatch` | 设为 `tool` 时，斜杠命令绕过模型直接调用工具 |
| `command-tool` | 当 `command-dispatch: tool` 时要调用的工具名称 |
| `command-arg-mode` | `raw`（默认）。工具调度时将原始参数字符串转发给工具 |

注意事项：
- 遵循 AgentSkills 规范的布局和意图
- 内嵌 agent 使用的解析器仅支持**单行** frontmatter 键
- `metadata` 应为**单行 JSON 对象**
- 在说明中使用 `{baseDir}` 引用技能文件夹路径

## 门控机制（加载时过滤）

OpenClaw 在加载时使用 `metadata`（单行 JSON）过滤技能：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图片
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

### `metadata.openclaw` 字段

| 字段 | 说明 |
|------|------|
| `always: true` | 始终包含该技能（跳过其他门控） |
| `emoji` | macOS 技能 UI 使用的可选 emoji |
| `homepage` | macOS 技能 UI 显示的可选网站 URL |
| `os` | 平台列表（`darwin`、`linux`、`win32`）。设置后仅在这些系统上可用 |
| `requires.bins` | 列表；所有二进制必须存在于 `PATH` |
| `requires.anyBins` | 列表；至少一个二进制存在于 `PATH` |
| `requires.env` | 列表；环境变量必须存在**或**在配置中提供 |
| `requires.config` | `openclaw.json` 路径列表，必须为真值 |
| `primaryEnv` | 与 `skills.entries.<name>.apiKey` 关联的环境变量名 |
| `install` | macOS 技能 UI 使用的安装器规格数组（brew/node/go/uv/download） |

### 沙箱注意事项

- `requires.bins` 在技能加载时在**宿主机**上检查
- 如果 agent 在沙箱中运行，二进制也必须存在于**容器内**
- 通过 `agents.defaults.sandbox.docker.setupCommand` 或自定义镜像安装
- `setupCommand` 在容器创建后运行一次
- 包安装还需要网络出口、可写根文件系统和容器内的 root 用户

### 安装器示例

```markdown
---
name: gemini
description: 使用 Gemini CLI 进行编码辅助和 Google 搜索查询
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "安装 Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

安装器说明：
- 列出多个安装器时，网关选择**单个**首选选项（有 brew 时优先，否则用 node）
- 如果所有安装器都是 `download` 类型，OpenClaw 会列出每个条目供查看
- 安装器规格可包含 `os: ["darwin"|"linux"|"win32"]` 按平台过滤
- Node 安装遵循 `skills.install.nodeManager` 配置（默认：npm）
- Go 安装：如果 `go` 缺失但有 `brew`，网关会先通过 Homebrew 安装 Go
- Download 安装：`url`（必需）、`archive`、`extract`、`stripComponents`、`targetDir`

如果没有 `metadata.openclaw`，技能始终可用（除非在配置中禁用或被 `skills.allowBundled` 阻止）。

## 配置覆盖

内置/托管技能可以在 `~/.openclaw/openclaw.json` 中切换和提供环境值：

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

配置规则：
- `enabled: false` 禁用技能（即使已内置/安装）
- `env`：仅当变量未设置时注入
- `apiKey`：便捷字段，用于声明了 `metadata.openclaw.primaryEnv` 的技能
- `config`：可选的自定义字段包；自定义键必须放在这里

注意：技能名含连字符时需引用键名（JSON5 允许带引号的键）。

## 环境注入（每次 agent 运行）

agent 运行启动时，OpenClaw：

1. 读取技能元数据
2. 将 `skills.entries.<key>.env` 或 `skills.entries.<key>.apiKey` 应用到 `process.env`
3. 使用**符合条件**的技能构建系统提示词
4. 运行结束后恢复原始环境

这是**仅限 agent 运行期间**的，不是全局 shell 环境。

## 会话快照（性能优化）

OpenClaw 在**会话开始时**快照符合条件的技能，并在同一会话的后续轮次中复用该列表。技能或配置的更改在下一个新会话生效。

当启用技能 watcher 或出现新的符合条件的远程节点时，技能也可以在会话中途刷新。可以理解为**热重载**：刷新的列表在下一个 agent 轮次生效。

## 远程 macOS 节点（Linux 网关）

如果网关运行在 Linux 上，但连接了一个**启用 `system.run`** 的 macOS 节点（Exec 审批安全设置不是 `deny`），OpenClaw 可以在该节点上存在所需二进制时将 macOS 专属技能视为可用。Agent 应通过 `nodes` 工具（通常是 `nodes.run`）执行这些技能。

这依赖于节点报告其命令支持和通过 `system.run` 的二进制探测。如果 macOS 节点后来离线，技能仍然可见；调用可能失败直到节点重新连接。

## 技能 Watcher（自动刷新）

默认情况下，OpenClaw 监视技能文件夹，当 `SKILL.md` 文件更改时更新技能快照。配置位于 `skills.load`：

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Token 影响（技能列表）

当有符合条件的技能时，OpenClaw 将可用技能的紧凑 XML 列表注入系统提示词。成本是确定的：

| 项目 | 字符数 |
|------|--------|
| 基础开销（仅当 ≥1 技能时） | 195 |
| 每个技能 | 97 + name + description + location 的 XML 转义长度 |

公式：
```
总计 = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

注意事项：
- XML 转义会将 `& < > " '` 扩展为实体（`&amp;`、`&lt;` 等），增加长度
- Token 数量因模型分词器而异。粗略估计约 4 字符/token，所以 **97 字符 ≈ 24 tokens** 每技能，加上实际字段长度

## 托管技能生命周期

OpenClaw 发布时包含一组**内置技能**（npm 包或 OpenClaw.app 的一部分）。`~/.openclaw/skills` 用于本地覆盖（例如，固定/修补技能而不更改内置副本）。工作区技能由用户拥有，在名称冲突时覆盖两者。

## 配置参考

完整配置 schema 详见 [技能配置](/zh-cn/tools/skills-config)。

## 寻找更多技能？

浏览 https://clawhub.com
