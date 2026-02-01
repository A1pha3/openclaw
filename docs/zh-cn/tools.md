---
summary: "工具和技能系统 - 扩展 AI 能力"
read_when:
  - 了解技能系统
  - 使用工具
  - 开发自定义技能
title: "工具和技能"
---

# 🛠️ 工具和技能

OpenClaw 通过**技能系统**扩展 AI 的能力，让代理能够执行各种任务。

---

## 🎯 什么是技能？

技能是**可插拔的功能模块**，为 AI 代理提供额外能力：

- 🔍 **Web 搜索** - 搜索互联网信息
- 🌐 **浏览器** - 控制浏览器自动化
- 📝 **文件操作** - 读写编辑文件
- 🐚 **命令执行** - 运行系统命令
- 🔗 **第三方集成** - GitHub、Slack 等

---

## 🚀 使用技能

### 查看可用技能

```bash
openclaw skills list
```

### 安装技能

```bash
openclaw skills install web
openclaw skills install github
```

### 使用技能

在对话中直接使用：

```
用户：搜索今天的 AI 新闻
代理：[使用 web 搜索技能] 这是今天的新闻...
```

---

## 📁 技能目录

技能存储在：
- 内置：`~/.openclaw/skills/bundled/`
- 本地：`~/.openclaw/skills/`
- 工作区：`<workspace>/skills/`

---

## 🔧 内置工具

| 工具 | 功能 | 示例 |
|------|------|------|
| **read** | 读取文件 | `read file.txt` |
| **write** | 写入文件 | `write file.txt "内容"` |
| **edit** | 编辑文件 | `edit file.txt "旧" "新"` |
| **exec** | 执行命令 | `exec ls -la` |
| **browser** | 浏览器控制 | `browser navigate https://...` |

---

## 📚 技能文档

- [创建技能](/zh-CN/tools/creating-skills) - 开发自定义技能
- [技能配置](/zh-CN/config/reference#技能配置) - 配置选项

---

## 🆘 故障排除

**技能无法加载？**
- 检查技能目录：`openclaw skills dir`
- 查看日志：`openclaw logs`
- 验证配置：`openclaw doctor`

---

## 📖 相关文档

- [配置参考](/zh-CN/config/reference)
- [CLI 技能命令](/zh-CN/cli/index#技能管理)
- [故障排除](/zh-CN/help/troubleshooting)
