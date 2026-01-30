# 测试指南

本文档介绍如何为 Moltbot 编写和运行测试。

## 测试框架

Moltbot 使用 **Vitest** 作为测试框架，配合 V8 覆盖率工具。

### 覆盖率要求

项目要求以下最低覆盖率阈值：

| 指标 | 阈值 |
|------|------|
| 行覆盖率 | 70% |
| 分支覆盖率 | 70% |
| 函数覆盖率 | 70% |
| 语句覆盖率 | 70% |

## 运行测试

### 基本命令

```bash
# 运行所有测试
pnpm test

# 带覆盖率报告
pnpm test:coverage

# 监视模式（开发时使用）
pnpm test:watch

# 运行特定文件
pnpm test src/channels/router.test.ts

# 运行匹配模式的测试
pnpm test -t "should route messages"
```

### 实时测试

需要真实 API 密钥的测试：

```bash
# Moltbot 实时测试
CLAWDBOT_LIVE_TEST=1 pnpm test:live

# 包含提供者实时测试
LIVE=1 pnpm test:live
```

### Docker 测试

```bash
# 实时模型测试
pnpm test:docker:live-models

# 实时网关测试
pnpm test:docker:live-gateway

# 引导流程 E2E 测试
pnpm test:docker:onboard
```

## 测试文件命名

| 类型 | 命名模式 | 示例 |
|------|----------|------|
| 单元测试 | `*.test.ts` | `router.test.ts` |
| E2E 测试 | `*.e2e.test.ts` | `gateway.e2e.test.ts` |
| 集成测试 | `*.integration.test.ts` | `channels.integration.test.ts` |

测试文件与源文件放在同一目录（共置）：

```
src/
├── channels/
│   ├── router.ts
│   └── router.test.ts
├── gateway/
│   ├── server.ts
│   └── server.test.ts
```

## 编写测试

### 基本测试结构

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest'

describe('MessageRouter', () => {
  let router: MessageRouter

  beforeEach(() => {
    router = new MessageRouter()
  })

  afterEach(() => {
    vi.restoreAllMocks()
  })

  describe('route()', () => {
    it('should route messages to correct agent', () => {
      const message = createTestMessage({ channel: 'telegram' })
      const result = router.route(message)
      
      expect(result.agentId).toBe('main')
    })

    it('should apply binding rules', () => {
      router.addBinding({
        match: { channel: 'discord' },
        agent: 'coder'
      })

      const message = createTestMessage({ channel: 'discord' })
      const result = router.route(message)
      
      expect(result.agentId).toBe('coder')
    })
  })
})
```

### Mock 和 Spy

```typescript
import { vi } from 'vitest'

describe('ChannelManager', () => {
  it('should call provider start method', async () => {
    const mockProvider = {
      id: 'test',
      start: vi.fn().mockResolvedValue(undefined),
      stop: vi.fn().mockResolvedValue(undefined)
    }

    const manager = new ChannelManager()
    manager.register(mockProvider)
    
    await manager.startAll()

    expect(mockProvider.start).toHaveBeenCalledTimes(1)
  })

  it('should log errors on failure', async () => {
    const logSpy = vi.spyOn(console, 'error')
    const error = new Error('Connection failed')

    const mockProvider = {
      id: 'test',
      start: vi.fn().mockRejectedValue(error)
    }

    const manager = new ChannelManager()
    manager.register(mockProvider)

    await expect(manager.startAll()).rejects.toThrow('Connection failed')
    expect(logSpy).toHaveBeenCalledWith(expect.stringContaining('test'))
  })
})
```

### 异步测试

```typescript
describe('Gateway', () => {
  it('should handle async message processing', async () => {
    const gateway = await createGateway()
    
    const response = await gateway.process({
      type: 'message',
      content: 'Hello'
    })

    expect(response.status).toBe('ok')
  })

  it('should timeout after 30 seconds', async () => {
    vi.useFakeTimers()
    
    const gateway = await createGateway()
    const promise = gateway.processLongRunning()

    vi.advanceTimersByTime(31000)

    await expect(promise).rejects.toThrow('Timeout')
    
    vi.useRealTimers()
  })
})
```

### 快照测试

```typescript
describe('Message formatting', () => {
  it('should format messages correctly', () => {
    const formatted = formatMessage({
      content: 'Hello **world**',
      sender: 'user123'
    })

    expect(formatted).toMatchSnapshot()
  })
})
```

## 测试工具函数

### 创建测试数据

```typescript
// tests/utils/factories.ts
export function createTestMessage(overrides = {}) {
  return {
    id: 'msg-123',
    channel: 'telegram',
    from: 'user-456',
    content: 'Test message',
    timestamp: Date.now(),
    ...overrides
  }
}

export function createTestConfig(overrides = {}) {
  return {
    agents: {
      defaults: {
        model: 'test-model'
      }
    },
    ...overrides
  }
}
```

### Mock 外部服务

```typescript
// tests/mocks/telegram.ts
export function mockTelegramAPI() {
  return {
    sendMessage: vi.fn().mockResolvedValue({ message_id: 123 }),
    getMe: vi.fn().mockResolvedValue({ id: 'bot123', username: 'testbot' }),
    setWebhook: vi.fn().mockResolvedValue(true)
  }
}
```

## 测试配置

### vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 70,
        branches: 70,
        functions: 70,
        statements: 70
      }
    },
    testTimeout: 10000,
    hookTimeout: 10000
  }
})
```

## 测试最佳实践

### 1. 测试应该独立

每个测试应该能够独立运行，不依赖其他测试的状态：

```typescript
// 好的做法
beforeEach(() => {
  db = createFreshDatabase()
})

afterEach(() => {
  db.cleanup()
})

// 避免
let sharedState = {}
it('test 1', () => { sharedState.value = 1 })
it('test 2', () => { expect(sharedState.value).toBe(1) }) // 依赖 test 1
```

### 2. 测试行为而非实现

```typescript
// 好的做法 - 测试行为
it('should add user to allowlist', () => {
  manager.allowUser('+15551234567')
  expect(manager.isAllowed('+15551234567')).toBe(true)
})

// 避免 - 测试实现细节
it('should add user to internal array', () => {
  manager.allowUser('+15551234567')
  expect(manager._allowlist).toContain('+15551234567')
})
```

### 3. 使用描述性的测试名称

```typescript
// 好的做法
it('should reject messages from non-allowlisted senders', () => {})
it('should return pairing code for unknown senders when policy is pairing', () => {})

// 避免
it('test1', () => {})
it('works', () => {})
```

### 4. 测试边界条件

```typescript
describe('Message chunking', () => {
  it('should handle empty string', () => {
    expect(chunk('')).toEqual([])
  })

  it('should handle exact limit length', () => {
    const text = 'a'.repeat(4000)
    expect(chunk(text)).toHaveLength(1)
  })

  it('should split at limit + 1', () => {
    const text = 'a'.repeat(4001)
    expect(chunk(text)).toHaveLength(2)
  })

  it('should handle unicode correctly', () => {
    const text = '你好'.repeat(2000)
    const chunks = chunk(text)
    expect(chunks.join('')).toBe(text)
  })
})
```

### 5. 避免过度 Mock

```typescript
// 好的做法 - 仅 mock 外部依赖
it('should send message via Telegram', async () => {
  const api = mockTelegramAPI()
  const channel = new TelegramChannel(api)
  
  await channel.send({ content: 'Hello' })
  
  expect(api.sendMessage).toHaveBeenCalled()
})

// 避免 - mock 太多内部逻辑
it('should process message', async () => {
  vi.mock('./router')
  vi.mock('./formatter')
  vi.mock('./validator')
  // 过度 mock 使测试变得脆弱
})
```

## 调试测试

### 使用 console.log

```typescript
it('debug test', () => {
  const result = complexOperation()
  console.log('Result:', JSON.stringify(result, null, 2))
  expect(result).toBeDefined()
})
```

### 使用 .only 隔离测试

```typescript
// 只运行这个测试
it.only('isolated test', () => {
  // ...
})

// 只运行这个测试组
describe.only('isolated group', () => {
  // ...
})
```

### 使用 --inspect 调试

```bash
# 使用 Node 调试器
node --inspect-brk ./node_modules/.bin/vitest run src/path/to/test.ts
```

## 移动端测试

在测试移动应用时：

1. **优先使用真机**：测试前检查连接的真实设备（iOS + Android）
2. **模拟器作为备选**：仅在没有真机时使用模拟器
3. **重启应用**：指"重新编译、安装并启动"而非仅杀死/启动进程

```bash
# 检查连接的设备
adb devices          # Android
xcrun xctrace list devices  # iOS
```

## 持续集成

测试在 GitHub Actions 中自动运行：

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: pnpm install
      - run: pnpm test:coverage
```

## 相关文档

- [开发指南](/zh-cn/developer)
- [项目结构](/zh-cn/developer/project-structure)
- [贡献指南](/zh-cn/developer/contributing)
