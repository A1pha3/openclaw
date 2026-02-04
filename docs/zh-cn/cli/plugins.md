---
summary: "CLI 参考 - `openclaw plugins` (列出、安装、启用/禁用、诊断插件)"
read_when:
  - 安装或管理进程内网关插件
  - 调试插件加载失败
title: "plugins"
---

# `openclaw plugins`

管理网关插件/扩展（进程内加载）。

## 为什么需要这个命令

插件扩展了 OpenClaw 的能力边界：

- **新渠道**：支持 MS Teams、Matrix、Zalo 等
- **新功能**：语音通话、自定义集成
- **模块化**：按需启用，保持核心精简

## 相关文档

- 插件系统：[插件](/zh-cn/plugin)
- 插件清单和 Schema：[插件清单](/zh-cn/plugins/manifest)
- 安全加固：[安全](/zh-cn/gateway/security)

## 命令

```bash
# 列出所有插件
openclaw plugins list

# 查看插件详情
openclaw plugins info <id>

# 启用/禁用插件
openclaw plugins enable <id>
openclaw plugins disable <id>

# 插件诊断
openclaw plugins doctor

# 更新插件
openclaw plugins update <id>
openclaw plugins update --all
```

## 内置插件

内置插件随 OpenClaw 一起安装，但默认禁用。使用 `plugins enable` 激活它们。

所有插件必须包含 `openclaw.plugin.json` 文件，其中包含内联 JSON Schema（`configSchema`，即使为空也需要）。缺失或无效的清单/Schema 会阻止插件加载并导致配置验证失败。

## 安装插件

```bash
openclaw plugins install <path-or-spec>
```

**安全提示**：将插件安装视为运行代码。建议使用固定版本。

支持的归档格式：`.zip`、`.tgz`、`.tar.gz`、`.tar`

### 链接本地插件

使用 `--link` 避免复制本地目录（添加到 `plugins.load.paths`）：

```bash
openclaw plugins install -l ./my-plugin
```

## 更新插件

```bash
# 更新特定插件
openclaw plugins update <id>

# 更新所有插件
openclaw plugins update --all

# 预览更新（不实际执行）
openclaw plugins update <id> --dry-run
```

更新仅适用于从 npm 安装的插件（记录在 `plugins.installs` 中）。

## 插件诊断

```bash
openclaw plugins doctor
```

检查：

- 插件清单有效性
- 依赖项完整性
- 配置 Schema 正确性
- 加载错误

## 使用场景

### 启用 MS Teams 支持

```bash
# 启用内置的 MS Teams 插件
openclaw plugins enable msteams

# 验证启用状态
openclaw plugins info msteams
```

### 安装第三方插件

```bash
# 从 npm 安装
openclaw plugins install some-plugin@1.0.0

# 从本地路径安装
openclaw plugins install ./my-custom-plugin
```

### 调试插件问题

```bash
# 运行插件诊断
openclaw plugins doctor

# 查看特定插件详情
openclaw plugins info <problematic-plugin>
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 插件无法加载 | 检查 `openclaw.plugin.json` 是否存在且有效 |
| 配置验证失败 | 确保 `configSchema` 字段存在（可为空对象） |
| 更新失败 | 确认插件是从 npm 安装的 |
| 启用后无效果 | 重启网关使更改生效 |
