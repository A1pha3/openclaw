---
summary: "CLI 参考 - `openclaw browser` (配置文件、标签页、操作、扩展中继)"
read_when:
  - 使用 `openclaw browser` 并需要常见任务示例
  - 通过节点主机控制另一台机器上的浏览器
  - 使用 Chrome 扩展中继（通过工具栏按钮附加/分离）
title: "browser"
---

# `openclaw browser`

管理 OpenClaw 的浏览器控制服务器并运行浏览器操作（标签页、快照、截图、导航、点击、输入）。

## 为什么需要这个命令

浏览器控制让代理能够：

- **网页自动化**：自动填表、点击、导航
- **信息采集**：截图、提取内容
- **远程控制**：控制另一台机器上的浏览器

## 相关文档

- 浏览器工具和 API：[浏览器工具](/zh-cn/tools/browser)
- Chrome 扩展中继：[Chrome 扩展](/zh-cn/tools/chrome-extension)

## 通用选项

| 选项 | 说明 |
|------|------|
| `--url <gatewayWsUrl>` | 网关 WebSocket URL（默认使用配置） |
| `--token <token>` | 网关令牌（如需要） |
| `--timeout <ms>` | 请求超时（毫秒） |
| `--browser-profile <name>` | 选择浏览器配置文件（默认使用配置） |
| `--json` | 机器可读输出（支持的命令） |

## 快速开始（本地）

```bash
# 查看标签页
openclaw browser --browser-profile chrome tabs

# 启动 OpenClaw 管理的浏览器
openclaw browser --browser-profile openclaw start

# 打开网址
openclaw browser --browser-profile openclaw open https://example.com

# 获取页面快照
openclaw browser --browser-profile openclaw snapshot
```

## 配置文件

配置文件是命名的浏览器路由配置。实际使用：

- **`openclaw`**：启动/附加到 OpenClaw 管理的专用 Chrome 实例（隔离的用户数据目录）
- **`chrome`**：通过 Chrome 扩展中继控制现有的 Chrome 标签页

### 管理配置文件

```bash
# 列出配置文件
openclaw browser profiles

# 创建新配置文件
openclaw browser create-profile --name work --color "#FF5A36"

# 删除配置文件
openclaw browser delete-profile --name work
```

使用特定配置文件：

```bash
openclaw browser --browser-profile work tabs
```

## 标签页操作

```bash
# 列出所有标签页
openclaw browser tabs

# 打开新标签页
openclaw browser open https://docs.openclaw.ai

# 聚焦标签页
openclaw browser focus <targetId>

# 关闭标签页
openclaw browser close <targetId>
```

## 快照、截图和操作

### 快照

```bash
# 获取页面 DOM 快照
openclaw browser snapshot
```

### 截图

```bash
# 截取当前页面
openclaw browser screenshot
```

### 导航/点击/输入

基于引用的 UI 自动化：

```bash
# 导航到 URL
openclaw browser navigate https://example.com

# 点击元素
openclaw browser click <ref>

# 输入文本
openclaw browser type <ref> "hello"
```

## Chrome 扩展中继

此模式让代理控制你手动附加的现有 Chrome 标签页（不会自动附加）。

### 安装扩展

```bash
# 安装扩展到稳定路径
openclaw browser extension install

# 获取扩展路径
openclaw browser extension path
```

然后：Chrome → `chrome://extensions` → 启用"开发者模式" → "加载已解压的扩展程序" → 选择打印的文件夹。

完整指南：[Chrome 扩展](/zh-cn/tools/chrome-extension)

## 远程浏览器控制（节点主机代理）

如果网关运行在与浏览器不同的机器上，在有 Chrome/Brave/Edge/Chromium 的机器上运行**节点主机**。网关会将浏览器操作代理到该节点（无需单独的浏览器控制服务器）。

配置选项：

| 配置项 | 说明 |
|--------|------|
| `gateway.nodes.browser.mode` | 控制自动路由 |
| `gateway.nodes.browser.node` | 固定到特定节点（多节点连接时） |

安全和远程设置：[浏览器工具](/zh-cn/tools/browser)、[远程访问](/zh-cn/gateway/remote)、[Tailscale](/zh-cn/gateway/tailscale)、[安全](/zh-cn/gateway/security)

## 使用场景

### 网页自动化

```bash
# 打开页面并截图
openclaw browser open https://example.com
openclaw browser screenshot
```

### 信息提取

```bash
# 获取页面快照用于分析
openclaw browser snapshot --json > page-content.json
```

### 多配置文件工作

```bash
# 工作配置文件
openclaw browser --browser-profile work open https://work-app.com

# 个人配置文件
openclaw browser --browser-profile personal open https://personal-site.com
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 无法连接浏览器 | 检查浏览器是否运行且扩展已安装 |
| 远程控制失败 | 确认节点主机已连接到网关 |
| 操作超时 | 增加 `--timeout` 值 |
| 配置文件不存在 | 使用 `openclaw browser profiles` 查看可用配置文件 |
