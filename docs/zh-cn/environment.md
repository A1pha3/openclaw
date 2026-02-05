---
summary: "OpenClaw 环境变量完全指南：加载机制、优先级规则、配置替换和安全最佳实践"
read_when:
  - 你需要理解 OpenClaw 加载环境变量的机制和优先级
  - 你在调试 API 密钥缺失问题
  - 你想掌握环境变量与配置文件的交互
  - 你在部署环境或配置敏感信息
title: "环境变量"
---

# 🔐 OpenClaw 环境变量完全指南

> **学习目标**：完成本章节学习后，你将能够深入理解 OpenClaw 的环境变量加载机制，掌握多层次的环境配置策略，理解配置与变量的交互关系，并能够安全高效地管理敏感凭据。

---

## 为什么要理解环境变量系统

在深入技术细节之前，我们需要先理解**为什么 OpenClaw 采用多层次的环境变量加载机制**。这个设计决策背后有深刻的考量：

### 设计目标

OpenClaw 的环境变量系统旨在解决以下核心问题：

1. **安全性**：敏感凭据不应硬编码在配置文件中
2. **灵活性**：支持多种凭据来源和注入方式
3. **可移植性**：环境变量便于在不同环境间迁移
4. **兼容性**：支持传统配置方式和现代最佳实践

### 设计演进历史

> **知识延伸**：OpenClaw 的环境变量系统经历了重要演进：
>
> - **v1.x**：仅支持进程环境变量，缺乏灵活性
> - **v2.x**：引入 dotenv 支持，但优先级规则混乱
> - **v3.x**（当前）：清晰的五层优先级体系，明确的覆盖规则

---

## 认知负荷管理：学习路径设计

```
环境变量系统认知金字塔：

                    ┌─────────────────────────┐
                    │   专家级：安全与审计     │  敏感信息管理、合规审计
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：配置替换       │  变量插值、环境感知配置
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：Shell 导入      │  登录 shell 集成
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：优先级规则      │  五层加载机制
                    └─────────────────────────┘
```

**建议学习路径**：
- 初学者：从优先级规则开始，理解基本加载机制
- 开发者：掌握配置块和环境变量替换
- 运维人员：重点关注 Shell 导入和安全最佳实践

---

## 第一部分：环境变量基础（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 OpenClaw 的五层环境变量加载机制
- [ ] 掌握各层的优先级顺序
- [ ] 解释「永不覆盖」规则的设计理由
- [ ] 定位常见的 API 密钥缺失问题

### 1.1 环境变量加载优先级

OpenClaw 采用**五层加载机制**，严格遵循「永不覆盖」规则。理解这个优先级体系是解决问题的关键：

**优先级图表**：

```
优先级顺序（高 → 低）

┌─────────────────────────────────────────────────────────────┐
│  1. 进程环境                                                 │
│     父 shell 或守护进程已有的环境变量                         │
│     优先级：最高（不被任何后续层覆盖）                         │
├─────────────────────────────────────────────────────────────┤
│  2. 当前目录 .env                                            │
│     项目根目录下的 dotenv 文件                               │
│     优先级：次高                                             │
├─────────────────────────────────────────────────────────────┤
│  3. 全局 .env                                                │
│     ~/.openclaw/.env                                        │
│     优先级：中等                                             │
├─────────────────────────────────────────────────────────────┤
│  4. 配置 env 块                                              │
│     openclaw.json 中的 env 配置                             │
│     优先级：较低                                             │
├─────────────────────────────────────────────────────────────┤
│  5. Shell 环境导入                                          │
│     登录 shell 导入（可选）                                  │
│     优先级：最低（仅填充缺失项）                             │
└─────────────────────────────────────────────────────────────┘
```

**「永不覆盖」规则详解**：

这是 OpenClaw 最重要的设计原则之一。**一旦某个环境变量在任何一层被设置，后续所有层都不会覆盖它的值**。

**示例说明**：

```bash
# 假设进程环境中已有 ANTHROPIC_API_KEY
export ANTHROPIC_API_KEY="process-key"

# 此时以下配置都不会覆盖它：
# 1. .env 文件中的 ANTHROPIC_API_KEY 被忽略
# 2. openclaw.json 中的 env 配置被忽略
# 3. Shell 导入的 ANTHROPIC_API_KEY 被忽略
```

**为什么采用这个规则**：

1. **安全性**：防止配置文件意外覆盖已设置的安全凭据
2. **可预测性**：调试时可以确定变量的最终来源
3. **灵活性**：可以在不同级别控制变量值

### 1.2 各层详细说明

**第 1 层：进程环境**

这是最高优先级的环境变量来源，通常来自：
- 父 shell 的 `export` 语句
- 守护进程配置（如 systemd 的 EnvironmentFile）
- Docker 容器的环境变量
- IDE 的运行配置

**诊断方法**：

```bash
# 查看 Gateway 进程的环境变量
cat /proc/$(pgrep -f openclaw-gateway)/environ | tr '\0' '\n' | grep API_KEY

# 或使用 ptrace（需要权限）
sudo cat /proc/$(pgrep -f openclaw-gateway)/environ | tr '\0' '\n'
```

**第 2 层：当前目录 .env**

当 Gateway 从特定目录运行时，会加载该目录下的 `.env` 文件：

```bash
# 从项目目录运行
cd /path/to/openclaw
OPENCLAW_STATE_DIR=. ./node_modules/.bin/openclaw gateway

# 此时会加载：
# /path/to/openclaw/.env
```

**第 3 层：全局 .env**

OpenClaw 专用的全局环境变量文件：

```bash
# 位置
~/.openclaw/.env

# 或通过环境变量指定
OPENCLAW_STATE_DIR=/custom/path
# 此时加载：/custom/path/.env
```

**文件格式**：

```bash
# ~/.openclaw/.env
ANTHROPIC_API_KEY=sk-ant-your-key
OPENROUTER_API_KEY=sk-or-your-key
TELEGRAM_BOT_TOKEN=123456:your-token

# 注释被支持
# 这是一条注释
ANOTHER_KEY=value  # 行内注释
```

**第 4 层：配置 env 块**

在 `openclaw.json` 中直接定义环境变量：

```json5
{
  env: {
    // 直接定义
    OPENROUTER_API_KEY: "sk-or-...",
    
    // 使用 vars 对象（历史兼容）
    vars: {
      GROQ_API_KEY: "gsk-...",
    }
  }
}
```

**第 5 层：Shell 环境导入**

从登录 shell 导入环境变量（仅当启用时）：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

**关键特性**：Shell 导入**仅填充缺失的预期密钥**。这意味着：
- 如果某个变量已存在，不会被覆盖
- 如果变量缺失，会从 shell 导入

### 1.3 故障排查：API 密钥缺失

**典型问题**：

```
错误信息：No API key found for provider 'anthropic'
```

**排查流程**：

```bash
# 1. 检查进程环境
env | grep ANTHROPIC

# 2. 检查 .env 文件
cat ~/.openclaw/.env | grep ANTHROPIC

# 3. 检查配置文件
openclaw config get env

# 4. 启用 Shell 导入测试
OPENCLAW_LOAD_SHELL_ENV=1 openclaw models list

# 5. 查看详细加载信息
OPENCLAW_DEBUG_ENV=1 openclaw gateway
```

**常见原因**：

| 原因 | 解决方案 |
|------|----------|
| API 密钥未设置 | 在对应层级添加密钥 |
| 密钥拼写错误 | 检查变量名拼写 |
| 密钥已过期 | 从提供商获取新密钥 |
| 优先级问题 | 确保密钥在最高优先级的层设置 |

---

## 第二部分：高级配置（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 掌握 Shell 环境导入的详细配置
- [ ] 理解环境变量替换语法
- [ ] 设计安全的环境变量管理策略
- [ ] 优化环境变量加载性能

### 2.1 Shell 环境导入详解

**启用方式**：

```json5
// 方式一：配置文件
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}

// 方式二：环境变量
OPENCLAW_LOAD_SHELL_ENV=1
OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000
```

**工作原理**：

```
┌─────────────────────────────────────────────────────────┐
│          Shell 环境导入流程                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 启动子进程执行登录 shell                             │
│     （bash -l、zsh -l 等）                              │
│           ↓                                              │
│  2. 加载 shell 初始化文件                               │
│     ~/.bashrc、~/.zshrc、~/.profile 等                  │
│           ↓                                              │
│  3. 收集所有导出的环境变量                              │
│           ↓                                              │
│  4. 与 OpenClaw 的「预期密钥」列表对比                  │
│           ↓                                              │
│  5. 仅导入缺失的密钥（永不覆盖已存在的值）               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**超时控制**：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      // 超时时间（毫秒），防止 shell 挂起
      timeoutMs: 15000
    }
  }
}
```

**预期密钥列表**：

OpenClaw 会尝试导入以下类别的密钥：

| 类别 | 前缀/模式 | 示例 |
|------|----------|------|
| Anthropic | `*ANTHROPIC*` | `ANTHROPIC_API_KEY` |
| OpenAI | `*OPENAI*` | `OPENAI_API_KEY` |
| Telegram | `*TELEGRAM*` | `TELEGRAM_BOT_TOKEN` |
| Discord | `*DISCORD*` | `DISCORD_BOT_TOKEN` |
| Google | `*GOOGLE*` | `GOOGLE_CLIENT_ID` |
| 自定义 | 配置中指定的键 | `CUSTOM_API_KEY` |

**性能影响**：

Shell 导入会引入额外的启动延迟：

| 配置 | 典型延迟 | 适用场景 |
|------|---------|---------|
| 禁用 | ~0ms | 生产环境，已使用其他方式配置 |
| 启用（默认） | 10-50ms | 开发环境，快速迭代 |
| 超时设置 | ≤ timeoutMs | 防止无限等待 |

### 2.2 环境变量替换语法

**基本语法**：

在 `openclaw.json` 中使用 `${VAR_NAME}` 语法引用环境变量：

```json5
{
  models: {
    providers: {
      "anthropic": {
        apiKey: "${ANTHROPIC_API_KEY}"
      },
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
        endpoint: "https://gateway.vercel.ai"
      }
    }
  }
}
```

**语法变体**：

```json5
// 基本引用
{ "apiKey": "${ANTHROPIC_API_KEY}" }

// 带默认值的引用
{ "apiKey": "${ANTHROPIC_API_KEY:-default-key}" }

// 条件引用（如果变量不存在则省略）
{ "apiKey": "${?ANTHROPIC_API_KEY}" }
```

**处理规则**：

| 语法 | 行为 |
|------|------|
| `${VAR}` | 如果 VAR 不存在，会报错 |
| `${VAR:-default}` | 如果 VAR 不存在，使用默认值 |
| `${VAR:-}` | 如果 VAR 不存在，使用空字符串 |
| `${?VAR}` | 如果 VAR 不存在，整个键被省略 |

**使用场景**：

```json5
{
  // 场景一：可选配置
  gateway: {
    customOption: "${?CUSTOM_OPTION}"
  },

  // 场景二：环境感知的端点
  models: {
    providers: {
      "custom": {
        endpoint: "${CUSTOM_ENDPOINT:-https://default.example.com}"
      }
    }
  },

  // 场景三：密钥回退机制
  models: {
    providers: {
      "openai": {
        apiKey: "${OPENAI_API_KEY:-${ANTHROPIC_API_KEY:-}}"
      }
    }
  }
}
```

### 2.3 安全最佳实践

**⚠️ 核心原则：敏感信息隔离**

```markdown
## 敏感信息管理 Checklist

### 不应该做的
- ❌ 不要在 openclaw.json 中硬编码 API 密钥
- ❌ 不要将密钥提交到版本控制
- ❌ 不要在日志中输出密钥（即使脱敏也可能被恢复）

### 应该做的
- ✅ 使用 ~/.openclaw/.env 存储密钥
- ✅ 使用凭据存储（credentials/ 目录）
- ✅ 定期轮换 API 密钥
- ✅ 使用环境变量注入（容器化部署）
```

**凭据存储目录结构**：

```
~/.openclaw/credentials/
├── anthropic/
│   ├── api-key.json
│   └── setup-token.json
├── openai/
│   └── api-key.json
└── telegram/
    └── bot-token.json
```

**文件格式**：

```json
{
  "key": "sk-ant-api-key",
  "expiresAt": "2025-12-31T23:59:59Z",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

**容器化部署策略**：

```yaml
# docker-compose.yml
services:
  openclaw:
    image: openclaw/openclaw:latest
    environment:
      # 通过环境变量注入
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      # 挂载凭据目录
      - ~/.openclaw/credentials:/home/openclaw/.openclaw/credentials
      - ~/.openclaw/.env:/home/openclaw/.openclaw/.env:ro
```

---

## 第三部分：专家级策略（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 设计多环境配置策略
- [ ] 实现环境变量的审计和监控
- [ ] 构建 CI/CD 中的环境变量管理流程
- [ ] 优化大规模部署的变量管理

### 3.1 多环境配置策略

**环境矩阵设计**：

| 环境 | 凭据来源 | 特点 |
|------|---------|------|
| 本地开发 | ~/.openclaw/.env | 灵活、可修改 |
| CI/CD | CI 环境变量 | 隔离、安全 |
| Staging | 密钥管理服务 | 接近生产 |
| Production | 密钥管理服务 | 最高安全级别 |

**实现方案**：

```bash
# ~/.bashrc 或 ~/.zshrc
# 根据目录自动切换环境

export OPENCLAW_ENVIRONMENT=$(basename $(pwd))

if [ "$OPENCLAW_ENVIRONMENT" = "prod" ]; then
    export ANTHROPIC_API_KEY=$(op read "op://Production/anthropic/api-key")
    export OPENAI_API_KEY=$(op read "op://Production/openai/api-key")
elif [ "$OPENCLAW_ENVIRONMENT" = "staging" ]; then
    export ANTHROPIC_API_KEY=$(op read "op://Staging/anthropic/api-key")
else
    # 开发环境使用本地 .env
    source ~/.openclaw/.env
fi
```

**配置文件模板**：

```json5
{
  // 环境标识
  _meta: {
    environment: "${OPENCLAW_ENVIRONMENT:-development}",
    version: "1.0.0"
  },

  // 通用配置（所有环境共享）
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace"
    }
  },

  // 环境特定配置
  logging: {
    level: {
      "development": "debug",
      "staging": "info",
      "production": "warn"
    }
  }
}
```

### 3.2 环境变量审计

**审计日志**：

```bash
# 启用环境变量审计日志
OPENCLAW_AUDIT_ENV=1 openclaw gateway
```

**审计输出格式**：

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "event": "env_loaded",
  "source": "process",
  "variable": "ANTHROPIC_API_KEY",
  "status": "set"
}
{
  "timestamp": "2024-01-15T10:30:00.050Z",
  "event": "env_loaded",
  "source": "global_dotenv",
  "variable": "OPENAI_API_KEY",
  "status": "skipped",
  "reason": "already_set_by_process"
}
```

**敏感变量检测**：

```bash
# 扫描配置文件中的潜在敏感信息
grep -rn "api.*key\|token\|secret\|password" ~/.openclaw/openclaw.json | \
  grep -v "env\|vars" | \
  grep -v "example\|placeholder"
```

### 3.3 CI/CD 集成

**GitHub Actions 示例**：

```yaml
name: OpenClaw CI

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      OPENCLAW_ENVIRONMENT: ci
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run tests
        run: pnpm test
      
      - name: Verify environment
        run: |
          echo "Environment variables loaded correctly"
          openclaw doctor
```

**密钥轮换流程**：

```bash
#!/bin/bash
# rotate-api-key.sh - API 密钥轮换脚本

set -e

PROVIDER=$1
NEW_KEY=$2

if [ -z "$PROVIDER" ] || [ -z "$NEW_KEY" ]; then
    echo "用法: $0 <provider> <new-key>"
    echo "示例: $0 anthropic sk-ant-new-key"
    exit 1
fi

# 1. 更新环境变量
export ${PROVIDER^^}_API_KEY="$NEW_KEY"

# 2. 更新 .env 文件（如果存在）
if [ -f ~/.openclaw/.env ]; then
    sed -i "s/${PROVIDER^^}_API_KEY=.*/${PROVIDER^^}_API_KEY=$NEW_KEY/" ~/.openclaw/.env
fi

# 3. 更新凭据存储（如果使用）
if [ -f ~/.openclaw/credentials/$PROVIDER/api-key.json ]; then
    cat > ~/.openclaw/credentials/$PROVIDER/api-key.json << EOF
{
  "key": "$NEW_KEY",
  "rotatedAt": "$(date -Iseconds)",
  "rotatedBy": "script"
}
EOF
fi

# 4. 验证新密钥
echo "正在验证新密钥..."
openclaw models list --provider $PROVIDER

echo "密钥轮换完成"
```

---

## 第四部分：故障排查速查

### 快速诊断命令

```bash
# 1. 完整环境诊断
openclaw doctor --env

# 2. 查看所有加载的环境变量（脱敏后）
OPENCLAW_DEBUG_ENV=1 openclaw gateway 2>&1 | grep -v "KEY\|TOKEN"

# 3. 测试特定密钥
export TEST_API_KEY=your-key
openclaw config set env.vars.TEST_API_KEY="test-value"
echo $TEST_API_KEY
echo $TEST_API_KEY | md5sum

# 4. 检查 Shell 导入
OPENCLAW_LOAD_SHELL_ENV=1 OPENCLAW_SHELL_ENV_TIMEOUT_MS=5000 openclaw models list
```

### 常见问题与解决方案

**问题一：密钥被意外覆盖**

**症状**：配置的 API 密钥不生效

**排查**：

```bash
# 检查进程环境
env | grep ANTHROPIC

# 查找覆盖源
OPENCLAW_DEBUG_ENV=1 openclaw gateway 2>&1 | grep "ANTHROPIC_API_KEY"
```

**解决**：移除更高优先级的设置，或调整策略

**问题二：Shell 导入超时**

**症状**：`OPENCLAW_LOAD_SHELL_ENV=1` 导致启动缓慢

**排查**：

```bash
# 测试 shell 加载时间
time bash -l -c 'export'

# 检查 .bashrc/.zshrc 中的耗时操作
grep -n "sleep\|wait\|curl.*timeout" ~/.bashrc ~/.zshrc
```

**解决**：优化 shell 初始化脚本，设置合理的超时

**问题三：变量替换失败**

**症状**：配置中的 `${VAR}` 未被替换

**排查**：

```bash
# 检查变量是否存在
echo $VAR_NAME

# 测试替换语法
openclaw config get models.providers.custom.endpoint
```

**解决**：确保变量已设置，或使用默认值语法

---

## 练习与自检

### 基础技能自检

- [ ] 能够列出五层环境变量加载优先级
- [ ] 理解「永不覆盖」规则的含义
- [ ] 能够在 .env 文件中配置 API 密钥
- [ ] 能够诊断密钥缺失问题

### 进阶技能自检

- [ ] 能够配置 Shell 环境导入
- [ ] 掌握 `${VAR:-default}` 语法
- [ ] 理解凭据存储与 .env 的区别
- [ ] 能够设计多环境配置策略

### 专家技能自检

- [ ] 能够构建密钥轮换流程
- [ ] 掌握 CI/CD 环境变量集成
- [ ] 理解审计日志的分析方法
- [ ] 能够优化大规模部署的变量管理

---

## 参考资料

### 核心文档

- [配置概述](/zh-CN/config) - 配置系统详解
- [配置参考](/zh-CN/config/reference) - 完整配置项
- [故障排除](/zh-CN/help/troubleshooting) - 常见问题

### 相关命令

```bash
# 环境变量诊断
openclaw doctor --env       # 完整诊断
OPENCLAW_DEBUG_ENV=1 ...   # 调试输出

# 凭据管理
openclaw credentials list   # 列出凭据
openclaw credentials add   # 添加凭据

# 模型配置
openclaw models list       # 列出可用模型
openclaw models auth      # 凭据认证
```

### 扩展阅读

- [1Password CLI 集成](https://developer.1password.com/docs/cli)
- [环境变量安全最佳实践](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [12-Factor App: Config](https://12factor.net/config)
