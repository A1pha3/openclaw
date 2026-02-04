---
summary: "ClawHub 指南：公共技能注册中心 + CLI 工作流"
read_when:
  - 向新用户介绍 ClawHub
  - 安装、搜索或发布技能
  - 了解 ClawHub CLI 参数和同步行为
title: "ClawHub"
---

# ClawHub

ClawHub 是 **OpenClaw 的公共技能注册中心**。这是一个免费服务：所有技能都是公开的、开放的，对所有人可见，便于分享和复用。技能就是一个包含 `SKILL.md` 文件（以及支持文本文件）的文件夹。你可以在 Web 应用中浏览技能，或使用 CLI 搜索、安装、更新和发布技能。

**网站**：[clawhub.com](https://clawhub.com)

## 为什么需要 ClawHub

| 需求 | ClawHub 如何满足 |
|------|------------------|
| 发现新能力 | 通过语义搜索找到合适的技能 |
| 快速集成 | 一条命令安装到工作区 |
| 保持更新 | 自动检测和更新已安装技能 |
| 分享成果 | 发布你的技能供社区使用 |

## 适用人群（新手友好）

如果你想为 OpenClaw 智能体添加新能力，ClawHub 是发现和安装技能最简单的方式。你不需要了解后端如何工作。你可以：

- 用自然语言搜索技能
- 将技能安装到工作区
- 稍后用一条命令更新技能
- 发布技能作为备份

## 快速开始（非技术用户）

**第一步：安装 CLI**（见下节）

**第二步：搜索你需要的功能**

```bash
clawhub search "calendar"
```

**第三步：安装技能**

```bash
clawhub install <skill-slug>
```

**第四步：启动新的 OpenClaw 会话**，让它加载新技能

## 安装 CLI

选择其一：

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## 与 OpenClaw 的集成

默认情况下，CLI 将技能安装到当前工作目录下的 `./skills`。如果配置了 OpenClaw 工作区，`clawhub` 会回退到该工作区，除非你用 `--workdir`（或 `CLAWHUB_WORKDIR`）覆盖。OpenClaw 从 `<workspace>/skills` 加载工作区技能，并在**下一个**会话中生效。如果你已经使用 `~/.openclaw/skills` 或捆绑技能，工作区技能优先级更高。

更多关于技能加载、共享和门控的细节，参见 [技能系统](/tools/skills)。

## 服务功能

| 功能 | 说明 |
|------|------|
| **公开浏览** | 查看技能及其 `SKILL.md` 内容 |
| **语义搜索** | 基于嵌入向量的搜索，不仅仅是关键词 |
| **版本管理** | semver 语义版本、更新日志和标签（包括 `latest`） |
| **下载** | 每个版本打包成 zip |
| **社区互动** | 点赞和评论 |
| **审核机制** | 审批和审计钩子 |
| **CLI 友好** | 支持自动化和脚本的 API |

## CLI 命令和参数

### 全局选项（适用于所有命令）

| 选项 | 说明 |
|------|------|
| `--workdir <dir>` | 工作目录（默认：当前目录；回退到 OpenClaw 工作区） |
| `--dir <dir>` | 技能目录，相对于 workdir（默认：`skills`） |
| `--site <url>` | 网站基础 URL（浏览器登录） |
| `--registry <url>` | 注册中心 API 基础 URL |
| `--no-input` | 禁用提示（非交互模式） |
| `-V, --cli-version` | 打印 CLI 版本 |

### 认证命令

```bash
# 浏览器登录流程
clawhub login

# 或使用 token
clawhub login --token <token>

# 登出
clawhub logout

# 查看当前用户
clawhub whoami
```

**认证选项：**

| 选项 | 说明 |
|------|------|
| `--token <token>` | 粘贴 API token |
| `--label <label>` | 浏览器登录 token 的标签（默认：`CLI token`） |
| `--no-browser` | 不打开浏览器（需要 `--token`） |

### 搜索命令

```bash
clawhub search "查询关键词"
clawhub search "postgres backups" --limit 10
```

### 安装命令

```bash
clawhub install <slug>
clawhub install my-skill-pack --version 1.2.0
clawhub install some-skill --force  # 覆盖已存在的文件夹
```

### 更新命令

```bash
# 更新单个技能
clawhub update <slug>

# 更新所有已安装技能
clawhub update --all

# 更新到特定版本
clawhub update <slug> --version 2.0.0
```

### 列表命令

```bash
clawhub list  # 读取 .clawhub/lock.json
```

### 发布命令

```bash
clawhub publish <path> \
  --slug my-skill \
  --name "My Skill" \
  --version 1.0.0 \
  --changelog "Initial release" \
  --tags latest
```

### 删除/恢复命令（仅所有者/管理员）

```bash
clawhub delete <slug> --yes
clawhub undelete <slug> --yes
```

### 同步命令（扫描本地技能 + 发布新的/更新的）

```bash
# 基本同步
clawhub sync

# 完整同步选项
clawhub sync \
  --root ~/extra-skills \
  --all \
  --bump minor \
  --changelog "Bug fixes" \
  --dry-run
```

**同步选项：**

| 选项 | 说明 |
|------|------|
| `--root <dir...>` | 额外扫描目录 |
| `--all` | 无提示上传所有 |
| `--dry-run` | 显示将要上传的内容 |
| `--bump <type>` | `patch\|minor\|major`（默认：`patch`） |
| `--changelog <text>` | 非交互更新的更新日志 |
| `--tags <tags>` | 逗号分隔的标签（默认：`latest`） |
| `--concurrency <n>` | 注册中心检查并发数（默认：4） |

## 智能体常用工作流

### 搜索技能

```bash
clawhub search "postgres backups"
```

### 下载新技能

```bash
clawhub install my-skill-pack
```

### 更新已安装技能

```bash
clawhub update --all
```

### 备份你的技能（发布或同步）

**单个技能文件夹：**

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

**批量扫描和备份：**

```bash
clawhub sync --all
```

## 高级细节（技术向）

### 版本管理和标签

- 每次发布创建一个新的 **semver** `SkillVersion`
- 标签（如 `latest`）指向一个版本；移动标签可以回滚
- 更新日志附加到每个版本，同步或发布更新时可以为空

### 本地变更 vs 注册中心版本

更新时通过内容哈希比较本地技能内容和注册中心版本。如果本地文件不匹配任何已发布版本，CLI 会在覆盖前询问（或在非交互运行中需要 `--force`）。

### 同步扫描和回退目录

`clawhub sync` 首先扫描当前工作目录。如果没找到技能，会回退到已知的旧位置（例如 `~/openclaw/skills` 和 `~/.openclaw/skills`）。这是为了无需额外参数就能找到旧的技能安装。

### 存储和锁定文件

- 已安装技能记录在工作目录下的 `.clawhub/lock.json`
- 认证 token 存储在 ClawHub CLI 配置文件中（通过 `CLAWHUB_CONFIG_PATH` 覆盖）

### 遥测（安装计数）

登录状态下运行 `clawhub sync` 时，CLI 发送最小快照以计算安装数。完全禁用：

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

## 环境变量

| 变量 | 说明 |
|------|------|
| `CLAWHUB_SITE` | 覆盖网站 URL |
| `CLAWHUB_REGISTRY` | 覆盖注册中心 API URL |
| `CLAWHUB_CONFIG_PATH` | 覆盖 CLI 存储 token/配置的位置 |
| `CLAWHUB_WORKDIR` | 覆盖默认工作目录 |
| `CLAWHUB_DISABLE_TELEMETRY=1` | 禁用 `sync` 时的遥测 |

## 相关文档

- [技能系统](/tools/skills) - 技能加载和门控规则
- [技能配置](/tools/skills-config) - 配置模式详解
