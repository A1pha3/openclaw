# 会话管理

本文档介绍 Moltbot 的会话系统，包括会话生命周期、上下文管理和配置选项。

## 会话概述

会话是用户与 AI 代理之间的对话容器，负责：

- 维护对话历史
- 管理上下文窗口
- 隔离不同对话
- 持久化状态

## 会话类型

### DM 会话（私信）

一对一的私人对话：

```
agent:main:whatsapp:dm:+15555550123
agent:main:telegram:dm:123456789
```

### 群组会话

群组或频道的对话：

```
agent:main:whatsapp:group:120363403215116621@g.us
agent:main:telegram:group:-1001234567890
agent:main:discord:channel:1234567890123456789
```

### WebChat 会话

浏览器控制台对话：

```
agent:main:webchat:session:abc123
```

## 会话键格式

```
agent:<agentId>:<channel>:<type>:<id>
```

| 组件 | 说明 | 示例 |
|------|------|------|
| `agent` | 固定前缀 | `agent` |
| `agentId` | 代理 ID | `main`, `work` |
| `channel` | 渠道名称 | `whatsapp`, `telegram` |
| `type` | 对话类型 | `dm`, `group`, `channel` |
| `id` | 对话标识 | 电话号码、群组 ID |

## 主会话合并

默认情况下，同一用户的多个 DM 会话合并到 `main` 会话：

```
用户 A (WhatsApp) ──┐
用户 A (Telegram) ──┼──→ agent:main:main
用户 A (Discord)  ──┘
```

### 配置主会话

```json5
{
  session: {
    mainKey: "main"  // 合并键
  }
}
```

### 禁用合并

要为每个渠道保持独立会话：

```json5
{
  session: {
    mainKey: null
  }
}
```

## 会话生命周期

### 创建

1. 收到新对话的第一条消息
2. 根据路由规则选择代理
3. 创建会话实例
4. 加载工作区上下文

### 活动

1. 接收消息
2. 加载历史上下文
3. 调用 AI 模型
4. 执行工具调用
5. 发送响应
6. 更新历史

### 持久化

会话自动保存到：

```
~/.clawdbot/agents/<agentId>/sessions/
```

### 过期

会话可以配置过期策略（未来功能）。

## 历史管理

### 群组历史限制

```json5
{
  messages: {
    groupChat: {
      historyLimit: 50  // 保留最近 50 条群组消息
    }
  }
}
```

### 按渠道配置

```json5
{
  channels: {
    telegram: {
      historyLimit: 100,      // 群组
      dmHistoryLimit: 30      // DM
    }
  }
}
```

### 禁用历史

```json5
{
  channels: {
    telegram: {
      historyLimit: 0  // 不包含历史上下文
    }
  }
}
```

## 上下文管理

### 上下文组成

1. **系统提示**: 代理身份和指令
2. **工作区文件**: AGENTS.md, SOUL.md 等
3. **会话历史**: 之前的对话
4. **当前消息**: 用户输入

### 上下文截断

当上下文过长时，自动截断：

```json5
{
  agents: {
    defaults: {
      bootstrapMaxChars: 20000  // 工作区文件最大字符
    }
  }
}
```

## 会话命令

### 查看会话

```bash
# 列出所有会话
moltbot sessions list

# 查看特定会话
moltbot sessions show <sessionKey>
```

### 查看历史

```bash
moltbot sessions history <sessionKey>
```

### 清除会话

```bash
# 清除特定会话
moltbot sessions clear <sessionKey>

# 清除所有会话
moltbot sessions clear --all
```

### 导出会话

```bash
moltbot sessions export <sessionKey> > session.json
```

## 聊天命令

### 新建会话

在聊天中发送：

```
/new
```

### 查看队列

```
/queue
```

### 切换代理

```
/agent <agentId>
```

## 消息队列

当代理正在处理时，新消息会排队：

```json5
{
  messages: {
    queue: {
      mode: "collect",      // steer | followup | collect
      debounceMs: 1000,     // 防抖
      cap: 20,              // 队列上限
      drop: "summarize"     // old | new | summarize
    }
  }
}
```

### 队列模式

| 模式 | 说明 |
|------|------|
| `steer` | 新消息中断当前处理 |
| `followup` | 新消息作为后续处理 |
| `collect` | 收集消息批量处理 |

## 入站消息防抖

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,  // 合并 2 秒内的消息
      byChannel: {
        whatsapp: 5000,   // WhatsApp 使用 5 秒
        slack: 1500
      }
    }
  }
}
```

## 线程/话题支持

### Telegram 话题

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "99": {
              requireMention: false,
              systemPrompt: "这是技术讨论话题。"
            }
          }
        }
      }
    }
  }
}
```

### Slack 线程

```json5
{
  channels: {
    slack: {
      thread: {
        historyScope: "thread",  // thread | channel
        inheritParent: false
      }
    }
  }
}
```

## 多代理会话

### 代理切换

用户可以在不同代理间切换：

```
/agent work
```

### 会话隔离

每个代理有独立的会话：

```
agent:personal:whatsapp:dm:+15555550123
agent:work:whatsapp:dm:+15555550123
```

### 子代理会话

```json5
{
  agents: {
    list: [
      {
        id: "main",
        subagents: {
          allowAgents: ["helper", "researcher"]
        }
      }
    ]
  }
}
```

## 会话持久化

### 存储位置

```
~/.clawdbot/agents/<agentId>/
├── agent/
│   └── sessions/
│       └── <sessionKey>.jsonl
```

### JSONL 格式

每行是一个会话事件：

```json
{"type":"message","role":"user","content":"你好"}
{"type":"message","role":"assistant","content":"你好！"}
{"type":"tool_use","tool":"read","params":{"path":"README.md"}}
```

## 监控与调试

### 查看活动会话

```bash
moltbot sessions list --active
```

### 会话统计

```bash
moltbot sessions stats
```

### 调试模式

```bash
moltbot gateway --verbose
```

## 最佳实践

1. **合理设置历史限制**: 平衡上下文质量和 token 使用
2. **使用消息防抖**: 避免快速连续消息产生多次响应
3. **定期清理旧会话**: 释放存储空间
4. **监控会话数量**: 避免资源耗尽

## 下一步

- [消息路由](/zh-cn/concepts/routing) - 路由配置
- [AI 代理](/zh-cn/concepts/agents) - 代理系统
- [配置参考](/zh-cn/config/reference) - 完整配置
