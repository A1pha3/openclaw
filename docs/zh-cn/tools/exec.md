---
summary: "Exec命令执行工具完整指南——安全风险、控制选项与权限管理"
read_when:
  - 使用或修改exec工具
  - 调试stdin或TTY行为
  - 配置执行批准和安全策略
title: "Exec命令执行工具"
---

# Exec命令执行工具

本指南全面介绍OpenClaw的exec命令执行工具，帮助您理解如何安全地在系统中执行Shell命令。exec是OpenClaw中最强大但也最危险的工具，需要谨慎配置和使用。完成本章节学习后，您将能够安全地使用exec工具，理解其权限模型，并掌握安全最佳实践。

## 学习目标

完成本章节学习后，您将能够：

### 基础目标（必掌握）

- 理解exec工具的核心功能和与bash工具的区别
- 掌握exec的基本用法和参数配置
- 理解执行主机（host）的概念：sandbox、gateway、node
- 认识exec工具的安全风险等级

### 进阶目标（建议掌握）

- 配置执行批准策略和白名单机制
- 实现前后台任务管理和进程控制
- 配置PATH处理和环境变量
- 为不同场景选择合适的执行主机

### 专家目标（挑战）

- 设计安全的工具权限体系
- 优化exec配置以平衡功能与安全
- 实现企业级的执行审计和审批流程

---

## 第一部分：核心概念解析

### 为什么exec是高风险工具

在深入技术细节之前，我们必须首先理解exec工具为何被视为高风险。

**核心问题**：exec工具允许AI执行任意Shell命令，这意味着：

| 风险类型 | 可能后果 |
|----------|----------|
| **数据破坏** | `rm -rf /` 可能删除系统文件 |
| **权限滥用** | 执行未经授权的管理操作 |
| **安全绕过** | 通过命令修改安全配置 |
| **资源耗尽** | 运行消耗大量CPU/内存的进程 |
| **持久化攻击** | 创建后门或修改启动脚本 |

**设计权衡**：
- 赋予AI执行命令的能力使其能够完成复杂任务
- 但这同时打开了安全风险的潘多拉魔盒
- OpenClaw通过多层次的权限控制来缓解这些风险

### exec与bash的区别

很多用户会混淆exec和bash工具，让我们明确它们的区别：

| 特性 | exec | bash |
|------|------|------|
| **执行模式** | 仅同步执行 | 支持同步和异步 |
| **yieldMs参数** | 被忽略 | 支持 |
| **background参数** | 被忽略 | 支持 |
| **进程控制** | 无 | 支持process工具 |
| **适用场景** | 快速单条命令 | 长时间运行任务 |

**简单理解**：
- exec是"fire and forget"——发射后不管
- bash是"interactive"——支持持续交互

**选择建议**：
- 只需要获取命令输出时使用exec
- 需要监控、发送按键或管理进程时使用bash

### 执行主机（Host）概念

exec工具支持三种执行主机，理解它们是安全配置的关键：

```
执行主机类型：

┌─────────────────────────────────────────────────────────────┐
│ host=sandbox（默认）                                         │
│ ├── 用途：在沙箱容器中执行（如果沙箱启用）                     │
│ ├── 安全级别：高                                             │
│ ├── 限制：受限的文件系统和网络访问                           │
│ └── 适用：不可信代码或外部用户                               │
├─────────────────────────────────────────────────────────────┤
│ host=gateway                                                │
│ ├── 用途：在网关主机上执行                                   │
│ ├── 安全级别：中等                                           │
│ ├── 限制：需要批准（除非配置为full）                         │
│ └── 适用：需要访问本地资源的操作                             │
├─────────────────────────────────────────────────────────────┤
│ host=node                                                   │
│ ├── 用途：在配对的节点主机上执行                             │
│ ├── 安全级别：取决于节点配置                                 │
│ ├── 限制：需要配对的节点                                     │
│ └── 适用：设备特定操作（如macOS系统命令）                    │
└─────────────────────────────────────────────────────────────┘
```

**重要说明**：沙箱默认关闭——这意味着`host=sandbox`可能直接在网关主机上运行。

---

## 第二部分：工具详解

### 2.1 核心参数

exec工具支持以下参数：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| command | string | 是 | 要执行的命令 |
| workdir | string | 否 | 工作目录，默认为当前目录 |
| env | object | 否 | 环境变量覆盖 |
| timeout | number | 否 | 超时时间（秒），默认1800 |
| pty | boolean | 否 | 是否使用伪终端 |
| host | string | 否 | 执行主机：sandbox/gateway/node |
| security | string | 否 | 安全模式：deny/allowlist/full |
| ask | string | 否 | 批准策略：off/on-miss/always |
| node | string | 否 | 节点ID（当host=node时） |
| elevated | boolean | 否 | 请求提权模式 |

### 2.2 前台执行

最基本的exec调用——同步执行命令并等待结果：

```json
{
  "tool": "exec",
  "command": "ls -la"
}
```

**返回结果示例**：

```json
{
  "success": true,
  "exitCode": 0,
  "stdout": "total 40\ndrwxr-xr-x   5 user  staff  170 Feb  5 10:00 docs\n-rw-r--r--   1 user  staff  120 Feb  5 10:00 README.md",
  "stderr": ""
}
```

### 2.3 带超时的执行

```json
{
  "tool": "exec",
  "command": "npm install",
  "timeout": 300
}
```

**超时行为**：
- 超时后进程被杀死
- 返回`exitCode: -1`和超时信息
- 建议长时间任务使用bash的background模式

### 2.4 指定工作目录和环境

```json
{
  "tool": "exec",
  "command": "npm run build",
  "workdir": "/path/to/project",
  "env": {
    "NODE_ENV": "production",
    "API_KEY": "secret"
  }
}
```

**注意事项**：
- `env`参数仅覆盖指定的环境变量，其他变量保持原值
- 不要在`env`中传递敏感信息——它们可能出现在日志中

### 2.5 伪终端（PTY）模式

```json
{
  "tool": "exec",
  "command": "top",
  "pty": true
}
```

**适用场景**：
- 需要TTY的CLI工具（如vim、nano）
- 需要交互式输入的程序
- 终端UI程序（如htop）

**限制**：
- 仅在TTY可用的环境中工作
- 可能影响输出格式

---

## 第三部分：高级控制

### 3.1 前后台执行（通过bash）

exec本身不支持前后台切换，需要通过bash工具实现：

```json
// 启动后台任务
{
  "tool": "bash",
  "command": "npm run build",
  "yieldMs": 1000,
  "background": true
}
```

**返回结果**：

```json
{
  "status": "accepted",
  "sessionId": "exec-session-123",
  "sessionKey": "process:abc123"
}
```

### 3.2 进程控制

启动后台任务后，可以使用process工具进行控制：

```json
// 轮询执行状态
{
  "tool": "process",
  "action": "poll",
  "sessionId": "exec-session-123"
}
```

```json
// 发送按键
{
  "tool": "process",
  "action": "send-keys",
  "sessionId": "exec-session-123",
  "keys": ["Enter"]
}
```

```json
// 发送Ctrl+C
{
  "tool": "process",
  "action": "send-keys",
  "sessionId": "exec-session-123",
  "keys": ["C-c"]
}
```

```json
// 提交表单（仅发送回车）
{
  "tool": "process",
  "action": "submit",
  "sessionId": "exec-session-123"
}
```

```json
// 粘贴文本
{
  "tool": "process",
  "action": "paste",
  "sessionId": "exec-session-123",
  "text": "line1\nline2\n"
}
```

### 3.3 会话覆盖（/exec命令）

可以通过`/exec`命令设置每个会话的默认参数：

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

**可用参数**：
- `host`——默认执行主机
- `security`——默认安全模式
- `ask`——默认批准策略
- `node`——默认节点ID

**查看当前设置**：

发送不带参数的`/exec`命令即可显示当前配置。

---

## 第四部分：安全模型详解

### 4.1 安全模式

exec工具支持三种安全模式：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **deny** | 默认拒绝所有命令 | 最安全，但限制功能 |
| **allowlist** | 仅允许白名单中的命令 | 平衡安全与功能 |
| **full** | 允许所有命令 | 高风险，仅可信环境 |

**allowlist模式详细说明**：
- 仅允许白名单中**完整路径**的命令
- 不支持basename匹配——必须指定完整路径
- 管道、链接（`;`、`&&`、`||`）在allowlist模式下被拒绝
- 支持安全bin（stdin-only命令，无需显式白名单）

### 4.2 执行批准机制

对于在gateway或node上执行的命令，可以配置执行批准：

```json5
{
  tools: {
    exec: {
      approvalRunningNoticeMs: 10000,  // 运行超过10秒时通知
      notifyOnExit: true                // 退出时通知
    }
  }
}
```

**批准流程**：

```
请求执行
    │
    ▼
需要批准吗？
    │
    ├─否→ 直接执行
    │
    └─是→ 返回 approval-pending
              │
              ▼
         用户审批
              │
              ├─批准→ 执行命令
              │
              ├─拒绝→ 返回拒绝
              │
              └─超时→ 返回超时
```

**批准配置存储位置**：~/.openclaw/exec-approvals.json

### 4.3 提权模式

对于需要管理员权限的操作，可以请求提权：

```json
{
  "tool": "exec",
  "command": "apt-get update",
  "elevated": true,
  "host": "gateway"
}
```

**注意事项**：
- 仅当提权策略解析为`full`时强制`security=full`
- 沙箱关闭时忽略`elevated`
- 需要预先配置提权策略

### 4.4 PATH处理

不同主机类型的PATH处理方式不同：

| 主机类型 | PATH行为 |
|----------|----------|
| **gateway** | 合并登录shell的PATH，但守护进程以最小PATH运行 |
| **sandbox** | 在容器内运行登录shell，/etc/profile可能重置PATH |
| **node** | 只发送env覆盖，macOS节点完全删除PATH覆盖 |

**macOS默认PATH**：
```
/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin
```

**Linux默认PATH**：
```
/usr/local/bin:/usr/bin:/bin
```

**自定义PATH**：

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

---

## 第五部分：节点执行

### 5.1 节点绑定

exec可以配置为在特定节点上执行：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          exec: {
            node: "macbook-pro"
          }
        }
      }
    ]
  }
}
```

### 5.2 节点配置

不同类型节点的PATH处理：

| 节点类型 | PATH处理 |
|----------|----------|
| **无头节点主机** | 预先添加节点PATH，不替换 |
| **macOS节点** | 完全删除PATH覆盖 |

### 5.3 使用示例

```json
{
  "tool": "exec",
  "command": "system_profiler SPSoftwareDataType",
  "host": "node",
  "node": "macbook-pro"
}
```

---

## 第六部分：安全最佳实践

### 6.1 开发环境配置

开发环境可以放宽限制，但仍需基本的安全控制：

```json5
{
  tools: {
    exec: {
      host: "gateway",
      security: "allowlist",
      ask: "on-miss",
      pathPrepend: ["~/bin"]
    }
  }
}
```

**推荐的白名单命令**（开发环境）：

```json5
{
  tools: {
    exec: {
      security: "allowlist",
      safeBins: ["cat", "head", "tail", "wc", "grep", "echo"]
    }
  }
}
```

### 6.2 生产环境配置

生产环境应使用最严格的限制：

```json5
{
  tools: {
    exec: {
      host: "sandbox",
      security: "deny",
      ask: "always"
    }
  }
}
```

### 6.3 受限环境配置

对外部用户或不信任的来源：

```json5
{
  agents: {
    list: [
      {
        id: "public",
        tools: {
          deny: ["exec", "bash"]
        }
      }
    ]
  }
}
```

### 6.4 安全检查清单

配置exec工具前，检查以下项目：

- [ ] 是否真正需要exec工具？如果不需要，直接禁用
- [ ] 执行主机是否正确选择？sandbox优先于gateway
- [ ] 安全模式是否适当？allowlist优于full
- [ ] 是否配置了执行批准？
- [ ] 是否记录了所有执行操作？
- [ ] 敏感信息（如API密钥）是否不会出现在命令中？

---

## 第七部分：使用场景

### 场景一：代码构建

**需求**：执行项目构建命令

**安全配置**：

```json5
{
  tools: {
    exec: {
      security: "allowlist",
      pathPrepend: ["/usr/local/bin", "/opt/homebrew/bin"]
    }
  }
}
```

**白名单配置**：

```json5
{
  tools: {
    exec: {
      security: "allowlist",
      allowedPaths: [
        "/usr/local/bin/npm",
        "/usr/local/bin/pnpm",
        "/usr/local/bin/docker"
      ]
    }
  }
}
```

### 场景二：文件操作

**需求**：使用系统命令处理文件

**推荐方式**：优先使用file工具而非exec

```
如果必须使用exec：
├── 使用read工具读取文件内容
├── 在代码中处理
└── 使用write工具写入结果
```

**如果必须执行shell命令**：

```json
{
  "tool": "exec",
  "command": "grep -r 'pattern' .",
  "workdir": "/path/to/project"
}
```

### 场景三：系统查询

**需求**：查询系统信息

**安全配置**：

```json5
{
  "agents": {
    "defaults": {
      "tools": {
        "exec": {
          "security": "allowlist",
          "allowedPaths": [
            "/bin/ps",
            "/usr/bin/top",
            "/bin/df",
            "/usr/bin/free"
          ]
        }
      }
    }
  }
}
```

### 场景四：容器操作

**需求**：管理Docker容器

**警告**：Docker操作风险极高，应严格控制

**最小权限配置**：

```json5
{
  tools: {
    exec: {
      security: "allowlist",
      allowedPaths: [
        "/usr/local/bin/docker",
        "/usr/bin/docker"
      ],
      ask: "always"
    }
  }
}
```

### 场景五：自动化脚本

**需求**：运行定时或事件驱动的脚本

**推荐做法**：
1. 将脚本写入文件
2. 审核脚本内容
3. 通过批准后执行

**执行审批流程**：

```
编写脚本 → 写入文件 → 管理员审核 → 批准执行
```

---

## 第八部分：故障排除

### 8.1 常见错误类型

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| 127 | 命令未找到 | 检查PATH或完整路径 |
| 126 | 权限不足 | 检查文件权限 |
| 1 | 一般性错误 | 检查命令输出 |
| -1 | 超时 | 检查timeout设置 |

### 8.2 命令未找到

**症状**：返回错误"command not found"

**解决方案**：

```bash
# 1. 使用完整路径
{
  "tool": "exec",
  "command": "/usr/local/bin/npm"
}

# 2. 或配置pathPrepend
{
  "tool": "exec",
  "command": "npm install",
  "env": {
    "PATH": "/usr/local/bin:/usr/bin:/bin"
  }
}
```

### 8.3 权限被拒绝

**症状**：返回错误"Permission denied"

**解决方案**：

```bash
# 1. 检查文件权限
ls -la /path/to/command

# 2. 如果需要，使用具有正确权限的路径
# 例如使用/usr/local/bin/而非~/bin/

# 3. 或请求提权（谨慎使用）
{
  "tool": "exec",
  "command": "apt-get update",
  "elevated": true
}
```

### 8.4 执行超时

**症状**：命令未在指定时间内完成

**解决方案**：

```json
{
  "tool": "exec",
  "command": "long-running-command",
  "timeout": 600  // 增加超时时间
}
```

**或使用后台执行**：

```json
{
  "tool": "bash",
  "command": "long-running-command",
  "yieldMs": 5000,
  "background": true
}
```

### 8.5 allowlist模式失败

**症状**：allowlist模式下命令被拒绝

**解决方案**：

```bash
# 1. 检查是否在白名单中
{
  "tool": "exec",
  "command": "which mycommand"
}

# 2. 添加到白名单
{
  "tool": "exec",
  "command": "/full/path/to/mycommand"
}

# 3. 或添加到safeBins（stdin-only命令）
{
  "tool": "exec",
  "command": "cat"
}
```

### 8.6 环境变量问题

**症状**：命令无法找到依赖的库或工具

**解决方案**：

```json
{
  "tool": "exec",
  "command": "mycommand",
  "env": {
    "LD_LIBRARY_PATH": "/path/to/libs",
    "PYTHONPATH": "/path/to/modules"
  }
}
```

### 8.7 调试执行问题

当exec出现问题时，使用以下调试流程：

```bash
# 1. 简化命令测试
{
  "tool": "exec",
  "command": "echo test"
}

# 2. 添加详细输出
{
  "tool": "exec",
  "command": "bash -x your-command"
}

# 3. 检查工作目录
{
  "tool": "exec",
  "command": "pwd"
}

# 4. 列出环境变量
{
  "tool": "exec",
  "command": "env | sort"
}
```

---

## 第九部分：专家思维模型

### 9.1 安全决策框架

```
需要执行命令？
    │
    ▼
使用file工具能完成吗？
    │
    ├─是→ 使用file工具（read/write/edit）
    │
    └─否→ 必须使用exec
              │
              ▼
         沙箱是否启用？
              │
              ├─是→ host=sandbox + minimal permissions
              │
              └─否→ 考虑启用沙箱或使用受限配置
                        │
                        ▼
                   命令是否可信？
                        │
                        ├─是→ allowlist + minimal ask
                        │
                        └─否→ full ask + 详细日志
```

### 9.2 风险评估矩阵

| 命令类型 | 风险等级 | 配置建议 |
|----------|----------|----------|
| 只读查询（ps、df、free） | 低 | allowlist |
| 文件读取（cat、head） | 低 | allowlist + safeBins |
| 文件写入（echo、tee） | 中 | allowlist + ask |
| 包管理（npm、pip） | 中 | allowlist + ask |
| 系统修改（chmod、chown） | 高 | full ask + 详细日志 |
| 网络操作（curl、wget） | 高 | full ask + 详细日志 |
| 系统管理（sudo、apt） | 极高 | 禁用或严格审批 |

### 9.3 最小权限原则实践

**步骤**：
1. 从禁用所有exec开始
2. 只添加明确需要的命令
3. 使用完整路径而非basename
4. 设置最严格的security模式
5. 启用执行批准
6. 定期审查和清理白名单

---

## 适用场景速查

| 场景 | 推荐配置 | 关键注意事项 |
|------|----------|---------------|
| 代码构建 | host=gateway, security=allowlist | 白名单构建命令 |
| 系统查询 | host=sandbox, security=allowlist | 限制为只读命令 |
| 文件处理 | host=sandbox, security=allowlist | 优先使用file工具 |
| 容器管理 | host=gateway, security=allowlist, ask=always | 严格审批 |
| 外部用户 | tools.deny: ["exec"] | 完全禁用 |
| 开发调试 | host=gateway, security=allowlist | 宽松但有控制 |

---

## 相关文档

- [工具配置参考](/zh-CN/config/reference)——完整配置选项
- [执行批准机制](/tools/exec-approvals)——审批策略配置
- [提权模式](/tools/elevated)——提权配置
- [会话管理](/concepts/session)——会话上下文

---

**安全警告**：exec工具是最强大的工具之一，但也带来最高的安全风险。在配置和使用exec时，始终遵循最小权限原则，记录和审计所有执行操作，定期审查和更新安全配置。不确定时，宁可保守也不冒险。