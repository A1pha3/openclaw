---
summary: "OpenClaw 故障排除中心：症状 → 检查 → 修复的系统性排查指南"
read_when:
  - 看到错误信息需要修复路径
  - 安装程序显示成功但 CLI 不工作
  - 系统不正常工作
title: "故障排除指南"
---

# 🔧 故障排除指南

本文档提供系统性的故障排除方法，帮助你快速定位和解决 OpenClaw 遇到的问题。我们采用「症状 → 检查 → 修复」的方法论，让你能够高效地解决 80% 的常见问题。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 掌握黄金 60 秒诊断流程
- [ ] 能够识别常见错误症状
- [ ] 理解故障排除的基本步骤
- [ ] 能够使用命令行工具收集诊断信息

### 进阶目标（建议掌握）

- [ ] 能够进行详细的故障隔离和分析
- [ ] 掌握渠道特定问题的排查方法
- [ ] 能够准备高质量的问题报告
- [ ] 理解常见错误的根本原因

### 专家目标（挑战）

- [ ] 能够设计预防性监控策略
- [ ] 能够建立团队级的故障响应流程
- [ ] 理解系统架构层面的问题排查
- [ ] 能够优化故障恢复时间

---

## ⏱️ 第一部分：黄金 60 秒诊断

### 1.1 为什么需要快速诊断？

**原理说明**：80% 的问题可以通过标准化的快速诊断流程解决。OpenClaw 提供了一系列诊断命令，按照「从外到内、从简单到复杂」的顺序检查系统。

**黄金 60 秒流程**：

```bash
# 第一步：查看整体状态（5 秒）
# 目的：确认网关是否运行、基本可达性
openclaw status

# 第二步：查看详细状态（10 秒）
# 目的：获取完整配置和状态信息
openclaw status --all

# 第三步：探测网关（10 秒）
# 目的：测试网关响应能力
openclaw gateway probe

# 第四步：查看实时日志（20 秒）
# 目的：观察实时运行信息
openclaw logs --follow

# 第五步：运行诊断工具（15 秒）
# 目的：执行全面健康检查
openclaw doctor
```

**流程图解**：

```
开始
  │
  ▼
┌────────────────────────────────────┐
│ 第一步：openclaw status           │
│ 检查：网关是否运行？               │
│ 结果：运行中 ──→ 第二步           │
│ 结果：未运行 ──→ 启动网关         │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ 第二步：openclaw status --all     │
│ 检查：详细配置和状态               │
│ 目的：获取完整信息用于后续分析      │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ 第三步：openclaw gateway probe     │
│ 检查：网关响应能力                 │
│ 目的：确认网络可达性               │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ 第四步：openclaw logs --follow    │
│ 检查：实时日志输出                 │
│ 目的：发现运行时错误和警告         │
└────────────────┬───────────────────┘
                 │
                 ▼
┌────────────────────────────────────┐
│ 第五步：openclaw doctor           │
│ 执行：全面健康检查                │
│ 目的：自动发现和修复常见问题       │
└────────────────┬───────────────────┘
                 │
                 ▼
完成 ──→ 根据输出进行针对性修复
```

**适用场景**：

| 症状 | 首选诊断命令 | 预期发现 |
|------|--------------|----------|
| CLI 不响应 | `openclaw status` | 网关运行状态 |
| 配置问题 | `status --all` | 配置错误 |
| 连接失败 | `gateway probe` | 网络问题 |
| 运行时错误 | `logs --follow` | 错误日志 |
| 复杂问题 | `doctor` | 综合诊断 |

**进阶诊断**：如果网关可达但需要深入排查，运行深度探测：

```bash
openclaw status --deep
```

**专家思维模型**：采用「分层诊断」策略，从最外层（CLI 可达性）逐步深入到最内层（具体错误原因）。每一步都基于前一步的结果进行决策。

---

## 🚨 第二部分：常见「坏了」场景

### 2.1 `openclaw: command not found`

**错误表现**：

```
bash: openclaw: command not found
```

**问题本质**：几乎总是 Node.js/npm PATH 问题。npm 存放全局二进制文件的目录不在你的 shell PATH 中。

**诊断流程**：

```bash
# 第一步：检查 Node 和 npm 是否可用
node -v
npm -v

# 第二步：查看 npm 全局路径
npm prefix -g

# 第三步：检查当前 PATH
echo "$PATH"
```

**输出示例与判断**：

| 场景 | node -v | npm -v | npm prefix -g | 问题 |
|------|---------|--------|---------------|------|
| Node 未安装 | command not found | command not found | - | Node 未安装 |
| npm 可用 | v22.0.0 | 10.0.0 | /usr/local | PATH 问题 |
| 全部正常 | v22.0.0 | 10.0.0 | /opt/homebrew | 其他问题 |

**解决方案**：

**快速修复（zsh）**：

```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**详细解决方案**：参见 [Node.js + npm PATH 配置文档](/zh-CN/install/node)。

**适用场景**：

| 场景 | 推荐解决方案 |
|------|--------------|
| 新安装后 | 添加 PATH |
| 使用 nvm | 初始化 nvm |
| 权限问题 | 配置 npm prefix |

---

### 2.2 安装程序失败

**错误表现**：安装脚本执行过程中途失败，输出错误信息或卡住。

**重新运行详细日志模式**：

```bash
# 详细模式重新运行
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

**Beta 版本测试**：

```bash
# 安装测试版
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

**环境变量方式**（适合 CI/自动化）：

```bash
export OPENCLAW_VERBOSE=1
curl -fsSL https://openclaw.ai/install.sh | bash
```

**常见安装错误**：

| 错误类型 | 表现 | 解决方式 |
|----------|------|----------|
| 网络超时 | Connection timeout | 检查网络、重试 |
| 权限不足 | EACCES permission denied | 使用 sudo 或配置 npm prefix |
| Node 版本不兼容 | Version mismatch | 升级到 Node.js 22+ |
| 磁盘空间不足 | No space left | 清理磁盘空间 |

**诊断信息收集**：

```bash
# 保存完整日志用于排查
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose 2>&1 | tee install.log

# 检查系统资源
df -h           # 磁盘空间
free -m         # 内存
node -v         # Node 版本
```

---

### 2.3 网关「未授权」、无法连接或不断重连

**错误表现**：

```
Error: Gateway not authorized
Error: Connection refused
Error: Reconnecting...
```

**问题分类**：

| 错误类型 | 原因 | 解决路径 |
|----------|------|----------|
| 未授权 | Token 无效或缺失 | 检查认证配置 |
| 连接拒绝 | 网关未运行 | 启动网关 |
| 不断重连 | 网络不稳定 | 检查网络配置 |

**诊断步骤**：

```bash
# 第一步：检查网关是否运行
openclaw status

# 第二步：查看网关日志
openclaw logs --follow

# 第三步：检查认证配置
openclaw config get gateway.auth

# 第四步：重新启动网关
openclaw gateway restart
```

**详细文档**：参见 [网关故障排除](/zh-CN/gateway/troubleshooting) 和 [网关认证](/zh-CN/gateway/authentication)。

---

### 2.4 控制界面在 HTTP 上失败

**错误表现**：

```
Access Denied: Device authentication required
```

**问题原因**：控制界面需要设备身份验证，无法通过不安全的 HTTP 访问。

**解决方案**：

| 场景 | 解决方案 |
|------|----------|
| 本地访问 | 使用 token 认证 |
| 远程访问 | 配置 HTTPS |
| 开发测试 | 临时禁用认证（不推荐） |

**配置认证**：

```json5
{
  gateway: {
    auth: {
      mode: "token",
      token: "your-gateway-token",
    },
  },
}
```

**详细文档**：参见 [控制界面文档](/zh-CN/web/control-ui)。

---

### 2.5 `docs.openclaw.ai` 显示 SSL 错误

**错误表现**：

```
SSL Certificate Error
Your connection is not private
```

**问题原因**：某些 Comcast/Xfinity 连接通过 Xfinity Advanced Security 阻止 `docs.openclaw.ai`。

**解决方案**：

1. **禁用 Advanced Security**（临时）
2. **将 docs.openclaw.ai 添加到允许列表**
3. **使用手机热点或 VPN**（验证是否为 ISP 过滤）

**验证诊断**：

```bash
# 使用 curl 测试连接
curl -I https://docs.openclaw.ai

# 如果 ISP 过滤，使用 VPN 验证
# 如果 VPN 正常，说明是 ISP 问题
```

**Xfinity Advanced Security 帮助**：https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security

---

### 2.6 服务显示运行中，但 RPC 探测失败

**错误表现**：

```
Status: Running
RPC Probe: Failed
```

**诊断流程**：

```bash
# 检查端口占用
lsof -i :18789

# 检查进程状态
ps aux | grep openclaw

# 检查系统服务
openclaw gateway status
```

**常见原因**：

| 原因 | 检查方法 | 解决方式 |
|------|----------|----------|
| 端口被占用 | `lsof -i :18789` | 释放端口或换端口 |
| 进程崩溃 | `ps aux | grep openclaw` | 重启进程 |
| 服务配置错误 | `openclaw doctor` | 运行诊断 |

**详细文档**：参见 [后台进程/服务文档](/zh-CN/gateway/background-process)。

---

### 2.7 模型/认证失败

**错误表现**：

```
Error: Rate limit exceeded
Error: Billing required
Error: All models failed
```

**分类排查**：

| 错误类型 | 原因 | 解决方式 |
|----------|------|----------|
| 速率限制 | 超过 API 限制 | 等待或升级套餐 |
| 计费问题 | 账户余额不足 | 充值或设置限额 |
| 模型失败 | 配置错误 | 检查模型配置 |

**诊断命令**：

```bash
# 检查模型状态
openclaw models status

# 检查认证配置
openclaw config get agents.defaults

# 查看详细错误
openclaw logs --lines 100
```

**详细文档**：参见 [模型命令](/zh-CN/cli/models) 和 [OAuth/认证概念](/zh-CN/concepts/oauth)。

---

### 2.8 `/model` 显示 `model not allowed`

**错误表现**：

```
Error: model not allowed
```

**问题原因**：配置了模型允许列表，当它非空时，只能选择列表中的模型。

**诊断与解决**：

```bash
# 检查当前允许的模型列表
openclaw config get agents.defaults.models

# 解决方案一：添加你想要的模型
openclaw config set agents.defaults.models "['anthropic/claude-sonnet-4']"

# 解决方案二：清除允许列表
openclaw config delete agents.defaults.models

# 查看所有允许的提供商和模型
/models
```

**配置说明**：

| 配置状态 | 行为 |
|----------|------|
| 未配置 | 可用所有支持的模型 |
| 空列表 | 可用所有支持的模型 |
| 有内容 | 只能使用列表中的模型 |

---

## 🔍 第三部分：详细故障排除流程

### 3.1 第一步：收集信息

**原则**：信息收集是故障排除的第一步，完整的诊断信息可以大幅缩短排查时间。

**收集命令**：

```bash
# 系统信息
openclaw status --all > debug-info.txt

# 网关日志
openclaw logs --lines 100 >> debug-info.txt

# 配置（去敏）
openclaw config get >> debug-info.txt

# 健康检查
openclaw health >> debug-info.txt 2>&1
```

**收集内容清单**：

| 信息类型 | 命令 | 重要性 |
|----------|------|--------|
| 系统状态 | `status --all` | ✅ 必需 |
| 配置信息 | `config get` | ✅ 必需 |
| 错误日志 | `logs --lines 100` | ✅ 推荐 |
| 健康报告 | `health` | ✅ 推荐 |

**信息去敏**：提交问题前，去除敏感信息（如 API Key）。

---

### 3.2 第二步：隔离问题

**原则**：将问题隔离到最小范围，定位根本原因。

**症状 → 检查项映射表**：

| 症状 | 检查项 | 命令 |
|------|--------|------|
| CLI 不响应 | Node/npm 安装 | `node -v && npm -v` |
| 网关不启动 | 端口占用 | `lsof -i :18789` |
| 无法连接 | 防火墙 | `sudo ufw status` |
| 认证失败 | 凭证配置 | `openclaw config get agents.defaults` |
| 渠道不工作 | 渠道状态 | `openclaw channels status` |
| 模型不响应 | 模型状态 | `openclaw models status` |

**隔离策略**：

```
复杂问题
  │
  ├── 子问题 1：CLI 是否正常？
  │       └── 测试：openclaw --version
  │
  ├── 子问题 2：网关是否运行？
  │       └── 测试：openclaw status
  │
  ├── 子问题 3：配置是否正确？
  │       └── 测试：openclaw config get
  │
  └── 子问题 4：渠道是否正常？
          └── 测试：openclaw channels status
```

---

### 3.3 第三步：常见修复

**原则**：从最简单的修复开始，逐步尝试更复杂的解决方案。

### 修复一：重启网关

```bash
openclaw gateway restart
```

**适用场景**：

| 场景 | 是否有效 |
|------|----------|
| 配置修改后 | ✅ 有效 |
| 内存泄漏 | ✅ 暂时有效 |
| 网络问题 | ⚠️ 视情况 |
| 认证过期 | ❌ 无效 |

### 修复二：重置配置

```bash
# 备份当前配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 重置向导
openclaw onboard --reset
```

**适用场景**：

| 场景 | 是否有效 |
|------|----------|
| 配置损坏 | ✅ 有效 |
| 引导问题 | ✅ 有效 |
| 不想丢失数据 | ❌ 需要备份 |

### 修复三：清除缓存

```bash
# 清除会话缓存
openclaw sessions clear

# 清除日志
openclaw logs clear
```

**适用场景**：

| 场景 | 是否有效 |
|------|----------|
| 会话混乱 | ✅ 有效 |
| 日志过大 | ✅ 有效 |
| 配置问题 | ❌ 无效 |

### 修复四：重新安装

```bash
# 卸载
npm uninstall -g openclaw

# 重新安装
curl -fsSL https://openclaw.ai/install.sh | bash
```

**适用场景**：

| 场景 | 是否有效 |
|------|----------|
| 安装损坏 | ✅ 有效 |
| 版本冲突 | ✅ 有效 |
| 其他方法无效 | ✅ 最后手段 |

**专家建议**：按照「重启 → 重置 → 清除 → 重装」的顺序尝试修复，每一步都验证效果后再进行下一步。

---

## 🐛 第四部分：渠道特定故障排除

### 4.1 WhatsApp 渠道

**常见问题**：

| 问题 | 原因 | 解决方式 |
|------|------|----------|
| 二维码扫描失败 | Node.js 版本问题 | 使用 Node.js（非 Bun） |
| 二维码扫描失败 | 网络连接问题 | 检查网络连接 |
| 连接断开 | 网络不稳定 | 保持网络稳定 |
| 消息丢失 | 会话过期 | 重新登录 |

**诊断命令**：

```bash
# 刷新二维码
openclaw channels login whatsapp --refresh

# 查看渠道日志
openclaw logs --channel whatsapp

# 检查渠道状态
openclaw channels status whatsapp
```

**详细文档**：参见 [WhatsApp 渠道文档](/zh-CN/channels/whatsapp)。

---

### 4.2 Telegram 渠道

**常见问题**：

| 问题 | 原因 | 解决方式 |
|------|------|----------|
| Bot 不响应 | 未配对 | 批准配对 |
| Bot 不响应 | Token 错误 | 检查 Token |
| Bot 不响应 | 权限不足 | 检查 allowlist |

**诊断命令**：

```bash
# 查看配对状态
openclaw pairing list telegram

# 确认 Token
openclaw config get channels.telegram.token

# 检查 allowlist
openclaw config get channels.telegram.allowFrom
```

**详细文档**：参见 [Telegram 渠道文档](/zh-CN/channels/telegram)。

---

### 4.3 Discord 渠道

**常见问题**：

| 问题 | 原因 | 解决方式 |
|------|------|----------|
| Bot 离线 | Token 错误 | 检查 Token |
| Bot 离线 | 权限不足 | 检查 Bot 权限 |
| 消息不响应 | Intents 未启用 | 启用 MESSAGE_CONTENT Intent |

**诊断命令**：

```bash
# 检查 Token
openclaw config get channels.discord.token

# 查看 Bot 状态
openclaw channels status discord
```

**详细文档**：参见 [Discord 渠道文档](/zh-CN/channels/discord)。

---

## 📝 第五部分：提交 GitHub Issue

### 5.1 准备信息

**必需信息清单**：

```bash
# 粘贴安全报告
openclaw status --all
```

| 信息类型 | 命令 | 为什么需要 |
|----------|------|------------|
| 系统状态 | `status --all` | 快速了解环境 |
| 配置信息 | `config get` | 排查配置问题 |
| 错误日志 | `logs --lines 50` | 定位错误原因 |
| 复现步骤 | 手动描述 | 帮助复现问题 |

---

### 5.2 Issue 模板

```markdown
**问题描述**
[清晰描述问题]

**复现步骤**
1. [步骤1]
2. [步骤2]
3. [步骤3]

**预期行为**
[描述预期发生什么]

**实际行为**
[描述实际发生什么]

**环境信息**
- OS: [例如 macOS 14, Ubuntu 22.04]
- Node: [输出 `node -v`]
- OpenClaw: [输出 `openclaw --version`]

**调试输出**
```
[粘贴 `openclaw status --all` 输出]
```

**日志**
```
[粘贴相关日志]
```
```

---

## 📚 第六部分：相关资源

**继续排查**：

- [FAQ](/zh-CN/help/faq) - 常见问题
- [网关故障排除](/zh-CN/gateway/troubleshooting) - 网关特定问题
- [渠道故障排除](/zh-CN/channels/troubleshooting) - 渠道问题
- [GitHub Issues](https://github.com/openclaw/openclaw/issues) - 提交问题
- [Discord](https://discord.gg/clawd) - 社区支持

---

## 🎓 章节总结

### 学习目标完成检查

#### 基础目标（必掌握）

- [ ] 掌握黄金 60 秒诊断流程
- [ ] 能够识别常见错误症状
- [ ] 理解故障排除的基本步骤
- [ ] 能够使用命令行工具收集诊断信息

#### 进阶目标（建议掌握）

- [ ] 能够进行详细的故障隔离和分析
- [ ] 掌握渠道特定问题的排查方法
- [ ] 能够准备高质量的问题报告
- [ ] 理解常见错误的根本原因

#### 专家目标（挑战）

- [ ] 能够设计预防性监控策略
- [ ] 能够建立团队级的故障响应流程
- [ ] 理解系统架构层面的问题排查
- [ ] 能够优化故障恢复时间

### 专家思维模型回顾

| 思维模型 | 应用场景 | 核心要点 |
|----------|----------|----------|
| 分层诊断 | 复杂问题 | 从外到内逐步深入 |
| 假设验证 | 故障排查 | 形成假设→最小成本验证 |
| 信息收集 | 所有问题 | 完整的诊断信息 |
| 隔离问题 | 多因素问题 | 定位根本原因 |

### 故障排除决策树

```
遇到问题
  │
  ├── 第一步：黄金 60 秒诊断
  │       ├── openclaw status
  │       ├── openclaw status --all
  │       ├── openclaw gateway probe
  │       ├── openclaw logs --follow
  │       └── openclaw doctor
  │
  ├── 第二步：根据症状分类
  │       ├── CLI 问题 → 检查 PATH
  │       ├── 安装问题 → 查看日志
  │       ├── 网关问题 → 检查状态
  │       └── 渠道问题 → 检查渠道
  │
  └── 第三步：尝试修复
          ├── 重启网关
          ├── 重置配置
          ├── 清除缓存
          └── 重新安装
```

---

**记住**：黄金 60 秒诊断流程能解决 80% 的问题！养成定期运行 `openclaw doctor` 的习惯，预防问题发生。
