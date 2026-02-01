# OpenClaw 中文文档翻译计划

## 执行概览

- **总文档数**: 299个英文文档
- **已完成**: 30个中文文档 (10%)
- **待翻译**: 269个文档
- **策略**: 四阶段渐进式翻译 + 结构优化

---

## 📚 第一阶段：基础入门 (Phase 1)
**目标**: 让新用户在30分钟内上手并发送第一条消息

### 核心文档 (25个)

#### 1. 主页与导航
- [x] `zh-CN/index.md` - 中文文档主页 ✓
- [ ] `zh-CN/start/getting-started.md` - 新用户入门 (增强版)
- [ ] `zh-CN/start/wizard.md` - 向导模式详解
- [ ] `zh-CN/start/showcase.md` - 功能展示

#### 2. 安装与配置
- [ ] `zh-CN/install/index.md` - 安装概述
- [ ] `zh-CN/install/installer.md` - 安装器使用
- [ ] `zh-CN/install/node.md` - Node.js 安装
- [ ] `zh-CN/install/docker.md` - Docker 部署
- [ ] `zh-CN/install/nix.md` - Nix 安装
- [ ] `zh-CN/install/ansible.md` - Ansible 自动化
- [ ] `zh-CN/install/migrating.md` - 迁移指南
- [ ] `zh-CN/install/updating.md` - 更新指南
- [ ] `zh-CN/install/uninstall.md` - 卸载指南

#### 3. 快速配置
- [x] `zh-CN/start/quick-start.md` - 5分钟快速上手 ✓
- [x] `zh-CN/start/installation.md` - 详细安装步骤 ✓
- [ ] `zh-CN/start/setup.md` - 初始配置
- [ ] `zh-CN/start/pairing.md` - 设备配对
- [ ] `zh-CN/start/onboarding.md` -  onboarding 流程
- [ ] `zh-CN/start/openclaw.md` - OpenClaw 助手设置

#### 4. 帮助与FAQ
- [ ] `zh-CN/help/index.md` - 帮助中心
- [ ] `zh-CN/help/faq.md` - 常见问题
- [ ] `zh-CN/help/troubleshooting.md` - 故障排除

---

## 📚 第二阶段：核心概念 (Phase 2)
**目标**: 深入理解系统架构与工作原理

### 架构与概念 (35个)

#### 1. 系统架构
- [x] `zh-CN/concepts/architecture.md` - 整体架构 ✓
- [x] `zh-CN/concepts/gateway.md` - 网关系统 ✓
- [ ] `zh-CN/concepts/agents.md` - AI 代理
- [ ] `zh-CN/concepts/agent.md` - 代理详解
- [ ] `zh-CN/concepts/agent-loop.md` - 代理循环
- [ ] `zh-CN/concepts/agent-workspace.md` - 代理工作区

#### 2. 消息与会话
- [x] `zh-CN/concepts/sessions.md` - 会话管理 ✓
- [x] `zh-CN/concepts/routing.md` - 消息路由 ✓
- [ ] `zh-CN/concepts/messages.md` - 消息系统
- [ ] `zh-CN/concepts/context.md` - 上下文管理
- [ ] `zh-CN/concepts/memory.md` - 记忆系统
- [ ] `zh-CN/concepts/session.md` - 会话详解
- [ ] `zh-CN/concepts/session-tool.md` - 会话工具
- [ ] `zh-CN/concepts/session-pruning.md` - 会话修剪
- [ ] `zh-CN/concepts/group-messages.md` - 群组消息
- [ ] `zh-CN/concepts/groups.md` - 群组管理

#### 3. 渠道系统
- [x] `zh-CN/channels/index.md` - 渠道概述 ✓
- [ ] `zh-CN/concepts/channel-routing.md` - 渠道路由

#### 4. 模型与AI
- [ ] `zh-CN/concepts/models.md` - 模型配置
- [ ] `zh-CN/concepts/model-providers.md` - 模型提供商
- [ ] `zh-CN/concepts/model-failover.md` - 模型故障转移
- [ ] `zh-CN/concepts/oauth.md` - OAuth 认证

#### 5. 其他核心概念
- [ ] `zh-CN/concepts/streaming.md` - 流式响应
- [ ] `zh-CN/concepts/presence.md` - 在线状态
- [ ] `zh-CN/concepts/typing-indicators.md` - 输入提示
- [ ] `zh-CN/concepts/retry.md` - 重试机制
- [ ] `zh-CN/concepts/queue.md` - 消息队列
- [ ] `zh-CN/concepts/compaction.md` - 数据压缩
- [ ] `zh-CN/concepts/timezone.md` - 时区处理
- [ ] `zh-CN/concepts/usage-tracking.md` - 用量追踪
- [ ] `zh-CN/concepts/multi-agent.md` - 多代理
- [ ] `zh-CN/concepts/system-prompt.md` - 系统提示词
- [ ] `zh-CN/concepts/markdown-formatting.md` - Markdown 格式
- [ ] `zh-CN/concepts/typebox.md` - TypeBox 类型系统

---

## 📚 第三阶段：功能与工具 (Phase 3)
**目标**: 掌握所有实用功能和工具

### CLI 命令 (41个)

#### 核心命令
- [ ] `zh-CN/cli/index.md` - CLI 概述
- [ ] `zh-CN/cli/onboard.md` -  onboard 命令
- [ ] `zh-CN/cli/gateway.md` -  gateway 命令
- [ ] `zh-CN/cli/config.md` -  config 命令
- [ ] `zh-CN/cli/setup.md` -  setup 命令
- [ ] `zh-CN/cli/doctor.md` -  doctor 命令
- [ ] `zh-CN/cli/status.md` -  status 命令

#### 消息与渠道
- [ ] `zh-CN/cli/message.md` -  message 命令
- [ ] `zh-CN/cli/channels.md` -  channels 命令
- [ ] `zh-CN/cli/pairing.md` -  pairing 命令

#### 代理与会话
- [ ] `zh-CN/cli/agent.md` -  agent 命令
- [ ] `zh-CN/cli/agents.md` -  agents 命令
- [ ] `zh-CN/cli/sessions.md` -  sessions 命令
- [ ] `zh-CN/cli/memory.md` -  memory 命令

#### 插件与技能
- [ ] `zh-CN/cli/plugins.md` -  plugins 命令
- [ ] `zh-CN/cli/skills.md` -  skills 命令

#### 系统工具
- [ ] `zh-CN/cli/health.md` -  health 命令
- [ ] `zh-CN/cli/logs.md` -  logs 命令
- [ ] `zh-CN/cli/update.md` -  update 命令
- [ ] `zh-CN/cli/uninstall.md` -  uninstall 命令
- [ ] `zh-CN/cli/reset.md` -  reset 命令
- [ ] `zh-CN/cli/dns.md` -  dns 命令
- [ ] `zh-CN/cli/tui.md` -  tui 命令
- [ ] `zh-CN/cli/dashboard.md` -  dashboard 命令
- [ ] `zh-CN/cli/cron.md` -  cron 命令
- [ ] `zh-CN/cli/hooks.md` -  hooks 命令
- [ ] `zh-CN/cli/node.md` -  node 命令
- [ ] `zh-CN/cli/nodes.md` -  nodes 命令
- [ ] `zh-CN/cli/devices.md` -  devices 命令
- [ ] `zh-CN/cli/webhooks.md` -  webhooks 命令
- [ ] `zh-CN/cli/security.md` -  security 命令
- [ ] `zh-CN/cli/approvals.md` -  approvals 命令
- [ ] `zh-CN/cli/browser.md` -  browser 命令
- [ ] `zh-CN/cli/sandbox.md` -  sandbox 命令
- [ ] `zh-CN/cli/directory.md` -  directory 命令
- [ ] `zh-CN/cli/system.md` -  system 命令
- [ ] `zh-CN/cli/acp.md` -  acp 命令
- [ ] `zh-CN/cli/voicecall.md` -  voicecall 命令
- [ ] `zh-CN/cli/configure.md` -  configure 命令
- [ ] `zh-CN/cli/docs.md` -  docs 命令

### 工具与技能 (22个)
- [ ] `zh-CN/tools/index.md` - 工具概述
- [ ] `zh-CN/tools/skills.md` - 技能系统
- [ ] `zh-CN/tools/creating-skills.md` - 创建技能
- [ ] `zh-CN/tools/skills-config.md` - 技能配置
- [ ] `zh-CN/tools/slash-commands.md` - 斜杠命令
- [ ] `zh-CN/tools/browser.md` - 浏览器工具
- [ ] `zh-CN/tools/exec.md` - 执行工具
- [ ] `zh-CN/tools/web.md` - Web 工具
- [ ] `zh-CN/tools/thinking.md` - 思考模式
- [ ] `zh-CN/tools/elevated.md` - 提升权限
- [ ] `zh-CN/tools/exec-approvals.md` - 执行审批
- [ ] `zh-CN/tools/subagents.md` - 子代理
- [ ] `zh-CN/tools/reactions.md` - 反应工具
- [ ] `zh-CN/tools/agent-send.md` - 代理发送
- [ ] `zh-CN/tools/clawhub.md` - ClawHub
- [ ] `zh-CN/tools/firecrawl.md` - Firecrawl
- [ ] `zh-CN/tools/apply-patch.md` - 应用补丁
- [ ] `zh-CN/tools/chrome-extension.md` - Chrome 扩展
- [ ] `zh-CN/tools/browser-login.md` - 浏览器登录
- [ ] `zh-CN/tools/browser-linux-troubleshooting.md` - Linux 浏览器故障排除
- [ ] `zh-CN/tools/lobster.md` - Lobster
- [ ] `zh-CN/tools/llm-task.md` - LLM 任务

---

## 📚 第四阶段：渠道配置 (Phase 4)
**目标**: 配置所有消息渠道

### 渠道文档 (22个)
- [x] `zh-CN/channels/whatsapp.md` - WhatsApp ✓
- [x] `zh-CN/channels/telegram.md` - Telegram ✓
- [x] `zh-CN/channels/discord.md` - Discord ✓
- [x] `zh-CN/channels/slack.md` - Slack ✓
- [x] `zh-CN/channels/signal.md` - Signal ✓
- [x] `zh-CN/channels/imessage.md` - iMessage ✓
- [x] `zh-CN/channels/matrix.md` - Matrix ✓
- [ ] `zh-CN/channels/bluebubbles.md` - BlueBubbles
- [ ] `zh-CN/channels/googlechat.md` - Google Chat
- [ ] `zh-CN/channels/grammy.md` - Grammy
- [ ] `zh-CN/channels/line.md` - Line
- [ ] `zh-CN/channels/location.md` - Location
- [ ] `zh-CN/channels/mattermost.md` - Mattermost
- [ ] `zh-CN/channels/msteams.md` - Microsoft Teams
- [ ] `zh-CN/channels/nextcloud-talk.md` - Nextcloud Talk
- [ ] `zh-CN/channels/nostr.md` - Nostr
- [ ] `zh-CN/channels/tlon.md` - Tlon
- [ ] `zh-CN/channels/twitch.md` - Twitch
- [ ] `zh-CN/channels/zalo.md` - Zalo
- [ ] `zh-CN/channels/zalouser.md` - Zalo User
- [ ] `zh-CN/channels/troubleshooting.md` - 渠道故障排除

---

## 📚 第五阶段：网关与运维 (Phase 5)
**目标**: 掌握网关配置与生产运维

### 网关文档 (27个)
- [ ] `zh-CN/gateway/index.md` - 网关概述
- [ ] `zh-CN/gateway/configuration.md` - 配置详解
- [ ] `zh-CN/gateway/configuration-examples.md` - 配置示例
- [ ] `zh-CN/gateway/protocol.md` - 协议说明
- [ ] `zh-CN/gateway/authentication.md` - 认证机制
- [ ] `zh-CN/gateway/pairing.md` - 配对流程
- [ ] `zh-CN/gateway/discovery.md` - 服务发现
- [ ] `zh-CN/gateway/remote.md` - 远程访问
- [ ] `zh-CN/gateway/tailscale.md` - Tailscale 集成
- [ ] `zh-CN/gateway/health.md` - 健康检查
- [ ] `zh-CN/gateway/heartbeat.md` - 心跳机制
- [ ] `zh-CN/gateway/logging.md` - 日志系统
- [ ] `zh-CN/gateway/doctor.md` - 诊断工具
- [ ] `zh-CN/gateway/troubleshooting.md` - 故障排除
- [ ] `zh-CN/gateway/background-process.md` - 后台进程
- [ ] `zh-CN/gateway/bonjour.md` - Bonjour 服务
- [ ] `zh-CN/gateway/bridge-protocol.md` - 桥接协议
- [ ] `zh-CN/gateway/cli-backends.md` - CLI 后端
- [ ] `zh-CN/gateway/multiple-gateways.md` - 多网关
- [ ] `zh-CN/gateway/gateway-lock.md` - 网关锁定
- [ ] `zh-CN/gateway/local-models.md` - 本地模型
- [ ] `zh-CN/gateway/openai-http-api.md` - OpenAI HTTP API
- [ ] `zh-CN/gateway/openresponses-http-api.md` - OpenResponses API
- [ ] `zh-CN/gateway/tools-invoke-http-api.md` - 工具调用 API
- [ ] `zh-CN/gateway/sandboxing.md` - 沙箱
- [ ] `zh-CN/gateway/sandbox-vs-tool-policy-vs-elevated.md` - 权限对比
- [ ] `zh-CN/gateway/remote-gateway-readme.md` - 远程网关说明

### 运维文档 (5个已有，需完善)
- [x] `zh-CN/operations/index.md` - 运维概述 ✓
- [x] `zh-CN/operations/deployment.md` - 部署指南 ✓
- [x] `zh-CN/operations/monitoring.md` - 监控 ✓
- [x] `zh-CN/operations/troubleshooting.md` - 故障排除 ✓

---

## 📚 第六阶段：提供商与平台 (Phase 6)
**目标**: 配置AI模型与部署平台

### AI提供商 (21个)
- [ ] `zh-CN/providers/index.md` - 提供商概述
- [ ] `zh-CN/providers/anthropic.md` - Anthropic
- [ ] `zh-CN/providers/openai.md` - OpenAI
- [ ] `zh-CN/providers/glm.md` - GLM
- [ ] `zh-CN/providers/moonshot.md` - Moonshot
- [ ] `zh-CN/providers/minimax.md` - MiniMax
- [ ] `zh-CN/providers/xiaomi.md` - Xiaomi
- [ ] `zh-CN/providers/qwen.md` - Qwen
- [ ] `zh-CN/providers/opencode.md` - Opencode
- [ ] `zh-CN/providers/openrouter.md` - OpenRouter
- [ ] `zh-CN/providers/ollama.md` - Ollama
- [ ] `zh-CN/providers/github-copilot.md` - GitHub Copilot
- [ ] `zh-CN/providers/deepgram.md` - Deepgram
- [ ] `zh-CN/providers/vercel-ai-gateway.md` - Vercel AI Gateway
- [ ] `zh-CN/providers/synthetic.md` - Synthetic
- [ ] `zh-CN/providers/venice.md` - Venice
- [ ] `zh-CN/providers/zai.md` - Z.ai
- [ ] `zh-CN/providers/models.md` - 模型参考
- [ ] `zh-CN/providers/claude-max-api-proxy.md` - Claude Max API 代理

### 部署平台 (17个)
- [ ] `zh-CN/platforms/index.md` - 平台概述
- [ ] `zh-CN/platforms/macos.md` - macOS
- [ ] `zh-CN/platforms/linux.md` - Linux
- [ ] `zh-CN/platforms/windows.md` - Windows
- [ ] `zh-CN/platforms/ios.md` - iOS
- [ ] `zh-CN/platforms/android.md` - Android
- [ ] `zh-CN/platforms/raspberry-pi.md` - Raspberry Pi
- [ ] `zh-CN/platforms/docker.md` - Docker
- [ ] `zh-CN/platforms/fly.md` - Fly.io
- [ ] `zh-CN/platforms/gcp.md` - Google Cloud
- [ ] `zh-CN/platforms/digitalocean.md` - DigitalOcean
- [ ] `zh-CN/platforms/hetzner.md` - Hetzner
- [ ] `zh-CN/platforms/oracle.md` - Oracle Cloud
- [ ] `zh-CN/platforms/exe-dev.md` - exe.dev
- [ ] `zh-CN/platforms/macos-vm.md` - macOS VM

### macOS 专项 (17个)
- [ ] `zh-CN/platforms/mac/bundled-gateway.md`
- [ ] `zh-CN/platforms/mac/canvas.md`
- [ ] `zh-CN/platforms/mac/child-process.md`
- [ ] `zh-CN/platforms/mac/dev-setup.md`
- [ ] `zh-CN/platforms/mac/health.md`
- [ ] `zh-CN/platforms/mac/icon.md`
- [ ] `zh-CN/platforms/mac/logging.md`
- [ ] `zh-CN/platforms/mac/menu-bar.md`
- [ ] `zh-CN/platforms/mac/peekaboo.md`
- [ ] `zh-CN/platforms/mac/permissions.md`
- [ ] `zh-CN/platforms/mac/release.md`
- [ ] `zh-CN/platforms/mac/remote.md`
- [ ] `zh-CN/platforms/mac/signing.md`
- [ ] `zh-CN/platforms/mac/skills.md`
- [ ] `zh-CN/platforms/mac/voice-overlay.md`
- [ ] `zh-CN/platforms/mac/voicewake.md`
- [ ] `zh-CN/platforms/mac/webchat.md`
- [ ] `zh-CN/platforms/mac/xpc.md`

---

## 📚 第七阶段：节点与自动化 (Phase 7)
**目标**: 移动设备与自动化工作流

### 节点 (9个)
- [ ] `zh-CN/nodes/index.md` - 节点概述
- [ ] `zh-CN/nodes/audio.md` - 音频节点
- [ ] `zh-CN/nodes/camera.md` - 相机节点
- [ ] `zh-CN/nodes/images.md` - 图片节点
- [ ] `zh-CN/nodes/location-command.md` - 位置命令
- [ ] `zh-CN/nodes/media-understanding.md` - 媒体理解
- [ ] `zh-CN/nodes/talk.md` - 语音对话
- [ ] `zh-CN/nodes/voicewake.md` - 语音唤醒

### 自动化 (6个)
- [ ] `zh-CN/automation/cron-jobs.md` - Cron 任务
- [ ] `zh-CN/automation/cron-vs-heartbeat.md` - Cron vs 心跳
- [ ] `zh-CN/automation/webhook.md` - Webhook
- [ ] `zh-CN/automation/poll.md` - 轮询
- [ ] `zh-CN/automation/auth-monitoring.md` - 认证监控
- [ ] `zh-CN/automation/gmail-pubsub.md` - Gmail Pub/Sub

---

## 📚 第八阶段：开发者文档 (Phase 8)
**目标**: 二次开发与贡献

### 开发者文档 (5个已有)
- [x] `zh-CN/developer/index.md` - 开发概述 ✓
- [x] `zh-CN/developer/project-structure.md` - 项目结构 ✓
- [x] `zh-CN/developer/plugin-development.md` - 插件开发 ✓
- [x] `zh-CN/developer/testing.md` - 测试指南 ✓
- [x] `zh-CN/developer/contributing.md` - 贡献指南 ✓

### 参考文档
- [ ] `zh-CN/reference/RELEASING.md` - 发布流程
- [ ] `zh-CN/reference/rpc.md` - RPC 协议
- [ ] `zh-CN/reference/api-usage-costs.md` - API 成本
- [ ] `zh-CN/reference/device-models.md` - 设备模型
- [ ] `zh-CN/reference/test.md` - 测试规范
- [ ] `zh-CN/reference/session-management-compaction.md` - 会话管理
- [ ] `zh-CN/reference/transcript-hygiene.md` - 记录卫生

### 模板文件
- [ ] `zh-CN/reference/templates/AGENTS.md`
- [ ] `zh-CN/reference/templates/BOOTSTRAP.md`
- [ ] `zh-CN/reference/templates/HEARTBEAT.md`
- [ ] `zh-CN/reference/templates/IDENTITY.md`
- [ ] `zh-CN/reference/templates/SOUL.md`
- [ ] `zh-CN/reference/templates/TOOLS.md`
- [ ] `zh-CN/reference/templates/USER.md`

---

## 📚 第九阶段：其他重要文档 (Phase 9)

### Web 界面
- [ ] `zh-CN/web/index.md` - Web 概述
- [ ] `zh-CN/web/dashboard.md` - 仪表盘
- [ ] `zh-CN/web/control-ui.md` - 控制界面
- [ ] `zh-CN/web/webchat.md` - Web 聊天

### 配置与安全
- [x] `zh-CN/config/index.md` - 配置概述 ✓
- [x] `zh-CN/config/reference.md` - 配置参考 ✓
- [x] `zh-CN/config/examples.md` - 配置示例 ✓
- [ ] `zh-CN/security/formal-verification.md` - 形式化验证

### 杂项
- [ ] `zh-CN/hooks.md` - Hooks
- [ ] `zh-CN/hooks/soul-evil.md` - Soul Evil
- [ ] `zh-CN/plugin.md` - 插件
- [ ] `zh-CN/plugins/agent-tools.md` - 代理工具
- [ ] `zh-CN/plugins/manifest.md` - 清单
- [ ] `zh-CN/plugins/voice-call.md` - 语音通话
- [ ] `zh-CN/plugins/zalouser.md` - Zalo User
- [ ] `zh-CN/tts.md` - TTS
- [ ] `zh-CN/tui.md` - TUI
- [ ] `zh-CN/pi.md` - Pi
- [ ] `zh-CN/pi-dev.md` - Pi 开发
- [ ] `zh-CN/bedrock.md` - Bedrock
- [ ] `zh-CN/brave-search.md` - Brave Search
- [ ] `zh-CN/broadcast-groups.md` - 广播组
- [ ] `zh-CN/date-time.md` - 日期时间
- [ ] `zh-CN/debugging.md` - 调试
- [ ] `zh-CN/debug/node-issue.md` - 节点问题
- [ ] `zh-CN/diagnostics/flags.md` - 诊断标志
- [ ] `zh-CN/environment.md` - 环境
- [ ] `zh-CN/logging.md` - 日志
- [ ] `zh-CN/multi-agent-sandbox-tools.md` - 多代理沙箱
- [ ] `zh-CN/network.md` - 网络
- [ ] `zh-CN/perplexity.md` - Perplexity
- [ ] `zh-CN/prose.md` - Prose
- [ ] `zh-CN/scripts.md` - 脚本
- [ ] `zh-CN/testing.md` - 测试
- [ ] `zh-CN/token-use.md` - Token 使用
- [ ] `zh-CN/vps.md` - VPS

### 重构相关
- [ ] `zh-CN/refactor/clawnet.md`
- [ ] `zh-CN/refactor/exec-host.md`
- [ ] `zh-CN/refactor/outbound-session-mirroring.md`
- [ ] `zh-CN/refactor/plugin-sdk.md`
- [ ] `zh-CN/refactor/strict-config.md`

### 实验性
- [ ] `zh-CN/experiments/onboarding-config-protocol.md`
- [ ] `zh-CN/experiments/plans/cron-add-hardening.md`
- [ ] `zh-CN/experiments/plans/group-policy-hardening.md`
- [ ] `zh-CN/experiments/plans/openresponses-gateway.md`
- [ ] `zh-CN/experiments/proposals/model-config.md`
- [ ] `zh-CN/experiments/research/memory.md`

---

## 📝 特殊要求

### 1. 新手友好增强
每个基础文档需要：
- 添加"为什么需要这个"解释
- 提供类比和比喻
- 包含常见错误和解决方案
- 提供学习检查点

### 2. 底层原理深度
概念文档需要：
- 架构图和流程图
- 数据流说明
- 设计决策解释
- 源码引用（关键部分）

### 3. 实用示例
操作文档需要：
- 完整的配置示例
- 实际使用场景
- 故障排查清单
- 最佳实践总结

---

## 🎯 执行策略

### 并行翻译
1. **Phase 1-2** (基础+概念): 60个文档，最高优先级
2. **Phase 3-4** (CLI+渠道): 63个文档，高优先级
3. **Phase 5-6** (网关+提供商): 65个文档，中优先级
4. **Phase 7-9** (其他): 81个文档，低优先级

### 质量检查点
每个阶段完成后：
- [ ] 链接检查
- [ ] 术语一致性
- [ ] 代码示例验证
- [ ] 格式规范检查

---

**计划创建完成！准备开始执行 Phase 1.**
