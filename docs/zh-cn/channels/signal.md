# Signal 配置

Signal 是一个以隐私和安全为核心的通信应用，OpenClaw 通过 `signal-cli` 工具支持 Signal 渠道。

## 概述

- **集成方式**: 通过 signal-cli 外部 CLI（JSON-RPC + SSE）
- **协议类型**: HTTP JSON-RPC + Server-Sent Events
- **路由模式**: 确定性路由，回复始终返回 Signal
- **会话模式**: 私信共享代理主会话，群组隔离

## 快速开始

### 前提条件

1. 安装 Java（signal-cli 依赖）
2. 安装 signal-cli
3. 准备一个单独的 Signal 号码用于机器人（推荐）

### 安装 signal-cli

```bash
# macOS (Homebrew)
brew install signal-cli

# Linux (手动安装)
# 下载最新版本: https://github.com/AsamK/signal-cli/releases
wget https://github.com/AsamK/signal-cli/releases/download/v0.X.X/signal-cli-0.X.X.tar.gz
tar xf signal-cli-0.X.X.tar.gz
sudo mv signal-cli-0.X.X /opt/signal-cli
sudo ln -s /opt/signal-cli/bin/signal-cli /usr/local/bin/signal-cli
```

### 链接设备

```bash
# 生成链接二维码
signal-cli link -n "OpenClaw"

# 用 Signal 应用扫描二维码完成链接
```

### 最小配置

```json5
{
  "channels": {
    "signal": {
      "enabled": true,
      "account": "+15551234567",        // 机器人的 Signal 号码
      "cliPath": "signal-cli",          // signal-cli 路径
      "dmPolicy": "pairing",            // 私信策略
      "allowFrom": ["+15557654321"]     // 允许的号码
    }
  }
}
```

### 启动网关

```bash
openclaw gateway run
```

## 号码模型

Signal 渠道的工作原理：

- 网关连接到一个 **Signal 设备**（signal-cli 账号）
- 如果使用**个人 Signal 账号**运行机器人，系统会忽略您自己的消息（循环保护）
- 若需要"给机器人发消息并收到回复"的体验，请使用**单独的机器人号码**

### 推荐设置

| 场景 | 推荐配置 |
|------|----------|
| 个人助手 | 使用单独的 Signal 号码作为机器人 |
| 家庭共享 | 使用家庭成员的备用号码 |
| 团队使用 | 使用专用的业务号码 |

## 访问控制

### 私信策略

```json5
{
  "channels": {
    "signal": {
      "dmPolicy": "pairing",           // pairing | allowlist | open | disabled
      "allowFrom": ["+15557654321"]    // 私信白名单
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `pairing` | 默认。未知发送者收到配对码，审批后才处理消息 |
| `allowlist` | 仅处理白名单中号码的消息 |
| `open` | 处理所有消息（需要 `allowFrom: ["*"]`） |
| `disabled` | 禁用所有私信 |

### 配对流程

当 `dmPolicy = "pairing"` 时：

1. 未知发送者发送消息
2. 机器人回复配对码（1小时过期）
3. 管理员审批配对

```bash
# 查看待审批的配对请求
openclaw pairing list signal

# 审批配对
openclaw pairing approve signal <CODE>
```

### 群组策略

```json5
{
  "channels": {
    "signal": {
      "groupPolicy": "allowlist",        // open | allowlist | disabled
      "groupAllowFrom": ["+15557654321"] // 群组中允许触发的号码
    }
  }
}
```

| 策略 | 说明 |
|------|------|
| `open` | 允许所有群组消息 |
| `allowlist` | 仅允许白名单中的群组/发送者 |
| `disabled` | 禁用群组消息处理 |

### UUID 标识

Signal 使用 UUID 作为用户唯一标识。仅有 UUID 的发送者（来自 `sourceUuid`）会以 `uuid:<id>` 格式存储在白名单中：

```json5
{
  "channels": {
    "signal": {
      "allowFrom": [
        "+15557654321",
        "uuid:123e4567-e89b-12d3-a456-426614174000"
      ]
    }
  }
}
```

## 外部守护进程模式

如果需要自行管理 signal-cli（避免 JVM 冷启动慢、容器初始化或共享 CPU 场景），可以独立运行守护进程：

```bash
# 手动启动 signal-cli 守护进程
signal-cli daemon --socket --no-receive-stdout
```

配置 OpenClaw 连接外部守护进程：

```json5
{
  "channels": {
    "signal": {
      "httpUrl": "http://127.0.0.1:8080",  // 守护进程 URL
      "autoStart": false                    // 禁用自动启动
    }
  }
}
```

### 启动超时

对于自动启动模式下的慢启动情况，可以调整超时：

```json5
{
  "channels": {
    "signal": {
      "startupTimeoutMs": 60000  // 60秒（最大 120000）
    }
  }
}
```

## 消息和媒体

### 文本分块

```json5
{
  "channels": {
    "signal": {
      "textChunkLimit": 4000,     // 单条消息最大字符数（默认 4000）
      "chunkMode": "length"       // length | newline
    }
  }
}
```

| 模式 | 说明 |
|------|------|
| `length` | 按长度分块（默认） |
| `newline` | 先按空行（段落边界）分块，再按长度分块 |

### 媒体限制

```json5
{
  "channels": {
    "signal": {
      "mediaMaxMb": 8,           // 媒体大小限制（MB，默认 8）
      "ignoreAttachments": false // 是否跳过附件下载
    }
  }
}
```

### 历史记录

```json5
{
  "channels": {
    "signal": {
      "historyLimit": 50,        // 群组历史消息数（默认 50，0 禁用）
      "dmHistoryLimit": 20       // 私信历史限制（用户轮次）
    }
  }
}
```

按用户配置私信历史：

```json5
{
  "channels": {
    "signal": {
      "dms": {
        "+15557654321": {
          "historyLimit": 30
        }
      }
    }
  }
}
```

## 已读回执和输入指示

### 输入指示

OpenClaw 在回复生成期间发送输入指示，并持续刷新直到回复完成。

### 已读回执

```json5
{
  "channels": {
    "signal": {
      "sendReadReceipts": true  // 发送已读回执
    }
  }
}
```

注意：signal-cli 不支持群组已读回执。

## 表情回复

使用 message 工具发送表情回复：

```
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=fire
```

### 回复示例

```bash
# 私信回复
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=fire

# 移除回复
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=fire remove=true

# 群组回复（需要指定消息作者）
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=check
```

### 回复配置

```json5
{
  "channels": {
    "signal": {
      "actions": {
        "reactions": true        // 启用/禁用回复功能
      },
      "reactionLevel": "minimal" // off | ack | minimal | extensive
    }
  }
}
```

| 级别 | 说明 |
|------|------|
| `off` | 禁用代理回复 |
| `ack` | 禁用代理回复 |
| `minimal` | 启用代理回复，最小化指导 |
| `extensive` | 启用代理回复，详细指导 |

## 投递目标

从 CLI 或 cron 发送消息时的目标格式：

| 类型 | 格式 | 示例 |
|------|------|------|
| 私信 | `signal:+<号码>` | `signal:+15551234567` |
| 私信（E.164） | `+<号码>` | `+15551234567` |
| UUID 私信 | `uuid:<id>` | `uuid:123e4567-e89b-12d3-a456-426614174000` |
| 群组 | `signal:group:<groupId>` | `signal:group:abc123def456` |
| 用户名 | `username:<name>` | `username:johndoe` |

```bash
# 发送消息示例
openclaw message send --to "signal:+15551234567" --message "你好"
openclaw message send --to "signal:group:abc123" --message "群组消息"
```

## 多账号配置

```json5
{
  "channels": {
    "signal": {
      "accounts": {
        "main": {
          "name": "主机器人",
          "account": "+15551234567",
          "cliPath": "signal-cli",
          "allowFrom": ["+15557654321"]
        },
        "alerts": {
          "name": "告警机器人",
          "account": "+15559876543",
          "cliPath": "signal-cli",
          "allowFrom": ["*"]
        }
      }
    }
  }
}
```

每个账号可以有独立的配置：

```json5
{
  "channels": {
    "signal": {
      "accounts": {
        "main": {
          "historyLimit": 100,
          "reactionLevel": "extensive",
          "actions": {
            "reactions": true
          }
        }
      }
    }
  }
}
```

## 配置写入

默认情况下，Signal 渠道允许通过 `/config set|unset` 命令触发配置更新（需要 `commands.config: true`）。

禁用配置写入：

```json5
{
  "channels": {
    "signal": {
      "configWrites": false
    }
  }
}
```

## 完整配置参考

### 提供者选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enabled` | boolean | false | 启用/禁用渠道 |
| `account` | string | - | 机器人的 E.164 号码 |
| `cliPath` | string | "signal-cli" | signal-cli 路径 |
| `httpUrl` | string | - | 外部守护进程 URL |
| `httpHost` | string | "127.0.0.1" | 守护进程绑定地址 |
| `httpPort` | number | 8080 | 守护进程端口 |
| `autoStart` | boolean | true | 自动启动守护进程 |
| `startupTimeoutMs` | number | - | 启动超时（毫秒，最大 120000） |
| `receiveMode` | string | "on-start" | on-start \| manual |
| `ignoreAttachments` | boolean | false | 跳过附件下载 |
| `ignoreStories` | boolean | false | 忽略动态消息 |
| `sendReadReceipts` | boolean | false | 发送已读回执 |

### 访问控制选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `dmPolicy` | string | "pairing" | pairing \| allowlist \| open \| disabled |
| `allowFrom` | string[] | [] | 私信白名单（E.164 或 uuid:<id>） |
| `groupPolicy` | string | "allowlist" | open \| allowlist \| disabled |
| `groupAllowFrom` | string[] | [] | 群组发送者白名单 |

### 消息选项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `textChunkLimit` | number | 4000 | 文本分块大小 |
| `chunkMode` | string | "length" | length \| newline |
| `mediaMaxMb` | number | 8 | 媒体大小限制（MB） |
| `historyLimit` | number | 50 | 群组历史消息数 |
| `dmHistoryLimit` | number | - | 私信历史限制 |

### 相关全局配置

| 配置项 | 说明 |
|--------|------|
| `agents.list[].groupChat.mentionPatterns` | 群组提及模式（Signal 不支持原生提及） |
| `messages.groupChat.mentionPatterns` | 全局回退提及模式 |
| `messages.responsePrefix` | 响应前缀 |

## 工作原理

1. `signal-cli` 作为守护进程运行
2. 网关通过 SSE 读取事件
3. 入站消息规范化为统一信封格式
4. 回复始终路由回相同的号码或群组

```
Signal App → signal-cli daemon → SSE → Gateway → Agent
                                          ↓
Signal App ← signal-cli daemon ← JSON-RPC ←─┘
```

## 故障排除

### signal-cli 无法启动

1. 确认 Java 已安装：`java -version`
2. 检查 signal-cli 路径
3. 查看日志：`tail -f ~/.local/share/signal-cli/logs/*.log`

### 消息收不到

1. 确认设备已链接
2. 检查 allowFrom 配置
3. 验证配对状态：`openclaw pairing list signal`

### 发送失败

1. 检查网络连接
2. 确认号码格式正确（E.164）
3. 验证守护进程状态

### 附件下载失败

1. 检查 `mediaMaxMb` 限制
2. 确认磁盘空间充足
3. 尝试设置 `ignoreAttachments: true` 排除问题

## 相关文档

- [渠道概述](/zh-CN/channels)
- [配置参考](/zh-CN/config/reference)
- [配对机制](/zh-CN/start/quick-start#配对)
- [故障排除](/zh-CN/operations/troubleshooting)
