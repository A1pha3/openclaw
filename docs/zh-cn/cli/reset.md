---
summary: "`openclaw reset` 命令参考（重置本地状态/配置）"
read_when:
  - 想要清除本地状态但保留 CLI
  - 想要预演会删除什么
title: "reset"
---

# `openclaw reset`

重置本地配置/状态（保留 CLI 安装）。

## 为什么需要这个命令

有时你需要从头开始：

- **排除问题**：配置损坏时重置到干净状态
- **隐私清理**：删除会话历史和凭据
- **测试环境**：为测试创建干净的环境
- **迁移准备**：迁移前清理旧数据

## 基本用法

```bash
# 交互式重置（会提示确认）
openclaw reset

# 预演模式（查看会删除什么）
openclaw reset --dry-run

# 非交互式重置（自动确认）
openclaw reset --scope config+creds+sessions --yes --non-interactive
```

## 选项

| 选项 | 说明 |
|------|------|
| `--dry-run` | 预演模式，只显示会删除什么 |
| `--scope <scope>` | 指定重置范围 |
| `--yes` | 跳过确认提示 |
| `--non-interactive` | 非交互模式 |

## 重置范围

可以组合多个范围，用 `+` 连接：

| 范围 | 包含内容 |
|------|----------|
| `config` | 主配置文件 (`~/.openclaw/openclaw.json`) |
| `creds` | 渠道凭据 (`~/.openclaw/credentials/`) |
| `sessions` | 会话数据 (`~/.openclaw/sessions/`) |
| `workspace` | 代理工作区 (`~/.openclaw/workspace/`) |
| `all` | 所有上述内容 |

## 示例

### 预演查看

```bash
$ openclaw reset --dry-run
Would remove:
  - ~/.openclaw/openclaw.json (config)
  - ~/.openclaw/credentials/ (credentials)
  - ~/.openclaw/sessions/ (sessions)

Run without --dry-run to actually remove.
```

### 只重置配置

```bash
openclaw reset --scope config --yes
```

### 完全重置

```bash
openclaw reset --scope all --yes --non-interactive
```

### 保留配置，只清除数据

```bash
openclaw reset --scope creds+sessions --yes
```

## 重要提醒

**不会删除**：
- CLI 程序本身（使用 `openclaw uninstall` 卸载服务）
- npm 全局安装的 openclaw 包

**会删除**：
- 配置文件（需要重新配置）
- 渠道凭据（需要重新登录）
- 会话历史（不可恢复）
- 工作区文件（如果包含在 scope 中）

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 权限错误 | 文件被锁定 | 停止网关后重试 |
| 文件未删除 | 路径不存在 | 正常，忽略即可 |
| 网关仍运行 | reset 不停止服务 | 先运行 `openclaw gateway stop` |

## 最佳实践

1. **始终先预演**：使用 `--dry-run` 确认范围
2. **备份重要数据**：工作区中的自定义配置
3. **停止网关**：重置前停止运行的网关
4. **记录凭据**：重置后需要重新登录渠道
