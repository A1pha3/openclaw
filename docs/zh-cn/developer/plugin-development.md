# 插件开发

本指南介绍如何为 OpenClaw 开发自定义插件，扩展其功能。

## 插件类型

OpenClaw 支持以下类型的插件：

| 类型 | 说明 | 示例 |
|------|------|------|
| 渠道插件 | 添加新的消息平台支持 | Mattermost, Matrix |
| 技能插件 | 添加新的 AI 工具 | 博客监控, 语音通话 |
| 存储插件 | 自定义数据存储 | LanceDB 向量存储 |
| 认证插件 | 自定义认证方式 | Google Gemini Auth |

## 快速开始

### 创建插件项目

```bash
mkdir openclaw-plugin-example
cd openclaw-plugin-example
npm init -y
```

### 安装 SDK

```bash
npm install openclaw --save-dev
```

### 基本结构

```
openclaw-plugin-example/
├── package.json
├── src/
│   └── index.ts
├── tsconfig.json
└── README.md
```

### package.json 配置

```json
{
  "name": "@yourorg/openclaw-example",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "exports": {
    ".": "./dist/index.js"
  },
  "peerDependencies": {
    "openclaw": ">=2026.1.0"
  },
  "devDependencies": {
    "openclaw": "^2026.1.0",
    "typescript": "^5.9.0"
  }
}
```

## 渠道插件开发

### 基本结构

```typescript
// src/index.ts
import {
  type ChannelPlugin,
  type ChannelMessage,
  type SendResult,
  defineChannelPlugin
} from 'openclaw/plugin-sdk';

export const myChannelPlugin = defineChannelPlugin({
  // 插件标识
  id: 'mychannel',
  name: 'My Channel',
  version: '1.0.0',

  // 配置 Schema
  configSchema: {
    type: 'object',
    properties: {
      apiKey: { type: 'string' },
      enabled: { type: 'boolean', default: true }
    },
    required: ['apiKey']
  },

  // 初始化
  async init(context) {
    const { config, logger } = context;
    logger.info('My Channel plugin initialized');
    
    // 返回渠道实例
    return {
      async start() {
        // 启动连接
      },
      
      async stop() {
        // 断开连接
      },
      
      async send(message: ChannelMessage): Promise<SendResult> {
        // 发送消息
        return { success: true };
      },
      
      async getStatus() {
        return { connected: true };
      }
    };
  }
});

export default myChannelPlugin;
```

### 处理入站消息

```typescript
async init(context) {
  const { config, logger, onMessage } = context;
  
  // 设置消息监听
  const client = new MyApiClient(config.apiKey);
  
  client.on('message', (msg) => {
    // 转换为标准格式
    onMessage({
      channel: 'mychannel',
      sender: {
        id: msg.from,
        name: msg.fromName
      },
      content: {
        type: 'text',
        text: msg.text
      },
      timestamp: new Date(),
      metadata: {
        messageId: msg.id
      }
    });
  });
  
  return { /* ... */ };
}
```

### 支持多账号

```typescript
export const myChannelPlugin = defineChannelPlugin({
  id: 'mychannel',
  supportsMultiAccount: true,
  
  async init(context) {
    const { config, accountId } = context;
    
    // accountId 用于区分不同账号
    const accountConfig = config.accounts?.[accountId] || config;
    
    // ...
  }
});
```

## 技能插件开发

### 基本结构

```typescript
// src/index.ts
import { defineSkillPlugin, type Tool } from 'openclaw/plugin-sdk';

export const mySkillPlugin = defineSkillPlugin({
  id: 'myskill',
  name: 'My Skill',
  version: '1.0.0',
  
  // 定义工具
  tools: [
    {
      name: 'my_tool',
      description: '执行某个操作',
      parameters: {
        type: 'object',
        properties: {
          query: {
            type: 'string',
            description: '查询内容'
          }
        },
        required: ['query']
      },
      
      async execute(params, context) {
        const { query } = params;
        const { logger } = context;
        
        logger.info(`Executing my_tool with query: ${query}`);
        
        // 执行操作
        const result = await doSomething(query);
        
        return {
          success: true,
          data: result
        };
      }
    }
  ]
});

export default mySkillPlugin;
```

### 工具参数 Schema

使用 JSON Schema 定义参数：

```typescript
{
  name: 'search_web',
  description: '搜索网页',
  parameters: {
    type: 'object',
    properties: {
      query: {
        type: 'string',
        description: '搜索关键词'
      },
      limit: {
        type: 'number',
        description: '结果数量',
        default: 10
      },
      language: {
        type: 'string',
        enum: ['zh', 'en', 'ja'],
        description: '语言'
      }
    },
    required: ['query']
  }
}
```

### 访问上下文

```typescript
async execute(params, context) {
  const {
    logger,          // 日志记录器
    config,          // 插件配置
    session,         // 当前会话
    agent,           // 当前代理
    gateway,         // Gateway 实例
    workspace        // 工作区路径
  } = context;
  
  // 使用上下文
  logger.info('Tool executed');
  
  // 读取文件
  const filePath = path.join(workspace, 'data.json');
  
  // ...
}
```

## 存储插件开发

```typescript
import { defineStoragePlugin } from 'openclaw/plugin-sdk';

export const myStoragePlugin = defineStoragePlugin({
  id: 'mystorage',
  name: 'My Storage',
  version: '1.0.0',
  
  async init(context) {
    const { config } = context;
    
    return {
      async get(key: string) {
        // 获取数据
      },
      
      async set(key: string, value: unknown) {
        // 存储数据
      },
      
      async delete(key: string) {
        // 删除数据
      },
      
      async list(prefix?: string) {
        // 列出键
      }
    };
  }
});
```

## 配置 Schema

### 定义配置

```typescript
configSchema: {
  type: 'object',
  properties: {
    apiKey: {
      type: 'string',
      description: 'API 密钥',
      // UI 提示
      'x-ui': {
        label: 'API 密钥',
        sensitive: true,  // 敏感字段
        group: '认证'
      }
    },
    maxRetries: {
      type: 'number',
      default: 3,
      minimum: 1,
      maximum: 10
    }
  },
  required: ['apiKey']
}
```

### 读取配置

配置会自动从 `channels.<pluginId>` 或 `plugins.<pluginId>` 读取：

```json5
{
  channels: {
    mychannel: {
      apiKey: "sk-...",
      maxRetries: 5
    }
  }
}
```

## 测试插件

### 单元测试

```typescript
// src/index.test.ts
import { describe, it, expect, vi } from 'vitest';
import { mySkillPlugin } from './index';

describe('mySkillPlugin', () => {
  it('should execute tool correctly', async () => {
    const context = {
      logger: {
        info: vi.fn(),
        error: vi.fn()
      },
      config: {},
      // ...
    };
    
    const tool = mySkillPlugin.tools[0];
    const result = await tool.execute({ query: 'test' }, context);
    
    expect(result.success).toBe(true);
  });
});
```

### 集成测试

```typescript
import { createTestGateway } from 'openclaw/testing';

describe('myChannelPlugin integration', () => {
  it('should handle messages', async () => {
    const gateway = await createTestGateway({
      plugins: [myChannelPlugin]
    });
    
    // 模拟消息
    await gateway.simulateMessage({
      channel: 'mychannel',
      sender: { id: 'user1' },
      content: { type: 'text', text: 'Hello' }
    });
    
    // 验证响应
    // ...
    
    await gateway.stop();
  });
});
```

## 发布插件

### 准备发布

1. 更新版本号
2. 构建项目
3. 编写文档

```bash
# 构建
npm run build

# 测试
npm test

# 发布
npm publish --access public
```

### 命名约定

- 官方插件：`@openclaw/<name>`
- 社区插件：`openclaw-plugin-<name>` 或 `@yourorg/openclaw-<name>`

## 最佳实践

### 错误处理

```typescript
async execute(params, context) {
  try {
    const result = await riskyOperation();
    return { success: true, data: result };
  } catch (error) {
    context.logger.error('Operation failed', error);
    return {
      success: false,
      error: error.message
    };
  }
}
```

### 日志记录

```typescript
// 使用结构化日志
context.logger.info('Processing request', {
  userId: params.userId,
  action: 'search'
});

// 不要记录敏感信息
// ❌ logger.info(`API key: ${apiKey}`);
// ✅ logger.info('Using configured API key');
```

### 资源清理

```typescript
async init(context) {
  const client = new ApiClient();
  const interval = setInterval(healthCheck, 60000);
  
  return {
    async stop() {
      clearInterval(interval);
      await client.disconnect();
    }
  };
}
```

### 类型安全

使用 TypeScript 并充分利用类型：

```typescript
import type { ToolExecuteContext } from 'openclaw/plugin-sdk';

interface MyToolParams {
  query: string;
  limit?: number;
}

async execute(params: MyToolParams, context: ToolExecuteContext) {
  // 类型安全的参数访问
}
```

## 示例插件

查看官方插件获取更多示例：

- [Mattermost](https://github.com/openclaw/openclaw/tree/main/extensions/mattermost)
- [Matrix](https://github.com/openclaw/openclaw/tree/main/extensions/matrix)
- [Voice Call](https://github.com/openclaw/openclaw/tree/main/extensions/voice-call)
- [Memory LanceDB](https://github.com/openclaw/openclaw/tree/main/extensions/memory-lancedb)

## 下一步

- [API 参考](/zh-cn/developer/api-reference) - 完整 API 文档
- [测试指南](/zh-cn/developer/testing) - 测试最佳实践
- [开发者手册](/zh-cn/developer/index) - 开发环境搭建
