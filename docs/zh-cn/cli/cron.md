---
summary: "CLI 参考 - `openclaw cron` (调度和运行后台任务)"
read_when:
  - 需要定时任务和唤醒
  - 调试定时任务执行和日志
title: "cron"
---

# `openclaw cron`

管理网关调度器的定时任务。

## 为什么需要这个命令

定时任务让你的代理自动工作：

- **定时报告**：每日总结、周报
- **主动唤醒**：定时检查邮件、提醒
- **自动化流程**：数据备份、清理任务

## 相关文档

- 定时任务详解：[定时任务](/zh-cn/automation/cron-jobs)

**提示**：运行 `openclaw cron --help` 查看完整命令列表。

## 基础用法

```bash
# 列出所有定时任务
openclaw cron list

# 添加任务
openclaw cron add --name "daily-summary" --schedule "0 9 * * *" --message "总结今日待办"

# 立即运行任务
openclaw cron run <job-id>

# 删除任务
openclaw cron remove <job-id>
```

## 常见编辑操作

### 更新投递设置

更新投递设置而不改变消息内容：

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

### 禁用投递

禁用特定任务的投递：

```bash
openclaw cron edit <job-id> --no-deliver
```

## Cron 表达式

使用标准 cron 表达式指定时间：

```
┌───────────── 分钟 (0-59)
│ ┌───────────── 小时 (0-23)
│ │ ┌───────────── 日期 (1-31)
│ │ │ ┌───────────── 月份 (1-12)
│ │ │ │ ┌───────────── 星期 (0-7，0 和 7 都是周日)
│ │ │ │ │
* * * * *
```

常用示例：

| 表达式 | 含义 |
|--------|------|
| `0 9 * * *` | 每天早上 9 点 |
| `0 9 * * 1-5` | 工作日早上 9 点 |
| `0 */4 * * *` | 每 4 小时 |
| `0 9 1 * *` | 每月 1 号早上 9 点 |
| `0 9 * * 0` | 每周日早上 9 点 |

## 使用场景

### 每日总结

```bash
openclaw cron add \
  --name "daily-summary" \
  --schedule "0 18 * * *" \
  --message "总结今天完成的工作" \
  --deliver \
  --channel telegram \
  --to "123456789"
```

### 早报推送

```bash
openclaw cron add \
  --name "morning-briefing" \
  --schedule "0 8 * * 1-5" \
  --message "今日日程和待办事项" \
  --deliver \
  --channel whatsapp \
  --to "+15555550123"
```

### 周报生成

```bash
openclaw cron add \
  --name "weekly-report" \
  --schedule "0 17 * * 5" \
  --message "生成本周工作周报"
```

## 任务管理

```bash
# 查看任务列表
openclaw cron list

# 查看任务详情
openclaw cron info <job-id>

# 测试运行（立即执行）
openclaw cron run <job-id>

# 暂停任务
openclaw cron disable <job-id>

# 恢复任务
openclaw cron enable <job-id>
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 任务没有执行 | 检查网关是否运行中 |
| 时间不对 | 确认时区设置正确 |
| 消息没有投递 | 检查 `--deliver` 和渠道配置 |
| 任务重复执行 | 检查是否有重复的任务定义 |

查看任务执行日志：

```bash
openclaw logs --follow
```
