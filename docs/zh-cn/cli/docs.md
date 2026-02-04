---
summary: "`openclaw docs` 命令参考 - 在终端搜索在线文档"
read_when:
  - 想在终端快速搜索 OpenClaw 文档
  - 不想打开浏览器查文档
title: "docs"
---

# `openclaw docs`

在终端搜索 OpenClaw 在线文档索引。

## 为什么需要这个

开发过程中经常需要查文档，但切换到浏览器会打断工作流程。`openclaw docs` 让你直接在终端搜索，快速找到需要的信息。

## 基本用法

```bash
# 搜索浏览器扩展相关文档
openclaw docs browser extension

# 搜索沙箱主机控制相关文档
openclaw docs sandbox allowHostControl

# 搜索配置相关文档
openclaw docs configuration
```

## 使用技巧

| 场景 | 搜索关键词 |
|------|-----------|
| 找某个命令的用法 | `openclaw docs <命令名>` |
| 找某个配置项 | `openclaw docs <配置键>` |
| 找某个功能 | `openclaw docs <功能名>` |

## 相关命令

- 查看帮助: `openclaw --help`
- 查看版本: `openclaw --version`
