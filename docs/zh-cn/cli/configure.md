---
summary: "`openclaw configure` 命令参考（交互式配置提示）"
read_when:
  - 想要交互式调整凭据、设备或代理默认值
title: "configure"
---

# `openclaw configure`

交互式提示来设置凭据、设备和代理默认值。

## 为什么需要这个命令

`configure` 提供了友好的交互式配置体验：

- **引导式设置**：一步步完成配置
- **多选支持**：轻松选择多个选项
- **验证输入**：确保配置值有效
- **无需手动编辑**：不用直接编辑 JSON 文件

## 相关链接

- 网关配置参考：[Configuration](/zh-cn/gateway/configuration)
- Config CLI：[Config](/zh-cn/cli/config)

## 基本用法

```bash
# 启动完整配置向导
openclaw configure

# 只配置特定部分
openclaw configure --section models --section channels
```

## 与 config 命令的区别

| 命令 | 适用场景 |
|------|----------|
| `openclaw configure` | 交互式设置，多个相关选项 |
| `openclaw config get/set` | 非交互式，单个配置值 |

**提示**：`openclaw config` 不带子命令会打开相同的向导。使用 `openclaw config get|set|unset` 进行非交互式编辑。

## 配置部分

### 网关运行位置

选择网关在哪里运行，总是会更新 `gateway.mode`。如果只需要这个，可以选择 "Continue" 跳过其他部分。

### 模型配置

**Model** 部分现在包含了 `agents.defaults.models` 允许列表的多选（出现在 `/model` 和模型选择器中的模型）。

### 渠道配置

面向渠道的服务（Slack/Discord/Matrix/Microsoft Teams）在设置期间会提示频道/房间允许列表。你可以输入名称或 ID；向导会在可能时将名称解析为 ID。

## 可配置部分

| 部分 | 内容 |
|------|------|
| `gateway` | 网关运行模式和设置 |
| `models` | 默认模型、别名、回退 |
| `channels` | 渠道配置和允许列表 |
| `credentials` | API 密钥和 OAuth |
| `devices` | 设备配对和权限 |
| `agents` | 代理默认值和工作区 |

## 示例

### 完整配置

```bash
openclaw configure
```

会依次引导你完成：
1. 网关模式选择
2. 模型配置
3. 渠道设置
4. 凭据输入
5. 设备配对
6. 代理默认值

### 只配置渠道

```bash
openclaw configure --section channels
```

### 配置多个部分

```bash
openclaw configure --section models --section credentials
```

## 配置流程示例

```
$ openclaw configure

? Where should the Gateway run?
  ○ Local (this machine)
  ● Remote (connect to an existing gateway)
  ○ macOS App

? Select default model
  ○ anthropic/claude-opus-4-5
  ● anthropic/claude-sonnet-4
  ○ openai/gpt-4o
  ○ Custom...

? Configure channels?
  ☑ WhatsApp
  ☑ Telegram
  ☐ Discord
  ☐ Slack

? WhatsApp: Enter allowed phone numbers (comma separated)
  +15555550123, +15555550456

✓ Configuration saved
```

## 选项

| 选项 | 说明 |
|------|------|
| `--section <name>` | 只配置指定部分（可重复） |
| `--json` | 以 JSON 输出当前配置 |

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 配置未保存 | 中断退出 | 完成向导或使用 Ctrl+C 确认 |
| 渠道名称未解析 | 渠道未连接 | 先连接渠道再配置 |
| 选项不可用 | 依赖未安装 | 检查依赖和插件 |
