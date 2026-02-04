---
summary: "`openclaw system` 命令参考（系统事件、心跳、在线状态）"
read_when:
  - 想要在不创建定时任务的情况下排队系统事件
  - 需要启用或禁用心跳
  - 想要检查系统在线状态条目
title: "system"
---

# `openclaw system`

网关的系统级辅助工具：排队系统事件、控制心跳、查看在线状态。

## 为什么需要这个命令

系统命令提供了底层控制能力：

- **即时触发**：无需等待定时任务
- **心跳控制**：管理代理的定期唤醒
- **状态监控**：查看系统和节点在线状态

## 常用命令

```bash
# 排队系统事件
openclaw system event --text "检查紧急跟进" --mode now

# 启用心跳
openclaw system heartbeat enable

# 查看上次心跳
openclaw system heartbeat last

# 查看在线状态
openclaw system presence
```

## system event

在 **main** 会话上排队一个系统事件。下一次心跳会将其作为 `System:` 行注入提示。

```bash
openclaw system event --text "检查待办事项" --mode now
```

### 选项

| 选项 | 说明 |
|------|------|
| `--text <text>` | 必需，系统事件文本 |
| `--mode <mode>` | `now` 或 `next-heartbeat`（默认） |
| `--json` | 机器可读输出 |

### 模式说明

| 模式 | 行为 |
|------|------|
| `now` | 立即触发心跳 |
| `next-heartbeat` | 等待下一次计划心跳 |

### 使用场景

```bash
# 立即提醒代理检查邮件
openclaw system event --text "检查新邮件" --mode now

# 下次心跳时提醒
openclaw system event --text "总结今日工作" --mode next-heartbeat
```

## system heartbeat

心跳控制命令。

### 查看上次心跳

```bash
openclaw system heartbeat last
```

### 启用心跳

```bash
openclaw system heartbeat enable
```

如果心跳被禁用，使用此命令重新启用。

### 禁用心跳

```bash
openclaw system heartbeat disable
```

暂停心跳（代理不会定期唤醒）。

### 选项

| 选项 | 说明 |
|------|------|
| `--json` | 机器可读输出 |

## system presence

列出网关知道的当前系统在线状态条目（节点、实例和类似状态行）。

```bash
openclaw system presence
```

### 输出示例

```
System Presence
───────────────
Type        Name          Status      Last Seen
────        ────          ──────      ─────────
node        MacBook Pro   online      2 seconds ago
node        iPhone 15     online      5 seconds ago
instance    gateway       running     now
```

### 选项

| 选项 | 说明 |
|------|------|
| `--json` | 机器可读输出 |

## 说明

- 需要运行中的网关（本地或远程）
- 系统事件是临时的，重启后不保留
- 心跳间隔在配置中设置

## 配置心跳间隔

```json5
{
  agent: {
    heartbeat: {
      enabled: true,
      intervalMinutes: 15,
    },
  },
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 事件未触发 | 网关未运行 | 启动网关 |
| 心跳未执行 | 已禁用 | `openclaw system heartbeat enable` |
| 在线状态为空 | 无连接设备 | 检查节点连接 |
