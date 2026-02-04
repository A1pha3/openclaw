---
summary: "后台执行和进程管理"
read_when:
  - "添加或修改后台执行行为"
  - "调试长时间运行的 exec 任务"
title: "后台执行和进程工具"
---

# 后台执行 + 进程工具

OpenClaw 通过 `exec` 工具运行 shell 命令，并将长时间运行的任务保留在内存中。`process` 工具管理这些后台会话。

## exec 工具

关键参数：

- `command`（必填）
- `yieldMs`（默认 10000）：此延迟后自动进入后台
- `background`（布尔值）：立即进入后台
- `timeout`（秒，默认 1800）：此超时后终止进程
- `elevated`（布尔值）：如果启用/允许提升模式则在主机上运行
- 需要真正的 TTY？设置 `pty: true`。
- `workdir`、`env`

行为：

- 前台运行直接返回输出。
- 进入后台后（显式或超时），工具返回 `status: "running"` + `sessionId` 和一个短尾。
- 输出保留在内存中，直到会话被轮询或清除。
- 如果 `process` 工具被禁用，`exec` 同步运行并忽略 `yieldMs`/`background`。

## 子进程桥接

当在 exec/process 工具外部生成长时间运行的子进程时（例如，CLI 重新生成或网关辅助工具），附加子进程桥接辅助工具，以便终止信号被转发，并且在退出/错误时分离监听器。这可以避免 systemd 上的孤立进程，并保持跨平台的关闭行为一致。

环境覆盖：

- `PI_BASH_YIELD_MS`：默认屈服（毫秒）
- `PI_BASH_MAX_OUTPUT_CHARS`：内存中输出上限（字符）
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`：每个流的待处理 stdout/stderr 上限（字符）
- `PI_BASH_JOB_TTL_MS`：完成会话的 TTL（毫秒，限制为 1 分钟-3 小时）

配置（首选）：

- `tools.exec.backgroundMs`（默认 10000）
- `tools.exec.timeoutSec`（默认 1800）
- `tools.exec.cleanupMs`（默认 1800000）
- `tools.exec.notifyOnExit`（默认 true）：当后台 exec 退出时，将系统事件加入队列并请求心跳。

## process 工具

操作：

- `list`：运行中 + 已完成的会话
- `poll`：排空会话的新输出（还报告退出状态）
- `log`：读取聚合输出（支持 `offset` + `limit`）
- `write`：发送 stdin（`data`，可选 `eof`）
- `kill`：终止后台会话
- `clear`：从内存中移除已完成的会话
- `remove`：如果正在运行则终止，否则清除已完成的

注意：

- 只有后台会话才在内存中列出/保留。
- 进程重启时会话会丢失（没有磁盘持久化）。
- 只有运行 `process poll/log` 并且工具结果被记录时，会话日志才会保存到聊天历史中。
- `process` 作用于每个代理；它只看到由该代理启动的会话。
- `process list` 包含一个派生的 `name`（命令动词 + 目标）以便快速扫描。
- `process log` 使用基于行的 `offset`/`limit`（省略 `offset` 以获取最后 N 行）。

## 示例

运行长时间任务并稍后轮询：

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

立即在后台启动：

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

发送 stdin：

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```
