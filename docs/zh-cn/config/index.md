# 配置概述

OpenClaw 使用 JSON5 格式的配置文件，支持注释和尾随逗号，便于阅读和维护。

## 配置文件位置

默认配置文件路径：

```
~/.clawdbot/openclaw.json
```

可通过环境变量覆盖：

```bash
export CLAWDBOT_CONFIG_PATH=~/custom/openclaw.json
```

## 配置格式

OpenClaw 使用 JSON5 格式，支持：

- 单行和多行注释
- 对象键不需要引号（无特殊字符时）
- 尾随逗号
- 单引号字符串

```json5
{
  // 这是注释
  agents: {
    defaults: {
      workspace: "~/clawd",  // 尾随逗号
    }
  },
  /* 多行
     注释 */
  channels: {
    whatsapp: {
      allowFrom: ['+15555550123'],
    }
  }
}
```

## 最小配置

如果没有配置文件，OpenClaw 使用安全的默认值。最小推荐配置：

```json5
{
  agents: { defaults: { workspace: "~/clawd" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

## 配置结构

### 顶层配置块

| 配置块 | 说明 |
|--------|------|
| `agents` | AI 代理配置 |
| `channels` | 消息渠道配置 |
| `bindings` | 消息路由绑定 |
| `gateway` | 网关服务配置 |
| `messages` | 消息处理配置 |
| `logging` | 日志配置 |
| `tools` | 工具配置 |
| `models` | 模型提供商配置 |
| `auth` | 认证配置 |
| `env` | 环境变量配置 |
| `commands` | 聊天命令配置 |
| `web` | WhatsApp Web 配置 |

### 完整配置示例

```json5
{
  // 代理配置
  agents: {
    defaults: {
      workspace: "~/clawd",
      model: "anthropic/claude-sonnet-4-20250514",
      sandbox: {
        mode: "non-main",
        scope: "session"
      }
    },
    list: [
      {
        id: "main",
        default: true,
        identity: {
          name: "Clawd",
          emoji: "🦞"
        }
      }
    ]
  },

  // 渠道配置
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    },
    telegram: {
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}"
    }
  },

  // 消息配置
  messages: {
    responsePrefix: "🦞",
    ackReaction: "👀"
  },

  // 日志配置
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log"
  }
}
```

## 配置验证

OpenClaw 使用严格的配置验证：

- 未知键会导致网关拒绝启动
- 类型不匹配会报错
- 无效值会被拒绝

### 验证失败时

```bash
# 诊断问题
openclaw doctor

# 自动修复
openclaw doctor --fix
```

## 配置管理

### 查看配置

```bash
# 查看当前配置
openclaw config get

# 查看特定配置项
openclaw config get channels.telegram.enabled
```

### 修改配置

```bash
# 设置配置项
openclaw config set channels.telegram.enabled true

# 删除配置项
openclaw config unset channels.discord
```

### 配置编辑

```bash
# 使用默认编辑器
openclaw config edit
```

## 环境变量

### 在配置中使用环境变量

支持 `${VAR_NAME}` 语法：

```json5
{
  channels: {
    telegram: {
      botToken: "${TELEGRAM_BOT_TOKEN}"
    }
  },
  gateway: {
    auth: {
      token: "${CLAWDBOT_GATEWAY_TOKEN}"
    }
  }
}
```

### 环境变量优先级

1. 进程环境变量（最高）
2. `.env` 文件（当前目录）
3. `~/.clawdbot/.env`（全局）
4. 配置文件中的 `env.vars`

### 配置内联环境变量

```json5
{
  env: {
    vars: {
      CUSTOM_API_KEY: "sk-..."
    }
  }
}
```

## 配置包含（$include）

将配置拆分到多个文件：

```json5
// ~/.clawdbot/openclaw.json
{
  gateway: { port: 18789 },
  
  // 包含单个文件
  agents: { "$include": "./agents.json5" },
  
  // 包含多个文件（深度合并）
  channels: { 
    "$include": [
      "./channels/whatsapp.json5",
      "./channels/telegram.json5"
    ]
  }
}
```

### 包含规则

- 相对路径：相对于包含文件的位置
- 绝对路径：直接使用
- 嵌套包含：最多 10 层深度
- 合并顺序：后面的文件覆盖前面的

## 配置热重载

修改配置后，网关会自动检测并重启。也可以手动触发：

```bash
# 重启网关应用新配置
openclaw gateway restart
```

### 通过 RPC 更新

```bash
# 获取当前配置（包含 hash）
openclaw gateway call config.get --params '{}'

# 应用新配置
openclaw gateway call config.apply --params '{
  "raw": "{...新配置...}",
  "baseHash": "<hash>",
  "restartDelayMs": 1000
}'
```

## 常见配置场景

### 个人使用

```json5
{
  agents: { defaults: { workspace: "~/clawd" } },
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"]
    }
  }
}
```

### 团队使用

```json5
{
  agents: {
    defaults: {
      workspace: "~/team-clawd",
      sandbox: { mode: "all", scope: "session" }
    }
  },
  channels: {
    slack: {
      botToken: "${SLACK_BOT_TOKEN}",
      appToken: "${SLACK_APP_TOKEN}",
      channels: {
        "#general": { allow: true, requireMention: true }
      }
    }
  }
}
```

### 多代理

```json5
{
  agents: {
    list: [
      { id: "personal", default: true, workspace: "~/clawd-personal" },
      { id: "work", workspace: "~/clawd-work" }
    ]
  },
  bindings: [
    { agentId: "personal", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "work" } }
  ]
}
```

## 下一步

- [配置参考](/zh-cn/config/reference) - 完整配置项说明
- [配置示例](/zh-cn/config/examples) - 更多配置示例
- [安全配置](/zh-cn/config/security) - 安全相关设置
