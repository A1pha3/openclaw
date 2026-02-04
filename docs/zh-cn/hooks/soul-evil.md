---
summary: "SOUL Evil hook（用 SOUL_EVIL.md 替换 SOUL.md）"
read_when:
  - 你想要启用或调整 SOUL Evil hook
  - 你想要清除窗口或随机机会人格切换
title: "SOUL Evil Hook"
---

# SOUL Evil Hook

SOUL Evil hook 在**清除窗口**或**随机机会**期间将**注入的** `SOUL.md` 内容替换为 `SOUL_EVIL.md`。
它**不**修改磁盘上的文件。

## 工作原理

当 `agent:bootstrap` 运行时，hook 可以在系统提示组装之前在内存中替换 `SOUL.md` 内容。
如果 `SOUL_EVIL.md` 缺失或为空，OpenClaw 会记录警告并保留正常的 `SOUL.md`。

子代理运行**不**在它们的引导文件中包含 `SOUL.md`，因此此 hook 对子代理没有影响。

## 启用

```bash
openclaw hooks enable soul-evil
```

然后设置配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

在代理工作区根目录（`SOUL.md` 旁边）创建 `SOUL_EVIL.md`。

## 选项

- `file`（字符串）：备用的 SOUL 文件名（默认：`SOUL_EVIL.md`）
- `chance`（数字 0–1）：每次运行使用 `SOUL_EVIL.md` 的随机机会
- `purge.at`（HH:mm）：每日清除开始时间（24 小时制）
- `purge.duration`（持续时间）：窗口长度（例如 `30s`、`10m`、`1h`）

**优先级：** 清除窗口优先于机会。

**时区：** 当设置时使用 `agents.defaults.userTimezone`；否则使用主机时区。

## 注意

- 磁盘上没有写入或修改任何文件。
- 如果 `SOUL.md` 不在引导列表中，hook 什么都不做。

## 另见

- [Hooks](/hooks)
