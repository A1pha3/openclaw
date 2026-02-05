---
summary: "OpenClaw 中文文档翻译计划：299 篇文档的四阶段渐进式翻译策略与执行指南"
read_when:
  - 参与 OpenClaw 中文文档翻译工作
  - 了解翻译工作的整体规划和进度
  - 需要添加新文档到翻译计划
title: "中文文档翻译计划（TRANSLATION_PLAN）"
---

# OpenClaw 中文文档翻译计划

## 执行概览

### 核心数据

| 指标 | 数值 | 说明 |
|------|------|------|
| 总文档数 | 299 篇 | 英文原文文档总量 |
| 已完成 | 30 篇 | 已翻译并发布的中文文档 |
| 待翻译 | 269 篇 | 尚未开始翻译的文档 |
| 完成率 | 10% | 当前进度 |
| 预计周期 | 4-6 个月 | 按当前资源估算 |

### 翻译策略总览

本计划采用**四阶段渐进式翻译策略**，遵循以下核心原则：

```
翻译优先级金字塔：

                    ┌─────────────────┐
                    │   基础设施       │  插件、脚本、TUI 等
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │   平台适配       │  各平台安装与配置指南
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │   功能工具       │  CLI 命令、工具使用
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │   核心概念       │  架构、原理、配置
                    └─────────────────┘
                         ▲
                    ┌─────────────────┐
                    │   入门指南       │  快速上手、新用户引导
                    └─────────────────┘

优先级说明：从金字塔顶端开始，越往下优先级越高
因为新用户首先接触的是入门内容
```

---

## 第一部分：学习目标

### 基础目标（必掌握）

- [ ] 理解 OpenClaw 文档的整体架构和分类体系
- [ ] 掌握四阶段翻译策略的设计原理
- [ ] 了解每个翻译阶段的目标和交付物
- [ ] 理解文档间的依赖关系和阅读路径

### 进阶目标（建议掌握）

- [ ] 能够为新增文档确定正确的翻译阶段
- [ ] 理解翻译质量检查的关键点
- [ ] 掌握术语一致性的管理方法
- [ ] 能够评估翻译工作的优先级

### 专家目标（挑战）

- [ ] 能够设计和优化大规模文档翻译工作流
- [ ] 理解国际化（i18n）与本地化（l10n）的区别
- [ ] 掌握文档版本同步和更新策略

---

## 第二部分：翻译阶段详解

### 第一阶段：基础入门（Phase 1）

**阶段目标**：让新用户在 30 分钟内完成上手并发送第一条消息

**设计原理**：用户旅程的起点

```
新用户旅程图：

    了解 OpenClaw
          │
          ▼
    安装与配置 ← Phase 1 覆盖区域
          │
          ▼
    连接第一个渠道
          │
          ▼
    发送第一条消息 ← 成功标准！
```

**核心文档清单（25 篇）**

| 序号 | 文档路径 | 优先级 | 说明 |
|------|----------|--------|------|
| 1 | `zh-CN/index.md` | 🔴 已完成 | 中文文档主页 |
| 2 | `zh-CN/start/getting-started.md` | 🔴 增强版 | 新用户入门（增强版） |
| 3 | `zh-CN/start/wizard.md` | 🟠 高 | 向导模式详解 |
| 4 | `zh-CN/start/showcase.md` | 🟠 高 | 功能展示 |
| 5 | `zh-CN/install/index.md` | 🔴 高 | 安装概述 |
| 6 | `zh-CN/install/installer.md` | 🟠 中 | 安装器使用 |
| 7 | `zh-CN/install/node.md` | 🟠 中 | Node.js 安装 |
| 8 | `zh-CN/install/docker.md` | 🟠 高 | Docker 部署 |
| 9 | `zh-CN/install/nix.md` | 🟡 低 | Nix 安装 |
| 10 | `zh-CN/install/ansible.md` | 🟡 低 | Ansible 自动化 |
| 11 | `zh-CN/install/migrating.md` | 🟠 中 | 迁移指南 |
| 12 | `zh-CN/install/updating.md` | 🔴 高 | 更新指南 |
| 13 | `zh-CN/install/uninstall.md` | 🟠 中 | 卸载指南 |
| 14 | `zh-CN/start/quick-start.md` | 🔴 已完成 | 5 分钟快速上手 |
| 15 | `zh-CN/start/installation.md` | 🔴 已完成 | 详细安装步骤 |
| 16 | `zh-CN/start/setup.md` | 🟠 高 | 初始配置 |
| 17 | `zh-CN/start/pairing.md` | 🟠 高 | 设备配对 |
| 18 | `zh-CN/start/onboarding.md` | 🟠 高 | Onboarding 流程 |
| 19 | `zh-CN/start/openclaw.md` | 🟠 中 | OpenClaw 助手设置 |
| 20 | `zh-CN/help/index.md` | 🟠 中 | 帮助中心 |
| 21 | `zh-CN/help/faq.md` | 🔴 高 | 常见问题 |
| 22 | `zh-CN/help/troubleshooting.md` | 🟠 高 | 故障排除 |

**已完成文档确认**：

| 文档路径 | 状态 | 完成日期 |
|----------|------|----------|
| `zh-CN/index.md` | ✅ 已完成 | - |
| `zh-CN/start/quick-start.md` | ✅ 已完成 | - |
| `zh-CN/start/installation.md` | ✅ 已完成 | - |

**阶段成功标准**：

```
用户测试清单：

□ 新用户能够根据文档独立完成安装（<15 分钟）
□ 新用户能够理解 OpenClaw 的核心概念（<10 分钟）
□ 新用户能够发送第一条消息（<5 分钟）
□ 遇到问题时能够找到帮助文档
```

### 第二阶段：核心概念（Phase 2）

**阶段目标**：深入理解系统架构与工作原理

**设计原理**：从"会用"到"理解"

```
知识深度金字塔：

           ┌─────────────────┐
           │   深度定制       │  高级配置、优化
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   问题解决       │  调试、故障排查
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   系统集成       │  与其他系统配合
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   核心概念 ← 本阶段 │  理解"为什么"
           └─────────────────┘
```

**架构与概念文档（35 篇）**

| 序号 | 文档路径 | 优先级 | 说明 |
|------|----------|--------|------|
| 1 | `zh-CN/concepts/architecture.md` | 🔴 已完成 | 整体架构 |
| 2 | `zh-CN/concepts/gateway.md` | 🔴 已完成 | 网关系统 |
| 3 | `zh-CN/concepts/agents.md` | 🟠 高 | AI 代理 |
| 4 | `zh-CN/concepts/agent.md` | 🟠 高 | 代理详解 |
| 5 | `zh-CN/concepts/agent-loop.md` | 🟠 高 | 代理循环 |
| 6 | `zh-CN/concepts/agent-workspace.md` | 🟠 中 | 代理工作区 |
| 7 | `zh-CN/concepts/sessions.md` | 🔴 已完成 | 会话管理 |
| 8 | `zh-CN/concepts/routing.md` | 🔴 已完成 | 消息路由 |
| 9 | `zh-CN/concepts/messages.md` | 🟠 高 | 消息系统 |
| 10 | `zh-CN/concepts/context.md` | 🟠 高 | 上下文管理 |
| 11 | `zh-CN/concepts/memory.md` | 🟠 高 | 记忆系统 |
| 12 | `zh-CN/concepts/session.md` | 🟠 中 | 会话详解 |
| 13 | `zh-CN/concepts/session-tool.md` | 🟠 中 | 会话工具 |
| 14 | `zh-CN/concepts/session-pruning.md` | 🟠 中 | 会话修剪 |
| 15 | `zh-CN/concepts/group-messages.md` | 🟠 中 | 群组消息 |
| 16 | `zh-CN/concepts/groups.md` | 🟠 中 | 群组管理 |
| 17 | `zh-CN/channels/index.md` | 🔴 已完成 | 渠道概述 |
| 18 | `zh-CN/concepts/channel-routing.md` | 🟠 中 | 渠道路由 |
| 19 | `zh-CN/concepts/models.md` | 🟠 高 | 模型配置 |
| 20 | `zh-CN/concepts/model-providers.md` | 🟠 高 | 模型提供商 |
| 21 | `zh-CN/concepts/model-failover.md` | 🟠 中 | 模型故障转移 |
| 22 | `zh-CN/concepts/oauth.md` | 🟠 中 | OAuth 认证 |
| 23 | `zh-CN/concepts/streaming.md` | 🟠 低 | 流式响应 |
| 24 | `zh-CN/concepts/presence.md` | 🟠 低 | 在线状态 |
| 25 | `zh-CN/concepts/typing-indicators.md` | 🟠 低 | 输入提示 |
| 26 | `zh-CN/concepts/retry.md` | 🟠 低 | 重试机制 |
| 27 | `zh-CN/concepts/queue.md` | 🟠 低 | 消息队列 |
| 28 | `zh-CN/concepts/compaction.md` | 🟠 低 | 数据压缩 |
| 29 | `zh-CN/concepts/timezone.md` | 🟠 低 | 时区处理 |
| 30 | `zh-CN/concepts/usage-tracking.md` | 🟠 低 | 用量追踪 |
| 31 | `zh-CN/concepts/multi-agent.md` | 🟠 低 | 多代理 |
| 32 | `zh-CN/concepts/system-prompt.md` | 🟠 低 | 系统提示词 |
| 33 | `zh-CN/concepts/markdown-formatting.md` | 🟠 低 | Markdown 格式 |
| 34 | `zh-CN/concepts/typebox.md` | 🟠 低 | TypeBox 类型系统 |

**已完成文档确认**：

| 文档路径 | 状态 | 完成日期 |
|----------|------|----------|
| `zh-CN/concepts/architecture.md` | ✅ 已完成 | - |
| `zh-CN/concepts/gateway.md` | ✅ 已完成 | - |
| `zh-CN/concepts/sessions.md` | ✅ 已完成 | - |
| `zh-CN/concepts/routing.md` | ✅ 已完成 | - |
| `zh-CN/channels/index.md` | ✅ 已完成 | - |

### 第三阶段：功能与工具（Phase 3）

**阶段目标**：掌握所有实用功能和工具

**设计原理**：从"理解"到"使用"

```
工具熟练度模型：

           ┌─────────────────┐
           │   专家           │  自定义工具、扩展
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   高级用户       │  复杂工作流、自动化
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   常规用户 ← 本阶段 │  日常使用
           └─────────────────┘
```

**CLI 命令文档（41 篇）**

| 分类 | 文档数量 | 优先级 |
|------|----------|--------|
| 核心命令 | 7 篇 | 🔴 高 |
| 消息与渠道 | 3 篇 | 🔴 高 |
| 代理与会话 | 4 篇 | 🟠 中 |
| 插件与技能 | 2 篇 | 🟠 中 |
| 系统工具 | 25 篇 | 🟡 低 |

**工具与技能文档（22 篇）**

| 序号 | 文档路径 | 优先级 | 说明 |
|------|----------|--------|------|
| 1 | `zh-CN/tools/index.md` | 🔴 高 | 工具概述 |
| 2 | `zh-CN/tools/skills.md` | 🔴 高 | 技能系统 |
| 3 | `zh-CN/tools/creating-skills.md` | 🟠 高 | 创建技能 |
| 4 | `zh-CN/tools/skills-config.md` | 🟠 中 | 技能配置 |
| 5 | `zh-CN/tools/slash-commands.md` | 🟠 中 | 斜杠命令 |
| 6 | `zh-CN/tools/browser.md` | 🟠 高 | 浏览器工具 |
| 7 | `zh-CN/tools/exec.md` | 🟠 高 | 执行工具 |
| 8 | `zh-CN/tools/web.md` | 🟠 中 | Web 工具 |
| 9 | `zh-CN/tools/thinking.md` | 🟠 中 | 思考模式 |
| 10 | `zh-CN/tools/elevated.md` | 🟠 中 | 提升权限 |
| 11 | `zh-CN/tools/exec-approvals.md` | 🟠 中 | 执行审批 |
| 12 | `zh-CN/tools/subagents.md` | 🟠 低 | 子代理 |
| 13 | `zh-CN/tools/reactions.md` | 🟠 低 | 反应工具 |
| 14 | `zh-CN/tools/agent-send.md` | 🟠 低 | 代理发送 |
| 15 | `zh-CN/tools/clawhub.md` | 🟠 低 | ClawHub |
| 16 | `zh-CN/tools/firecrawl.md` | 🟠 低 | Firecrawl |
| 17 | `zh-CN/tools/apply-patch.md` | 🟠 低 | 应用补丁 |
| 18 | `zh-CN/tools/chrome-extension.md` | 🟠 低 | Chrome 扩展 |
| 19 | `zh-CN/tools/browser-login.md` | 🟠 低 | 浏览器登录 |
| 20 | `zh-CN/tools/browser-linux-troubleshooting.md` | 🟠 低 | Linux 浏览器故障排除 |
| 21 | `zh-CN/tools/lobster.md` | 🟠 低 | Lobster |
| 22 | `zh-CN/tools/llm-task.md` | 🟠 低 | LLM 任务 |

### 第四阶段：渠道配置（Phase 4）

**阶段目标**：配置所有消息渠道

**设计原理**：连接即价值

```
渠道价值矩阵：

           ┌─────────────────────────────────────────┐
           │              用户常用渠道                │
           │  WhatsApp / Telegram / Slack / Discord │
           └─────────────────────────────────────────┘
                              │
                              ▼
           ┌─────────────────────────────────────────┐
           │            增强型渠道                   │
           │  Signal / iMessage / Google Chat        │
           └─────────────────────────────────────────┘
                              │
                              ▼
           ┌─────────────────────────────────────────┐
           │            专业型渠道                     │
           │  Microsoft Teams / Matrix / Zalo        │
           └─────────────────────────────────────────┘
```

**渠道文档清单（22 篇）**

| 序号 | 文档路径 | 优先级 | 状态 |
|------|----------|--------|------|
| 1 | `zh-CN/channels/whatsapp.md` | 🔴 高 | ✅ 已完成 |
| 2 | `zh-CN/channels/telegram.md` | 🔴 高 | ✅ 已完成 |
| 3 | `zh-CN/channels/discord.md` | 🔴 高 | ✅ 已完成 |
| 4 | `zh-CN/channels/slack.md` | 🔴 高 | ✅ 已完成 |
| 5 | `zh-CN/channels/signal.md` | 🔴 高 | ✅ 已完成 |
| 6 | `zh-CN/channels/imessage.md` | 🔴 高 | ✅ 已完成 |
| 7 | `zh-CN/channels/matrix.md` | 🟠 高 | ✅ 已完成 |
| 8 | `zh-CN/channels/bluebubbles.md` | 🟠 中 | ⏳ 待翻译 |
| 9 | `zh-CN/channels/googlechat.md` | 🟠 中 | ⏳ 待翻译 |
| 10 | `zh-CN/channels/grammy.md` | 🟡 低 | ⏳ 待翻译 |
| 11 | `zh-CN/channels/line.md` | 🟡 低 | ⏳ 待翻译 |
| 12 | `zh-CN/channels/location.md` | 🟡 低 | ⏳ 待翻译 |
| 13 | `zh-CN/channels/mattermost.md` | 🟡 低 | ⏳ 待翻译 |
| 14 | `zh-CN/channels/msteams.md` | 🟠 高 | ⏳ 待翻译 |
| 15 | `zh-CN/channels/nextcloud-talk.md` | 🟡 低 | ⏳ 待翻译 |
| 16 | `zh-CN/channels/nostr.md` | 🟡 低 | ⏳ 待翻译 |
| 17 | `zh-CN/channels/tlon.md` | 🟡 低 | ⏳ 待翻译 |
| 18 | `zh-CN/channels/twitch.md` | 🟡 低 | ⏳ 待翻译 |
| 19 | `zh-CN/channels/zalo.md` | 🟠 中 | ⏳ 待翻译 |
| 20 | `zh-CN/channels/zalouser.md` | 🟠 中 | ⏳ 待翻译 |
| 21 | `zh-CN/channels/troubleshooting.md` | 🟠 高 | ⏳ 待翻译 |

---

## 第三部分：进阶阶段

### 第五阶段：网关与运维（Phase 5）

**阶段目标**：掌握网关配置与生产运维

**运维成熟度模型**：

```
运维成熟度阶段：

Stage 1: 初始安装
└── 能够运行基本功能

Stage 2: 正常运维 ← Phase 5 覆盖
└── 稳定运行、基础监控

Stage 3: 优化运维
└── 性能调优、成本优化

Stage 4: 专业运维
└── 高可用、自动扩缩容
```

**网关文档清单（27 篇）**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/gateway/index.md` | 🔴 高 |
| 2 | `zh-CN/gateway/configuration.md` | 🔴 高 |
| 3 | `zh-CN/gateway/configuration-examples.md` | 🟠 高 |
| 4 | `zh-CN/gateway/protocol.md` | 🟠 高 |
| 5 | `zh-CN/gateway/authentication.md` | 🟠 高 |
| 6 | `zh-CN/gateway/pairing.md` | 🟠 高 |
| 7 | `zh-CN/gateway/discovery.md` | 🟠 中 |
| 8 | `zh-CN/gateway/remote.md` | 🔴 高 |
| 9 | `zh-CN/gateway/tailscale.md` | 🟠 高 |
| 10 | `zh-CN/gateway/health.md` | 🟠 高 |
| 11 | `zh-CN/gateway/heartbeat.md` | 🟠 中 |
| 12 | `zh-CN/gateway/logging.md` | 🟠 高 |
| 13 | `zh-CN/gateway/doctor.md` | 🟠 高 |
| 14 | `zh-CN/gateway/troubleshooting.md` | 🔴 高 |
| 15 | `zh-CN/gateway/background-process.md` | 🟠 中 |
| 16 | `zh-CN/gateway/bonjour.md` | 🟠 低 |
| 17 | `zh-CN/gateway/bridge-protocol.md` | 🟠 低 |
| 18 | `zh-CN/gateway/cli-backends.md` | 🟠 低 |
| 19 | `zh-CN/gateway/multiple-gateways.md` | 🟠 低 |
| 20 | `zh-CN/gateway/gateway-lock.md` | 🟠 低 |
| 21 | `zh-CN/gateway/local-models.md` | 🟠 中 |
| 22 | `zh-CN/gateway/openai-http-api.md` | 🟠 低 |
| 23 | `zh-CN/gateway/openresponses-http-api.md` | 🟠 低 |
| 24 | `zh-CN/gateway/tools-invoke-http-api.md` | 🟠 低 |
| 25 | `zh-CN/gateway/sandboxing.md` | 🟠 高 |
| 26 | `zh-CN/gateway/sandbox-vs-tool-policy-vs-elevated.md` | 🟠 中 |
| 27 | `zh-CN/gateway/remote-gateway-readme.md` | 🟠 低 |

**运维文档（5 篇）**

| 序号 | 文档路径 | 状态 |
|------|----------|------|
| 1 | `zh-CN/operations/index.md` | ✅ 已完成 |
| 2 | `zh-CN/operations/deployment.md` | ✅ 已完成 |
| 3 | `zh-CN/operations/monitoring.md` | ✅ 已完成 |
| 4 | `zh-CN/operations/troubleshooting.md` | ✅ 已完成 |

### 第六阶段：提供商与平台（Phase 6）

**阶段目标**：配置 AI 模型与部署平台

**AI 提供商生态图**：

```
AI 提供商分类：

           ┌─────────────────┐
           │   主流提供商       │
           │  Anthropic / OpenAI │
           └─────────────────┘
                ▲
     ┌─────────┼─────────┐
     ▼         ▼         ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│  国内   │ │  开源   │ │  专业   │
│ 提供商  │ │  模型   │ │  场景   │
└─────────┘ └─────────┘ └─────────┘
```

**AI 提供商文档（21 篇）**

| 序号 | 提供商 | 优先级 |
|------|--------|--------|
| 1 | Anthropic | 🔴 高 |
| 2 | OpenAI | 🔴 高 |
| 3 | GLM | 🟠 中 |
| 4 | Moonshot | 🟠 中 |
| 5 | MiniMax | 🟠 中 |
| 6 | Xiaomi | 🟡 低 |
| 7 | Qwen | 🟠 中 |
| 8 | Opencode | 🟡 低 |
| 9 | OpenRouter | 🟠 中 |
| 10 | Ollama | 🟠 高 |
| 11 | GitHub Copilot | 🟠 中 |
| 12 | Deepgram | 🟡 低 |
| 13 | Vercel AI Gateway | 🟡 低 |
| 14 | Synthetic | 🟡 低 |
| 15 | Venice | 🟡 低 |
| 16 | Z.ai | 🟡 低 |
| 17 | Models | 🟠 中 |
| 18 | Claude Max API Proxy | 🟡 低 |

**部署平台文档（17 篇）**

| 序号 | 平台 | 优先级 |
|------|------|--------|
| 1 | macOS | 🔴 高 |
| 2 | Linux | 🔴 高 |
| 3 | Windows | 🟠 高 |
| 4 | iOS | 🟠 高 |
| 5 | Android | 🟠 高 |
| 6 | Raspberry Pi | 🟠 中 |
| 7 | Docker | 🔴 高 |
| 8 | Fly.io | 🟠 中 |
| 9 | Google Cloud | 🟠 中 |
| 10 | DigitalOcean | 🟠 中 |
| 11 | Hetzner | 🟠 中 |
| 12 | Oracle Cloud | 🟠 中 |
| 13 | exe.dev | 🟠 中 |
| 14 | macOS VM | 🟡 低 |

### 第七阶段：节点与自动化（Phase 7）

**阶段目标**：移动设备与自动化工作流

```
自动化复杂度金字塔：

           ┌─────────────────┐
           │   复杂编排       │  跨系统集成
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   工作流         │  条件触发
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   定时任务 ← 本阶段 │  Cron 触发
           └─────────────────┘
```

**节点文档（9 篇）**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/nodes/index.md` | 🟠 高 |
| 2 | `zh-CN/nodes/audio.md` | 🟠 高 |
| 3 | `zh-CN/nodes/camera.md` | 🟠 高 |
| 4 | `zh-CN/nodes/images.md` | 🟠 中 |
| 5 | `zh-CN/nodes/location-command.md` | 🟠 中 |
| 6 | `zh-CN/nodes/media-understanding.md` | 🟠 中 |
| 7 | `zh-CN/nodes/talk.md` | 🔴 高 |
| 8 | `zh-CN/nodes/voicewake.md` | 🔴 高 |

**自动化文档（6 篇）**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/automation/cron-jobs.md` | 🔴 高 |
| 2 | `zh-CN/automation/cron-vs-heartbeat.md` | 🟠 中 |
| 3 | `zh-CN/automation/webhook.md` | 🔴 高 |
| 4 | `zh-CN/automation/poll.md` | 🟠 中 |
| 5 | `zh-CN/automation/auth-monitoring.md` | 🟠 中 |
| 6 | `zh-CN/automation/gmail-pubsub.md` | 🟠 中 |

### 第八阶段：开发者文档（Phase 8）

**阶段目标**：二次开发与贡献

```
开发者旅程图：

           ┌─────────────────┐
           │   核心贡献       │  PR/Issue
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   插件开发 ← 本阶段 │  自定义扩展
           └─────────────────┘
                ▲
           ┌─────────────────┐
           │   基础开发       │  环境搭建
           └─────────────────┘
```

**开发者文档（5 篇）**

| 序号 | 文档路径 | 状态 |
|------|----------|------|
| 1 | `zh-CN/developer/index.md` | ✅ 已完成 |
| 2 | `zh-CN/developer/project-structure.md` | ✅ 已完成 |
| 3 | `zh-CN/developer/plugin-development.md` | ✅ 已完成 |
| 4 | `zh-CN/developer/testing.md` | ✅ 已完成 |
| 5 | `zh-CN/developer/contributing.md` | ✅ 已完成 |

**参考文档与模板**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/reference/RELEASING.md` | 🟠 高 |
| 2 | `zh-CN/reference/rpc.md` | 🟠 高 |
| 3 | `zh-CN/reference/api-usage-costs.md` | 🟠 中 |
| 4 | `zh-CN/reference/device-models.md` | 🟡 低 |
| 5 | `zh-CN/reference/test.md` | 🟠 中 |
| 6 | `zh-CN/reference/session-management-compaction.md` | 🟡 低 |
| 7 | `zh-CN/reference/transcript-hygiene.md` | 🟡 低 |

**模板文件**

| 序号 | 模板 | 状态 |
|------|------|------|
| 1 | `zh-CN/reference/templates/AGENTS.md` | ✅ 本文档 |
| 2 | `zh-CN/reference/templates/BOOTSTRAP.md` | ✅ 本文档 |
| 3 | `zh-CN/reference/templates/HEARTBEAT.md` | ⏳ 待翻译 |
| 4 | `zh-CN/reference/templates/IDENTITY.md` | ⏳ 待翻译 |
| 5 | `zh-CN/reference/templates/SOUL.md` | ⏳ 待翻译 |
| 6 | `zh-CN/reference/templates/TOOLS.md` | ✅ 本文档 |
| 7 | `zh-CN/reference/templates/USER.md` | ⏳ 待翻译 |

### 第九阶段：其他重要文档（Phase 9）

**阶段目标**：补充遗漏的重要文档

**Web 界面**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/web/index.md` | 🟠 中 |
| 2 | `zh-CN/web/dashboard.md` | 🟠 中 |
| 3 | `zh-CN/web/control-ui.md` | 🟠 高 |
| 4 | `zh-CN/web/webchat.md` | 🟠 高 |

**配置与安全**

| 序号 | 文档路径 | 状态 |
|------|----------|------|
| 1 | `zh-CN/config/index.md` | ✅ 已完成 |
| 2 | `zh-CN/config/reference.md` | ✅ 已完成 |
| 3 | `zh-CN/config/examples.md` | ✅ 已完成 |
| 4 | `zh-CN/security/formal-verification.md` | 🟡 低 |

**杂项文档**

| 序号 | 文档路径 | 优先级 |
|------|----------|--------|
| 1 | `zh-CN/hooks.md` | 🟠 高 |
| 2 | `zh-CN/hooks/soul-evil.md` | 🟡 低 |
| 3 | `zh-CN/plugin.md` | 🔴 高 |
| 4 | `zh-CN/plugins/agent-tools.md` | 🟠 高 |
| 5 | `zh-CN/plugins/manifest.md` | 🟠 中 |
| 6 | `zh-CN/plugins/voice-call.md` | 🟠 高 |
| 7 | `zh-CN/plugins/zalouser.md` | 🟠 中 |
| 8 | `zh-CN/tts.md` | 🟠 高 |
| 9 | `zh-CN/tui.md` | 🔴 高 |
| 10 | `zh-CN/pi.md` | 🟠 中 |
| 11 | `zh-CN/pi-dev.md` | 🟡 低 |
| 12 | `zh-CN/bedrock.md` | 🟡 低 |
| 13 | `zh-CN/brave-search.md` | 🟡 低 |
| 14 | `zh-CN/broadcast-groups.md` | 🟠 中 |
| 15 | `zh-CN/date-time.md` | 🟡 低 |
| 16 | `zh-CN/debugging.md` | 🟠 高 |
| 17 | `zh-CN/debug/node-issue.md` | 🟠 中 |
| 18 | `zh-CN/diagnostics/flags.md` | 🟡 低 |
| 19 | `zh-CN/environment.md` | 🟠 中 |
| 20 | `zh-CN/logging.md` | 🟠 中 |
| 21 | `zh-CN/multi-agent-sandbox-tools.md` | 🟠 中 |
| 22 | `zh-CN/network.md` | 🟠 中 |
| 23 | `zh-CN/perplexity.md` | 🟡 低 |
| 24 | `zh-CN/prose.md` | 🟡 低 |
| 25 | `zh-CN/scripts.md` | 🔴 高 |
| 26 | `zh-CN/testing.md` | 🟠 高 |
| 27 | `zh-CN/token-use.md` | 🟠 中 |
| 28 | `zh-CN/vps.md` | 🔴 高 |

---

## 第四部分：特殊要求

### 4.1 新手友好增强

**基础文档必须包含**：

| 要求 | 说明 | 示例 |
|------|------|------|
| "为什么需要这个" | 解释功能的价值和必要性 | "TTS 让助手能够用语音回复..." |
| 类比和比喻 | 用熟悉的概念解释新概念 | "网关就像交通枢纽..." |
| 常见错误 | 列出新手容易犯的错误 | "忘记重启 Gateway..." |
| 学习检查点 | 阶段性验证理解 | "现在你应该能够..." |

### 4.2 底层原理深度

**概念文档必须包含**：

| 要求 | 说明 | 示例 |
|------|------|------|
| 架构图 | 可视化展示结构 | 组件关系图 |
| 数据流说明 | 解释信息如何流动 | 请求处理流程 |
| 设计决策解释 | 解释为什么这样设计 | "选择 HTTP 而非 WebSocket..." |
| 源码引用 | 指向关键代码位置 | `src/gateway/...` |

### 4.3 操作文档必须包含**：

| 要求 | 说明 | 示例 |
|------|------|------|
| 完整配置示例 | 可直接使用的配置 | JSON/YAML 代码块 |
| 实际使用场景 | 真实的应用场景 | "当你想监控服务器时..." |
| 故障排查清单 | 常见问题及解决方案 | "如果连接失败，检查..." |
| 最佳实践总结 | 专家建议 | "建议使用密钥而非密码..." |

---

## 第五部分：执行策略

### 5.1 并行翻译优先级

```
翻译任务优先级矩阵：

                    紧急度
                    低 ────────── 高
              ┌─────────────────────────────────┐
        高    │  Phase 1     │  Phase 2       │
   重          │  基础入门     │  核心概念      │
   要          │  ─────────    │  ─────────     │
   度          │  30篇        │  35篇         │
              ├─────────────────────────────────┤
        低    │  Phase 9     │  Phase 3       │
              │  其他文档     │  CLI + 工具    │
              │  ─────────    │  ─────────     │
              │  81篇        │  63篇         │
              └─────────────────────────────────┘
```

**推荐执行顺序**：

| 批次 | 包含阶段 | 优先级 | 预计工作量 |
|------|----------|--------|------------|
| 批次 1 | Phase 1 | 🔴 最高 | 25 篇 |
| 批次 2 | Phase 2 | 🔴 高 | 35 篇 |
| 批次 3 | Phase 3 | 🟠 中 | 63 篇 |
| 批次 4 | Phase 4 | 🟠 中 | 22 篇 |
| 批次 5 | Phase 5-6 | 🟡 低 | 65 篇 |
| 批次 6 | Phase 7-9 | 🟡 低 | 81 篇 |

### 5.2 质量检查清单

**每个阶段完成后必须执行**：

| 检查项 | 说明 | 通过标准 |
|--------|------|----------|
| 链接检查 | 确保所有内部链接有效 | 无死链 |
| 术语一致性 | 确认术语翻译统一 | 术语表对照 |
| 代码示例验证 | 测试代码片段可运行 | 无语法错误 |
| 格式规范检查 | 符合文档格式标准 | 清单验证通过 |

---

## 总结与进阶

### 知识点回顾

本章节的核心知识点包括：

1. **四阶段翻译策略**：从入门到高级的渐进式覆盖
2. **文档分类体系**：按功能和优先级组织
3. **依赖关系**：理解文档间的阅读路径
4. **质量要求**：新手友好、原理深度、实用示例

### 学习成果检验

完成本章节学习后，你应该能够：

- [ ] 说出翻译计划的整体结构和各阶段目标
- [ ] 识别当前已完成的文档和待翻译的文档
- [ ] 理解文档优先级排序的原理
- [ ] 掌握翻译质量检查的关键点

### 下一步学习

- **参与翻译**：选择感兴趣的文档开始翻译
- **术语管理**：[术语表](/references/terminology.md) — 统一翻译标准
- **格式规范**：[文档风格指南](/references/style.md) — 翻译格式要求

---

## 附录

### 文档状态速查表

| 状态 | 含义 | 数量 |
|------|------|------|
| ✅ 已完成 | 翻译完成并发布 | 30 篇 |
| 🔴 待翻译（高优先级） | 核心文档 | ~80 篇 |
| 🟠 待翻译（中优先级） | 重要文档 | ~100 篇 |
| 🟡 待翻译（低优先级） | 专业文档 | ~89 篇 |

### 翻译进度追踪

```
当前进度条：

Phase 1: ████████████████████ 22/25 (88%)
Phase 2: ████████████████████ 5/35 (14%)
Phase 3: ████████████████████ 0/63 (0%)
Phase 4: ████████████████████ 7/22 (32%)
Phase 5: ████████████████████ 4/32 (13%)
Phase 6: ████████████████████ 0/38 (0%)
Phase 7: ████████████████████ 0/15 (0%)
Phase 8: ████████████████████ 5/12 (42%)
Phase 9: ████████████████████ 0/48 (0%)

总计：███████████████ 30/299 (10%)
```

---

_文档版本：v3.0_
_最后更新：2026-02-05_
