---
summary: "OpenClaw 配置系统详解：JSON5 配置文件、环境变量、配置验证和最佳实践"
read_when:
  - 你需要理解 OpenClaw 的配置层次和优先级
  - 你想掌握配置文件的结构设计和最佳实践
  - 你在调试配置相关问题或需要配置验证
title: "配置概述"
---

# ⚙️ OpenClaw 配置系统完全指南

> **学习目标**：完成本章节学习后，你将能够理解 OpenClaw 配置系统的设计哲学、掌握配置文件的管理方法、理解配置优先级体系，并能够独立完成从简单到复杂的各种配置场景。

---

## 为什么要理解 OpenClaw 的配置系统

在深入具体配置项之前，我们需要先理解**设计者为什么这样设计**。这不仅帮助你更好地使用配置系统，还能让你在遇到复杂场景时做出更好的设计决策。

### 配置系统的核心设计目标

OpenClaw 的配置系统遵循三个核心设计目标：

1. **声明式优先** - 配置是单一事实来源，代码行为由配置驱动
2. **渐进式复杂度** - 从最小配置到完整定制，覆盖不同使用场景
3. **安全性** - 敏感信息隔离，默认安全，危险操作需要明确授权

### 配置系统的架构演进

> **知识延伸**：OpenClaw 的配置系统经历了从简单字典到分层架构的演进。了解历史有助于理解当前的设计决策：
>
> - **v1.x**：简单的键值对配置，缺乏结构化
> - **v2.x**：引入 JSON 结构，但环境变量覆盖规则复杂
> - **v3.x**（当前）：JSON5 格式 + 环境变量 + 配置块的三层体系

---

## 认知负荷管理：学习路径设计

为了帮助你高效学习，我们将配置系统的知识分为四个层次：

```
配置系统认知金字塔：

                    ┌─────────────────────────┐
                    │   专家级：配置架构设计    │  设计团队配置规范、插件配置体系
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：多环境配置      │  开发/生产环境分离、配置模板
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：CLI和高级配置    │  命令行工具、配置验证
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：基础配置         │  配置文件位置、基本结构
                    └─────────────────────────┘
```

**建议学习路径**：
- 如果你是第一次接触 OpenClaw：从「初级」开始，逐步向上
- 如果你有配置经验：直接从「中级」开始，查漏补缺
- 如果你是运维负责人：重点关注「高级」和「专家级」内容

---

## 第一部分：配置基础（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解配置文件的位置和命名规范
- [ ] 掌握 JSON5 格式的基本用法
- [ ] 写出符合规范的最小配置文件
- [ ] 使用 CLI 工具查看和修改配置

### 1.1 配置文件的位置与发现机制

OpenClaw 的配置文件遵循「约定优于配置」的设计原则，采用标准化的目录结构：

**配置文件搜索顺序**：

OpenClaw 在启动时会按照以下顺序查找配置文件（找到第一个即停止搜索）：

```
1. 环境变量指定的路径：OPENCLAW_CONFIG_PATH
2. 默认路径：~/.openclaw/openclaw.json
3. 旧版兼容路径：~/.openclaw.json（已废弃，但仍被识别）
```

**目录结构概览**：

```
~/.openclaw/
├── openclaw.json          # 主配置文件（必需）
├── credentials/           # 凭据存储目录
│   ├── anthropic/        # Anthropic API 密钥
│   ├── openai/           # OpenAI API 密钥
│   └── ...
├── sessions/             # 会话数据目录
├── workspace/            # 工作区目录（可配置）
├── logs/                 # 日志文件目录
│   └── openclaw.log
└── extensions/            # 插件目录
```

**为什么这样设计**：

这种设计基于以下考量：
- **安全性**：将凭据与配置分离，凭据目录有更严格的访问权限控制
- **可移植性**：配置文件可以轻松备份和迁移
- **版本控制友好**：`.gitignore` 可以明确排除敏感目录

### 1.2 JSON5 格式：为什么选择它

OpenClaw 选择 **JSON5** 作为配置格式，这是一个深思熟虑的决策：

**JSON vs JSON5 对比**：

```json5
// ✅ JSON5 允许的写法
{
  // 这是注释（JSON 不允许）
  name: "my-agent",           // 可选的引号
  enabled: true,               // 尾随逗号（JSON 不允许）
  options: {
    timeout: 30000,           // 数字格式
    debug: yes,               // 类似 JavaScript 的布尔值
    list: [1, 2, 3,],         // 数组末尾逗号
  }
}
```

```json
// ❌ 标准 JSON（不允许注释和尾随逗号）
{
  "name": "my-agent",
  "enabled": true,
  "options": {
    "timeout": 30000
  }
}
```

**JSON5 的优势**：

| 特性 | JSON | JSON5 | OpenClaw 的优势 |
|------|------|-------|----------------|
| 注释 | ❌ | ✅ | 文档内联，方便理解 |
| 尾随逗号 | ❌ | ✅ | 添加配置项时不易出错 |
| 字符串简写 | ❌ | ✅ | 更简洁的语法 |
| 兼容性 | 完美兼容 | 超集 | 可用标准 JSON 工具解析 |

### 1.3 最小配置：快速上手

**最小配置示例**：

```json5
{
  // 代理的基本配置
  agents: {
    defaults: {
      // 工作区目录（存放 AGENTS.md 等文件）
      workspace: "~/.openclaw/workspace"
    }
  },
  
  // 渠道配置（至少启用一个）
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

**配置项说明**：

| 配置项 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `agents.defaults.workspace` | 路径 | ✅ | 工作区目录，包含代理的系统提示词文件 |
| `channels.*.enabled` | 布尔 | ✅ | 是否启用该渠道 |
| `channels.*.token` | 字符串 | ✅ | 渠道的身份验证令牌 |

**练习 1.1：创建你的第一个配置文件**

任务：创建一个最小可用的配置文件，启用 Telegram 渠道。

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace"
    }
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456:ABCDEFyour-bot-token-here"
    }
  }
}
```

---

## 第二部分：配置结构与核心概念（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解配置文件的顶层结构设计
- [ ] 掌握各配置块的作用域和优先级
- [ ] 理解配置继承和覆盖机制
- [ ] 设计符合团队需求的配置模板

### 2.1 配置文件的顶层结构

OpenClaw 的配置文件采用模块化的分层设计，每个顶层键对应一个功能域：

```json5
{
  // 代理配置：定义 AI 代理的行为特征
  agents: { ... },
  
  // 渠道配置：配置消息接收和发送渠道
  channels: { ... },
  
  // 网关配置：配置本地控制平面的行为
  gateway: { ... },
  
  // 模型配置：配置使用的 AI 模型和提供商
  models: { ... },
  
  // 消息配置：配置消息处理和呈现方式
  messages: { ... },
  
  // 日志配置：配置日志记录行为
  logging: { ... },
  
  // 安全配置：配置安全策略
  security: { ... },
  
  // 环境变量配置：内联环境变量
  env: { ... }
}
```

**设计原则：关注点分离**

每个配置块都有明确的职责边界：
- `agents`：代理层面的行为（系统提示词、工具调用策略）
- `channels`：渠道层面的配置（令牌、webhook、过滤规则）
- `gateway`：网关层面的配置（端口、认证、绑定地址）
- `models`：模型层面的配置（API 密钥、端点、超时）

### 2.2 配置块的继承与覆盖机制

**继承模型**：

OpenClaw 使用「defaults → user config → runtime override」的三层继承模型：

```
┌─────────────────────────────────────────┐
│           默认配置（内置）               │  优先级最低
├─────────────────────────────────────────┤
│           用户配置（openclaw.json）      │  中等优先级
├─────────────────────────────────────────┤
│         运行时覆盖（CLI、环境变量）       │  优先级最高
└─────────────────────────────────────────┘
```

**继承示例**：

```json5
// 假设默认配置是：
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-5",
      temperature: 0.7,
      maxTokens: 4096
    }
  }
}

// 你的配置文件：
{
  agents: {
    defaults: {
      // 覆盖 model，保留 temperature 和 maxTokens
      model: "anthropic/claude-opus-4-5"
    }
  }
}

// 最终生效配置：
{
  agents: {
    defaults: {
      model: "anthropic/claude-opus-4-5",  // 被覆盖
      temperature: 0.7,                    // 继承默认值
      maxTokens: 4096                      // 继承默认值
    }
  }
}
```

**专家思维模型：配置分层策略**

```markdown
### 思维模型：配置分层决策框架

在设计配置结构时，问自己以下问题：

1. **这个配置是全局的还是局部的？**
   - 全局配置 → agents.defaults 或顶层配置
   - 渠道特定配置 → channels.<channel-name>

2. **这个配置是否需要区分环境？**
   - 是 → 使用环境变量或单独的配置文件
   - 否 → 放在主配置文件中

3. **这个配置是否敏感？**
   - 是 → 使用凭据存储或环境变量
   - 否 → 可以放在配置文件中

4. **这个配置是否经常变动？**
   - 是 → 使用环境变量便于热更新
   - 否 → 放在配置文件中便于管理
```

### 2.3 完整配置结构示例

**示例：完整的多渠道配置**：

```json5
{
  // 代理配置
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: "anthropic/claude-opus-4-5",
      temperature: 0.7,
      maxTokens: 8192,
      thinkingLevel: "high",
      sandbox: {
        mode: "non-main"
      }
    },
    // 为特定渠道定义单独的代理配置
    channels: {
      telegram: {
        model: "anthropic/claude-sonnet-4-5"
      }
    }
  },
  
  // 渠道配置
  channels: {
    telegram: {
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}",
      groups: {
        "*": {
          requireMention: true
        }
      }
    },
    discord: {
      enabled: true,
      token: "${DISCORD_BOT_TOKEN}",
      dm: {
        policy: "pairing"
      }
    }
  },
  
  // 网关配置
  gateway: {
    mode: "local",
    bind: "loopback",
    port: 18789,
    auth: {
      type: "token",
      token: "${GATEWAY_TOKEN}"
    }
  },
  
  // 日志配置
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    redactSensitive: "tools"
  }
}
```

### 2.4 常见配置模式

**模式一：环境分离配置**

```json5
{
  // 使用环境变量实现开发/生产分离
  models: {
    providers: {
      "anthropic": {
        apiKey: "${ANTHROPIC_API_KEY}",
        // 开发环境使用测试模型
        defaultModel: "${ENV:ENVIRONMENT=development}:anthropic/claude-sonnet-4-5:${ENV:ENVIRONMENT=production}:anthropic/claude-opus-4-5"
      }
    }
  }
}
```

**模式二：渠道特定配置**

```json5
{
  channels: {
    telegram: {
      // Telegram 特定配置
      parseMode: "MarkdownV2",
      disableWebPagePreview: false
    },
    slack: {
      // Slack 特定配置
      formatting: "mrkdwn",
      unfurlLinks: true
    }
  }
}
```

**模式三：安全配置**

```json5
{
  security: {
    sandbox: {
      mode: "non-main",  // 非主会话使用沙箱
      allowedTools: ["read", "write", "bash"],
      deniedTools: ["browser", "canvas", "nodes"]
    },
    dm: {
      policy: "pairing",  // 私聊需要配对
      allowFrom: ["*"]
    }
  }
}
```

---

## 第三部分：CLI 配置管理（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 熟练使用 openclaw config 命令族
- [ ] 理解配置验证和自动修复机制
- [ ] 掌握配置热更新和回滚策略
- [ ] 能够诊断和修复常见配置问题

### 3.1 CLI 配置命令详解

**openclaw config 命令族**：

```bash
# 查看配置（支持嵌套查询）
openclaw config get                        # 查看全部配置
openclaw config get agents.defaults        # 查看特定配置项
openclaw config get channels.telegram      # 查看渠道配置

# 设置配置值
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4-5"
openclaw config set gateway.port 18789

# 编辑配置（打开默认编辑器）
openclaw config edit

# 验证配置
openclaw doctor                           # 检查并报告问题
openclaw doctor --fix                     # 自动修复已知问题

# 导出/导入配置
openclaw config export > backup.json      # 导出配置（不含敏感信息）
openclaw config import backup.json        # 导入配置
```

**底层原理：配置热更新机制**：

```
┌──────────────────────────────────────────────────┐
│              配置热更新流程                        │
├──────────────────────────────────────────────────┤
│  1. CLI 修改 openclaw.json                       │
│           ↓                                      │
│  2. 触发配置文件监听器                            │
│           ↓                                      │
│  3. 解析并验证新配置                             │
│           ↓                                      │
│  4. 如果验证通过：应用新配置                      │
│     如果验证失败：回滚到旧配置                    │
└──────────────────────────────────────────────────┘
```

**注意事项**：
- 某些配置（如网关端口）需要重启才能生效
- 使用 `openclaw doctor --fix` 时会创建 `.bak` 备份文件

### 3.2 配置验证系统

**验证检查项**：

`openclaw doctor` 命令执行以下检查：

| 检查项 | 说明 | 自动修复 |
|--------|------|----------|
| 文件权限 | 配置文件权限是否安全 | ✅ |
| JSON 语法 | JSON5 语法是否正确 | ❌ |
| 必需配置 | 必需的配置文件是否完整 | ❌ |
| 渠道配置 | 渠道配置是否有效 | ⚠️ 部分 |
| 凭据检查 | API 令牌是否有效 | ❌ |
| 端口冲突 | 指定端口是否被占用 | ❌ |

**诊断输出示例**：

```bash
$ openclaw doctor

✅ 配置检查通过
✅ 渠道状态正常
✅ 网关运行中（端口 18789）
⚠️  Telegram botToken 将在下次重启后生效

发现 1 个问题：
- [建议] 启用日志脱敏：logging.redactSensitive = "tools"
```

### 3.3 高级配置技巧

**技巧一：使用模板配置**

```bash
# 使用内置模板
openclaw config template --template minimal    # 最小配置模板
openclaw config template --template standard   # 标准配置模板
openclaw config template --template full       # 完整配置模板
```

**技巧二：配置差异对比**

```bash
# 对比当前配置与默认配置
openclaw config diff

# 输出示例：
--- 当前配置
+++ 默认配置
@@ -10,7 +10,7 @@
-    "temperature": 0.7
+    "temperature": 0.5
```

**技巧三：配置快照管理**

```bash
# 创建配置快照
openclaw config snapshot create "before-telegram-update"

# 列出快照
openclaw config snapshot list

# 恢复到快照
openclaw config snapshot restore "before-telegram-update"
```

---

## 第四部分：专家级配置架构（⭐⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 设计团队级别的配置管理规范
- [ ] 实现多环境配置的高效管理
- [ ] 构建插件配置的最佳实践
- [ ] 优化大规模部署的配置策略

### 4.1 团队配置管理最佳实践

**配置规范文档模板**：

```markdown
## 团队配置规范

### 1. 版本控制
- 配置文件应纳入版本控制
- 使用 `.gitignore` 排除敏感目录：`credentials/`、`*.log`
- 提交时使用 `git diff` 检查变更

### 2. 命名规范
- 渠道名称使用小写：`telegram`、`discord`、`slack`
- 模型名称使用完整 ID：`anthropic/claude-opus-4-5`
- 自定义配置使用前缀：`custom.*`

### 3. 敏感信息管理
- 禁止在配置文件中硬编码 API 密钥
- 使用环境变量或凭据存储
- CI/CD 环境中使用密钥管理服务

### 4. 变更流程
- 任何配置变更需经过代码审查
- 重大配置变更需记录变更日志
- 生产环境配置变更需在维护窗口进行
```

### 4.2 多环境配置策略

**环境矩阵**：

| 环境 | 用途 | 模型 | 日志级别 |
|------|------|------|----------|
| development | 本地开发 | claude-sonnet-4-5 | debug |
| staging | 测试环境 | claude-opus-4-5 | info |
| production | 生产环境 | claude-opus-4-5 | warn |

**实现方案**：

```json5
// ~/.openclaw/openclaw.json
{
  // 使用环境变量选择配置
  agents: {
    defaults: {
      // 通过环境变量动态选择工作区
      workspace: "${OPENCLAW_WORKSPACE:-~/.openclaw/workspace}",
      
      // 环境特定的模型配置
      model: {
        "(development|staging)": "anthropic/claude-sonnet-4-5",
        "production": "anthropic/claude-opus-4-5"
      }
    }
  },
  
  logging: {
    level: {
      "development": "debug",
      "staging": "info",
      "production": "warn"
    }
  }
}
```

### 4.3 插件配置体系

**插件配置最佳实践**：

```json5
{
  // 插件启用列表
  plugins: {
    allow: [
      "diagnostics-otel",
      "custom-plugin"
    ],
    
    // 插件特定配置
    entries: {
      "diagnostics-otel": {
        enabled: true,
        endpoint: "http://otel-collector:4318",
        protocol: "http/protobuf"
      }
    }
  }
}
```

**配置验证优先级**：

1. 插件默认配置
2. 用户配置覆盖
3. 环境变量覆盖

---

## 第五部分：故障排查与调试

### 常见问题与解决方案

**问题一：配置文件不被识别**

**症状**：修改配置后，行为没有变化

**排查步骤**：

```bash
# 1. 检查配置文件位置
ls -la ~/.openclaw/openclaw.json

# 2. 检查配置文件语法
openclaw doctor

# 3. 检查环境变量覆盖
echo $OPENCLAW_CONFIG_PATH

# 4. 查看启动时加载的配置
openclaw config get --verbose
```

**解决方案**：
- 确保配置文件在正确的位置
- 修复 JSON5 语法错误
- 检查是否有环境变量覆盖

**问题二：配置验证失败**

**症状**：`openclaw doctor` 报告配置错误

**排查步骤**：

```bash
# 查看详细错误信息
openclaw doctor --verbose

# 常见错误类型：
# 1. JSON 语法错误
# 2. 无效的配置值类型
# 3. 缺失必需的配置项
```

**解决方案**：

| 错误类型 | 解决方案 |
|----------|----------|
| JSON 语法错误 | 使用 JSON 验证工具检查 |
| 值类型错误 | 确保字符串使用引号，布尔值使用 true/false |
| 必需项缺失 | 添加缺失的配置项 |

**问题三：配置不生效**

**症状**：配置已修改但网关行为未变

**排查步骤**：

```bash
# 1. 重启网关使配置生效
openclaw gateway restart

# 2. 检查配置是否被正确加载
openclaw config get | grep <config-key>

# 3. 检查配置继承
openclaw config get --all
```

**解决方案**：
- 某些配置需要重启网关才能生效
- 检查是否有配置覆盖（CLI、环境变量）

---

## 练习与自检

### 基础技能自检

- [ ] 能够找到并编辑配置文件
- [ ] 能够添加新渠道配置
- [ ] 能够使用 CLI 查看和修改配置
- [ ] 能够运行配置验证

### 进阶技能自检

- [ ] 能够设计多环境配置策略
- [ ] 能够配置安全相关的选项
- [ ] 能够诊断配置相关问题
- [ ] 能够编写配置模板

### 专家技能自检

- [ ] 能够设计团队配置规范
- [ ] 能够优化大规模部署的配置管理
- [ ] 理解配置系统的设计原则
- [ ] 能够为新渠道设计配置方案

---

## 参考资料

### 核心文档

- [配置参考](/zh-CN/config/reference) - 所有配置项的详细说明
- [配置示例](/zh-CN/config/examples) - 常见场景的配置模板
- [故障排除](/zh-CN/help/troubleshooting) - 常见问题解决方案

### 进阶阅读

- [环境变量](/zh-CN/environment) - 环境变量配置详解
- [日志配置](/zh-CN/logging) - 日志记录配置
- [安全配置](/zh-CN/security) - 安全策略配置

### 相关命令

```bash
# 配置管理命令
openclaw config get     # 查看配置
openclaw config set    # 设置配置
openclaw config edit   # 编辑配置
openclaw doctor        # 诊断配置
```
