---
summary: 收集常见问题的诊断和解决方法，涵盖诊断工具、安装问题、网关问题、认证问题、渠道问题、消息问题、性能问题、服务问题和错误代码
read_when:
  - 遇到问题时需要诊断和解决
  - 使用 openclaw doctor 等工具时
  - 排查各种错误代码时
title: 故障排除
---

# 故障排除

本指南系统性地收集了 OpenClaw 运维过程中常见的问题及其解决方案。有效的故障排除需要系统化的方法和丰富的经验，本指南旨在帮助运维人员快速定位问题、采取正确的应对措施，并从每次故障中学习改进。在开始排查之前，建议您先了解 OpenClaw 的整体架构和各组件的依赖关系，这将有助于建立正确的诊断思路。

## 学习目标

完成本章节学习后，您将能够：

### 核心能力

- 掌握系统化的故障诊断方法和决策流程
- 熟练使用各类诊断工具定位问题根因
- 识别常见问题模式并采取正确的修复措施
- 建立预防性维护策略，减少故障发生概率
- 从故障中学习并持续改进运维实践

### 适用场景

| 场景 | 问题类型 | 预期解决时间 |
|------|---------|-------------|
| 服务启动失败 | 配置/权限/端口 | 5-15 分钟 |
| 渠道连接异常 | 网络/凭证/API | 10-30 分钟 |
| 消息处理问题 | 队列/路由/格式 | 15-30 分钟 |
| 性能下降 | 资源/配置/外部 | 30-60 分钟 |
| 复杂故障 | 多因素 | 1-4 小时 |

## 故障诊断方法论

### 为什么需要系统化的故障诊断

当 OpenClaw 出现问题时，运维人员面临的压力通常来自于多个方面：业务影响的不确定性、解决时间的不确定性以及根因的隐蔽性。系统化的故障诊断方法能够将不确定性转化为可管理的步骤，通过结构化的流程快速缩小问题范围，最终定位根因并采取有效措施。

**故障诊断的核心原则**：

| 原则 | 说明 | 应用示例 |
|------|------|----------|
| 分而治之 | 将复杂问题分解为简单子问题 | 先检查服务状态，再检查渠道，最后检查消息 |
| 从现象到本质 | 从表象症状推断内部原因 | 连接断开 → 凭证过期 |
| 先恢复后诊断 | 优先恢复服务，再深入分析 | 重启服务确认恢复，再查看日志 |
| 记录复盘 | 记录故障过程以供学习 | 保存诊断结果，更新文档 |

### 故障诊断决策框架

```
故障诊断思维模型：

                    ┌─────────────────────────┐
                    │      故障现象识别         │
                    │  (收集症状、影响范围)     │
                    └───────────┬─────────────┘
                                │
                                ▼
              ┌─────────────────────────────────┐
              │       影响范围评估              │
              │  P0: 全局不可用                │
              │  P1: 主要功能受损               │
              │  P2: 部分用户受影响             │
              │  P3: 轻微问题                   │
              └───────────────┬─────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │  快速恢复   │     │  深度诊断   │     │  升级处理   │
   │  (优先)    │     │  (同时进行) │     │  (必要时)   │
   └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
          │                   │                   │
          ├─ 重启服务         ├─ 查看日志         ├─ 联系支持
          ├─ 回滚配置         ├─ 检查配置         ├─ 社区求助
          ├─ 清理缓存         ├─ 分析指标         └─ 提交 Issue
          └─ 切换备机         └─ 网络诊断
```

### 信息收集流程

**第一步：收集基础信息**：

```bash
#!/bin/bash
# collect-diagnostics.sh - 故障信息收集脚本

OUTPUT_FILE="openclaw-diagnostics-$(date +%Y%m%d_%H%M%S).tar.gz"

echo "开始收集诊断信息..."

# 创建临时目录
TEMP_DIR=$(mktemp -d)
cd "$TEMP_DIR"

# 1. 服务状态
echo "[1/8] 收集服务状态..."
openclaw status --all > status.txt 2>&1
openclaw health > health.json 2>&1
openclaw doctor > doctor.txt 2>&1 || true

# 2. 进程信息
echo "[2/8] 收集进程信息..."
ps aux | grep openclaw > ps.txt
pgrep -f openclaw | xargs -I {} sh -c 'ps -p {} -o pid,ppid,cmd,%mem,%cpu,etime' > process-detail.txt

# 3. 端口和网络
echo "[3/8] 收集网络信息..."
ss -ltnp | grep 18789 > ports.txt
netstat -tnp 2>/dev/null | grep 18789 >> ports.txt || true

# 4. 日志收集
echo "[4/8] 收集日志..."
mkdir -p logs
openclaw logs --level error --since '24 hours' > logs/errors.log 2>&1 || true
openclaw logs --since '1 hour' > logs/recent.log 2>&1 || true
tail -1000 /tmp/openclaw-*.log 2>/dev/null > logs/gateway.log || true

# 5. 配置信息
echo "[5/8] 收集配置..."
openclaw config get > config.json 2>&1

# 6. 渠道状态
echo "[6/8] 收集渠道状态..."
openclaw channels status --probe > channels.txt 2>&1

# 7. 系统信息
echo "[7/8] 收集系统信息..."
uname -a > system.txt
df -h > disk.txt
free -h > memory.txt 2>/dev/null || true

# 8. 环境信息
echo "[8/8] 收集环境信息..."
env | grep -i openclaw > env.txt 2>/dev/null || true
env | grep -i anthropic >> env.txt 2>/dev/null || true
env | grep -i api >> env.txt 2>/dev/null || true

# 打包
cd ..
tar -czf "$OUTPUT_FILE" "$(basename "$TEMP_DIR")"
rm -rf "$TEMP_DIR"

echo "诊断信息已收集: $OUTPUT_FILE"
echo "请将此文件附加到 Issue 或发送给支持团队"
```

## 诊断工具箱

### openclaw doctor：综合诊断工具

`openclaw doctor` 是最重要的诊断工具，能够自动检测和修复常见问题：

```bash
# 综合诊断（首选）
openclaw doctor

# 自动修复发现的问题
openclaw doctor --fix

# 自动确认所有修复（无人值守场景）
openclaw doctor --yes

# 检查特定类别
openclaw doctor --check config          # 配置检查
openclaw doctor --check channels        # 渠道检查
openclaw doctor --check permissions    # 权限检查
openclaw doctor --check security        # 安全检查
openclaw doctor --check storage        # 存储检查
openclaw doctor --check dependencies   # 依赖检查
```

**诊断输出解读**：

```
========================================
OpenClaw 诊断报告
========================================
检查时间: 2026-02-05 10:30:00
版本: 2026.1.27

[✓] Node.js 版本检查
    版本: v22.12.0 (满足要求 >= 22.0.0)

[✓] 配置文件检查
    路径: /home/user/.clawdbot/openclaw.json
    语法: 有效

[✓] Gateway 服务检查
    状态: 运行中 (PID: 12345)
    端口: 18789
    绑定: 127.0.0.1

[✓] 磁盘空间检查
    可用: 45.2GB
    状态: 正常

[⚠] 建议优化
    - 日志级别可调整为 warn 以减少 I/O

========================================
诊断结果: 通过
========================================
```

### 健康检查命令

```bash
# 快速健康检查（秒级响应）
openclaw health

# 完整状态报告（适合诊断和分享）
openclaw status --all

# 深度检查（验证渠道连接）
openclaw status --deep

# HTTP 健康检查端点
curl http://127.0.0.1:18789/health
```

### 日志查看工具

```bash
# 实时日志
openclaw logs --tail 100
openclaw logs --follow

# 按级别过滤
openclaw logs --level error
openclaw logs --level warn

# 按渠道过滤
openclaw logs --channel telegram
openclaw logs --channel whatsapp

# 组合过滤
openclaw logs --level error --channel telegram --tail 50

# 搜索特定内容
openclaw logs --grep "error\|failed\|timeout"
openclaw logs --grep "whatsapp" --level info

# 按时间范围
openclaw logs --since "2026-02-05 00:00:00"
openclaw logs --since "1 hour ago"
```

## 常见问题与解决方案

### 安装问题

#### Node.js 版本不兼容

**症状表现**：

- 安装 npm 包时报错
- 运行时出现意外的语法错误
- 某些功能无法正常工作

**诊断方法**：

```bash
# 检查当前版本
node --version
npm --version

# 期望输出：v22.x.x (>= 22.12.0)
```

**解决方案**：

```bash
# 使用 nvm 安装指定版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc  # 或 ~/.zshrc

# 安装 Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# 验证
node --version
```

#### 权限错误

**症状表现**：

- 安装全局包时报 `EACCES` 错误
- 创建配置文件时权限被拒绝
- 日志文件无法写入

**诊断方法**：

```bash
# 检查 npm 前缀
npm config get prefix

# 检查当前用户
whoami

# 检查目录权限
ls -la ~/.npm
ls -la ~/.clawdbot
```

**解决方案**：

```bash
# 方案一：修复 npm 权限（推荐）
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 方案二：使用 Node 版本管理器（推荐）
# nvm 或 fnm 自动处理权限问题

# 方案三：为现有目录设置正确权限
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) ~/.clawdbot
```

### 网关问题

#### Gateway 无法启动

**症状表现**：

- `openclaw gateway` 命令无响应或立即退出
- 端口 18789 未被监听
- 健康检查返回失败

**诊断流程**：

```
Gateway 启动失败排查流程：

1. 检查配置文件
   └── openclaw doctor --check config

2. 检查端口占用
   └── lsof -i :18789
   └── ss -ltnp | grep 18789

3. 查看详细错误
   └── openclaw gateway --verbose
   └── journalctl --user -u openclaw-gateway -n 100

4. 常见原因
   ├── 配置文件语法错误 → 修复配置
   ├── 端口被占用 → 更换端口或终止占用进程
   ├── 认证信息缺失 → 配置必要环境变量
   ├── Node.js 版本问题 → 升级到 >= 22
   └── 磁盘空间不足 → 清理磁盘
```

**详细诊断命令**：

```bash
# 1. 检查配置
openclaw doctor --check config

# 2. 检查端口
lsof -i :18789

# 3. 详细日志启动
openclaw gateway --verbose

# 4. 检查系统日志
journalctl --user -u openclaw-gateway -n 100
tail -100 /tmp/openclaw-gateway.log

# 5. 手动测试 Node.js 环境
node -e "console.log('Node.js works')"
npm -v
```

#### 端口占用

**症状表现**：

- 启动时报 `EADDRINUSE` 错误
- 提示端口 18789 已被占用

**诊断方法**：

```bash
# 查找占用端口的进程
lsof -i :18789
ss -ltnp | grep 18789

# 查找所有 openclaw 相关进程
ps aux | grep openclaw
pgrep -f openclaw
```

**解决方案**：

```bash
# 方案一：终止占用进程
lsof -i :18789
# 假设输出显示 PID 为 12345
kill -9 12345

# 方案二：使用其他端口
openclaw gateway --port 18790

# 或在配置中修改
# ~/.clawdbot/openclaw.json:
# { "gateway": { "port": 18790 } }
```

#### 配置验证失败

**症状表现**：

- 启动时报 "Config validation failed"
- 配置修改后服务无法启动

**诊断方法**：

```bash
# 诊断配置问题
openclaw doctor --check config

# 查看详细错误
openclaw config validate

# 备份并检查配置文件
cp ~/.clawdbot/openclaw.json ~/.clawdbot/openclaw.json.bak
cat ~/.clawdbot/openclaw.json | python3 -m json.tool
```

**解决方案**：

```bash
# 自动修复
openclaw doctor --fix

# 手动修复
openclaw config edit

# 重置为默认配置（谨慎使用）
cp ~/.clawdbot/openclaw.json.bak ~/.clawdbot/openclaw.json
```

### 认证问题

#### "No auth configured"

**症状表现**：

- 健康检查显示没有配置认证
- 外部工具无法连接到 Gateway

**诊断方法**：

```bash
# 检查配置
openclaw config get gateway.auth

# 查看健康检查输出
openclaw health
```

**解决方案**：

```bash
# 方案一：运行配置向导
openclaw onboard

# 方案二：手动配置
# 编辑配置文件
openclaw config edit

# 添加认证配置
# ~/.clawdbot/openclaw.json:
# {
#   "gateway": {
#     "auth": {
#       "token": "your-secure-token-here"
#     }
#   }
# }

# 生成安全令牌
openssl rand -base64 32
```

#### OAuth 令牌过期

**症状表现**：

- AI 模型调用返回认证错误
- 提示令牌无效或过期

**诊断方法**：

```bash
# 检查凭证状态
ls -la ~/.clawdbot/credentials/

# 查看 OAuth 令牌文件
cat ~/.clawdbot/credentials/oauth.json | jq '.'
```

**解决方案**：

```bash
# 重新认证模型提供商
openclaw configure --section models

# 或手动更新令牌
openclaw config set models.providers.anthropic.apiKey "new-api-key"

# 验证令牌有效性
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
    https://api.anthropic.com/v1/messages -X POST -d '{}'
```

#### API 密钥无效

**症状表现**：

- 模型调用返回 401 错误
- 提示认证失败

**诊断方法**：

```bash
# 检查 API 密钥配置
openclaw config get models.providers

# 验证密钥格式
echo $ANTHROPIC_API_KEY | head -c 10

# 测试 API 访问
curl -s -H "x-api-key: $ANTHROPIC_API_KEY" \
    https://api.anthropic.com/v1/complete \
    -H "anthropic-version: 2023-01-01" | head -100
```

**解决方案**：

```bash
# 确认 API 密钥正确
# 1. 登录提供商控制台确认密钥状态
# 2. 检查密钥是否过期
# 3. 验证账户余额是否充足

# 更新 API 密钥
export ANTHROPIC_API_KEY="sk-ant-api03-xxx"
# 或
openclaw config set models.providers.anthropic.apiKey "sk-ant-api03-xxx"
```

### 渠道问题

#### WhatsApp 无法连接

**诊断流程**：

```
WhatsApp 连接问题排查：

1. 基础状态检查
   └── openclaw channels status whatsapp

2. 带探测的深度检查
   └── openclaw channels status whatsapp --probe

3. 查看 WhatsApp 渠道日志
   └── openclaw logs --channel whatsapp --level error --tail 50

4. 检查凭证状态
   └── ls -la ~/.clawdbot/credentials/whatsapp/

5. 常见原因
   ├── 会话过期 → 重新登录
   ├── 手机离线 → 确保 WhatsApp 在线
   ├── 网络问题 → 检查网络连接
   └── API 限制 → 检查是否被限制
```

**详细诊断命令**：

```bash
# 1. 检查渠道状态
openclaw channels status whatsapp
openclaw channels status whatsapp --probe
openclaw channels status whatsapp --verbose

# 2. 查看日志
openclaw logs --channel whatsapp --level error --tail 100
openclaw logs --channel whatsapp --since "1 hour ago"

# 3. 检查凭证
ls -la ~/.clawdbot/credentials/whatsapp/
cat ~/.clawdbot/credentials/whatsapp/*.json 2>/dev/null | head -50

# 4. 测试网络连通性
curl -v https://web.whatsapp.com 2>&1 | head -20
```

**解决方案**：

```bash
# 方案一：重新登录 WhatsApp
openclaw channels logout whatsapp
openclaw channels login whatsapp

# 方案二：完全重置会话
rm -rf ~/.clawdbot/credentials/whatsapp/
openclaw channels login whatsapp

# 方案三：检查网络
ping web.whatsapp.com
curl -I https://web.whatsapp.com
```

#### WhatsApp 频繁断开

**可能原因分析**：

| 原因 | 诊断方法 | 解决方案 |
|------|---------|----------|
| 网络不稳定 | 检查网络延迟和丢包 | 使用稳定的网络连接 |
| 手机 WhatsApp 离线 | 检查手机状态 | 确保手机在线 |
| 同一账号多设备登录 | 检查登录设备 | 移除其他设备 |
| WhatsApp 服务器限制 | 查看错误日志 | 等待解除限制 |
| 代理/IP 变更 | 检查 IP 变化 | 保持 IP 稳定 |

**诊断命令**：

```bash
# 检查网络质量
ping -c 10 web.whatsapp.com
mtr web.whatsapp.com

# 检查连接日志
openclaw logs --channel whatsapp --grep "disconnect\|reconnect" --tail 100

# 查看系统日志中的网络相关错误
journalctl --user -u openclaw-gateway | grep -i network
```

#### Telegram Bot 无响应

**诊断流程**：

```
Telegram Bot 问题排查：

1. 检查 Bot 状态
   └── openclaw channels status telegram --probe

2. 验证 Bot Token
   └── curl https://api.telegram.org/bot<TOKEN>/getMe

3. 检查配置
   └── openclaw config get channels.telegram

4. 检查 allowlist
   └── openclaw config get channels.telegram.allowFrom

5. 查看日志
   └── openclaw logs --channel telegram --level error
```

**诊断命令**：

```bash
# 1. 验证 Bot Token
curl https://api.telegram.org/bot$TELEGRAM_TOKEN/getMe

# 2. 检查 Bot 状态
openclaw channels status telegram --verbose

# 3. 查看配置
openclaw config get channels.telegram

# 4. 检查 allowlist
openclaw config get channels.telegram.allowFrom

# 5. 查看日志
openclaw logs --channel telegram --level error --tail 100
```

**常见解决方案**：

```bash
# 1. 验证 Token 正确
# 从 @BotFather 获取正确的 Token

# 2. 检查 Bot 权限
# 确保 Bot 有发送消息的权限

# 3. 检查配置
# ~/.clawdbot/openclaw.json:
# {
#   "channels": {
#     "telegram": {
#       "botToken": "your-bot-token",
#       "allowFrom": ["user-id-1", "user-id-2"]
#     }
#   }
# }

# 4. 重新配置 Bot
# 重新与 @BotFather 对话获取 Token
```

#### Discord Bot 无响应

**诊断流程**：

```
Discord Bot 问题排查：

1. 检查 Bot 状态
   └── openclaw channels status discord --probe

2. 验证 Bot Token
   └── curl -H "Authorization: Bot $TOKEN" \
       https://discord.com/api/v10/users/@me

3. 检查 Bot 权限
   └── 在 Discord Developer Portal 检查权限

4. 检查 Guild 配置
   └── openclaw config get channels.discord

5. 查看日志
   └── openclaw logs --channel discord --level error
```

**诊断命令**：

```bash
# 1. 验证 Bot Token
curl -H "Authorization: Bot $DISCORD_TOKEN" \
    https://discord.com/api/v10/users/@me

# 2. 检查 Bot 状态
openclaw channels status discord --verbose

# 3. 查看配置
openclaw config get channels.discord

# 4. 查看日志
openclaw logs --channel discord --level error --tail 100
```

### 消息问题

#### 收不到消息

**检查清单**：

```
消息接收问题排查：

1. 渠道连接状态
   └── openclaw channels status
   └── 所有渠道应显示 "connected"

2. DM 策略检查
   └── openclaw config get channels.<channel>.dmPolicy
   └── 确认策略允许接收消息

3. 配对请求处理
   └── openclaw pairing list <channel>
   └── 批准所有待处理配对

4. Allowlist 检查
   └── openclaw config get channels.<channel>.allowFrom
   └── 确认发送者在允许列表中

5. 群组策略检查（如果是群组消息）
   └── openclaw config get channels.<channel>.groupPolicy
   └── openclaw config get channels.<channel>.groups
```

**诊断命令**：

```bash
# 1. 检查渠道状态
openclaw channels status
openclaw channels status --probe

# 2. 检查 DM 策略
openclaw config get channels.whatsapp.dmPolicy
openclaw config get channels.telegram.dmPolicy

# 3. 查看配对请求
openclaw pairing list whatsapp
openclaw pairing list telegram

# 4. 检查 allowlist
openclaw config get channels.whatsapp.allowFrom

# 5. 测试发送消息
openclaw message send --channel whatsapp --target "+1234567890" --message "test"
```

#### 消息发送失败

**诊断流程**：

```
消息发送失败排查：

1. 检查渠道连接
   └── openclaw channels status <channel>
   └── 应显示 "connected"

2. 检查目标格式
   └── WhatsApp: +1234567890
   └── Telegram: @username 或数字 ID
   └── Discord: 用户名#1234 或 ID

3. 查看详细日志
   └── openclaw logs --level error --tail 50

4. 测试发送
   └── openclaw message send --verbose
```

**诊断命令**：

```bash
# 1. 查看最近日志
openclaw logs --tail 50

# 2. 详细模式发送测试
openclaw message send \
    --channel whatsapp \
    --target "+1234567890" \
    --message "test" \
    --verbose

# 3. 检查消息队列
openclaw status | grep queue
```

#### 群组消息不响应

**检查要点**：

| 检查项 | 配置路径 | 说明 |
|-------|---------|------|
| 群组策略 | `channels.<channel>.groupPolicy` | "open" 或 "restricted" |
| 群组白名单 | `channels.<channel>.groups` | 允许的群组 ID 列表 |
| 提及要求 | `channels.<channel>.groups."*".requireMention` | 是否需要 @Bot |
| 激活模式 | `channels.<channel>.groups."*".activation` | "mention" 或 "always" |

**诊断命令**：

```bash
# 1. 检查群组策略
openclaw config get channels.discord.groupPolicy
openclaw config get channels.telegram.groupPolicy

# 2. 检查群组白名单
openclaw config get channels.discord.guilds

# 3. 检查提及要求
openclaw config get channels.discord.groups."*".requireMention

# 4. 测试群组消息
# 在群组中发送 @Bot help
```

### 性能问题

#### 响应缓慢

**诊断流程**：

```
性能问题排查：

1. 基础状态检查
   └── openclaw status
   └── 检查内存和 CPU 使用

2. 资源使用分析
   └── top -p $(pgrep -f openclaw)
   └── 查看内存增长趋势

3. 队列状态检查
   └── openclaw status | grep queue

4. 外部 API 响应时间
   └── curl -w "\ntime: %{time_total}s" https://api.anthropic.com

5. 常见原因
   ├── 资源不足 → 增加内存/CPU
   ├── 历史会话过长 → 清理或限制
   ├── 消息队列积压 → 调整队列参数
   └── 外部 API 慢 → 检查网络/API
```

**诊断命令**：

```bash
# 1. 查看状态
openclaw status
openclaw status --deep

# 2. 资源使用
top -p $(pgrep -f openclaw)
htop -p $(pgrep -f openclaw)

# 3. 清理会话历史
openclaw sessions prune --older-than 7d

# 4. 测试 API 响应
time curl -s -o /dev/null -w "%{time_total}" \
    https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2023-01-01" \
    -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[]}'
```

**优化建议**：

```json5
{
  "messages": {
    "queue": {
      "mode": "collect",
      "debounceMs": 500,      // 减少收集窗口
      "cap": 10               // 减小批量大小
    }
  },
  "agents": {
    "defaults": {
      "historyLimit": {
        "messages": 50,       // 减少历史消息数
        "days": 7            // 限制历史保存天数
      }
    }
  }
}
```

#### 内存占用高

**诊断方法**：

```bash
# 1. 查看内存使用
openclaw status
ps -p $(pgrep -f openclaw) -o pid,vsz,rss,%mem,%cpu,etime

# 2. 内存增长趋势
# 多次执行并对比
ps -p $(pgrep -f openclaw) -o rss=

# 3. 检查活跃会话
openclaw sessions list
openclaw sessions stats

# 4. 查看历史记录配置
openclaw config get agents.defaults.historyLimit
```

**解决方案**：

```bash
# 1. 重启服务释放内存
openclaw service restart

# 2. 清理会话历史
openclaw sessions prune --older-than 1d
openclaw sessions prune --older-than 7d --force

# 3. 减少历史限制
# ~/.clawdbot/openclaw.json:
# {
#   "agents": {
#     "defaults": {
#       "historyLimit": {
#         "messages": 20,
#         "days": 3
#       }
#     }
#   }
# }

# 4. 调整日志级别减少 I/O
# "logging": { "level": "warn" }
```

### 服务问题

#### macOS 服务无法启动

**诊断命令**：

```bash
# 1. 检查 launchd 服务
launchctl print gui/$UID | grep openclaw

# 2. 查看服务日志
cat ~/Library/Logs/openclaw-gateway.log
tail -100 ~/Library/Logs/openclaw-gateway.log

# 3. 手动加载服务
launchctl load ~/Library/LaunchAgents/openclaw.gateway.plist

# 4. 查看 plist 配置
cat ~/Library/LaunchAgents/openclaw.gateway.plist

# 5. 检查进程
ps aux | grep openclaw
```

**解决方案**：

```bash
# 1. 重新加载服务
launchctl unload ~/Library/LaunchAgents/openclaw.gateway.plist
launchctl load ~/Library/LaunchAgents/openclaw.gateway.plist

# 2. 使用 CLI 管理
openclaw service stop
openclaw service start
openclaw service restart

# 3. 强制重启
pkill -9 -f openclaw
openclaw service start

# 4. 检查权限
ls -la ~/Library/LaunchAgents/openclaw.gateway.plist
```

#### Linux 服务无法启动

**诊断命令**：

```bash
# 1. 检查 systemd 服务
systemctl --user status openclaw-gateway

# 2. 查看服务日志
journalctl --user -u openclaw-gateway -n 100
journalctl --user -u openclaw-gateway -f

# 3. 重新加载配置
systemctl --user daemon-reload

# 4. 手动测试运行
/usr/local/bin/openclaw gateway --verbose
```

**解决方案**：

```bash
# 1. 重新加载并启动
systemctl --user daemon-reload
systemctl --user restart openclaw-gateway

# 2. 查看详细错误
journalctl --user -u openclaw-gateway -xe

# 3. 检查服务配置
cat ~/.config/systemd/user/openclaw-gateway.service

# 4. 确保用户登录
loginctl list-sessions
loginctl show-session $(loginctl | grep $(whoami) | awk '{print $1}') | grep State
```

#### 服务崩溃后不重启

**诊断方法**：

```bash
# 1. 检查重启策略
cat ~/.config/systemd/user/openclaw-gateway.service | grep Restart

# 2. 查看崩溃日志
journalctl --user -u openclaw-gateway --since "1 hour ago"

# 3. 检查是否有循环崩溃
# 多次快速崩溃会触发 systemd 的保护机制
systemctl --user status openclaw-gateway | grep -i "fail\|crash\|limit"
```

**解决方案**：

```bash
# 1. 手动启动服务
systemctl --user start openclaw-gateway

# 2. 禁用然后启用服务
systemctl --user disable openclaw-gateway
systemctl --user enable openclaw-gateway

# 3. 重置服务状态
systemctl --user reset-failed openclaw-gateway

# 4. 检查重启策略配置
# ~/.config/systemd/user/openclaw-gateway.service:
# [Service]
# Restart=always
# RestartSec=10
```

## 错误代码参考

### 常见错误代码

| 错误代码 | 含义 | 常见原因 | 解决方案 |
|---------|------|---------|----------|
| `EACCES` | 权限不足 | 文件权限、npm 权限 | 检查并修复权限 |
| `EADDRINUSE` | 端口占用 | 端口已被其他进程使用 | 更换端口或终止占用进程 |
| `ECONNREFUSED` | 连接被拒绝 | 服务未运行、认证失败 | 检查服务状态 |
| `ETIMEDOUT` | 连接超时 | 网络问题、防火墙 | 检查网络连接 |
| `E_CONFIG_INVALID` | 配置无效 | JSON 语法错误、缺少必需项 | 运行 `openclaw doctor` |
| `E_AUTH_FAILED` | 认证失败 | Token/API Key 错误 | 重新配置认证 |
| `E_CHANNEL_OFFLINE` | 渠道离线 | 凭证过期、网络问题 | 重新登录渠道 |
| `E_MESSAGE_FAILED` | 消息发送失败 | 目标无效、权限不足 | 检查目标格式 |
| `E_MODEL_ERROR` | 模型调用错误 | API 限制、余额不足 | 检查 API 状态 |

### 错误处理建议

```
错误处理最佳实践：

1. 优先查看日志
   └── 大多数错误都有详细的日志记录

2. 使用 doctor 诊断
   └── openclaw doctor 自动检测常见问题

3. 检查配置
   └── 很多问题源于配置错误

4. 搜索已知问题
   └── 查看 GitHub Issues
   └── 搜索类似问题的解决方案
```

## 获取帮助

### 问题排查流程

```
获取帮助前的自查流程：

1. 收集信息
   ├── 运行诊断命令
   ├── 保存状态输出
   ├── 收集相关日志
   └── 记录错误信息

2. 自助排查
   ├── 查阅本文档
   ├── 查看配置示例
   └── 尝试常见解决方案

3. 社区求助
   ├── GitHub Issues
   ├── Discord 社区
   └── 搜索类似问题

4. 提交新问题
   ├── 提供详细信息
   └── 附上诊断数据
```

### 收集诊断信息

```bash
#!/bin/bash
# collect-support-info.sh - 收集支持所需信息

OUTPUT="openclaw-support-$(date +%Y%m%d_%H%M%S).txt"

{
    echo "OpenClaw 支持信息"
    echo "================="
    echo "时间: $(date)"
    echo ""

    echo "1. 版本信息"
    echo "-----------"
    openclaw --version 2>&1
    node --version 2>&1 || echo "Node.js 检查失败"
    echo ""

    echo "2. 服务状态"
    echo "-----------"
    openclaw health 2>&1
    echo ""

    echo "3. 诊断结果"
    echo "-----------"
    openclaw doctor 2>&1 || echo "诊断失败"
    echo ""

    echo "4. 渠道状态"
    echo "-----------"
    openclaw channels status 2>&1
    echo ""

    echo "5. 配置摘要"
    echo "----------"
    openclaw config get 2>&1 | head -100
    echo ""

    echo "6. 最近错误"
    echo "----------"
    openclaw logs --level error --since "24 hours" 2>&1 | tail -50
    echo ""

    echo "7. 系统信息"
    echo "----------"
    uname -a 2>&1
    df -h ~/.clawdbot 2>&1
    echo ""

} > "$OUTPUT"

echo "支持信息已保存到: $OUTPUT"
echo "请将此文件附加到您的 Issue 或支持请求中"
```

### 提交 Issue

**Issue 模板**：

```markdown
## 问题描述
[清晰描述您遇到的问题]

## 环境信息
- OpenClaw 版本: [运行 openclaw --version]
- 操作系统: [uname -a]
- Node.js 版本: [node --version]

## 复现步骤
1. [步骤 1]
2. [步骤 2]
3. [...]

## 预期行为
[描述您期望的行为]

## 实际行为
[描述实际发生的行为]

## 错误信息
[如果有错误信息，粘贴在这里]

## 日志
[相关日志摘要]

## 排查尝试
[您已经尝试过的解决方法]

## 附加信息
[任何其他有帮助的信息]
```

## 下一步

| 文档 | 说明 |
|------|------|
| [运维手册](/zh-CN/operations/index) | 运维最佳实践和整体架构 |
| [部署指南](/zh-CN/operations/deployment) | 详细部署步骤和配置 |
| [监控指南](/zh-CN/operations/monitoring) | 监控配置和告警策略 |
| [配置参考](/zh-CN/gateway/configuration) | 配置选项详细说明 |
| [快速入门](/zh-CN/start/quick-start) | 重新开始安装流程 |
