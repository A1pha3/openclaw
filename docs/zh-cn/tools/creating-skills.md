---
title: "创建技能"
---

# 创建自定义技能 🛠

OpenClaw 旨在易于扩展。"技能"是向你的助手添加新功能的主要方式。

## 什么是技能？

技能是一个目录，包含一个 `SKILL.md` 文件（向 LLM 提供指令和工具定义）和可选的一些脚本或资源。

## 逐步指南：你的第一个技能

### 1. 创建目录

技能位于你的工作区，通常是 `~/.openclaw/workspace/skills/`。为你的技能创建一个新文件夹：

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. 定义 `SKILL.md`

在该目录中创建一个 `SKILL.md` 文件。该文件使用 YAML 前置元数据来获取元数据，使用 Markdown 来获取指令。

```markdown
---
name: hello_world
description: 一个说 hello 的简单技能。
---

# Hello World 技能

当用户要求问候时，使用 `echo` 工具说 "Hello from your custom skill!"。
```

### 3. 添加工具（可选）

你可以在前置元数据中定义自定义工具，或指示代理使用现有的系统工具（如 `bash` 或 `browser`）。

### 4. 刷新 OpenClaw

要求你的代理"刷新技能"或重启网关。OpenClaw 将发现新目录并索引 `SKILL.md`。

## 最佳实践

- **简洁**：指示模型_做什么_，而不是如何成为 AI。
- **安全第一**：如果你的技能使用 `bash`，确保提示不允许来自不受信任用户输入的任意命令注入。
- **本地测试**：使用 `openclaw agent --message "use my new skill"` 进行测试。

## 共享技能

你也可以浏览并向 [ClawHub](https://clawhub.com) 贡献技能。
