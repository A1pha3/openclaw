---
summary: "工具和技能系统完整指南——浏览器控制、命令执行、Web 搜索、文件操作等工具的使用和配置，以及自定义技能的开发方法"
read_when:
  - 了解工具系统的工作原理
  - 使用各种内置工具扩展 AI 能力
  - 配置和管理技能
  - 开发自定义工具和技能
title: "工具和技能"
---

# 🛠️ 工具和技能

OpenClaw 通过**工具系统**扩展 AI 代理的能力边界，使 AI 不仅能够对话和思考，还能够执行实际操作。工具系统是连接 AI 智慧与现实世界的桥梁——AI 可以通过工具浏览网页、读写文件、执行命令、搜索信息，让你的 AI 助手真正成为得力的工作伙伴。本文将详细介绍工具系统的架构设计、内置工具的使用方法以及自定义技能的开发指南。

> **工具系统的核心价值**
>
> 想象一下，如果你有一个非常聪明的助手，但他不能使用电脑、不能查阅资料、不能发送邮件，那他的能力将大打折扣。OpenClaw 的工具系统正是为了解决这个限制。通过工具，AI 代理获得了"动手能力"：它可以代表你执行操作、获取外部信息、操作外部系统。工具系统的设计遵循最小权限原则，确保在扩展能力的同时保持安全性。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解工具系统的基本架构和工作原理
- 掌握内置工具的基本使用方法
- 配置和管理已安装的技能
- 理解工具调用的安全边界

### 进阶目标（建议掌握）

- 优化工具调用策略以提高效率
- 配置工具使用的权限和限制
- 集成第三方 API 和服务
- 实现工具调用的缓存和错误处理

### 专家目标（挑战）

- 开发完整的自定义工具模块
- 设计复杂的多工具协同工作流
- 构建可复用的技能包
- 优化工具性能和安全审计

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速上手** | 首次使用工具系统 | 30 分钟 | 启用工具，执行第一次工具调用 |
| **日常使用** | 普通用户日常使用 | 1 小时 | 掌握常用工具，高效使用 |
| **高级配置** | 需要深度定制 | 2 小时 | 权限配置、性能优化 |
| **开发扩展** | 开发者扩展功能 | 4 小时 | 开发自定义工具和技能 |

---

## 🧠 原理解释：工具系统架构

### 工具系统的核心组件

OpenClaw 的工具系统由以下几个核心组件构成：

**工具定义（Tool Definition）**

每个工具都有一个标准化的定义，描述了工具的名称、功能、参数和返回值。工具定义是 AI 了解如何使用该工具的唯一依据。良好的工具定义包含：清晰的功能描述，让 AI 理解何时应该使用这个工具；明确的参数规范，包括参数类型、是否必填、取值范围等；详细的返回格式说明，帮助 AI 正确解析结果；使用示例，展示工具的典型用法。

**工具执行器（Tool Executor）**

执行器负责实际执行工具调用。对于不同类型的工具，执行器的实现方式也不同：内置工具的执行器集成在 OpenClaw 核心中，直接调用系统资源；外部工具的执行器通过子进程或 API 调用执行；远程工具的执行器通过网络协议与外部服务通信。

**工具注册表（Tool Registry）**

注册表维护了所有可用工具的索引，AI 可以根据名称查找工具并获取其定义。注册表还负责管理工具的生命周期，包括加载、卸载和更新。

**安全控制器（Security Controller）**

安全控制器是工具系统的守门人，负责：验证工具调用的权限；检查参数的有效性和安全性；记录工具调用的审计日志；在必要时阻止危险操作。

### 工具调用流程

当 AI 代理决定使用工具时，整个调用流程如下：

```
步骤一：AI 决定使用工具
     │
     ▼
步骤二：从注册表获取工具定义
     │
     ▼
步骤三：安全控制器检查权限
     │
     ├── 通过 ──→ 步骤四：执行器执行工具
     │
     └── 拒绝 ──→ 返回安全错误
                │
                ▼
            记录审计日志
```

理解这个流程有助于诊断工具调用问题：如果工具未被调用，可能是 AI 的决策逻辑问题；如果权限被拒绝，需要检查安全配置；如果执行失败，可能是工具本身的错误。

---

## 📦 技能系统概述

### 什么是技能

**技能（Skill）** 是工具的集合和封装，一个技能可以包含多个相关工具，提供更完整的功能。技能系统提供了比单个工具更高层次的抽象：预打包的功能——技能将相关的工具和配置打包在一起，用户可以一键启用完整功能；版本管理——技能支持版本控制，可以方便地更新和回滚；依赖管理——技能可以声明依赖，自动安装所需组件；配置模板——技能提供预置的配置模板，降低使用门槛。

### 技能目录结构

技能存储在多个位置，OpenClaw 按以下优先级搜索：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | `~/.openclaw/skills/` | 用户全局技能目录 |
| 2 | `<workspace>/skills/` | 工作区技能目录 |
| 3 | `~/.openclaw/skills/bundled/` | 内置技能目录 |
| 4 | `~/.openclaw/extensions/*/skills/` | 插件内置技能 |

### 技能的标准结构

每个技能都是一个标准化的目录结构：

```
my-skill/
├── SKILL.md           # 技能元数据和配置
├── tools/            # 工具实现
│   ├── tool1.ts
│   └── tool2.ts
├── package.json      # 依赖声明（可选）
└── README.md         # 技能文档
```

---

## 🚀 内置工具详解

### 工具一：文件读取（read）

文件读取工具允许 AI 读取本地文件内容，是最基础也是最常用的工具之一。

**功能描述**：读取指定路径的文件内容，支持文本文件和二进制文件。AI 可以使用这个工具查阅文档、分析代码、读取配置文件等。

**使用场景**：

- 读取项目文档，了解项目背景和结构
- 分析源代码，理解函数逻辑
- 查看配置文件，获取系统设置
- 读取日志文件，排查问题

**使用示例**：

```
用户：请帮我查看项目的 README 文件内容
AI：[使用 read 工具读取 README.md]
```

**参数配置**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | string | 是 | 要读取的文件路径 |
| encoding | string | 否 | 编码格式，默认 utf-8 |
| offset | number | 否 | 起始行号 |
| limit | number | 否 | 读取行数限制 |

**安全考虑**：文件读取工具默认只能访问工作区和配置目录中的文件，无法读取系统敏感文件。如果需要访问其他位置，需要在配置中明确授权。

### 工具二：文件写入（write）

文件写入工具允许 AI 创建或修改文件，是实现文件操作的主要工具。

**功能描述**：创建新文件或覆盖现有文件内容，支持文本和二进制数据。

**使用场景**：

- 生成代码文件
- 创建文档和报告
- 写入配置文件
- 生成数据文件

**使用示例**：

```
用户：创建一个新的 Python 脚本来计算斐波那契数列
AI：[使用 write 工具创建 fibonacci.py]
```

**参数配置**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | string | 是 | 要写入的文件路径 |
| content | string | 是 | 文件内容 |
| mode | string | 否 | 写入模式：overwrite/append |

**安全考虑**：写入操作默认只允许在工作区内进行。危险操作（如覆盖系统文件）会被阻止。

### 工具三：文件编辑（edit）

文件编辑工具提供精确的文件修改能力，相比写入工具更加精细。

**功能描述**：对文件的指定部分进行修改，支持精确的范围定位和替换。

**使用场景**：

- 修改配置文件中的特定设置
- 修复代码中的 bug
- 更新文档的某一部分
- 重构代码的小改动

**使用示例**：

```
用户：把配置中的端口号从 8080 改成 3000
AI：[使用 edit 工具修改 config.json]
```

**参数配置**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | string | 是 | 要编辑的文件路径 |
| oldString | string | 是 | 要替换的原文（必须精确匹配） |
| newString | string | 是 | 替换后的内容 |

**使用技巧**：oldString 应该足够长以确保唯一性，但又不能包含太多动态内容。最佳实践是包含至少 3-5 行上下文。

### 工具四：命令执行（exec）

命令执行工具允许 AI 运行系统命令，是最强大但也最需要谨慎的工具。

**功能描述**：在系统 shell 中执行命令，返回命令的输出结果。

**使用场景**：

- 运行构建脚本和开发命令
- 执行 Git 操作
- 管理进程和服务
- 运行测试和代码检查

**使用示例**：

```
用户：运行项目的测试套件
AI：[使用 exec 工具运行 npm test]
```

**参数配置**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| command | string | 是 | 要执行的完整命令 |
| timeout | number | 否 | 超时时间（毫秒） |
| workdir | string | 否 | 工作目录 |

**安全考虑**：命令执行工具默认需要显式授权。在配置中需要列出允许执行的命令白名单。

**危险命令警告**：以下命令在未经明确授权时会被阻止：

- rm -rf（递归删除）
- sudo（提权操作）
- chmod 777（权限修改）
- > /dev/null（输出丢弃）

### 工具五：浏览器控制（browser）

浏览器控制工具让 AI 能够自动化浏览器操作，是进行 Web 自动化和信息采集的核心工具。

**功能描述**：通过 CDP（Chrome DevTools Protocol）控制浏览器，执行导航、点击、填写表单等操作。

**使用场景**：

- 访问网页获取信息
- 自动化表单填写
- 截图和页面分析
- Web 应用测试

**使用示例**：

```
用户：搜索 OpenClaw 的最新版本
AI：[使用 browser 工具打开搜索引擎，搜索关键词]
```

**可用操作**：

| 操作 | 说明 |
|------|------|
| navigate | 导航到指定 URL |
| click | 点击页面元素 |
| type | 输入文本 |
| scroll | 滚动页面 |
| screenshot | 截取页面截图 |
| evaluate | 执行 JavaScript |

### 工具六：Web 搜索（web）

Web 搜索工具提供互联网信息检索能力，让 AI 能够获取最新信息。

**功能描述**：调用搜索引擎获取网络信息，支持多种搜索引擎和搜索策略。

**使用场景**：

- 查询最新新闻和信息
- 搜索技术文档和解决方案
- 查找产品评测和比较
- 获取实时数据

**使用示例**：

```
用户：OpenClaw 的最新版本有什么新功能？
AI：[使用 web 搜索工具查找信息]
```

---

## 🛠️ 技能管理

### 查看可用技能

```bash
# 列出所有已安装的技能
openclaw skills list

# 查看技能详情
openclaw skills info <skill-name>

# 搜索可用技能
openclaw skills search <keyword>
```

### 安装和卸载技能

```bash
# 安装技能
openclaw skills install web
openclaw skills install github

# 安装指定版本
openclaw skills install web@1.2.0

# 卸载技能
openclaw skills uninstall web
```

### 技能配置

每个技能可以在配置文件中进行自定义：

```json5
{
  skills: {
    web: {
      enabled: true,
      options: {
        searchEngine: "google",
        maxResults: 10,
        timeout: 30000
      }
    },
    
    github: {
      enabled: true,
      options: {
        token: "${GITHUB_TOKEN}",
        defaultOwner: "openclaw"
      }
    }
  }
}
```

---

## 📋 工具配置详解

### 全局工具配置

```json5
{
  tools: {
    // 是否启用工具系统
    enabled: true,
    
    // 默认超时时间
    defaultTimeout: 30000,
    
    // 允许的工具列表（空列表表示允许所有）
    allowedTools: [
      "read",
      "write",
      "edit",
      "exec"
    ],
    
    // 禁止的工具列表
    blockedTools: [
      "format_disk",
      "modify_system"
    ],
    
    // 执行选项
    execution: {
      // 最大并发数
      maxConcurrency: 5,
      // 重试次数
      retryCount: 3,
      // 重试间隔
      retryDelay: 1000
    }
  }
}
```

### 工具特定配置

**浏览器工具配置**：

```json5
{
  tools: {
    browser: {
      enabled: true,
      options: {
        // Chrome 可执行文件路径
        executablePath: null,  // 自动检测
        // 启动参数
        args: [
          "--no-sandbox",
          "--disable-setuid-sandbox"
        ],
        // 用户数据目录
        userDataDir: null,
        // 视窗大小
        viewport: {
          width: 1280,
          height: 800
        },
        // 截图保存目录
        screenshotDir: "/tmp/openclaw-screenshots"
      }
    }
  }
}
```

**命令执行工具配置**：

```json5
{
  tools: {
    exec: {
      enabled: true,
      // 命令白名单
      allowedCommands: [
        "git *",
        "npm *",
        "pnpm *",
        "ls *",
        "cat *"
      ],
      // 工作目录限制
      workdir: "${WORKSPACE}",
      // 环境变量
      env: {
        PATH: "${PATH}",
        HOME: "${HOME}"
      }
    }
  }
}
```

---

## 🔧 自定义工具开发

### 开发第一个自定义工具

假设我们需要创建一个天气查询工具：

**步骤一：创建工具定义**

```typescript
// tools/weather.ts
import type { ToolDefinition, ToolExecutor, ToolResult } from "openclaw/tools";

export const weatherTool: ToolDefinition = {
  name: "weather",
  description: "获取指定城市的天气信息",
  parameters: {
    type: "object",
    properties: {
      city: {
        type: "string",
        description: "城市名称，如 北京、上海"
      },
      days: {
        type: "number",
        description: "查询天数，默认 1",
        default: 1
      }
    },
    required: ["city"]
  },
  returns: {
    type: "object",
    properties: {
      temperature: { type: "number", description: "温度（摄氏度）" },
      humidity: { type: "number", description: "湿度（百分比）" },
      condition: { type: "string", description: "天气状况" },
      forecast: {
        type: "array",
        items: {
          type: "object",
          properties: {
            date: { type: "string" },
            temperature: { type: "number" },
            condition: { type: "string" }
          }
        }
      }
    }
  }
};

export const weatherExecutor: ToolExecutor = async (params: {
  city: string;
  days?: number;
}): Promise<ToolResult> => {
  const { city, days = 1 } = params;
  
  // 调用天气 API
  const response = await fetch(
    `https://api.weather.example.com?city=${encodeURIComponent(city)}&days=${days}`
  );
  
  const data = await response.json();
  
  return {
    success: true,
    data: {
      temperature: data.temperature,
      humidity: data.humidity,
      condition: data.condition,
      forecast: data.forecast
    }
  };
};
```

**步骤二：注册工具**

```typescript
// tools/index.ts
import { registerTool } from "openclaw/tools";
import { weatherTool, weatherExecutor } from "./weather";

registerTool({
  definition: weatherTool,
  executor: weatherExecutor,
  // 工具分类
  category: "information",
  // 权限要求
  permissions: ["network"]
});
```

**步骤三：配置技能**

```yaml
# skills/weather/SKILL.md
name: weather
description: 提供天气查询功能
version: 1.0.0
author: Your Name

tools:
  - weather

config:
  - name: apiKey
    description: 天气 API 密钥
    required: true
```

### 工具开发最佳实践

**实践一：清晰的错误处理**

```typescript
try {
  const result = await performOperation();
  return {
    success: true,
    data: result
  };
} catch (error) {
  return {
    success: false,
    error: {
      message: error.message,
      code: error.code,
      recoverable: error.recoverable
    }
  };
}
```

**实践二：输入验证**

```typescript
function validateParams(params: Record<string, unknown>): void {
  if (params.city && typeof params.city !== "string") {
    throw new ToolError("city 参数必须是字符串", "INVALID_PARAM");
  }
  
  if (params.city.length > 100) {
    throw new ToolError("城市名称过长", "PARAM_TOO_LONG");
  }
}
```

**实践三：超时控制**

```typescript
export const executor: ToolExecutor = async (params, context) => {
  const abortController = new AbortController();
  const timeoutId = setTimeout(() => abortController.abort(), 30000);
  
  try {
    const result = await fetchWithSignal(params, abortController.signal);
    return { success: true, data: result };
  } finally {
    clearTimeout(timeoutId);
  }
};
```

---

## 🔒 安全配置

### 权限级别

OpenClaw 的工具系统实现了精细的权限控制：

| 级别 | 说明 | 示例 |
|------|------|------|
| **无限制** | 所有会话都可使用 | read, write |
| **需确认** | 首次使用需用户确认 | exec, browser |
| **仅主会话** | 仅主会话可用 | system operations |
| **需显式启用** | 需在配置中显式启用 | dangerous tools |

### 安全策略配置

```json5
{
  tools: {
    security: {
      // 信任级别
      trustLevel: "collaborative",  // collaborative / restricted / locked
      
      // 是否允许危险工具
      allowDangerousTools: false,
      
      // 是否需要用户确认
      requireConfirmation: true,
      
      // 敏感操作白名单
      sensitiveOperations: [
        {
          pattern: "rm -rf *",
          allowed: false
        },
        {
          pattern: "git push *",
          allowed: true,
          confirmBefore: true
        }
      ],
      
      // 文件访问限制
      fileAccess: {
        allowedPaths: [
          "${WORKSPACE}",
          "${CONFIG_DIR}"
        ],
        deniedPaths: [
          "/etc/passwd",
          "~/.ssh/*",
          "/root/*"
        ]
      },
      
      // 网络访问限制
      networkAccess: {
        allowedDomains: [
          "api.github.com",
          "api.weather.example.com"
        ],
        blockedDomains: [
          "malicious.com"
        ]
      }
    }
  }
}
```

---

## 🚨 故障排除

### 问题一：工具无法使用

**症状**：AI 尝试使用工具但返回错误。

**诊断步骤**：

```bash
# 检查工具是否启用
openclaw tools list

# 查看工具状态
openclaw tools status <tool-name>

# 检查配置
openclaw doctor --tools
```

**常见原因**：工具未在配置中启用；工具依赖未安装；权限不足。

### 问题二：权限被拒绝

**症状**：调用工具时返回权限错误。

**解决方案**：检查安全配置；在配置中添加白名单；联系管理员获取权限。

### 问题三：执行超时

**症状**：工具执行很长时间后超时。

**解决方案**：增加超时设置；优化工具逻辑；检查外部服务状态。

### 问题四：工具返回错误

**症状**：工具执行成功但返回错误结果。

**解决方案**：查看工具返回的详细错误信息；检查输入参数是否正确；验证外部服务是否正常。

---

## 📋 适用场景分析

### 场景一：代码开发

**推荐工具组合**：read、write、edit、exec、browser

**典型工作流**：

```
1. 读取现有代码文件
2. 分析代码结构
3. 修改代码
4. 运行测试验证
5. 查看测试结果
```

### 场景二：信息收集

**推荐工具组合**：web、browser、read

**典型工作流**：

```
1. 使用 Web 搜索查找信息
2. 浏览相关网页
3. 读取和整理信息
4. 生成汇总报告
```

### 场景三：自动化任务

**推荐工具组合**：exec、read、write、browser

**典型工作流**：

```
1. 执行定时任务脚本
2. 读取执行结果
3. 分析数据
4. 生成报告并发送
```

---

## 📖 相关文档

- [配置参考](/zh-CN/config/reference)——完整配置选项
- [开发者指南](/zh-CN/developer)——开发环境搭建
- [插件开发](/zh-CN/developer/plugin-development)——开发扩展插件
- [故障排除](/zh-CN/help/troubleshooting)——问题解决指南
