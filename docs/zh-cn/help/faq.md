---
summary: "OpenClaw 常见问题解答：安装、配置、使用、故障排除"
read_when:
  - 遇到问题需要快速答案
  - 想要深入了解某个功能
  - 寻找最佳实践
title: "常见问题 (FAQ)"
---

# ❓ 常见问题 (FAQ)

快速答案 + 真实场景的深度故障排除（本地开发、VPS、多代理、OAuth/API Key、模型故障转移）。

运行时诊断请参见 [故障排除](/zh-CN/gateway/troubleshooting)。完整配置参考请参见 [配置](/zh-CN/gateway/configuration)。

---

## 📑 目录

### 🚀 快速开始与首次设置
- [我卡住了，最快的解决方法是什么？](#我卡住了最快的解决方法是什么)
- [推荐的安装和设置方式是什么？](#推荐的安装和设置方式是什么)
- [引导后如何打开仪表盘？](#引导后如何打开仪表盘)
- [需要什么运行时？](#需要什么运行时)
- [它能在树莓派上运行吗？](#它能在树莓派上运行吗)
- [安装通常需要多长时间？](#安装通常需要多长时间)

### 🤖 什么是 OpenClaw？
- [一句话解释 OpenClaw？](#一句话解释-openclaw)
- [主要用途是什么？](#主要用途是什么)
- [刚设置好，应该先做什么？](#刚设置好应该先做什么)

### 🔧 配置基础
- [配置是什么格式？在哪里？](#配置是什么格式在哪里)
- [修改配置后需要重启吗？](#修改配置后需要重启吗)
- [如何启用网页搜索？](#如何启用网页搜索)
- [最小的"合理"配置是什么样的？](#最小的合理配置是什么样的)

### 🔐 认证与模型
- [需要 Claude 或 OpenAI 订阅吗？](#需要-claude-或-openai-订阅吗)
- [Anthropic "setup-token" 认证如何工作？](#anthropic-setup-token-认证如何工作)
- [支持 Claude 订阅认证吗？](#支持-claude-订阅认证吗)
- [为什么会出现 HTTP 429 速率限制错误？](#为什么会出现-http-429-速率限制错误)
- [支持 AWS Bedrock 吗？](#支持-aws-bedrock-吗)
- [可以使用本地模型吗？](#可以使用本地模型吗)

### 💬 渠道与消息
- [Telegram 的 `allowFrom` 填什么？](#telegram-的-allowfrom-填什么)
- [WhatsApp 群组需要添加机器人账号吗？](#whatsapp-群组需要添加机器人账号吗)
- [为什么 OpenClaw 在群组中不回复？](#为什么-openclaw-在群组中不回复)
- [群组/线程与 DM 共享上下文吗？](#群组线程与-dm-共享上下文吗)

### 🛠️ 技能与自动化
- [如何自定义技能？](#如何自定义技能)
- [可以从自定义文件夹加载技能吗？](#可以从自定义文件夹加载技能吗)
- [如何为不同任务使用不同模型？](#如何为不同任务使用不同模型)
- [机器人做重活时卡住怎么办？](#机器人做重活时卡住怎么办)
- [Cron 或提醒不触发怎么办？](#cron-或提醒不触发怎么办)

### 🔒 沙箱与记忆
- [有专门的沙箱文档吗？](#有专门的沙箱文档吗)
- [记忆如何工作？](#记忆如何工作)
- [记忆总是忘记事情，如何让记忆持久？](#记忆总是忘记事情如何让记忆持久)
- [语义记忆搜索需要 OpenAI API Key 吗？](#语义记忆搜索需要-openai-api-key-吗)

### 📁 文件位置
- [所有数据都保存在本地吗？](#所有数据都保存在本地吗)
- [OpenClaw 数据存储在哪里？](#openclaw-数据存储在哪里)
- [AGENTS.md / SOUL.md / USER.md / MEMORY.md 应该放在哪里？](#agentsmd-soulmd-usermd-memorymd-应该放在哪里)
- [推荐的备份策略是什么？](#推荐的备份策略是什么)
- [如何完全卸载 OpenClaw？](#如何完全卸载-openclaw)

---

## 🚀 快速开始与首次设置

### 我卡住了，最快的解决方法是什么？

**60秒诊断流程：**

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

如果网关可达，深度探测：
```bash
openclaw status --deep
```

### 推荐的安装和设置方式是什么？

**最快路径：**
1. 使用安装脚本：`curl -fsSL https://openclaw.ai/install.sh | bash`
2. 运行向导：`openclaw onboard --install-daemon`
3. 打开仪表盘：`openclaw dashboard`

最快聊天：无需配置渠道，直接在浏览器控制界面聊天。

### 引导后如何打开仪表盘？

本地默认：http://127.0.0.1:18789/

如果配置了 token，在控制界面设置中粘贴 token。

远程访问：[Web 界面](/zh-CN/web) 和 [Tailscale](/zh-CN/gateway/tailscale)

### 需要什么运行时？

- **Node.js >= 22**（推荐）
- **Bun**（不推荐用于网关，WhatsApp/Telegram 有兼容性问题）

### 它能在树莓派上运行吗？

可以！查看 [树莓派](/zh-CN/platforms/raspberry-pi)。

提示：
- 使用 64 位操作系统
- 至少 2GB RAM
- 外置存储（SD 卡寿命问题）

### 安装通常需要多长时间？

- 安装脚本：2-5 分钟
- 引导向导：5-10 分钟
- 首次渠道配置：5-15 分钟（取决于渠道）

**总计：15-30 分钟**

---

## 🤖 什么是 OpenClaw？

### 一句话解释 OpenClaw

OpenClaw 是**你的个人 AI 助手网关**，连接 WhatsApp、Telegram、Discord 等聊天应用与 Claude、GPT 等 AI，让 AI 通过你熟悉的聊天界面为你服务。

### 主要用途是什么？

**前五大日常用例：**

1. **编程助手** - 通过聊天界面让 AI 帮你写代码、 review PR
2. **知识问答** - 询问任何问题，获得即时回答
3. **任务自动化** - 定时任务、提醒、数据抓取
4. **多平台消息** - 统一回复所有聊天应用的消息
5. **个人知识库** - 存储和检索个人记忆

### 刚设置好，应该先做什么？

**5分钟快速测试：**

1. 打开仪表盘：`openclaw dashboard`
2. 发送测试消息：`你好，OpenClaw！`
3. 观察 AI 是否回复
4. 配置第一个渠道（推荐 Telegram，最简单）
5. 从手机发送消息测试

**第一个技能：**
```bash
openclaw skills install web
```

然后问：`搜索今天的 AI 新闻`

---

## 🔧 配置基础

### 配置是什么格式？在哪里？

**位置**：`~/.openclaw/openclaw.json`

**格式**：JSON5（支持注释、尾随逗号）

**示例**：
```json5
{
  // 网关配置
  gateway: {
    port: 18789,
    bind: 'loopback',
    auth: {
      type: 'token',
      token: 'your-token-here',
    },
  },
  
  // 默认代理配置
  agents: {
    defaults: {
      model: 'anthropic/claude-sonnet-4',
      workspace: '~/.openclaw/workspace',
    },
  },
}
```

### 修改配置后需要重启吗？

**网关配置**：需要重启
```bash
openclaw gateway restart
```

**代理配置**：热重载（无需重启）

### 如何启用网页搜索？

1. 获取 Brave Search API Key：https://api.search.brave.com/app/dashboard
2. 配置：
```bash
openclaw configure --section web
```

或手动编辑配置：
```json5
{
  tools: {
    web: {
      search: {
        apiKey: 'your-brave-api-key',
      },
    },
  },
}
```

`web_fetch` 无需 API Key 即可工作。

### 最小的"合理"配置是什么样的？

```json5
{
  gateway: {
    port: 18789,
    bind: 'loopback',
    auth: {
      type: 'token',
      token: 'auto-generated-or-your-own',
    },
  },
  agents: {
    defaults: {
      model: 'anthropic/claude-sonnet-4',
      workspace: '~/.openclaw/workspace',
    },
  },
}
```

---

## 🔐 认证与模型

### 需要 Claude 或 OpenAI 订阅吗？

**不需要**，但必须有一种认证方式：

| 方式 | 成本 | 推荐度 |
|------|------|--------|
| Anthropic API Key | 按量付费 | ⭐⭐⭐ |
| OpenAI API Key | 按量付费 | ⭐⭐⭐ |
| Claude Code OAuth | 订阅制 | ⭐⭐⭐ |
| OpenAI Codex OAuth | 订阅制 | ⭐⭐⭐ |
| 本地模型 (Ollama) | 免费 | ⭐⭐ |

### Anthropic "setup-token" 认证如何工作？

1. 在任何机器上运行 `claude setup-token`
2. 复制生成的 token
3. 在向导中粘贴 token
4. 向导自动配置

### 支持 Claude 订阅认证吗？

支持！两种方式：

1. **Claude Code CLI OAuth**：向导检查 Keychain/credentials.json
2. **Setup-token**：运行 `claude setup-token` 获取

### 为什么会出现 HTTP 429 速率限制错误？

**原因：**
- 超过了 Anthropic API 速率限制
- 免费 tier 限制更严格
- 并发请求过多

**解决：**
- 等待并重试
- 升级到付费 tier
- 减少并发请求
- 检查是否有循环调用

### 支持 AWS Bedrock 吗？

支持！详见 [Bedrock](/zh-CN/bedrock)。

### 可以使用本地模型吗？

可以！支持：
- Ollama
- llama.cpp
- vLLM

详见 [本地模型](/zh-CN/gateway/local-models) 和 [Ollama 提供商](/zh-CN/providers/ollama)。

**适合**：隐私敏感场景、离线使用、成本控制

**注意**：本地模型性能不如云端，适合简单对话，不推荐复杂编程任务。

---

## 💬 渠道与消息

### Telegram 的 `allowFrom` 填什么？

填你的 Telegram 用户名或 ID：

```json5
{
  channels: {
    telegram: {
      allowFrom: ['your_username', 'friend_username'],
    },
  },
}
```

获取你的 ID：在 Telegram 中发送 `/start` 给 @userinfobot。

### WhatsApp 群组需要添加机器人账号吗？

**不需要！** OpenClaw 使用 WhatsApp Web 协议，通过你登录的 WhatsApp 账号直接发送消息。

### 为什么 OpenClaw 在群组中不回复？

**常见原因：**
1. 未批准配对（首次 DM 需要）
2. 群组不在允许列表中
3. 消息格式不符合触发规则
4. 上下文限制

**检查：**
```bash
# 查看配对状态
openclaw pairing list whatsapp

# 批准配对
openclaw pairing approve whatsapp <code>
```

### 群组/线程与 DM 共享上下文吗？

**默认不共享**。每个会话是独立的：
- DM：私聊上下文
- 群组：群组上下文
- 线程：线程上下文

可以配置共享，但不推荐（隐私问题）。

---

## 🛠️ 技能与自动化

### 如何自定义技能？

**方式一：修改现有技能**
```bash
# 找到技能目录
openclaw skills dir

# 编辑技能文件（在 skills/ 目录下）
```

**方式二：创建新技能**
详见 [创建技能](/zh-CN/tools/creating-skills)。

### 可以从自定义文件夹加载技能吗？

可以！配置 `skills.path`：

```json5
{
  skills: {
    path: '/path/to/your/skills',
  },
}
```

### 如何为不同任务使用不同模型？

配置模型别名和路由：

```json5
{
  models: {
    aliases: {
      fast: 'openai/gpt-4.1-mini',
      coding: 'anthropic/claude-opus-4',
      cheap: 'moonshot/kimi-k2',
    },
  },
  routing: {
    agents: {
      coding: {
        model: 'coding',
      },
    },
  },
}
```

### 机器人做重活时卡住怎么办？

**原因**：同步执行阻塞了消息循环。

**解决**：
1. 使用子代理（`delegate_task`）
2. 启用流式响应
3. 增加超时设置

### Cron 或提醒不触发怎么办？

**检查清单：**
1. 网关是否运行？`openclaw gateway status`
2. 守护进程是否启用？`openclaw cron list`
3. 时区设置是否正确？
4. Cron 表达式是否正确？

查看 [Cron 任务](/zh-CN/automation/cron-jobs)。

---

## 🔒 沙箱与记忆

### 有专门的沙箱文档吗？

有！详见：
- [沙箱](/zh-CN/gateway/sandboxing)
- [沙箱 vs 工具策略 vs 提升权限](/zh-CN/gateway/sandbox-vs-tool-policy-vs-elevated)

### 记忆如何工作？

**两层记忆系统：**

1. **短期记忆（会话上下文）**
   - 当前对话的最近消息
   - LLM 的上下文窗口
   - 自动管理

2. **长期记忆（语义搜索）**
   - 存储在 `memory/` 目录
   - 向量数据库（需要 OpenAI API Key 用于 embedding）
   - 可搜索的历史信息

### 记忆总是忘记事情，如何让记忆持久？

**方法：**
1. 使用 `/remember` 命令（显式存储）
2. 增加 `memory/` 目录的笔记
3. 配置自动记忆提取
4. 定期回顾和整理 MEMORY.md

### 语义记忆搜索需要 OpenAI API Key 吗？

**是的**，用于生成文本的向量 embedding。

如果不想用 OpenAI，可以：
- 使用本地 embedding 模型
- 禁用语义搜索（仅用关键词搜索）

---

## 📁 文件位置

### 所有数据都保存在本地吗？

**是的！** OpenClaw 的设计理念是**数据主权**：
- 所有数据本地存储
- 无需云服务
- 可选：自行选择 AI 提供商（部分可能需要 API 调用）

### OpenClaw 数据存储在哪里？

**默认位置**：`~/.openclaw/`

| 目录/文件 | 用途 |
|-----------|------|
| `openclaw.json` | 主配置文件 |
| `credentials/` | OAuth 凭证 |
| `agents/` | 代理配置和数据 |
| `workspace/` | 默认工作区 |
| `sessions/` | 会话存储 |
| `skills/` | 已安装技能 |
| `logs/` | 日志文件 |

### AGENTS.md / SOUL.md / USER.md / MEMORY.md 应该放在哪里？

**默认工作区**：`~/.openclaw/workspace/`

或代理特定工作区：`~/.openclaw/agents/<agentId>/workspace/`

### 推荐的备份策略是什么？

**备份内容：**
```bash
# 配置文件
cp ~/.openclaw/openclaw.json backup/

# 凭证
cp -r ~/.openclaw/credentials backup/

# 工作区
cp -r ~/.openclaw/workspace backup/

# 会话（可选）
cp -r ~/.openclaw/sessions backup/
```

**自动化备份：**
```bash
# 使用 cron 每天备份
openclaw cron add --name "backup" --schedule "0 2 * * *" --command "tar czf ~/backups/openclaw-$(date +%Y%m%d).tar.gz ~/.openclaw/"
```

### 如何完全卸载 OpenClaw？

详见 [卸载指南](/zh-CN/install/uninstall)。

快速版：
```bash
# 停止服务
openclaw gateway stop

# 卸载 CLI
npm uninstall -g openclaw

# 删除数据（谨慎！）
rm -rf ~/.openclaw
```

---

## 🆘 提交问题

### 当提交 GitHub Issue 时

粘贴安全报告：
```bash
openclaw status --all
```

如果可能，包含相关日志：
```bash
openclaw logs --follow
```

---

**还有问题？** 查看完整文档或加入 [Discord](https://discord.gg/clawd)。
