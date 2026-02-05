---
summary: OpenClaw RPC 适配器完全指南——HTTP 守护进程模式与 stdio 子进程模式的实现详解
read_when:
  - 添加或修改外部 CLI 集成
  - 调试 RPC 适配器（signal-cli、imsg）
  - 理解 JSON-RPC 与外部进程的通信机制
title: "OpenClaw RPC 适配器完全指南"
---

# OpenClaw RPC 适配器完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 OpenClaw 使用的两种 RPC 模式
- 掌握 HTTP 守护进程模式（signal-cli）的配置和使用
- 掌握 stdio 子进程模式（imsg）的配置和使用

### 进阶目标（建议掌握）

- 理解 RPC 适配器的设计模式和最佳实践
- 能够为新的外部 CLI 开发 RPC 适配器
- 掌握 RPC 客户端的弹性设计原则

### 专家目标（挑战）

- 优化 RPC 通信的性能和可靠性
- 设计复杂的外部集成架构
- 解决跨进程通信的边界情况

## 核心概念

### 什么是 RPC 适配器？

RPC（Remote Procedure Call，远程过程调用）适配器是 OpenClaw 与外部命令行工具通信的桥梁。通过 JSON-RPC 协议，OpenClaw 可以调用外部 CLI 的功能，并将结果整合到自己的系统中。

**为什么需要 RPC 适配器？**

许多消息平台提供的是命令行工具而非编程接口：

| 平台 | 提供方式 | 集成方式 |
|------|----------|----------|
| Signal | signal-cli（CLI） | JSON-RPC HTTP |
| iMessage | imsg（CLI） | JSON-RPC stdio |
| Matrix | 多种 CLI | 多种方式 |

### 两种 RPC 模式对比

```
┌─────────────────────────────────────────────────────────────────┐
│                    两种 RPC 模式对比                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   模式 A：HTTP 守护进程模式                                      │
│   ──────────────────────────────────────────────────────────   │
│                                                                 │
│   ┌─────────────┐      HTTP       ┌─────────────┐              │
│   │  OpenClaw   │ ◄────────────► │ signal-cli  │              │
│   │  (客户端)    │   JSON-RPC     │ (守护进程)   │              │
│   └─────────────┘                └─────────────┘              │
│                                                                 │
│   特点：                                                         │
│   ├── 独立的进程生命周期                                          │
│   ├── 支持 SSE 事件流                                            │
│   └── 需要端口监听                                               │
│                                                                 │
│   ──────────────────────────────────────────────────────────   │
│                                                                 │
│   模式 B：stdio 子进程模式                                       │
│   ──────────────────────────────────────────────────────────   │
│                                                                 │
│   ┌─────────────┐    stdin/stdout    ┌─────────────┐          │
│   │  OpenClaw   │ ◄────────────────► │    imsg     │          │
│   │  (父进程)    │    JSON Lines     │  (子进程)    │          │
│   └─────────────┘                    └─────────────┘          │
│                                                                 │
│   特点：                                                         │
│   ├── 进程随父进程管理                                           │
│   ├── 简单可靠                                                  │
│   └── 无需网络配置                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 模式 A：HTTP 守护进程模式

### 适用场景

HTTP 守护进程模式适用于：

- **signal-cli**：Signal 的官方 CLI 工具
- 任何支持 HTTP JSON-RPC 的外部服务
- 需要长时间运行的守护进程

### 架构设计

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP 守护进程模式架构                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   外部进程层                                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │ signal-cli daemon                                        │  │
│   │                                                         │  │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐          │  │
│   │  │ JSON-RPC  │  │   SSE     │  │  健康检查  │          │  │
│   │  │  端点     │  │  事件流   │  │   端点    │          │  │
│   │  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘          │  │
│   │        │              │              │                   │  │
│   │        └──────────────┼──────────────┘                   │  │
│   │                       │                                  │  │
│   │                       ▼                                  │  │
│   │              ┌─────────────────┐                         │  │
│   │              │   信号处理/消息  │                         │  │
│   │              │     管理        │                         │  │
│   │              └─────────────────┘                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                    HTTP (JSON-RPC/SSE)                        │
│                              │                                  │
│   OpenClaw 层               ▼                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  RPC 客户端适配器                                        │  │
│   │                                                         │  │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐          │  │
│   │  │ 请求队列  │  │ 重试逻辑  │  │  响应解析  │          │  │
│   │  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘          │  │
│   │        │              │              │                   │  │
│   │        └──────────────┼──────────────┘                   │  │
│   │                       ▼                                  │  │
│   │              ┌─────────────────┐                         │  │
│   │              │   OpenClaw     │                         │  │
│   │              │    核心        │                         │  │
│   │              └─────────────────┘                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 端点说明

**JSON-RPC 端点：**

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/v1/json-rpc` | POST | 发送 JSON-RPC 请求 |

**SSE 事件流端点：**

| 端点 | 说明 |
|------|------|
| `/api/v1/events` | 服务器发送事件流，用于实时消息通知 |

**健康检查端点：**

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/v1/check` | GET | 检查守护进程健康状态 |

### 请求格式

**JSON-RPC 请求：**

```json
{
  "jsonrpc": "2.0",
  "id": "req-123",
  "method": "send",
  "params": {
    "message": {
      "recipients": ["+1234567890"],
      "message": "Hello from OpenClaw"
    }
  }
}
```

**JSON-RPC 响应：**

```json
{
  "jsonrpc": "2.0",
  "id": "req-123",
  "result": {
    "success": true,
    "messageId": "msg-456"
  }
}
```

**错误响应：**

```json
{
  "jsonrpc": "2.0",
  "id": "req-123",
  "error": {
    "code": -32600,
    "message": "Invalid Request",
    "data": {
      "detail": "Missing required parameter: recipients"
    }
  }
}
```

### SSE 事件格式

```json
{
  "event": "message",
  "data": {
    "envelope": {
      "source": "+1234567890",
      "sourceNumber": "+1234567890",
      "sourceName": "Test User",
      "timestamp": 1707148800000,
      "message": {
        "body": "Hello!"
      }
    }
  }
}
```

### 生命周期管理

**自动启动配置：**

```json
{
  "channels": {
    "signal": {
      "autoStart": true
    }
  }
}
```

**生命周期状态机：**

```
┌─────────────────────────────────────────────────────────────────┐
│                    进程生命周期状态机                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   stopped → starting → running → stopping → stopped            │
│                                                                 │
│   状态转换：                                                     │
│   ├── stopped → starting：autoStart=true 且未运行              │
│   ├── starting：等待进程初始化完成                              │
│   ├── running：健康检查通过                                      │
│   ├── stopping：收到停止信号或配置变更                          │
│   └── stopping → stopped：进程已终止                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 模式 B：stdio 子进程模式

### 适用场景

stdio 子进程模式适用于：

- **imsg**：iMessage 的 CLI 工具
- 任何通过 stdin/stdout 通信的命令行工具
- 轻量级的外部集成

### 架构设计

```
┌─────────────────────────────────────────────────────────────────┐
│                    stdio 子进程模式架构                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   进程通信层                                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                        imsg rpc                        │  │
│   │                                                     │    │  │
│   │   stdin:  ◄─────────────────────────────────────────┼───│──│── JSON-RPC 请求
│   │   stdout: ─────────────────────────────────────────►│   │  │
│   │                                                     │    │  │
│   │                        stdin/stdout                   │    │  │
│   │                        (逐行 JSON)                   │    │
│   │                                                     │    │  │
│   │                        特点：                       │    │  │
│   │                        ├── 无需网络                  │    │  │
│   │                        ├── 进程随父终止              │    │  │
│   │                        └── 简单可靠                  │    │  │
│   └─────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                         stdio 通信                             │
│                              │                                  │
│   OpenClaw 层               ▼                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  RPC 客户端适配器                                        │  │
│   │                                                         │  │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐          │  │
│   │  │  行解析   │  │  请求队列  │  │  响应处理  │          │  │
│   │  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘          │  │
│   │        │              │              │                   │  │
│   │        └──────────────┼──────────────┘                   │  │
│   │                       ▼                                  │  │
│   │              ┌─────────────────┐                         │  │
│   │              │   OpenClaw     │                         │  │
│   │              │    核心        │                         │  │
│   │              └─────────────────┘                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 通信协议

**请求格式（通过 stdin 发送）：**

```json
{"jsonrpc":"2.0","id":"req-001","method":"send","params":{"recipients":["chat123"],"message":"Hello"}}
```

**响应格式（通过 stdout 接收）：**

```json
{"jsonrpc":"2.0","id":"req-001","result":{"success":true}}
```

**事件格式（消息通知）：**

```json
{"method":"message","params":{"envelope":{"source":"chat123","sourceNumber":"+1234567890","message":{"body":"Reply!"}}}}
```

### 进程管理

**生成命令：**

```bash
# imsg 启动命令
imsg rpc -p 0
```

**参数说明：**

| 参数 | 说明 |
|------|------|
| `-p 0` | 使用随机可用端口（避免冲突） |
| `-n` | 不自动启动（手动管理） |

## 核心方法参考

### 通用方法

**watch.subscribe**

订阅消息通知：

```json
{
  "jsonrpc": "2.0",
  "method": "watch.subscribe",
  "params": {}
}
```

**watch.unsubscribe**

取消订阅：

```json
{
  "jsonrpc": "2.0",
  "method": "watch.unsubscribe",
  "params": {}
}
```

**send**

发送消息：

```json
{
  "jsonrpc": "2.0",
  "method": "send",
  "params": {
    "recipients": ["chat_id_or_number"],
    "message": "消息内容"
  }
}
```

### imsg 特定方法

**chats.list**

获取会话列表（诊断用途）：

```json
{
  "jsonrpc": "2.0",
  "method": "chats.list",
  "params": {}
}
```

## 适配器开发指南

### 设计原则

**原则一：进程拥有权**

```
OpenClaw 拥有子进程的完整生命周期管理权

职责：
├── 启动：根据配置自动启动
├── 停止：随 OpenClaw 终止而终止
├── 重启：检测到异常时自动重启
└── 监控：持续监控进程健康状态
```

**原则二：弹性通信**

```
RPC 客户端必须能够从故障中恢复

策略：
├── 超时设置：避免无限等待
├── 重试机制：临时故障自动重试
├── 断线检测：及时发现通信中断
└── 平滑重启：避免服务中断
```

**原则三：稳定标识**

```
优先使用稳定 ID 而非显示字符串

推荐：
├── chat_id：稳定的内部标识符
└── phone_number：可能变更的标识符

示例：
├── ✅ "chat.12345.67890" → chat_id
└── ✅ "+1234567890" → phone_number
```

### 代码模板

**基础适配器结构：**

```typescript
import { EventEmitter } from 'events';

export class RpcAdapter extends EventEmitter {
  private process: ChildProcess | null = null;
  private pendingRequests: Map<string, (result: any) => void> = new Map();
  private requestId = 0;
  private reconnectTimeout: NodeJS.Timeout | null = null;

  async start(): Promise<void> {
    // 1. 启动子进程
    this.process = this.spawnProcess();

    // 2. 设置输出监听
    this.process.stdout.on('data', (data) => {
      this.handleStdout(data);
    });

    // 3. 设置错误处理
    this.process.on('error', (error) => {
      this.handleError(error);
    });

    // 4. 发送初始化请求
    await this.initialize();
  }

  async stop(): Promise<void> {
    // 清理资源
    if (this.process) {
      this.process.kill();
      this.process = null;
    }
  }

  async sendRequest(method: string, params: any): Promise<any> {
    const id = String(++this.requestId);
    const request = {
      jsonrpc: '2.0',
      id,
      method,
      params
    };

    // 发送请求
    this.process?.stdin.write(JSON.stringify(request) + '\n');

    // 等待响应
    return new Promise((resolve, reject) => {
      this.pendingRequests.set(id, resolve);
      setTimeout(() => {
        this.pendingRequests.delete(id);
        reject(new Error('Request timeout'));
      }, 30000);
    });
  }

  private handleStdout(data: Buffer): void {
    // 解析 JSON 行
    const lines = data.toString().split('\n').filter(Boolean);

    for (const line of lines) {
      try {
        const parsed = JSON.parse(line);

        if (parsed.id && this.pendingRequests.has(parsed.id)) {
          // 响应处理
          const resolve = this.pendingRequests.get(parsed.id)!;
          this.pendingRequests.delete(parsed.id);

          if (parsed.error) {
            reject(parsed.error);
          } else {
            resolve(parsed.result);
          }
        } else if (parsed.method) {
          // 事件处理
          this.emit(parsed.method, parsed.params);
        }
      } catch (e) {
        console.error('Failed to parse JSON:', e);
      }
    }
  }

  private handleError(error: Error): void {
    console.error('Process error:', error);
    this.emit('error', error);

    // 自动重启
    this.scheduleRestart();
  }

  private scheduleRestart(): void {
    if (this.reconnectTimeout) return;

    this.reconnectTimeout = setTimeout(async () => {
      this.reconnectTimeout = null;
      await this.start();
    }, 5000);
  }
}
```

## 最佳实践

### 错误处理

**重试策略：**

```typescript
async function sendWithRetry(
  request: Request,
  maxRetries = 3,
  baseDelay = 1000
): Promise<Response> {
  let lastError: Error | null = null;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await sendRequest(request);
    } catch (error) {
      lastError = error as Error;

      // 指数退避
      const delay = baseDelay * Math.pow(2, attempt);
      await sleep(delay);

      // 只重试临时错误
      if (!isTransientError(error)) {
        throw error;
      }
    }
  }

  throw lastError;
}

function isTransientError(error: Error): boolean {
  return (
    error.message.includes('timeout') ||
    error.message.includes('ECONNRESET') ||
    error.message.includes('ECONNREFUSED')
  );
}
```

### 健康检查

**守护进程模式健康检查：**

```typescript
async function checkHealth(endpoint: string): Promise<boolean> {
  try {
    const response = await fetch(`${endpoint}/api/v1/check`);
    const data = await response.json();

    return data.status === 'healthy';
  } catch {
    return false;
  }
}
```

### 监控指标

```
建议收集的指标：

├── 进程状态
│   ├── running (bool)
│   ├── uptime (seconds)
│   └── restarts (count)
│
├── 通信指标
│   ├── requests_total (counter)
│   ├── requests_success (counter)
│   ├── requests_error (counter)
│   ├── request_duration_seconds (histogram)
│   └── pending_requests (gauge)
│
└── 资源指标
    ├── memory_usage_bytes (gauge)
    └── cpu_usage_percent (gauge)
```

## 故障排查

### 常见问题

**问题一：连接超时**

```
症状：请求超时，无响应

排查步骤：
├── 检查目标进程是否运行
├── 验证端口/路径配置是否正确
├── 检查网络连接
└── 查看防火墙规则
```

**问题二：进程意外终止**

```
症状：进程在运行中突然停止

排查步骤：
├── 检查进程日志
├── 验证内存/CPU 使用情况
├── 检查系统资源限制
└── 查看系统日志
```

**问题三：响应格式错误**

```
症状：无法解析响应 JSON

排查步骤：
├── 验证响应格式是否符合 JSON-RPC 2.0
├── 检查是否有乱码或截断
├── 验证编码设置
└── 查看详细错误日志
```

### 调试技巧

**启用调试日志：**

```json
{
  "diagnostics": {
    "flags": ["rpc.*", "external.*"]
  }
}
```

**手动测试通信：**

```bash
# HTTP 模式
curl -X POST http://localhost:8080/api/v1/json-rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"send","params":{"recipients":["test"],"message":"ping"},"id":"1"}'

# stdio 模式
echo '{"jsonrpc":"2.0","method":"send","params":{"recipients":["test"],"message":"ping"},"id":"1"}' | imsg rpc
```

## 相关文档

### 通道集成

- [Signal 通道配置](/channels/signal)
- [iMessage 通道配置](/channels/imessage)

### 配置参考

- [网关配置](/gateway/configuration)
- [通道配置](/channels)

### 外部资源

- [JSON-RPC 2.0 规范](https://www.jsonrpc.org/specification)
- [Signal CLI 文档](https://github.com/AsamK/signal-cli)
- [iMsg 项目](https://github.com/gradyrobbins/imsg)
