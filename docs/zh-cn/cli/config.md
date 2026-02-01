---
summary: "CLI config 命令 - 配置管理"
read_when:
  - 查看配置
  - 修改配置
  - 配置操作
title: "config 命令"
---

# config 命令

管理 OpenClaw 配置。

---

## 用法

```bash
openclaw config <子命令>
```

---

## 子命令

| 子命令 | 说明 |
|--------|------|
| `get <键>` | 获取配置值 |
| `set <键> <值>` | 设置配置值 |
| `unset <键>` | 删除配置项 |

---

## 示例

### 查看配置

```bash
openclaw config get agents.defaults.model
```

### 设置配置

```bash
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4"
```

### 删除配置

```bash
openclaw config unset channels.discord
```

---

## 相关文档

- [配置参考](/zh-CN/config/reference)
- [配置示例](/zh-CN/config/examples)
