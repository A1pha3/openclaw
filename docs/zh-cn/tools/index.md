---
summary: "工具与技能系统概述 - 浏览器、文件操作、命令执行等能力"
read_when:
  - 了解 OpenClaw 的工具能力
  - 配置技能系统
  - 自定义工具使用
title: "工具与技能"
---

# 🛠️ 工具与技能

本文档介绍 OpenClaw 的工具系统和技能机制，帮助您理解 AI 助手可以执行的操作。

## 🎯 什么是工具？

**工具（Tools）** 是 AI 代理可以调用的外部能力，包括：

- 📁 **文件操作** - 读取、写入、编辑文件
- 🔧 **命令执行** - 运行 Shell 命令
- 🌐 **网页浏览** - 搜索和访问网页
- 📝 **代码编辑** - 修改代码文件
- 🤖 **子代理** - 调用其他 AI 代理

---

## 📦 技能系统

### 什么是技能？

**技能（Skills）** 是预配置的工具体验，针对特定任务优化：

```
技能 = 工具组合 + 专用提示词 + 使用约束
```

### 技能类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **内置技能** | 核心功能 | `default`、`read`、`write` |
| **管理技能** | OpenClaw 维护 | `memory`、`sessions` |
| **外部技能** | 社区贡献 | `web`、`github` |

### 查看可用技能

```bash
# 列出所有技能
openclaw skills list

# 查看技能详情
openclaw skills info web

# 查看技能目录
openclaw skills dir
```

---

## 🔧 核心工具

### 文件操作工具

| 工具 | 功能 | 示例 |
|------|------|------|
| `read` | 读取文件内容 | `read(path: "file.txt")` |
| `write` | 创建/覆盖文件 | `write(path: "file.txt", content: "...")` |
| `edit` | 编辑文件部分内容 | `edit(path: "file.txt", old: "...", new: "...")` |
| `glob` | 查找文件 | `glob(pattern: "**/*.ts")` |
| `grep` | 搜索文件内容 | `grep(pattern: "function.*")` |

### 命令执行工具

| 工具 | 功能 | 示例 |
|------|------|------|
| `bash` | 执行 Shell 命令 | `bash(command: "npm install")` |
| `exec` | 执行命令并获取输出 | `exec(command: "ls -la")` |

### 网页工具

| 工具 | 功能 | 示例 |
|------|------|------|
| `browser` | 浏览器自动化 | `browser.go(url: "https://...")` |
| `web` | 网页搜索 | `web.search(query: "OpenClaw 文档")` |
| `firecrawl` | 网页抓取 | `firecrawl.scrape(url: "https://...")` |

### 代码工具

| 工具 | 功能 | 示例 |
|------|------|------|
| `lsp` | 语言服务器协议 | `lsp.go_to_definition(...)` |
| `edit` | 代码编辑 | `edit(...)` |

### 代理工具

| 工具 | 功能 | 示例 |
|------|------|------|
| `subagents` | 调用子代理 | `subagents.invoke(agent: "helper")` |
| `agent` | 调用 AI 代理 | `agent.send(message: "...")` |

---

## ⚙️ 工具配置

### 启用/禁用工具

```json5
{
  tools: {
    enabled: {
      read: true,
      write: true,
      edit: true,
      glob: true,
      grep: true,
      bash: true,
      browser: true,
      web: true
    }
  }
}
```

### 工具限制

为特定代理配置工具限制：

```json5
{
  agents: {
    list: [
      {
        id: "restricted",
        tools: {
          allow: ["read", "write", "edit"],
          deny: ["bash", "browser", "exec"]
        }
      }
    ]
  }
}
```

### 高级工具配置

```json5
{
  tools: {
    // Bash 工具配置
    bash: {
      timeout: 60000,        // 超时时间（毫秒）
      shell: "/bin/bash"     // Shell 路径
    },
    
    // 浏览器工具配置
    browser: {
      headless: true,        // 无头模式
      windowSize: [1280, 720]
    },
    
    // Web 工具配置
    web: {
      search: {
        timeout: 30000
      }
    }
  }
}
```

---

## 🎨 技能配置

### 安装技能

```bash
# 从 ClawHub 安装
openclaw skills install web
openclaw skills install github

# 查看可用的技能
openclaw skills search
```

### 技能配置

```json5
{
  skills: {
    // 技能配置
    web: {
      enabled: true,
      config: {
        search: {
          provider: "brave"
        }
      }
    },
    
    // 禁用技能
    github: {
      enabled: false
    }
  }
}
```

### 技能目录结构

```
~/.openclaw/workspace/skills/
└── <skill-name>/
    ├── SKILL.md         # 技能说明
    ├── PROMPT.md        # 专用提示词
    ├── config.json5     # 技能配置
    └── tools/           # 技能专用工具
```

---

## 🛡️ 安全与权限

### 工具安全级别

| 级别 | 工具 | 风险 |
|------|------|------|
| **安全** | `read`、`glob`、`grep` | 低 |
| **中等** | `write`、`edit`、`web` | 中 |
| **高风险** | `bash`、`exec`、`browser` | 高 |

### 沙箱中的工具

在沙箱模式下，高风险工具可能受限：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        tools: {
          allow: ["read", "write", "edit", "web"],
          deny: ["bash", "exec", "browser"]
        }
      }
    }
  }
}
```

### 提升权限

需要高风险工具时，可以临时提升权限：

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15551234567"]
      }
    }
  }
}
```

> **注意**：提升权限存在安全风险，请谨慎使用。

---

## 📊 工具使用统计

### 查看工具使用

```bash
# 查看当前会话的工具调用
openclaw sessions history <session-key>

# 查看工具调用统计
openclaw status --all
```

### 日志记录

```json5
{
  logging: {
    tools: {
      enabled: true,
      level: "info",
      redactSensitive: "tools"  // 脱敏敏感数据
    }
  }
}
```

---

## 🐛 故障排除

### 工具调用失败

```bash
# 检查工具是否启用
openclaw config get tools.enabled

# 查看工具日志
openclaw logs --category tools

# 诊断问题
openclaw doctor
```

### 常见问题

**问题：Bash 工具不可用**
```bash
# 检查是否被禁用
openclaw config get agents.defaults.tools.deny

# 启用 bash
openclaw config unset agents.defaults.tools.deny
```

**问题：浏览器无法启动**
```bash
# 检查 Chrome/Chromium
which google-chrome
which chromium

# 配置浏览器路径
openclaw config set tools.browser.executablePath "/usr/bin/chromium"
```

**问题：Web 搜索失败**
```bash
# 检查 API Key
openclaw config get tools.web.search.apiKey

# 配置 API Key
openclaw config set tools.web.search.apiKey "YOUR_API_KEY"
```

---

## 📝 最佳实践

### 开发环境配置

```json5
{
  agents: {
    defaults: {
      tools: {
        allow: ["read", "write", "edit", "glob", "grep", "bash", "browser", "web"],
        deny: []
      }
    }
  }
}
```

### 生产环境配置

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session"
      },
      tools: {
        allow: ["read", "write", "edit", "glob", "grep"],
        deny: ["bash", "exec", "browser"]
      }
    }
  }
}
```

### 受限环境配置

```json5
{
  agents: {
    list: [
      {
        id: "public",
        tools: {
          allow: ["read", "glob"],
          deny: ["bash", "exec", "browser", "write", "edit", "web"]
        }
      }
    ]
  }
}
```

---

## 🔧 相关命令

| 命令 | 说明 |
|------|------|
| `openclaw skills list` | 列出技能 |
| `openclaw skills install <name>` | 安装技能 |
| `openclaw skills uninstall <name>` | 卸载技能 |
| `openclaw skills dir` | 技能目录 |
| `openclaw config` | 配置工具和技能 |
| `openclaw sessions` | 查看工具调用历史 |

---

## 📚 相关文档

- [CLI 参考](/zh-CN/cli) - 所有命令
- [配置参考](/zh-CN/config/reference) - 完整配置
- [技能开发](/zh-CN/tools/creating-skills) - 创建自定义技能
- [浏览器工具](/zh-CN/tools/browser) - 浏览器自动化
- [执行工具](/zh-CN/tools/exec) - 命令执行

---

**工具和技能是 AI 助手能力的延伸。合理配置，让您的助手既强大又安全！** 🦞
