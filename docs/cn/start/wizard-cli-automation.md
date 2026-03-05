---
summary: "OpenClaw CLI 的脚本化配置和代理自动化设置指南"
read_when:
  - 在脚本或 CI 中自动化配置时
  - 需要特定提供商的非交互式示例时
title: "CLI 自动化配置"
sidebarTitle: "CLI 自动化"
---

# CLI 自动化配置

使用 `--non-interactive` 参数实现 `openclaw onboard` 的自动化。

> **⚠️ 重要提示**：`--json` 参数不会隐式启用非交互式模式。脚本中请使用 `--non-interactive`（和 `--workspace`）。

---

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 编写非交互式自动化配置脚本
- [ ] 为不同 AI 提供商配置自动化参数
- [ ] 理解密钥存储模式（明文 vs 引用）

### 进阶目标（建议掌握）

- [ ] 创建和管理多个独立的 AI 代理
- [ ] 在 CI/CD 流水线中集成 OpenClaw
- [ ] 实现安全的密钥引用模式

---

## 💡 为什么需要自动化配置？

自动化配置解决了以下场景的需求：

| 场景 | 说明 |
|------|------|
| **批量部署** | 在多台服务器上快速部署 OpenClaw |
| **CI/CD 集成** | 在持续集成流程中自动配置测试环境 |
| **容器化部署** | Docker/Kubernetes 镜像构建时的自动配置 |
| **多环境管理** | 开发、测试、生产环境的标准化配置 |
| **无人值守安装** | 无需手动干预的完全自动化安装 |

---

## 📋 非交互式配置基础示例

### 最小化配置

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `--non-interactive` | 启用非交互式模式（必需） |
| `--mode local` | 本地模式（vs 远程模式） |
| `--auth-choice apiKey` | 使用 API Key 认证 |
| `--anthropic-api-key` | Anthropic API Key |
| `--secret-input-mode plaintext` | 明文存储密钥 |
| `--gateway-port` | 网关端口 |
| `--gateway-bind loopback` | 仅本地监听 |
| `--install-daemon` | 安装后台服务 |
| `--daemon-runtime node` | 使用 Node.js 运行时 |
| `--skip-skills` | 跳过技能安装 |

### 添加机器可读输出

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --json
```

`--json` 参数会输出 JSON 格式的配置摘要，便于脚本解析。

### 使用密钥引用模式

`--secret-input-mode ref` 在认证配置文件中存储环境变量引用，而非明文密钥值。

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

**引用模式 vs 明文模式**：

| 模式 | 存储内容 | 优势 | 安全性 |
|------|----------|------|--------|
| **明文（plaintext）** | 实际密钥值 | 简单直接 | ⚠️ 较低 |
| **引用（ref）** | 环境变量引用 | 密钥不落地，易于轮换 | ✅ 较高 |

> **🔐 安全建议**：生产环境推荐使用引用模式，避免密钥明文存储在配置文件中。

**非交互式引用模式要求**：

- 必须在进程环境中设置提供商环境变量
- 如果没有匹配的环境变量，内联密钥参数现在会快速失败

---

## 🏢 提供商特定示例

以下是各主流 AI 提供商的非交互式配置示例：

<details>
<summary><strong>Gemini (Google)</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Z.AI</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Vercel AI Gateway</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Cloudflare AI Gateway</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Moonshot (Kimi)</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Mistral</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>Synthetic (Anthropic 兼容)</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>OpenCode Zen</strong></summary>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

</details>

<details>
<summary><strong>自定义提供商</strong></summary>

**标准模式（明文密钥）**：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

**兼容性选项**：
- `openai` - OpenAI 兼容 API（默认）
- `anthropic` - Anthropic 兼容 API

> **💡 提示**：`--custom-api-key` 是可选的。如果省略，配置会检查 `CUSTOM_API_KEY` 环境变量。

**引用模式（推荐）**：

```bash
export CUSTOM_API_KEY="your-key"

openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --secret-input-mode ref \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

在这种模式下，配置会将 `apiKey` 存储为：
```json
{
  "source": "env",
  "provider": "default",
  "id": "CUSTOM_API_KEY"
}
```

</details>

---

## 🤖 添加另一个代理

使用 `openclaw agents add <name>` 创建具有独立工作区、会话和认证配置的代理。

不带 `--workspace` 运行会启动向导。

### 非交互式创建代理

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

**配置内容**：

| 字段 | 说明 |
|------|------|
| `agents.list[].name` | 代理名称 |
| `agents.list[].workspace` | 工作区路径 |
| `agents.list[].agentDir` | 代理目录 |

**注意事项**：

| 项目 | 说明 |
|------|------|
| **默认工作区** | 遵循 `~/.openclaw/workspace-<agentId>` 模式 |
| **消息绑定** | 添加 `bindings` 以路由入站消息（向导可以完成此操作） |
| **非交互式参数** | `--model`、`--agent-dir`、`--bind`、`--non-interactive` |

### 多代理场景示例

```
┌─────────────────────────────────────────────────────────────┐
│                    多代理架构示例                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  OpenClaw CLI                                               │
│                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  个人代理      │  │   工作代理     │  │  测试代理     │  │
│  │  (personal)   │  │   (work)       │  │  (testing)   │  │
│  │               │  │               │  │              │  │
│  │ 模型: Claude   │  │ 模型: GPT-5.2  │  │ 模型: GPT-4  │  │
│  │ 绑定: 私人渠道 │  │ 绑定: 工作群组 │  │ 绑定: 测试环境│  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**创建多个代理的脚本**：

```bash
#!/bin/bash

# 个人代理
openclaw agents add personal \
  --workspace ~/.openclaw/workspace-personal \
  --model anthropic/claude-opus-4-5 \
  --non-interactive

# 工作代理
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind slack:work-team \
  --non-interactive

# 测试代理
openclaw agents add testing \
  --workspace ~/.openclaw/workspace-testing \
  --model openai/gpt-4-turbo \
  --bind discord:test-server \
  --non-interactive

echo "所有代理创建完成！"
openclaw agents list
```

---

## 🚀 CI/CD 集成示例

### GitHub Actions 示例

```yaml
name: OpenClaw Setup

on: [push]

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '22'

      - name: Install OpenClaw
        run: npm install -g openclaw@latest

      - name: Configure OpenClaw
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          openclaw onboard --non-interactive \
            --mode local \
            --auth-choice apiKey \
            --anthropic-api-key "$ANTHROPIC_API_KEY" \
            --secret-input-mode ref \
            --skip-daemon \
            --skip-skills \
            --json

      - name: Verify installation
        run: openclaw health
```

### Dockerfile 示例

```dockerfile
FROM node:22-bullseye

# 安装 OpenClaw
RUN npm install -g openclaw@latest

# 设置环境变量（运行时传入）
ENV ANTHROPIC_API_KEY=""
ENV OPENCLAW_HOME="/root/.openclaw"

# 创建配置脚本
COPY configure.sh /tmp/configure.sh
RUN chmod +x /tmp/configure.sh

# 非交互式配置
RUN /tmp/configure.sh || echo "Configuration will be completed at runtime"

CMD ["openclaw", "gateway", "--port", "18789"]
```

**configure.sh**:

```bash
#!/bin/bash
if [ -n "$ANTHROPIC_API_KEY" ]; then
  openclaw onboard --non-interactive \
    --mode local \
    --auth-choice apiKey \
    --anthropic-api-key "$ANTHROPIC_API_KEY" \
    --secret-input-mode ref \
    --skip-daemon \
    --skip-skills
fi
```

---

## ⚠️ 常见问题与解决方案

### 问题一：非交互式模式仍然要求输入

**症状**：使用 `--non-interactive` 后仍然提示输入

**可能原因与解决方案**：

| 检查项 | 操作 |
|--------|------|
| 1. 环境变量未设置 | 确保密钥环境变量已导出 |
| 2. 缺少必需参数 | 检查是否所有必需的 `--*` 参数都已提供 |
| 3. 参数拼写错误 | 使用 `openclaw onboard --help` 验证参数名 |

### 问题二：密钥引用模式不工作

**症状**：配置后提示 "API key not found"

**解决方案**：

```bash
# 1. 确认环境变量已设置
echo $CUSTOM_API_KEY

# 2. 检查配置文件中的引用格式
cat ~/.openclaw/agents/*/agent/auth-profiles.json | grep -A5 "apiKey"

# 3. 重新配置（确保环境变量存在）
export CUSTOM_API_KEY="your-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --secret-input-mode ref \
  --accept-risk
```

### 问题三：多代理消息路由混乱

**症状**：消息发送到错误的代理

**解决方案**：

```bash
# 1. 检查代理绑定配置
openclaw agents list --json | jq '.[].bindings'

# 2. 验证消息路由规则
openclaw config get agents.list

# 3. 如需要，重新配置绑定
openclaw agents bind <agent-name> <channel>:<target>
```

---

## 📚 核心概念回顾

| 概念 | 说明 |
|------|------|
| **非交互式模式** | 脚本友好的自动化配置方式 |
| **密钥存储模式** | 明文（plaintext）vs 引用（ref） |
| **多代理** | 独立工作区和配置的多个 AI 实例 |
| **消息绑定** | 将特定渠道路由到指定代理 |

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| **配置中心** | [CLI 配置向导](/zh-CN/start/wizard) |
| **完整参考** | [CLI 配置完整参考](/zh-CN/start/wizard-cli-reference) |
| **命令参考** | [`openclaw onboard`](/zh-CN/cli/onboard) |

---

## 🎯 下一步学习

| 学习路径 | 文档链接 | 预计时间 |
|----------|----------|----------|
| **多代理深入** | [代理管理](/zh-CN/cli/agents) | 20 分钟 |
| **消息路由** | [路由配置](/zh-CN/concepts/routing) | 30 分钟 |
| **生产部署** | [运维指南](/zh-CN/operations/index) | 45 分钟 |

---

> **💡 专家提示**：自动化配置时，建议使用 `--json` 参数获取结构化输出，便于在脚本中进行解析和错误处理。
