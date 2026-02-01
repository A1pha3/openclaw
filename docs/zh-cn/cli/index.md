---
summary: "OpenClaw CLI 参考 - 所有命令、子命令和选项的完整指南"
read_when:
  - 学习 CLI 命令
  - 查找特定命令用法
  - 理解命令组织结构
title: "CLI 参考"
---

# 🖥️ CLI 参考

本文档介绍 OpenClaw 命令行工具 (`openclaw`) 的所有命令和用法。

---

## 🎯 快速导航

| 你想要... | 使用命令 |
|-----------|----------|
| **首次设置** | `openclaw onboard` |
| **查看状态** | `openclaw status` |
| **发送消息** | `openclaw message send` |
| **管理配置** | `openclaw config` |
| **诊断问题** | `openclaw doctor` |
| **启动网关** | `openclaw gateway` |
| **查看日志** | `openclaw logs` |

---

## 📋 命令分类

### 🚀 设置与配置

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`setup`](#setup) | 初始化配置和工作区 | ⭐⭐⭐ |
| [`onboard`](#onboard) | 引导向导（推荐） | ⭐⭐⭐⭐⭐ |
| [`configure`](#configure) | 重新配置 | ⭐⭐⭐ |
| [`config`](#config) | 配置操作（get/set/unset） | ⭐⭐⭐⭐ |
| [`doctor`](#doctor) | 诊断检查 | ⭐⭐⭐⭐ |
| [`reset`](#reset) | 重置配置 | ⭐⭐ |
| [`uninstall`](#uninstall) | 卸载 OpenClaw | ⭐⭐ |
| [`update`](#update) | 更新到最新版 | ⭐⭐⭐ |

### 🤖 代理管理

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`agent`](#agent) | 单个代理操作 | ⭐⭐⭐ |
| [`agents`](#agents) | 多代理管理 | ⭐⭐⭐ |
| [`sessions`](#sessions) | 会话管理 | ⭐⭐⭐ |
| [`memory`](#memory) | 记忆管理 | ⭐⭐ |
| [`models`](#models) | 模型管理 | ⭐⭐⭐ |
| [`skills`](#skills) | 技能管理 | ⭐⭐⭐⭐ |

### 💬 消息与渠道

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`message`](#message) | 发送消息 | ⭐⭐⭐ |
| [`channels`](#channels) | 渠道管理 | ⭐⭐⭐⭐ |
| [`pairing`](#pairing) | 配对管理 | ⭐⭐⭐ |
| [`approvals`](#approvals) | 审批管理 | ⭐⭐ |

### 🌐 网关与系统

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`gateway`](#gateway) | 网关控制 | ⭐⭐⭐⭐⭐ |
| [`status`](#status) | 查看状态 | ⭐⭐⭐⭐⭐ |
| [`health`](#health) | 健康检查 | ⭐⭐⭐⭐ |
| [`logs`](#logs) | 查看日志 | ⭐⭐⭐⭐ |
| [`system`](#system) | 系统操作 | ⭐⭐ |
| [`nodes`](#nodes) | 节点管理 | ⭐⭐⭐ |
| [`devices`](#devices) | 设备管理 | ⭐⭐ |

### 🛠️ 工具与扩展

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`cron`](#cron) | 定时任务 | ⭐⭐⭐ |
| [`browser`](#browser) | 浏览器控制 | ⭐⭐ |
| [`sandbox`](#sandbox) | 沙箱管理 | ⭐⭐ |
| [`plugins`](#plugins) | 插件管理 | ⭐⭐⭐ |
| [`hooks`](#hooks) | Hook 管理 | ⭐⭐ |
| [`webhooks`](#webhooks) | Webhook 管理 | ⭐⭐ |
| [`voicecall`](#voicecall) | 语音通话（需插件） | ⭐⭐ |

### 🎨 其他

| 命令 | 用途 | 常用程度 |
|------|------|----------|
| [`dashboard`](#dashboard) | 打开仪表盘 | ⭐⭐⭐⭐ |
| [`tui`](#tui) | TUI 界面 | ⭐⭐ |
| [`dns`](#dns) | DNS 工具 | ⭐ |
| [`docs`](#docs) | 打开文档 | ⭐⭐ |
| [`acp`](#acp) | ACP 工具 | ⭐ |
| [`security`](#security) | 安全审计 | ⭐⭐⭐ |

---

## 🔧 常用命令详解

### setup

初始化配置和工作区。

```bash
openclaw setup [options]
```

**选项**：
- `--workspace <path>` - 指定工作区路径

**示例**：
```bash
# 默认设置
openclaw setup

# 指定工作区
openclaw setup --workspace ~/my-workspace
```

---

### onboard

引导向导（推荐首次使用）。

```bash
openclaw onboard [options]
```

**选项**：
- `--install-daemon` - 安装后台服务
- `--reset` - 重置配置
- `--non-interactive` - 非交互模式
- `--workspace <path>` - 指定工作区

**示例**：
```bash
# 完整引导（推荐）
openclaw onboard --install-daemon

# 仅配置（不安装服务）
openclaw onboard

# 非交互模式
openclaw onboard --non-interactive --workspace ~/workspace
```

---

### config

配置操作（get/set/unset）。

```bash
openclaw config get <key>
openclaw config set <key> <value>
openclaw config unset <key>
```

**示例**：
```bash
# 查看配置
openclaw config get agents.defaults.model

# 设置模型
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4"

# 删除配置项
openclaw config unset channels.telegram.enabled
```

---

### doctor

诊断检查。

```bash
openclaw doctor [options]
```

**选项**：
- `--fix` - 尝试自动修复问题

**示例**：
```bash
# 诊断检查
openclaw doctor

# 诊断并修复
openclaw doctor --fix
```

---

### status

查看系统状态。

```bash
openclaw status [options]
```

**选项**：
- `--all` - 显示完整状态（含配置）
- `--deep` - 深度检查（探测网关）

**示例**：
```bash
# 快速状态
openclaw status

# 完整状态
openclaw status --all

# 深度检查
openclaw status --deep
```

**输出示例**：
```
🦞 OpenClaw Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Version:     1.0.0
Gateway:     running (pid 1234)
Channels:    whatsapp, telegram, discord
Agents:      main
Sessions:    5 active
Health:      ✅ healthy
```

---

### health

健康检查。

```bash
openclaw health [options]
```

**示例**：
```bash
openclaw health
```

---

### gateway

网关控制。

```bash
openclaw gateway <subcommand>
```

**子命令**：
- `run` - 启动网关（前台）
- `start` - 启动服务
- `stop` - 停止服务
- `restart` - 重启服务
- `status` - 查看状态

**示例**：
```bash
# 前台启动（调试）
openclaw gateway run --verbose

# 服务管理
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway status
```

---

### logs

查看日志。

```bash
openclaw logs [options]
```

**选项**：
- `--follow, -f` - 实时跟踪
- `--lines <n>` - 显示行数（默认 100）
- `--category <name>` - 按类别过滤

**示例**：
```bash
# 查看最新 100 行
openclaw logs

# 实时跟踪
openclaw logs --follow

# 查看特定类别
openclaw logs --category gateway

# 查看最近 500 行
openclaw logs --lines 500
```

---

### message

发送消息。

```bash
openclaw message send [options]
```

**选项**：
- `--target <id>` - 目标（电话号码、聊天 ID 等）
- `--message <text>` - 消息内容
- `--channel <name>` - 指定渠道

**示例**：
```bash
# 发送消息
openclaw message send --target "+15555550123" --message "Hello!"

# 指定渠道
openclaw message send --channel whatsapp --target "+15555550123" --message "Hi"
```

---

### channels

渠道管理。

```bash
openclaw channels <subcommand>
```

**子命令**：
- `list` - 列出渠道
- `status` - 查看渠道状态
- `login <channel>` - 登录渠道
- `logout <channel>` - 登出渠道

**示例**：
```bash
# 查看渠道列表
openclaw channels list

# 查看渠道状态
openclaw channels status

# 登录 WhatsApp
openclaw channels login whatsapp

# 登出
openclaw channels logout whatsapp
```

---

### pairing

配对管理（DM 安全）。

```bash
openclaw pairing <subcommand>
```

**子命令**：
- `list <channel>` - 列出待批准配对
- `approve <channel> <code>` - 批准配对
- `reject <channel> <code>` - 拒绝配对

**示例**：
```bash
# 查看待批准列表
openclaw pairing list whatsapp

# 批准配对
openclaw pairing approve whatsapp ABC123

# 拒绝配对
openclaw pairing reject whatsapp ABC123
```

---

### agents

多代理管理。

```bash
openclaw agents <subcommand>
```

**子命令**：
- `list` - 列出代理
- `add <name>` - 添加代理
- `delete <name>` - 删除代理
- `set-identity` - 设置身份

**示例**：
```bash
# 列出代理
openclaw agents list

# 添加工作代理
openclaw agents add work --workspace ~/.openclaw/workspace-work

# 设置身份
openclaw agents set-identity --agent main --name "Clawd" --emoji "🦞"

# 删除代理
openclaw agents delete work
```

---

### sessions

会话管理。

```bash
openclaw sessions <subcommand>
```

**子命令**：
- `list` - 列出会话
- `history <key>` - 查看会话历史
- `clear` - 清除会话缓存

**示例**：
```bash
# 列出会话
openclaw sessions list

# 查看历史
openclaw sessions history agent:main:whatsapp:dm:+15555550123

# 清除缓存
openclaw sessions clear
```

---

### skills

技能管理。

```bash
openclaw skills <subcommand>
```

**子命令**：
- `list` - 列出可用技能
- `install <name>` - 安装技能
- `uninstall <name>` - 卸载技能
- `dir` - 查看技能目录

**示例**：
```bash
# 列出技能
openclaw skills list

# 安装技能
openclaw skills install web
openclaw skills install github

# 卸载
openclaw skills uninstall web

# 查看目录
openclaw skills dir
```

---

### cron

定时任务管理。

```bash
openclaw cron <subcommand>
```

**子命令**：
- `list` - 列出任务
- `add` - 添加任务
- `remove <id>` - 删除任务
- `run <id>` - 立即运行

**示例**：
```bash
# 列出定时任务
openclaw cron list

# 添加每日任务
openclaw cron add --name "backup" --schedule "0 2 * * *" --command "backup.sh"

# 删除任务
openclaw cron remove backup

# 立即运行
openclaw cron run backup
```

---

### dashboard

打开仪表盘。

```bash
openclaw dashboard [options]
```

**选项**：
- `--port <n>` - 指定端口

**示例**：
```bash
# 打开仪表盘
openclaw dashboard

# 指定端口
openclaw dashboard --port 8080
```

---

## 🚩 全局选项

所有命令都支持的选项：

| 选项 | 说明 |
|------|------|
| `--dev` | 隔离状态到 `~/.openclaw-dev`，偏移默认端口 |
| `--profile <name>` | 隔离状态到 `~/.openclaw-<name>` |
| `--no-color` | 禁用 ANSI 颜色 |
| `--json` | JSON 输出（禁用样式） |
| `--update` | 简写为 `openclaw update`（仅源码安装） |
| `-V, --version` | 打印版本 |
| `-h, --help` | 显示帮助 |

**示例**：
```bash
# 开发模式（不污染主配置）
openclaw --dev gateway run

# 使用特定 profile
openclaw --profile work status

# JSON 输出（脚本友好）
openclaw status --json

# 无颜色（日志文件）
openclaw logs --no-color
```

---

## 🎨 输出样式

### TTY 检测

- ANSI 颜色和进度指示器仅在 TTY 会话渲染
- 管道或重定向时自动禁用

### 超链接

- OSC-8 超链接在支持终端显示为可点击链接
- 不支持时回退到纯 URL

### 颜色方案

OpenClaw 使用龙虾配色方案：

| 颜色 | 用途 |
|------|------|
| `accent` (#FF5A2D) | 标题、标签、主高亮 |
| `accentBright` (#FF7A3D) | 命令名、强调 |
| `success` (#2FBF71) | 成功状态 |
| `warn` (#FFB020) | 警告、注意 |
| `error` (#E23D2D) | 错误、失败 |
| `muted` (#8B7F77) | 次要、元数据 |

---

## 📝 命令树

```
openclaw [--dev] [--profile <name>] <command>

系统命令:
  setup                    初始化配置
  onboard                  引导向导
  configure                重新配置
  config                   配置操作
    get <key>              获取值
    set <key> <value>      设置值
    unset <key>            删除值
  doctor                   诊断检查
  reset                    重置配置
  update                   更新
  uninstall                卸载

代理命令:
  agent                    代理操作
  agents                   多代理管理
    list                   列出代理
    add <name>             添加代理
    delete <name>          删除代理
    set-identity           设置身份
  sessions                 会话管理
  memory                   记忆管理
  models                   模型管理
  skills                   技能管理

消息命令:
  message                  消息发送
    send                   发送消息
  channels                 渠道管理
    list                   列出渠道
    status                 渠道状态
    login <channel>        登录渠道
    logout <channel>       登出渠道
  pairing                  配对管理
    list                   列出待批准
    approve                批准配对
    reject                 拒绝配对

网关命令:
  gateway                  网关控制
    run                    前台运行
    start                  启动服务
    stop                   停止服务
    restart                重启服务
    status                 查看状态
  status                   系统状态
  health                   健康检查
  logs                     查看日志
  system                   系统操作
  nodes                    节点管理
  devices                  设备管理

工具命令:
  cron                     定时任务
  browser                  浏览器控制
  sandbox                  沙箱管理
  plugins                  插件管理
  hooks                    Hook 管理
  webhooks                 Webhook 管理
  voicecall                语音通话

其他命令:
  dashboard                打开仪表盘
  tui                      TUI 界面
  dns                      DNS 工具
  docs                     打开文档
  security                 安全审计
  acp                      ACP 工具
```

---

## 💡 快速参考卡

### 每日必用
```bash
openclaw status              # 查看状态
openclaw health              # 健康检查
openclaw logs -f             # 实时日志
openclaw gateway status      # 网关状态
```

### 配置管理
```bash
openclaw config get agents.defaults.model
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4"
openclaw doctor              # 检查配置
```

### 渠道管理
```bash
openclaw channels list
openclaw channels status
openclaw pairing list whatsapp
openclaw pairing approve whatsapp ABC123
```

### 调试诊断
```bash
openclaw status --all        # 完整状态
openclaw status --deep       # 深度探测
openclaw logs --lines 500    # 更多日志
openclaw doctor --fix        # 自动修复
```

---

## 📚 相关文档

- [新手上路](/zh-CN/start/getting-started) - 首次使用指南
- [配置参考](/zh-CN/config/reference) - 所有配置项
- [故障排除](/zh-CN/help/troubleshooting) - 问题解决

---

**掌握 CLI，你就掌握了 OpenClaw 的全部力量！** 🦞
