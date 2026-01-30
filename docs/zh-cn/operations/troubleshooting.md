# 故障排除

本文档收集了常见问题的诊断和解决方法。

## 诊断工具

### moltbot doctor

这是最重要的诊断工具，可以自动检测和修复常见问题：

```bash
# 诊断问题
moltbot doctor

# 自动修复
moltbot doctor --fix

# 或使用 --yes 自动确认
moltbot doctor --yes
```

### 健康检查

```bash
# 基础健康检查
moltbot health

# 深度检查
moltbot status --deep

# 完整状态报告（适合分享）
moltbot status --all
```

### 查看日志

```bash
# 实时日志
moltbot logs --tail 100

# 查看特定日志文件
cat /tmp/moltbot/moltbot-$(date +%Y-%m-%d).log

# 搜索错误
grep -i error /tmp/moltbot/moltbot-*.log | tail -50
```

## 常见问题

### 安装问题

#### Node.js 版本不兼容

**症状**: 安装失败或运行时错误

**解决方案**:
```bash
# 检查版本
node --version

# 需要 Node.js >= 22
# 使用 nvm 安装
nvm install 22
nvm use 22
```

#### 权限错误

**症状**: `EACCES` 错误

**解决方案**:
```bash
# 不要使用 sudo 安装全局包
# 修复 npm 权限
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 网关问题

#### 网关无法启动

**症状**: `moltbot gateway` 启动失败

**诊断**:
```bash
# 检查配置
moltbot doctor

# 检查端口占用
lsof -i :18789

# 详细日志启动
moltbot gateway --verbose
```

**常见原因**:
1. 配置文件语法错误
2. 端口被占用
3. 认证信息缺失

#### 配置验证失败

**症状**: "Config validation failed"

**解决方案**:
```bash
# 诊断配置问题
moltbot doctor

# 自动修复
moltbot doctor --fix

# 手动编辑修复
moltbot config edit
```

#### 端口占用

**症状**: "EADDRINUSE"

**解决方案**:
```bash
# 找出占用进程
lsof -i :18789

# 终止进程
kill -9 <PID>

# 或使用其他端口
moltbot gateway --port 18790
```

### 认证问题

#### "No auth configured"

**症状**: 健康检查显示没有认证

**解决方案**:
```bash
# 重新运行配置向导
moltbot onboard

# 或手动配置
moltbot configure --section auth
```

#### OAuth 令牌过期

**症状**: 认证失败

**解决方案**:
```bash
# 重新认证
moltbot configure --section auth

# 检查令牌文件
ls -la ~/.clawdbot/credentials/
```

#### API 密钥无效

**症状**: 401 错误

**解决方案**:
1. 确认 API 密钥正确
2. 检查密钥是否过期
3. 验证账户余额

```bash
moltbot config set models.providers.openai.apiKey "sk-..."
```

### 渠道问题

#### WhatsApp 无法连接

**诊断**:
```bash
moltbot channels status whatsapp --probe
```

**常见解决方案**:

1. **重新登录**:
```bash
moltbot channels logout whatsapp
moltbot channels login
```

2. **清除会话**:
```bash
rm -rf ~/.clawdbot/credentials/whatsapp/
moltbot channels login
```

3. **检查网络**: 确保能访问 WhatsApp 服务器

#### WhatsApp 频繁断开

**可能原因**:
- 网络不稳定
- 手机 WhatsApp 离线
- 同一账号在其他设备登录

**解决方案**:
1. 确保手机在线并有网络
2. 检查是否有其他设备使用同一账号
3. 查看日志分析断开原因

#### Telegram Bot 无响应

**诊断**:
```bash
moltbot channels status telegram --probe
```

**常见解决方案**:

1. **验证 Token**:
```bash
# 使用 curl 测试
curl https://api.telegram.org/bot<TOKEN>/getMe
```

2. **检查配置**:
```bash
moltbot config get channels.telegram
```

3. **检查 allowlist**: 确认发送者在允许列表中

#### Discord Bot 无响应

**诊断**:
```bash
moltbot channels status discord --probe
```

**常见解决方案**:

1. **检查 Bot 权限**: 确保 Bot 有必要的权限
2. **验证 Token**: 确认 Token 正确
3. **检查 Guild 配置**: 确认服务器 ID 正确

### 消息问题

#### 收不到消息

**检查清单**:

1. 渠道是否连接？
```bash
moltbot channels status
```

2. DM 策略是否正确？
```bash
moltbot config get channels.<channel>.dmPolicy
```

3. 是否有待处理的配对？
```bash
moltbot pairing list <channel>
```

4. allowlist 是否包含发送者？
```bash
moltbot config get channels.<channel>.allowFrom
```

#### 消息发送失败

**诊断**:
```bash
# 查看日志
moltbot logs --tail 50

# 测试发送
moltbot message send --channel whatsapp --target +15555550123 --message "test" --verbose
```

**常见原因**:
- 目标格式错误
- 渠道未连接
- 权限不足

#### 群组消息不响应

**检查**:

1. 群组策略:
```bash
moltbot config get channels.<channel>.groupPolicy
```

2. 群组白名单:
```bash
moltbot config get channels.<channel>.groups
```

3. 提及要求:
```bash
moltbot config get channels.<channel>.groups."*".requireMention
```

### 性能问题

#### 响应缓慢

**诊断**:
```bash
moltbot status
moltbot health
```

**优化建议**:
1. 检查网络延迟
2. 减少历史记录限制
3. 优化消息队列设置

```json5
{
  messages: {
    queue: {
      debounceMs: 500,
      cap: 10
    }
  }
}
```

#### 内存占用高

**诊断**:
```bash
# 查看内存使用
moltbot status

# 系统级别
ps aux | grep moltbot
```

**解决方案**:
1. 重启服务
2. 减少会话历史
3. 调整日志级别

### 服务问题

#### 服务无法启动

**macOS (launchd)**:
```bash
# 检查状态
launchctl print gui/$UID | grep moltbot

# 查看日志
cat ~/Library/Logs/moltbot-gateway.log

# 手动加载
launchctl load ~/Library/LaunchAgents/bot.molt.gateway.plist
```

**Linux (systemd)**:
```bash
# 检查状态
systemctl --user status moltbot-gateway

# 查看日志
journalctl --user -u moltbot-gateway -n 100

# 重新加载
systemctl --user daemon-reload
```

#### 服务崩溃后不重启

**检查服务配置**:
```bash
# macOS
cat ~/Library/LaunchAgents/bot.molt.gateway.plist

# Linux
cat ~/.config/systemd/user/moltbot-gateway.service
```

确保有重启策略配置。

## 错误代码

| 代码 | 含义 | 解决方案 |
|------|------|----------|
| `EACCES` | 权限不足 | 检查文件权限 |
| `EADDRINUSE` | 端口占用 | 更换端口或终止占用进程 |
| `ECONNREFUSED` | 连接被拒绝 | 检查目标服务是否运行 |
| `ETIMEDOUT` | 连接超时 | 检查网络连接 |
| `E_CONFIG_INVALID` | 配置无效 | 运行 `moltbot doctor` |
| `E_AUTH_FAILED` | 认证失败 | 重新配置认证 |

## 获取帮助

如果以上方法都无法解决问题：

1. **收集信息**:
```bash
moltbot status --all > status.txt
moltbot doctor 2>&1 | tee doctor.txt
moltbot logs --tail 200 > logs.txt
```

2. **提交 Issue**: [GitHub Issues](https://github.com/moltbot/moltbot/issues)

3. **包含信息**:
   - Moltbot 版本
   - 操作系统
   - 错误信息
   - 复现步骤
   - 相关日志

## 下一步

- [运维手册](/zh-cn/operations/index) - 运维最佳实践
- [配置参考](/zh-cn/config/reference) - 配置选项
- [快速入门](/zh-cn/start/quick-start) - 重新开始
