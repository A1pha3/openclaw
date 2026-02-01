---
summary: "故障排除中心：症状 → 检查 → 修复"
read_when:
  - 看到错误信息需要修复路径
  - 安装程序显示"成功"但 CLI 不工作
  - 系统不正常工作
title: "故障排除"
---

# 🔧 故障排除

## ⏱️ 黄金60秒

按顺序运行这些命令：

```bash
# 1. 查看整体状态
openclaw status

# 2. 查看详细状态（包含配置）
openclaw status --all

# 3. 探测网关
openclaw gateway probe

# 4. 查看实时日志
openclaw logs --follow

# 5. 运行诊断工具
openclaw doctor
```

如果网关可达，深度探测：
```bash
openclaw status --deep
```

---

## 🚨 常见"坏了"场景

### `openclaw: command not found`

**几乎总是 Node/npm PATH 问题。**

**诊断：**
```bash
# 检查 Node 和 npm
node -v
npm -v

# 查看 npm 全局路径
npm prefix -g

# 检查 PATH
echo "$PATH"
```

**解决：** [安装指南 - PATH 问题](/zh-CN/install#故障排除命令找不到)

快速修复（zsh）：
```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 安装程序失败（或需要完整日志）

在详细模式下重新运行安装程序：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

Beta 版本：
```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

或设置环境变量：
```bash
export OPENCLAW_VERBOSE=1
curl -fsSL https://openclaw.ai/install.sh | bash
```

### 网关 "未授权"、无法连接或不断重连

- [网关故障排除](/zh-CN/gateway/troubleshooting)
- [网关认证](/zh-CN/gateway/authentication)

### 控制界面在 HTTP 上失败（需要设备身份）

- [网关故障排除](/zh-CN/gateway/troubleshooting)
- [控制界面](/zh-CN/web/control-ui#不安全的-http)

### `docs.openclaw.ai` 显示 SSL 错误（Comcast/Xfinity）

某些 Comcast/Xfinity 连接通过 Xfinity Advanced Security 阻止 `docs.openclaw.ai`。

**解决：**
1. 禁用 Advanced Security
2. 或将 `docs.openclaw.ai` 添加到允许列表

Xfinity Advanced Security 帮助：https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security

快速验证：尝试手机热点或 VPN 确认是 ISP 级别的过滤。

### 服务显示运行中，但 RPC 探测失败

- [网关故障排除](/zh-CN/gateway/troubleshooting)
- [后台进程/服务](/zh-CN/gateway/background-process)

### 模型/认证失败（速率限制、计费问题、"所有模型失败"）

- [模型命令](/zh-CN/cli/models)
- [OAuth/认证概念](/zh-CN/concepts/oauth)

### `/model` 显示 `model not allowed`

这通常意味着 `agents.defaults.models` 配置为允许列表。当它非空时，只能选择这些 provider/model key。

**检查：**
```bash
openclaw config get agents.defaults.models
```

**解决：**
1. 添加你想要的模型，或
2. 清除允许列表，然后重试 `/model`
3. 使用 `/models` 浏览允许的提供商/模型

---

## 🔍 详细故障排除流程

### 第一步：收集信息

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

### 第二步：隔离问题

| 症状 | 检查项 | 命令 |
|------|--------|------|
| CLI 不响应 | Node/npm 安装 | `node -v && npm -v` |
| 网关不启动 | 端口占用 | `lsof -i :18789` |
| 无法连接 | 防火墙 | `sudo ufw status` |
| 认证失败 | 凭证 | `openclaw config get agents.defaults` |
| 渠道不工作 | 渠道状态 | `openclaw channels status` |
| 模型不响应 | 模型状态 | `openclaw models status` |

### 第三步：常见修复

#### 重启网关

```bash
openclaw gateway restart
```

#### 重置配置

```bash
# 备份当前配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 重置向导
openclaw onboard --reset
```

#### 清除缓存

```bash
# 清除会话缓存
openclaw sessions clear

# 清除日志
openclaw logs clear
```

#### 重新安装

```bash
# 卸载
npm uninstall -g openclaw

# 重新安装
curl -fsSL https://openclaw.ai/install.sh | bash
```

---

## 🐛 渠道特定故障排除

### WhatsApp

**问题：二维码扫描失败**
- 确保使用 Node.js（非 Bun）
- 检查网络连接
- 尝试刷新二维码：`openclaw channels login whatsapp --refresh`

**问题：连接断开**
- 检查网络稳定性
- 查看日志：`openclaw logs --channel whatsapp`

详见 [WhatsApp 渠道](/zh-CN/channels/whatsapp)。

### Telegram

**问题：Bot 不响应**
- 检查配对：`openclaw pairing list telegram`
- 确认 token 正确：`openclaw config get channels.telegram.token`
- 检查 allowlist：`openclaw config get channels.telegram.allowFrom`

详见 [Telegram 渠道](/zh-CN/channels/telegram)。

### Discord

**问题：Bot 离线**
- 检查 token：`openclaw config get channels.discord.token`
- 确认权限：在 Discord Developer Portal 检查 Bot 权限
- 检查 intents：确保启用了 MESSAGE_CONTENT intent

详见 [Discord 渠道](/zh-CN/channels/discord)。

---

## 📝 提交 GitHub Issue

### 准备信息

粘贴安全报告：
```bash
openclaw status --all
```

### 创建 Issue 模板

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
[paste `openclaw status --all` output]
```

**日志**
```
[paste relevant logs]
```
```

---

## 📚 更多资源

- [FAQ](/zh-CN/help/faq) - 常见问题
- [网关故障排除](/zh-CN/gateway/troubleshooting) - 网关特定问题
- [渠道故障排除](/zh-CN/channels/troubleshooting) - 渠道问题
- [GitHub Issues](https://github.com/openclaw/openclaw/issues) - 提交问题
- [Discord](https://discord.gg/clawd) - 社区支持

---

**记住**：黄金60秒诊断流程能解决80%的问题！
