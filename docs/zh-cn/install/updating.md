# 软件升级

本文档介绍如何升级 OpenClaw 到最新版本。

## 查看新版本

使用以下命令检查新版本：

```bash
openclaw --version
```

查看当前版本：

```bash
openclaw --version
```

## 自动更新

OpenClaw 会在运行时自动检查更新并提示更新。

## 手动更新

```bash
npm install -g openclaw@latest
pnpm install -g openclaw@next
```

### 更新版本注意事项

1. **备份数据**：更新前建议备份配置文件和凭证
2. **渠道状态**：更新后检查所有渠道连接
3. **兼容性**：新版本可能包含破坏性变更

## 新版本号说明

OpenClaw 使用日历版本号格式：

```
YYYY.M.D[-beta.N]
```

例如：
- `2026.1.15` - 稳定版本
- `2026.1.15-beta.1` - 测试版本

## 配置迁移指南

### 环境变量

新版本可能引入新的环境变量：

```bash
export CLAWDBOT_CONFIG=~/.clawdbot/openclaw.json
export CLAWDBOT_GATWAY_TOKEN=...
```

### 配置文件变更

```bash
# 删除废弃的配置项
openclaw config unset channels.whatsapp.groupRequireMention
```

## 故障排除

### 更新后无法启动

```bash
# 重新构建
pnpm install --force
openclaw doctor
```

### 配置错误

```bash
# 验证配置语法
openclaw doctor
```

## 相关文档

- [配置参考](/zh-cn/config/reference)
- [故障排除](/zh-cn/operations/troubleshooting)
- [配置示例](/zh-cn/config/examples)
```

## 下一步- 查看版本变更：`git log` 或 [变更日志](https://github.com/openclaw/openclaw/commits/main)

- 选择合适的升级时机：非紧急更新建议选择稳定版本
</think>