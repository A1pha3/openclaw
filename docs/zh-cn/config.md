---
summary: "配置根索引 - 配置系统入口"
read_when:
  - 了解配置系统
  - 查找配置文档
  - 开始配置 OpenClaw
title: "配置概述"
---

# ⚙️ 配置概述

OpenClaw 使用 JSON5 格式的配置文件，支持注释和尾随逗号。

---

## 📁 配置文件

**默认位置**：`~/.openclaw/openclaw.json`

**格式**：JSON5（支持注释、尾随逗号）

---

## 🎯 配置结构

```json5
{
  // 代理配置
  agents: { ... },
  
  // 渠道配置
  channels: { ... },
  
  // 网关配置
  gateway: { ... },
  
  // 模型配置
  models: { ... }
}
```

---

## 📚 配置文档

| 文档 | 内容 |
|------|------|
| [**配置参考**](/zh-CN/config/reference) | 所有配置项详解 |
| [**配置示例**](/zh-CN/config/examples) | 常见场景模板 |
| [配置概述](/zh-CN/config/index) | 基础配置说明 |

---

## 🚀 快速配置

### 最小配置

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace"
    }
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TOKEN"
    }
  }
}
```

### 使用 CLI 配置

```bash
# 查看配置
openclaw config get

# 设置值
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4"

# 编辑配置
openclaw config edit
```

---

## 🔧 配置验证

```bash
# 检查配置
openclaw doctor

# 自动修复
openclaw doctor --fix
```

---

## 📖 相关文档

- [配置参考](/zh-CN/config/reference)
- [配置示例](/zh-CN/config/examples)
- [故障排除](/zh-CN/help/troubleshooting)
