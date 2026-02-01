---
summary: "自动化 - Cron 任务、Webhook、定时执行"
read_when:
  - 设置定时任务
  - 配置 Webhook
  - 自动化工作流
title: "自动化"
---

# ⏰ 自动化

OpenClaw 支持**自动化工作流**，包括定时任务和 Webhook。

---

## 🎯 自动化类型

| 类型 | 用途 | 示例 |
|------|------|------|
| **Cron 任务** | 定时执行 | 每日备份 |
| **Webhook** | HTTP 触发 | 接收外部事件 |
| **轮询** | 定期检查 | 监控服务状态 |

---

## 🚀 Cron 任务

### 添加任务

```bash
openclaw cron add \
  --name "backup" \
  --schedule "0 2 * * *" \
  --command "backup.sh"
```

### 查看任务

```bash
openclaw cron list
```

### 删除任务

```bash
openclaw cron remove backup
```

---

## 🔧 Webhook

配置 Webhook 接收外部事件：

```json5
{
  webhooks: {
    github: {
      secret: "your-secret",
      events: ["push", "pull_request"]
    }
  }
}
```

---

## 📖 相关文档

- [CLI Cron 命令](/zh-CN/cli/index#定时任务)
- [配置参考](/zh-CN/config/reference)
