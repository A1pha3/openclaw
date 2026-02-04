---
summary: "OpenClaw 如何构建提示词上下文并报告 token 使用量与成本"
read_when:
  - 解释 token 使用量、成本或上下文窗口
  - 调试上下文增长或压缩行为
title: "Token 使用与成本"
---

# Token 使用与成本

OpenClaw 跟踪 **token**，而不是字符。Token 是特定于模型的，但大多数 OpenAI 风格的模型对于英文文本平均约 4 个字符 per token。

## 系统提示词如何构建

OpenClaw 在每次运行时组装自己的系统提示词，包括：

- 工具列表 + 简短描述
- 技能列表（仅元数据；指令按需通过 `read` 加载）
- 自我更新指令
- 工作区 + 引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`，新消息时包括 `BOOTSTRAP.md`）。大文件按 `agents.defaults.bootstrapMaxChars`（默认：20000）截断
- 时间（UTC + 用户时区）
- 回复标签 + 心跳行为
- 运行时元数据（主机/操作系统/模型/思考）

完整分解请参阅 [系统提示词](/concepts/system-prompt)。

## 什么计入上下文窗口

模型接收到的所有内容都计入上下文限制：

- 系统提示词（上面列出的所有部分）
- 对话历史（用户 + 助手消息）
- 工具调用和工具结果
- 附件/转录（图像、音频、文件）
- 压缩摘要和剪枝产物
- 提供商包装器或安全头（不可见，但仍计入）

关于实际分解（每个注入的文件、工具、技能和系统提示词大小），使用 `/context list` 或 `/context detail`。请参阅 [上下文](/concepts/context)。

## 如何查看当前 token 使用量

在聊天中使用：

- `/status` → **表情符号丰富的状态卡**，显示会话模型、上下文使用量、最后响应输入/输出 token，以及**估计成本**（仅 API 密钥）
- `/usage off|tokens|full` → 将**每响应使用量页脚**附加到每个回复
  - 按会话持久化（存储为 `responseUsage`）
  - OAuth 认证**隐藏成本**（仅 token）
- `/usage cost` → 显示来自 OpenClaw 会话日志的本地成本摘要

其他界面：

- **TUI/Web TUI：** 支持 `/status` + `/usage`
- **CLI：** `openclaw status --usage` 和 `openclaw channels list` 显示提供商配额窗口（不是每响应成本）

## 成本估算（当显示时）

成本根据你的模型定价配置估算：

```
models.providers.<provider>.models[].cost
```

这些是 `input`、`output`、`cacheRead` 和 `cacheWrite` 的 **USD per 1M token**。如果缺少定价，OpenClaw 只显示 token。OAuth token 从不显示美元成本。

## 缓存 TTL 和剪枝影响

提供商提示缓存仅在缓存 TTL 窗口内适用。OpenClaw 可以选择运行**缓存 TTL 剪枝**：它在缓存 TTL 过期后压缩会话，然后重置缓存窗口，以便后续请求可以重用新鲜缓存的上下文，而不是重新缓存完整历史。这可以在会话在 TTL 之后空闲时保持缓存写入成本更低。

在 [Gateway 配置](/gateway/configuration) 中配置它，并在 [会话剪枝](/concepts/session-pruning) 中查看行为详情。

心跳可以保持缓存**温暖**跨越空闲间隔。如果你的模型缓存 TTL 是 `1h`，将心跳间隔设置在略低于该值（例如 `55m`）可以避免重新缓存完整提示，减少缓存写入成本。

对于 Anthropic API 定价，缓存读取比输入 token 便宜得多，而缓存写入按更高的倍数计费。请参阅 Anthropic 的提示缓存定价以了解最新费率和 TTL 倍数：

https://docs.anthropic.com/docs/build-with-claude/prompt-caching

### 示例：使用心跳保持 1h 缓存温暖

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

## 减少 token 压力的提示

- 使用 `/compact` 压缩长会话
- 在你的工作流程中修剪大型工具输出
- 保持技能描述简短（技能列表被注入到提示词中）
- 对于冗长、探索性的工作，偏好较小的模型

请参阅 [技能](/tools/skills) 了解确切的技能列表开销公式。

## 相关文档

- [上下文管理](/concepts/context) - 上下文构建详解
- [会话管理](/concepts/session) - 会话状态
- [系统提示词](/concepts/system-prompt) - 提示词组成
- [成本管理](/operations/costs) - 成本优化策略
