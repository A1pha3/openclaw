---
summary: "跨信封、提示、工具和连接器的时间和日期处理"
read_when:
  - "你正在更改时间戳向用户或模型显示的方式"
  - "你正在调试消息或系统提示输出中的时间格式"
title: "日期和时间"
---

# 日期和时间

OpenClaw 默认**传输时间戳使用主机本地时间**以及**系统提示中仅使用用户时区**。提供商时间戳被保留，以便工具保持其本地语义（当前时间可通过 `session_status` 获得）。

## 消息信封（默认本地）

入站消息带有时间戳包装（分钟精度）：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

这个信封时间戳**默认是主机本地的**，无论提供商时区如何。

你可以覆盖此行为：

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA 时区
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

- `envelopeTimezone: "utc"` 使用 UTC。
- `envelopeTimezone: "local"` 使用主机时区。
- `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（回退到主机时区）。
- 对固定时区使用明确的 IANA 时区（例如 `"America/Chicago"`）。
- `envelopeTimestamp: "off"` 从信封头中移除绝对时间戳。
- `envelopeElapsed: "off"` 移除经过时间后缀（`+2m` 样式）。

### 示例

**本地（默认）：**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**用户时区：**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**启用的经过时间：**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

## 系统提示：当前日期和时间

如果用户时区已知，系统提示包含一个专门的**当前日期和时间**部分，只有**时区**（没有时钟/时间格式）以保持提示缓存稳定：

```
Time zone: America/Chicago
```

当代理需要当前时间时，使用 `session_status` 工具；状态卡包含时间戳行。

## 系统事件行（默认本地）

插入到代理上下文中的排队系统事件使用与消息信封相同的时区选择（默认：主机本地）加上前缀：

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

### 配置用户时区 + 格式

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

- `userTimezone` 设置提示上下文的**用户本地时区**。
- `timeFormat` 控制提示中的**12 小时/24 小时显示**。`auto` 遵循操作系统偏好。

## 时间格式检测（自动）

当 `timeFormat: "auto"` 时，OpenClaw 检查操作系统偏好（macOS/Windows）并回退到区域设置格式。检测到的值**按进程缓存**以避免重复的系统调用。

## 工具负载 + 连接器（原始提供商时间 + 标准化字段）

频道工具返回**提供商原生时间戳**并添加标准化字段以保持一致性：

- `timestampMs`：自纪元以来的毫秒（UTC）
- `timestampUtc`：ISO 8601 UTC 字符串

原始提供商字段被保留，因此不会丢失任何内容。

- Slack：来自 API 的类似纪元的字符串
- Discord：UTC ISO 时间戳
- Telegram/WhatsApp：提供商特定的数字/ISO 时间戳

如果你需要本地时间，使用已知的时区在下游转换。

## 相关文档

- [系统提示](/concepts/system-prompt)
- [时区](/concepts/timezone)
- [消息](/concepts/messages)
