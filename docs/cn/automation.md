---
summary: "自动化完整指南——Cron 定时任务、Webhook 触发、工作流自动化，让 OpenClaw 自动执行预设任务"
read_when:
  - 配置定时任务自动执行
  - 设置 Webhook 接收外部事件
  - 实现复杂的自动化工作流
  - 了解自动化的最佳实践
title: "自动化"
---

# ⏰ 自动化

自动化是 OpenClaw 帮助你从繁琐重复的任务中解放出来的核心能力。通过自动化，你可以在特定时间自动执行任务、通过外部事件触发操作、设置条件判断实现智能响应。想象一下：每天早上 8 点自动发送天气摘要；收到 GitHub webhook 时自动部署代码；检测到异常时立即发送告警——这些都是自动化能够实现的功能。本文将详细介绍 OpenClaw 的自动化系统，帮助你建立高效的工作流程。

> **自动化的核心价值**
>
> 手动重复的任务不仅消耗时间，还容易出错。自动化将这些任务交给系统执行，确保每次执行都准确无误。更重要的是，自动化释放了你的注意力，让你专注于真正需要人类智慧的工作。在 AI 助手的场景下，自动化让 AI 能够主动提供服务，而不是被动等待指令——这是一种从"工具"到"助手"的本质转变。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解自动化的基本概念和类型
- 配置 Cron 定时任务
- 设置 Webhook 接收外部事件
- 创建简单的自动化规则

### 进阶目标（建议掌握）

- 设计复杂的多条件自动化工作流
- 实现任务的错误处理和重试
- 优化自动化任务的性能
- 集成多个外部服务

### 专家目标（挑战）

- 开发自定义自动化模块
- 实现分布式任务调度
- 设计企业级自动化架构
- 建立完善的监控和告警系统

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **定时任务** | 需要定时执行任务 | 30 分钟 | Cron 表达式、定时任务配置 |
| **事件触发** | 需要响应外部事件 | 45 分钟 | Webhook 配置、事件处理 |
| **工作流** | 需要复杂自动化 | 2 小时 | 条件判断、串并行执行 |
| **高级扩展** | 开发者扩展自动化 | 4 小时 | 自定义触发器、执行器 |

---

## 🧠 原理解释：自动化系统架构

### 自动化系统的核心组件

OpenClaw 的自动化系统由以下核心组件构成：

**触发器（Trigger）**

触发器是自动化流程的起点，决定了何时启动自动化任务。OpenClaw 支持多种触发器类型：时间触发器——基于 Cron 表达式或固定间隔触发；事件触发器——基于 Webhook、消息到达等事件触发；条件触发器——基于系统状态或外部条件触发；手动触发器——由用户显式触发的任务。

**执行器（Executor）**

执行器负责实际执行自动化任务中的操作：执行动作——如发送消息、调用工具、执行命令；错误处理——处理执行过程中的异常和失败；结果处理——根据执行结果决定后续操作。

**调度器（Scheduler）**

调度器管理所有定时任务的执行时间：维护任务队列，确保按时执行；处理时区和夏令时问题；支持任务的暂停、恢复和取消。

**工作流引擎（Workflow Engine）**

工作流引擎支持复杂的多步骤自动化流程：顺序执行——按顺序执行一系列操作；并行执行——同时执行多个独立操作；条件分支——根据条件选择执行路径；循环执行——重复执行直到满足条件。

### 自动化执行流程

```
事件/时间触发器激活
          │
          ▼
┌─────────────────────────────────────┐
│         条件检查                      │
│    （如果配置了条件）                  │
└─────────────────────────────────────┘
          │
          ├── 条件不满足 → 结束
          │
          ▼
┌─────────────────────────────────────┐
│         加载工作流                     │
│     读取自动化规则定义                 │
└─────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────┐
│         执行步骤                      │
│     按顺序/并行执行每个步骤            │
└─────────────────────────────────────┘
          │
          ├── 执行失败 → 错误处理
          │
          ▼
┌─────────────────────────────────────┐
│         完成处理                      │
│     记录日志、发送通知、清理资源        │
└─────────────────────────────────────┘
```

---

## ⏱️ Cron 定时任务

### Cron 表达式基础

Cron 表达式是用于描述时间调度的标准格式，由五个字段组成：

```
┌───────────── 分钟 (0 - 59)
│ ┌─────────── 小时 (0 - 23)
│ │ ┌───────── 月内日期 (1 - 31)
│ │ │ ┌─────── 月份 (1 - 12)
│ │ │ │ ┌───── 周内日期 (0 - 6) (星期日 = 0)
│ │ │ │ │
* * * * *
```

**常用示例**：

| 表达式 | 说明 |
|--------|------|
| `* * * * *` | 每分钟执行 |
| `0 * * * *` | 每小时执行（第 0 分钟） |
| `0 9 * * *` | 每天早上 9 点执行 |
| `0 9 * * 1` | 每周一早上 9 点执行 |
| `0 0 1 * *` | 每月 1 号午夜执行 |
| `*/5 * * * *` | 每 5 分钟执行 |
| `0 0 * * 0` | 每周日午夜执行 |

**高级语法**：

| 语法 | 说明 | 示例 |
|------|------|------|
| `,` | 多个值 | `1,3,5 * * * *`（每小时的 1、3、5 分钟执行） |
| `-` | 范围 | `0 9-17 * * *`（每天 9 点到 17 点每小时执行） |
| `/` | 步长 | `*/15 * * * *`（每 15 分钟执行） |

### 创建定时任务

**方法一：命令行创建**

```bash
# 创建每日天气提醒任务
openclaw cron add \
  --name "daily-weather" \
  --schedule "0 8 * * *" \
  --command "tools.weather --location Beijing"

# 创建每周报告任务
openclaw cron add \
  --name "weekly-summary" \
  --schedule "0 18 * * 5" \
  --command "agents.summary --type weekly"

# 创建每小时健康检查
openclaw cron add \
  --name "hourly-health" \
  --schedule "0 * * * *" \
  --command "system.health-check"
```

**方法二：配置文件创建**

```yaml
# ~/.openclaw/cron/daily-backup.yaml
name: daily-backup
description: 每日自动备份工作区数据

schedule: "0 3 * * *"  # 每天凌晨 3 点执行

# 执行动作
actions:
  - type: exec
    command: "restic backup /workspace --repo /backup/openclaw"
    timeout: 3600000  # 1 小时超时

  - type: notification
    title: "备份完成"
    body: "每日备份已成功完成"

# 错误处理
onError:
  - type: notification
    title: "备份失败"
    body: "备份任务执行失败，请检查日志"
```

### 管理定时任务

```bash
# 列出所有定时任务
openclaw cron list

# 查看任务详情
openclaw cron info <task-name>

# 暂停任务
openclaw cron pause <task-name>

# 恢复任务
openclaw cron resume <task-name>

# 删除任务
openclaw cron remove <task-name>

# 手动执行任务（测试）
openclaw cron run <task-name>
```

---

## 🔗 Webhook 触发

### Webhook 基础

Webhook 是一种通过 HTTP 请求触发自动化任务的机制。当外部系统发生特定事件时，它会向 OpenClaw 发送一个 HTTP 请求，OpenClaw 收到请求后执行相应的自动化任务。

**Webhook 的工作原理**：

```
GitHub 代码推送事件
        │
        ▼
HTTP POST 请求到 OpenClaw Webhook 端点
        │
        ▼
OpenClaw 验证请求签名
        │
        ├── 签名无效 → 返回 401/403
        │
        ▼
解析请求体中的事件数据
        │
        ▼
匹配对应的自动化规则
        │
        ▼
执行预设的任务序列
```

### 配置 Webhook

**步骤一：启用 Webhook 服务**

```json5
{
  automation: {
    webhook: {
      enabled: true,
      // 监听端口
      port: 18790,
      // 路径前缀
      pathPrefix: "/webhook",
      // 是否启用 HTTPS
      https: false,
      
      // 认证配置
      auth: {
        // 是否验证请求签名
        verifySignature: true,
        // 密钥（用于签名验证）
        secret: "${WEBHOOK_SECRET}"
      }
    }
  }
}
```

**步骤二：创建 Webhook 自动化规则**

```yaml
# ~/.openclaw/automation/github-push.yaml
name: github-push-handler
description: 处理 GitHub 代码推送事件

# Webhook 配置
webhook:
  # 事件类型
  events:
    - push
  # GitHub 配置
  github:
    # 验证签名
    verifySignature: true
    # 仓库白名单
    allowedRepos:
      - "owner/repo1"
      - "owner/repo2"

# 执行的动作
actions:
  - type: exec
    command: "./scripts/deploy.sh"
    env:
      REPO: "{{event.repository.full_name}}"
      BRANCH: "{{event.ref}}"
      COMMIT: "{{event.head_commit.id}}"

  - type: sendMessage
    channel: "deployments"
    text: |
      🚀 代码已部署！
      
      仓库：{{event.repository.full_name}}
      分支：{{event.ref}}
      提交：{{event.head_commit.message}}
```

**步骤三：在外部服务配置 Webhook URL**

```
Webhook URL 格式：
https://your-domain.com/webhook/github/push

Headers：
Content-Type: application/json
X-GitHub-Event: push
X-Hub-Signature-256: sha256=...

Body（示例）：
{
  "action": "opened",
  "number": 42,
  "repository": {
    "full_name": "owner/repo",
    "name": "repo"
  }
}
```

### 支持的 Webhook 提供商

| 提供商 | 事件类型 | 签名验证 | 说明 |
|--------|----------|----------|------|
| **GitHub** | push、pull_request、issues | SHA256 | 需配置 Webhook Secret |
| **GitLab** | Push Hook、Merge Request | SHA256 | 需配置 Secret Token |
| **Slack** | message、reaction_added | SHA256 | 需配置 Signing Secret |
| **Discord** | MESSAGE_CREATE、INTERACTION_CREATE | SHA256 | 需配置 Public Key |
| **Custom** | 任意 | 可配置 | 自定义 webhook 端点 |

### 自定义 Webhook 端点

```json5
{
  automation: {
    webhook: {
      enabled: true,
      customEndpoints: [
        {
          // 端点路径
          path: "/webhook/custom/myapp",
          // HTTP 方法
          method: "POST",
          // 认证方式
          auth: {
            type: "bearer",
            token: "${CUSTOM_WEBHOOK_TOKEN}"
          },
          // 响应处理
          handler: {
            type: "automation",
            automation: "my-custom-automation"
          }
        }
      ]
    }
  }
}
```

---

## 🔄 工作流自动化

### 工作流定义

工作流是复杂自动化任务的核心，通过组合多个操作、条件判断和错误处理，实现复杂的业务逻辑。

**工作流的基本结构**：

```yaml
name: daily-report-workflow
description: 每日数据报告生成工作流

# 触发条件
trigger:
  type: cron
  schedule: "0 8 * * *"  # 每天早上 8 点

# 输入参数
inputs:
  - name: date
    type: string
    default: "${CURRENT_DATE}"
  - name: recipients
    type: array
    default: ["team@example.com"]

# 工作流步骤
steps:
  - name: fetch-data
    description: 从数据库获取昨日数据
    action:
      type: tool
      tool: "database.query"
      params:
        query: "SELECT * FROM metrics WHERE date = '{{inputs.date}}'"
        
  - name: generate-report
    description: 生成分析报告
    action:
      type: tool
      tool: "report.generate"
      params:
        data: "${steps.fetch-data.result}"
        template: "daily-summary"
        
  - name: send-email
    description: 发送邮件给团队
    action:
      type: tool
      tool: "email.send"
      params:
        to: "${inputs.recipients}"
        subject: "每日报告 - {{inputs.date}}"
        attachment: "${steps.generate-report.output}"
```

### 条件分支

```yaml
steps:
  - name: check-status
    description: 检查服务状态
    action:
      type: tool
      tool: "monitor.check"
      params:
        service: "api-server"
    outputVariable: "status"
    
  - name: status-check
    description: 根据状态决定后续步骤
    type: conditional
    condition: "${steps.check-status.output.status}"
    branches:
      - condition: "healthy"
        steps:
          - name: log-healthy
            action:
              type: log
              message: "服务状态正常"
              
      - condition: "warning"
        steps:
          - name: log-warning
            action:
              type: notification
              title: "服务警告"
              body: "服务状态为警告"
          - name: send-alert
            action:
              type: email.send
              to: ["ops@example.com"]
              body: "服务出现警告状态"
              
      - condition: "critical"
        steps:
          - name: critical-alert
            action:
              type: notification
              title: "🚨 严重告警"
              body: "服务已不可用"
          - name: page-oncall
            action:
              type: tool
              tool: "pagerduty.trigger"
              params:
                severity: "critical"
```

### 并行执行

```yaml
steps:
  - name: fetch-all-data
    description: 并行获取多个数据源
    type: parallel
    maxConcurrency: 5
    tasks:
      - name: sales-data
        action:
          type: tool
          tool: "database.query"
          params:
            query: "SELECT * FROM sales"
            
      - name: user-data
        action:
          type: tool
          tool: "database.query"
          params:
            query: "SELECT * FROM users"
            
      - name: product-data
        action:
          type: tool
          tool: "database.query"
          params:
            query: "SELECT * FROM products"
```

### 错误处理

```yaml
steps:
  - name: risky-operation
    description: 可能失败的操作
    action:
      type: tool
      tool: "api.call"
      params:
        url: "https://external-service/data"
        
  - name: handle-error
    description: 错误处理分支
    type: error-handler
    on:
      - error
    steps:
      - name: log-error
        action:
          type: log
          message: "操作失败：${error.message}"
      - name: notify
        action:
          type: notification
          title: "自动化任务失败"
          body: "任务 'risky-operation' 执行失败"
      - name: fallback
        action:
          type: tool
          tool: "cache.get"
          params:
            key: "fallback-data"
```

---

## 📊 自动化监控

### 查看执行历史

```bash
# 查看所有执行记录
openclaw automation history

# 查看特定自动化的执行历史
openclaw automation history <automation-name>

# 查看执行详情
openclaw automation history <automation-name> --execution-id <id>
```

### 监控配置

```json5
{
  automation: {
    monitoring: {
      // 是否启用执行日志
      logExecutions: true,
      
      // 日志保留时间（天）
      retentionDays: 30,
      
      // 指标收集
      metrics: {
        enabled: true,
        // 收集间隔（秒）
        interval: 60
      },
      
      // 告警配置
      alerts: {
        // 执行失败告警
        onFailure: {
          enabled: true,
          // 连续失败次数阈值
          threshold: 3,
          // 通知方式
          notify: ["notification", "email"]
        },
        
        // 执行超时告警
        onTimeout: {
          enabled: true,
          // 超时时间（毫秒）
          threshold: 300000,
          notify: ["notification"]
        }
      }
    }
  }
}
```

---

## 🚨 故障排除

### 问题一：定时任务未执行

**症状**：Cron 任务没有在预期时间执行。

**诊断步骤**：

```bash
# 检查任务是否存在
openclaw cron list

# 查看任务状态
openclaw cron info <task-name>

# 检查 Gateway 是否在运行
openclaw gateway status

# 查看调度日志
openclaw logs | grep cron
```

**常见原因**：Gateway 服务未运行；任务被暂停；Cron 表达式配置错误；系统时间不正确。

### 问题二：Webhook 未触发

**症状**：外部服务发送了请求但自动化未执行。

**诊断步骤**：

```bash
# 查看 Webhook 请求日志
openclaw logs | grep webhook

# 验证 Webhook 配置
openclaw config get automation.webhook

# 测试 Webhook 端点
curl -X POST http://localhost:18790/webhook/test -d '{}'
```

**常见原因**：Webhook 端点配置错误；签名验证失败；自动化规则未匹配；网络连接问题。

### 问题三：工作流执行失败

**症状**：工作流中的某个步骤失败导致整个流程中断。

**解决方案**：添加错误处理步骤；增加重试机制；检查前置依赖步骤的输出；查看详细错误日志。

---

## 📋 适用场景分析

### 场景一：每日工作流自动化

**目标**：每天自动完成重复性工作

**配置**：
- Cron 任务：每日早上 8 点
- 工作流：获取数据 → 分析 → 生成报告 → 发送邮件

**收益**：节省每日 30 分钟重复工作；确保报告准时发送。

### 场景二：DevOps 自动化

**目标**：代码推送后自动部署

**配置**：
- Webhook：GitHub push 事件
- 工作流：拉取代码 → 运行测试 → 构建镜像 → 部署到服务器 → 发送通知

**收益**：缩短部署周期；减少人为错误；提高发布频率。

### 场景三：监控告警自动化

**目标**：异常情况自动响应

**配置**：
- 触发器：监控告警
- 工作流：分析告警 → 尝试自动修复 → 通知相关人员

**收益**：缩短故障响应时间；减少人工干预；提高系统可用性。

### 场景四：跨系统集成

**目标**：多个系统之间自动同步数据

**配置**：
- Webhook：多个外部系统事件
- 工作流：接收事件 → 数据转换 → 同步到目标系统

**收益**：消除手动数据同步；确保数据一致性；提高协作效率。

---

## 📖 相关文档

- [Cron 任务命令](/zh-CN/cli/cron)——CLI 命令详解
- [配置参考](/zh-CN/config/reference)——完整配置选项
- [消息系统](/zh-CN/concepts/messages)——消息处理详解
- [工具系统](/zh-CN/tools)——可用工具列表
- [故障排除](/zh-CN/help/troubleshooting)——问题解决指南
