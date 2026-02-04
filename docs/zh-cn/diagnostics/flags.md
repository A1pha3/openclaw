---
summary: 用于定向调试日志的诊断标志
read_when:
  - 需要在不提高全局日志级别的情况下获取定向调试日志
  - 需要捕获子系统特定的日志以获取支持信息
title: "诊断标志"
---

# 诊断标志

诊断标志允许您启用定向调试日志，而无需在所有地方都开启详细日志。标志是选择加入的，除非有子系统检查它们，否则不会生效。

## 工作原理

- 标志是字符串（不区分大小写）。
- 您可以通过配置或环境变量覆盖来启用标志。
- 支持通配符：
  - `telegram.*` 匹配 `telegram.http`
  - `*` 启用所有标志

## 通过配置启用

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

多个标志：

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

更改标志后需要重新启动网关。

## 环境变量覆盖（一次性使用）

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

禁用所有标志：

```bash
OPENCLAW_DIAGNOSTICS=0
```

## 日志输出位置

标志会将日志发射到标准诊断日志文件中。默认位置：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

如果设置了 `logging.file`，请使用该路径。日志格式为 JSONL（每行一个 JSON 对象）。根据 `logging.redactSensitive` 仍会应用脱敏处理。

## 提取日志

获取最新的日志文件：

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

过滤 Telegram HTTP 诊断日志：

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

或在复现问题时实时查看：

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

对于远程网关，您也可以使用 `openclaw logs --follow`（请参阅 [/cli/logs](/cli/logs)）。

## 注意事项

- 如果 `logging.level` 设置高于 `warn`，这些日志可能会被抑制。默认的 `info` 级别没有问题。
- 标志可以安全地保留启用状态；它们只会影响特定子系统的日志量。
- 使用 [/logging](/logging) 更改日志目的地、级别和脱敏处理。
