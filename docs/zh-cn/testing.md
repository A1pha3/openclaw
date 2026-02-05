---
summary: "OpenClaw 测试完全指南：单元测试、端到端测试、实时测试、Docker 运行器和模型矩阵的完整测试体系"
read_when:
  - 你需要理解 OpenClaw 的测试体系和最佳实践
  - 你想在本地或 CI 中运行测试
  - 你需要为模型和提供商问题添加回归测试
  - 你在调试网关和代理行为
title: "测试指南"
---

# 🧪 OpenClaw 测试完全指南

> **学习目标**：完成本章节学习后，你将能够深入理解 OpenClaw 的三层测试体系，掌握从单元测试到实时测试的完整方法论，理解测试策略的选择依据，并能够为各种场景设计和实施有效的测试方案。

---

## 为什么要建立测试体系

在深入技术细节之前，我们需要先理解**为什么 OpenClaw 需要如此复杂的测试体系**。作为连接多渠道、多模型的智能助手系统，OpenClaw 的复杂性决定了测试不是可选的「额外工作」，而是确保系统质量的「基础设施」。

### 测试金字塔

OpenClaw 采用经典的**测试金字塔**模型：

```
┌─────────────────────────────────────────────────────────────────┐
│                      测试金字塔                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                         ▲                                       │
│                    ┌─────────────────────────┐                 │
│                    │      实时测试 (Live)     │   比例：5-10%   │
│                    │  真实提供商 + 真实模型    │                 │
│                    │  最高信任度、最高成本    │                 │
│                    └───────────┬─────────────┘                 │
│                                │                                 │
│                    ┌───────────┴─────────────┐                 │
│                    │    端到端测试 (E2E)     │   比例：15-25%   │
│                    │   多实例 + 网络交互     │                 │
│                    │   较高信任度、中等成本  │                 │
│                    └───────────┬─────────────┘                 │
│                                │                                 │
│                    ┌───────────┴─────────────┐                 │
│                    │     单元/集成测试         │   比例：65-80%  │
│                    │   纯单元 + 进程内集成     │                 │
│                    │   最高频率、最低成本     │                 │
│                    └─────────────────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| 测试类型 | 频率 | 速度 | 隔离度 | 信任度 | 成本 |
|----------|------|------|--------|--------|------|
| **单元测试** | 每提交 | 毫秒级 | 完全隔离 | 低 | $ |
| **集成测试** | 每提交 | 秒级 | 进程隔离 | 中 | $$ |
| **E2E 测试** | 每 PR | 分钟级 | 环境隔离 | 高 | $$$ |
| **实时测试** | 按需 | 分钟级 | 无隔离 | 最高 | $$$$ |

### OpenClaw 测试体系的设计目标

1. **快速反馈**：单元测试毫秒级反馈，不阻塞开发流程
2. **安全回归**：集成测试确保核心功能不受影响
3. **端到端验证**：E2E 测试验证完整用户流程
4. **真实验证**：实时测试验证与真实服务的集成

---

## 认知负荷管理：学习路径设计

```
测试技能金字塔：

                    ┌─────────────────────────┐
                    │   专家级：测试策略设计   │  CI/CD 集成、测试覆盖优化
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：实时测试       │  模型矩阵、探针测试
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：E2E 测试       │  网关冒烟测试、CI 集成
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：单元测试       │  基础测试编写、运行
                    └─────────────────────────┘
```

**建议学习路径**：
- 新手：从单元测试开始，理解测试基础
- 开发者：掌握集成测试和 E2E 测试
- 高级用户：深入实时测试和模型验证
- 架构师：设计测试策略和 CI/CD 流程

---

## 第一部分：快速开始（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 OpenClaw 的测试命令
- [ ] 运行基础测试套件
- [ ] 理解测试结果的含义
- [ ] 掌握基本的测试调试方法

### 1.1 快速命令参考

**常用测试命令**：

```bash
# 完整门禁测试（推送前必跑）
pnpm lint && pnpm build && pnpm test

# 仅运行单元/集成测试
pnpm test

# 运行单元测试并生成覆盖率
pnpm test:coverage

# 运行端到端测试
pnpm test:e2e

# 运行实时测试（需要真实凭据）
pnpm test:live

# 运行特定测试文件
pnpm test src/gateway/gateway.test.ts

# 运行特定测试
pnpm test -- -t "test name"
```

**测试配置位置**：

| 文件 | 用途 |
|------|------|
| `vitest.config.ts` | 单元/集成测试配置 |
| `vitest.e2e.config.ts` | E2E 测试配置 |
| `vitest.live.config.ts` | 实时测试配置 |

### 1.2 测试文件结构

```
src/
├── *.test.ts           # 单元/集成测试
├── *.e2e.test.ts       # 端到端测试
├── *.live.test.ts      # 实时测试
└── gateway/
    ├── gateway.test.ts              # Gateway 基础测试
    ├── gateway.tool-calling.test.ts # 工具调用测试
    └── gateway-models.live.test.ts  # 模型实时测试
```

### 1.3 测试输出解读

**测试结果示例**：

```
Test Files    15 passed, 15 total
Tests         247 passed, 247 total
Time          12.45s
```

**结果含义**：

| 指标 | 说明 | 期望值 |
|------|------|--------|
| Test Files | 测试文件数 | 根据变更调整 |
| Tests | 测试用例数 | 根据变更调整 |
| Time | 运行时间 | < 30s（单元测试） |
| Coverage | 代码覆盖率 | > 70% |

---

## 第二部分：测试套件详解（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解各层测试的范围和职责
- [ ] 选择合适的测试类型
- [ ] 理解测试的预期行为
- [ ] 配置测试环境

### 2.1 单元/集成测试

**范围**：

| 测试类型 | 说明 | 示例 |
|----------|------|------|
| **纯单元测试** | 测试单个函数/类 | `add(2, 3) === 5` |
| **进程内集成** | 测试组件交互 | Gateway 认证流程 |
| **回归测试** | 已知 Bug 的确定性测试 | 修复后的 Bug 重现 |

**配置**：

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    // 测试环境
    environment: 'node',
    
    // 全局测试超时
    testTimeout: 10000,
    
    // 并发配置
    pool: 'forks',
    poolOptions: {
      forks: {
        maxForks: 16  // 最大并发数
      }
    },
    
    // 覆盖率配置
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 70,
        functions: 70,
        branches: 70,
        statements: 70
      }
    }
  }
})
```

**预期行为**：

```
┌─────────────────────────────────────────────────────────┐
│              单元/集成测试预期行为                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ✅ 在 CI 中运行                                        │
│   ✅ 不需要真实 API 密钥                                 │
│   ✅ 快速且稳定（< 30s）                                 │
│   ✅ 确定性结果（无随机失败）                              │
│   ✅ 隔离性好（不影响其他测试）                            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 2.2 端到端测试（E2E）

**范围**：

```
┌─────────────────────────────────────────────────────────────┐
│                    E2E 测试范围                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. 多实例 Gateway 行为                                      │
│      - 实例间通信                                            │
│      - 状态同步                                              │
│                                                              │
│   2. WebSocket/HTTP 表面                                    │
│      - 连接建立                                              │
│      - 消息传递                                              │
│      - 断开处理                                              │
│                                                              │
│   3. 节点配对                                               │
│      - 发现流程                                              │
│      - 认证流程                                              │
│                                                              │
│   4. 网络交互                                               │
│      - 端口绑定                                              │
│      - 连接池                                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**配置**：

```typescript
// vitest.e2e.config.ts
export default defineConfig({
  test: {
    environment: 'node',
    
    // E2E 测试专用配置
    e2e: {
      // 测试间等待时间
      waitForDeployTimeout: 60000,
      
      // 全局测试超时
      testTimeout: 120000
    },
    
    // 更严格的资源限制
    pool: 'threads',
    poolOptions: {
      threads: {
        maxThreads: 4,
        minThreads: 1
      }
    }
  }
})
```

**预期行为**：

```
┌─────────────────────────────────────────────────────────┐
│              E2E 测试预期行为                              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ✅ 在 CI 中运行（当启用时）                              │
│   ✅ 不需要真实 API 密钥                                   │
│   ✅ 移动部件较多（可能较慢）                               │
│   ✅ 需要环境隔离                                          │
│   ✅ 可能受网络影响                                        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 2.3 实时测试（Live）

**核心设计**：

实时测试是 OpenClaw 测试体系的**最后一公里**，验证与真实服务的集成：

```
┌─────────────────────────────────────────────────────────────┐
│                    实时测试设计                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   设计原则：                                                  │
│   1. 按需运行（不强制在每次提交时）                           │
│   2. 真实凭据（使用 ~/.profile 或配置文件）                   │
│   3. 成本意识（注意 API 配额和费用）                         │
│   4. 失败容忍（网络和服务可能不稳定）                         │
│                                                              │
│   ⚠️ 重要：实时测试不保证在 CI 中稳定                         │
│   - 真实网络延迟                                             │
│   - 提供商策略变化                                           │
│   - 速率限制                                                 │
│   - 服务中断                                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**启用方式**：

```bash
# 方法一：环境变量
OPENCLAW_LIVE_TEST=1 pnpm test:live

# 方法二：直接配置
pnpm test:live
#（自动设置 OPENCLAW_LIVE_TEST=1）
```

**凭据发现**：

```
┌─────────────────────────────────────────────────────────────┐
│                    实时测试凭据发现                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   优先级（与 CLI 相同）：                                     │
│   1. 进程环境变量                                            │
│   2. ~/.profile 导出                                         │
│   3. 配置文件存储                                             │
│   4. 交互式输入（如果未找到）                                 │
│                                                              │
│   建议：确保 ~/.profile 包含你的 API 密钥                     │
│   export ANTHROPIC_API_KEY="sk-ant-..."                      │
│   export OPENAI_API_KEY="sk-..."                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第三部分：实时测试深入（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解实时测试的两层架构
- [ ] 配置模型选择和测试探针
- [ ] 理解 Anthropic 设置令牌测试
- [ ] 掌握 CLI 后端测试

### 3.1 实时测试的两层架构

**设计理念**：

```
┌─────────────────────────────────────────────────────────────┐
│                  实时测试两层架构                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Layer 1：直接模型测试                                       │
│   ┌─────────────────────────────────────────────────────┐    │
│   │  目标：验证提供商/模型 API 是否正常                   │    │
│   │  范围：API 调用、响应解析、错误处理                   │    │
│   │  文件：src/agents/models.profiles.live.test.ts     │    │
│   │  速度：快速（无需完整网关栈）                         │    │
│   └─────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           ▼                                  │
│   Layer 2：网关冒烟测试                                       │
│   ┌─────────────────────────────────────────────────────┐    │
│   │  目标：验证完整网关 + 代理管道是否正常                │    │
│   │  范围：会话、历史、工具调用、工具调用流程              │    │
│   │  文件：src/gateway/gateway-models.profiles.live.test.ts│    │
│   │  速度：较慢（完整栈）                                 │    │
│   └─────────────────────────────────────────────────────┘    │
│                                                              │
│   分离原因：                                                  │
│   - 快速定位问题层级（API vs 网关）                           │
│   - 减少调试时间                                             │
│   - 降低测试成本                                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 模型选择与环境配置

**模型选择控制**：

```bash
# 使用现代模型列表（推荐）
OPENCLAW_LIVE_MODELS=modern pnpm test:live

# 运行所有测试模型
OPENCLAW_LIVE_MODELS=all pnpm test:live

# 自定义模型列表
OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5" pnpm test:live

# 指定提供商
OPENCLAW_LIVE_PROVIDERS="google,openai,anthropic" pnpm test:live
```

**提供商选择**：

| 提供商 | ID | 特点 |
|--------|-----|------|
| OpenAI | `openai` | 稳定、功能全 |
| Anthropic | `anthropic` | 高质量、长上下文 |
| Google | `google` | Gemini 系列 |
| OpenRouter | `openrouter` | 模型聚合 |
| Z.AI | `zai` | GLM 系列 |

**密钥来源控制**：

```bash
# 仅使用配置文件存储的密钥
OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1 pnpm test:live

# 允许环境变量
pnpm test:live
```

### 3.3 测试探针详解

**工具调用探针**：

```
┌─────────────────────────────────────────────────────────────┐
│                    工具调用探针                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Read 探针：                                                │
│   1. 在工作区写入随机数文件                                  │
│   2. 要求代理读取并回显内容                                  │
│   3. 验证内容正确性                                          │
│                                                              │
│   Exec+Read 探针：                                          │
│   1. 要求代理执行命令写入随机数                              │
│   2. 读取并回显内容                                          │
│   3. 验证命令执行和文件读取                                  │
│                                                              │
│   图像探针：                                                 │
│   1. 生成带随机码的 PNG 图像                                 │
│   2. 作为附件发送给代理                                      │
│   3. 要求代理识别并返回码                                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**探针测试示例**：

```typescript
// Read 探针测试
test('read probe', async () => {
  // 1. 写入随机数文件
  const randomCode = generateRandomCode();
  await writeFile('probe-test.txt', randomCode);
  
  // 2. 发送读取请求
  const response = await sendToAgent(`Read probe-test.txt and echo: ${randomCode}`);
  
  // 3. 验证响应
  expect(response).toContain(randomCode);
});
```

### 3.4 Anthropic 设置令牌测试

**测试范围**：

```
┌─────────────────────────────────────────────────────────────┐
│                  Anthropic 设置令牌测试                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   目标：                                                      │
│   验证 Claude Code CLI 设置令牌能够正常完成 Anthropic 提示    │
│                                                              │
│   文件：src/agents/anthropic.setup-token.live.test.ts       │
│                                                              │
│   启用方式：                                                  │
│   OPENCLAW_LIVE_SETUP_TOKEN=1 pnpm test:live               │
│                                                              │
│   令牌来源：                                                  │
│   - 配置文件：OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=profile-id   │
│   - 原始令牌：OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-...    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.5 CLI 后端测试

**测试范围**：

```
┌─────────────────────────────────────────────────────────────┐
│                    CLI 后端测试                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   目标：                                                      │
│   验证网关 + 代理管道使用本地 CLI 后端正常工作                  │
│                                                              │
│   文件：src/gateway/gateway-cli-backend.live.test.ts        │
│                                                              │
│   启用方式：                                                  │
│   OPENCLAW_LIVE_CLI_BACKEND=1 pnpm test:live               │
│                                                              │
│   默认配置：                                                  │
│   - 模型：claude-cli/claude-sonnet-4-5                      │
│   - 命令：claude                                             │
│   - 参数：["-p", "--output-format", "json"]                 │
│                                                              │
│   覆盖选项：                                                  │
│   - 模型：OPENCLAW_LIVE_CLI_BACKEND_MODEL                   │
│   - 命令：OPENCLAW_LIVE_CLI_BACKEND_COMMAND                 │
│   - 图像探针：OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第四部分：模型矩阵（⭐⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解推荐测试的模型列表
- [ ] 根据场景选择测试模型
- [ ] 掌握模型发现命令
- [ ] 理解不同提供商的特点

### 4.1 推荐模型测试矩阵

**现代冒烟集（工具调用 + 图像）**：

| 提供商 | 模型 ID | 用途 | 必需性 |
|--------|---------|------|--------|
| OpenAI | `openai/gpt-5.2` | 主要测试 | ✅ 推荐 |
| OpenAI | `openai-codex/gpt-5.2` | Codex 测试 | ⭐ 可选 |
| Anthropic | `anthropic/claude-opus-4-5` | 主要测试 | ✅ 推荐 |
| Anthropic | `anthropic/claude-sonnet-4-5` | 备用测试 | ⭐ 可选 |
| Google | `google/gemini-3-pro-preview` | Gemini API | ✅ 推荐 |
| Google | `google/gemini-3-flash-preview` | 快速测试 | ✅ 推荐 |
| Google Antigravity | `google-antigravity/claude-opus-4-5-thinking` | OAuth 测试 | ⭐ 可选 |
| Z.AI | `zai/glm-4.7` | 国产模型 | ⭐ 可选 |
| MiniMax | `minimax/minimax-m2.1` | 国产模型 | ⭐ 可选 |

**推荐测试命令**：

```bash
# 完整冒烟测试
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts
```

### 4.2 模型发现

**查看可用模型**：

```bash
# 列出所有可用模型
openclaw models list

# JSON 格式输出（程序处理）
openclaw models list --json

# 按提供商筛选
openclaw models list --provider anthropic

# 筛选支持图像的模型
openclaw models list | grep -i image
```

### 4.3 提供商特点对比

| 提供商 | 优势 | 劣势 | 适用场景 |
|--------|------|------|----------|
| **OpenAI** | 稳定、功能全、文档完善 | 成本较高 | 通用场景 |
| **Anthropic** | 长上下文、高质量 | 配额限制 | 复杂推理 |
| **Google** | 多模态优秀、价格低 | 快速发展中 | 图像任务 |
| **OpenRouter** | 模型多样、灵活 | 延迟较高 | 实验性测试 |

---

## 第五部分：Docker 运行器

### 学习目标

完成本节学习后，你将能够：
- [ ] 使用 Docker 运行测试
- [ ] 配置 Docker 测试环境
- [ ] 理解 Docker 测试的价值
- [ ] 解决 Docker 相关问题

### 5.1 Docker 测试命令

| 命令 | 用途 | 说明 |
|------|------|------|
| `pnpm test:docker:live-models` | 直接模型测试 | 挂载配置目录 |
| `pnpm test:docker:live-gateway` | 网关测试 | 完整栈测试 |
| `pnpm test:docker:onboard` | 向导测试 | TTY 交互测试 |
| `pnpm test:docker:gateway-network` | 网络测试 | 双容器测试 |
| `pnpm test:docker:plugins` | 插件测试 | 扩展测试 |

**环境变量配置**：

```bash
# 配置目录（默认：~/.openclaw）
OPENCLAW_CONFIG_DIR=/path/to/config pnpm test:docker:live-gateway

# 工作区目录（默认：~/.openclaw/workspace）
OPENCLAW_WORKSPACE_DIR=/path/to/workspace pnpm test:docker:live-gateway

# Profile 文件
OPENCLAW_PROFILE_FILE=~/.profile pnpm test:docker:live-gateway

# 限制测试范围
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:docker:live-gateway

# 强制使用配置文件密钥
OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1 pnpm test:docker:live-gateway
```

### 5.2 Docker 测试的价值

```
┌─────────────────────────────────────────────────────────────┐
│                  Docker 测试的优势                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. 环境一致性                                              │
│      - 标准化测试环境                                         │
│      - 消除「在我机器上能运行」问题                            │
│                                                              │
│   2. 隔离性                                                  │
│      - 不影响主机配置                                         │
│      - 干净的依赖环境                                         │
│                                                              │
│   3. 可重复性                                                │
│      - 多次运行结果一致                                       │
│      - 便于问题复现                                           │
│                                                              │
│   4. CI/CD 集成                                              │
│      - 无缝集成 CI 流程                                      │
│      - 标准化部署验证                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第六部分：添加回归测试

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解何时需要添加回归测试
- [ ] 掌握回归测试的编写方法
- [ ] 理解测试分层策略
- [ ] 优化测试覆盖

### 6.1 何时添加回归测试

**触发条件**：

```
┌─────────────────────────────────────────────────────────────┐
│                  回归测试触发条件                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ✅ 发现并修复了提供商/模型 问题                             │
│   ✅ 添加了新的功能模块                                       │
│   ✅ 重构了核心逻辑                                          │
│   ✅ 修复了用户报告的 Bug                                     │
│   ✅ 变更影响多个组件                                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 测试分层策略

```
┌─────────────────────────────────────────────────────────────┐
│                  回归测试分层策略                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   问题类型              →  测试层  →  选择理由               │
│   ──────────────────────────────────────────────────────── │
│   提供商请求转换问题    →  直接模型 →  最小依赖              │
│   网关管道问题         →  网关测试 →  验证完整栈             │
│   集成问题             →  E2E 测试 →  端到端验证              │
│   速率限制/认证        →  实时测试 →  真实环境验证           │
│                                                              │
│   原则：                                                    │
│   - 优先选择依赖最少、速度最快的测试层                        │
│   - 仅当需要真实环境时才使用实时测试                          │
│   - 保持测试套件的平衡                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 测试模板

**单元测试模板**：

```typescript
// src/utils/math.test.ts
import { describe, it, expect } from 'vitest';
import { add } from '../math';

describe('math utils', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });
    
    it('should handle negative numbers', () => {
      expect(add(-1, 1)).toBe(0);
    });
  });
});
```

**实时回归测试模板**：

```typescript
// src/providers/openai-format.live.test.ts
import { test, expect } from 'vitest';

test('openai response format regression', async () => {
  // 设置测试凭据
  process.env.OPENAI_API_KEY = process.env.OPENCLAW_LIVE_OPENAI_KEY;
  
  // 执行测试
  const response = await callOpenAI('Hello');
  
  // 验证格式（这是我们修复的问题）
  expect(response).toHaveProperty('choices');
  expect(response.choices[0]).toHaveProperty('message');
});
```

---

## 第七部分：故障排查速查

### 常见问题与解决方案

**问题一：测试超时**

**排查步骤**：

```bash
# 增加超时时间
pnpm test -- --testTimeout=30000

# 检查网络连接
curl -I https://api.anthropic.com

# 验证 API 密钥
echo $ANTHROPIC_API_KEY | head -c 10
```

**问题二：实时测试找不到凭据**

**排查步骤**：

```bash
# 检查环境变量
env | grep API_KEY

# 检查配置文件
cat ~/.openclaw/credentials/openai/api-key.json

# 测试凭据获取
OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1 pnpm test:live

# 检查 ~/.profile
cat ~/.profile | grep API_KEY
```

**问题三：测试覆盖率不达标**

**解决方案**：

```bash
# 运行覆盖率报告
pnpm test:coverage

# 查看详细报告
open coverage/index.html

# 添加缺失测试
# 根据报告补全测试用例
```

**问题四：E2E 测试不稳定**

**排查步骤**：

```bash
# 单独运行失败的测试
pnpm test:e2e src/gateway/failing-test.e2e.test.ts

# 增加等待时间
pnpm test:e2e --testTimeout=120000

# 检查资源使用
htop  # 确保有足够内存
```

---

## 练习与自检

### 基础技能自检

- [ ] 能够运行基本测试命令
- [ ] 理解测试结果含义
- [ ] 能够定位测试文件
- [ ] 掌握测试调试方法

### 进阶技能自检

- [ ] 理解测试套件分层
- [ ] 能够配置测试环境
- [ ] 理解实时测试凭据发现
- [ ] 能够选择测试类型

### 专家技能自检

- [ ] 能够设计回归测试策略
- [ ] 理解 Docker 测试价值
- [ ] 掌握模型矩阵选择
- [ ] 能够优化测试覆盖

---

## 参考资料

### 核心文档

- [调试指南](/zh-CN/debugging) - 调试工具详解
- [日志记录](/zh-CN/logging) - 日志分析
- [Vitest 文档](https://vitest.dev/)

### 相关命令速查

```bash
# 测试命令
pnpm test              # 单元/集成测试
pnpm test:coverage    # 覆盖率测试
pnpm test:e2e         # 端到端测试
pnpm test:live        # 实时测试

# Docker 测试
pnpm test:docker:live-models     # Docker 模型测试
pnpm test:docker:live-gateway    # Docker 网关测试

# 模型发现
openclaw models list             # 列出模型
openclaw models list --json     # JSON 格式
```

### 扩展阅读

- [测试金字塔](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Vitest 最佳实践](https://vitest.dev/guide/best-practices.html)
- [持续集成测试策略](https://docs.github.com/en/actions/guides/choosing-when-to-use-github-hosted-runners)
