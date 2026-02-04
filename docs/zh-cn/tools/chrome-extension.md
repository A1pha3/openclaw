---
summary: "Chrome 扩展：让 OpenClaw 控制你现有的 Chrome 标签页"
read_when:
  - 想让智能体控制现有的 Chrome 标签页（工具栏按钮）
  - 需要通过 Tailscale 实现远程 Gateway + 本地浏览器自动化
  - 想了解浏览器接管的安全影响
title: "Chrome 扩展"
---

# Chrome 扩展（浏览器中继）

OpenClaw Chrome 扩展让智能体可以控制你**现有的 Chrome 标签页**（你的日常 Chrome 窗口），而不是启动一个单独的 OpenClaw 托管 Chrome 配置文件。

通过**单个 Chrome 工具栏按钮**进行附加/分离操作。

## 这是什么（概念）

整个系统由三部分组成：

| 组件 | 功能 |
|------|------|
| **浏览器控制服务**（Gateway 或节点） | 智能体/工具调用的 API（通过 Gateway） |
| **本地中继服务器**（回环 CDP） | 连接控制服务器和扩展的桥梁（默认 `http://127.0.0.1:18792`） |
| **Chrome MV3 扩展** | 使用 `chrome.debugger` 附加到活动标签页，将 CDP 消息传递给中继 |

然后 OpenClaw 通过标准的 `browser` 工具界面（选择正确的配置文件）控制已附加的标签页。

## 安装/加载（开发者模式）

**第一步：安装扩展到本地路径**

```bash
openclaw browser extension install
```

**第二步：获取安装目录路径**

```bash
openclaw browser extension path
```

**第三步：在 Chrome 中加载**

1. 打开 Chrome → `chrome://extensions`
2. 启用「开发者模式」
3. 点击「加载已解压的扩展程序」→ 选择上面打印的目录
4. 固定扩展到工具栏

## 更新（无需构建）

扩展作为静态文件随 OpenClaw 发布（npm 包）发布，没有单独的「构建」步骤。

升级 OpenClaw 后：

1. 重新运行 `openclaw browser extension install` 刷新安装文件
2. Chrome → `chrome://extensions` → 点击扩展上的「重新加载」

## 使用方法（无需额外配置）

OpenClaw 内置了一个名为 `chrome` 的浏览器配置文件，它指向默认端口的扩展中继。

使用方式：

| 方式 | 命令 |
|------|------|
| CLI | `openclaw browser --browser-profile chrome tabs` |
| 智能体工具 | `browser`，设置 `profile="chrome"` |

如果想用不同的名称或不同的中继端口，创建自己的配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

## 附加/分离（工具栏按钮）

1. 打开你想让 OpenClaw 控制的标签页
2. 点击扩展图标
   - 徽章显示 `ON` 表示已附加
3. 再次点击分离

## 控制哪个标签页

- 它**不会**自动控制「你正在查看的任何标签页」
- 它**只**控制你通过点击工具栏按钮**明确附加**的标签页
- 要切换：打开另一个标签页并点击扩展图标

## 徽章状态说明

| 徽章 | 含义 |
|------|------|
| `ON` | 已附加；OpenClaw 可以控制该标签页 |
| `…` | 正在连接本地中继 |
| `!` | 无法访问中继（通常：浏览器中继服务器未在本机运行） |

如果看到 `!`：

- 确保 Gateway 在本地运行（默认设置），或者如果 Gateway 在别处运行，在本机运行一个节点主机
- 打开扩展选项页面；它会显示中继是否可达

## 远程 Gateway（使用节点主机）

### 本地 Gateway（与 Chrome 在同一台机器）— 通常**无需额外步骤**

如果 Gateway 与 Chrome 在同一台机器上运行，它会在回环地址启动浏览器控制服务并自动启动中继服务器。扩展与本地中继通信；CLI/工具调用发送到 Gateway。

### 远程 Gateway（Gateway 在其他地方）— **运行节点主机**

如果 Gateway 在另一台机器上运行，需要在运行 Chrome 的机器上启动节点主机。Gateway 会将浏览器操作代理到该节点；扩展 + 中继保持在浏览器机器本地。

如果连接了多个节点，使用 `gateway.nodes.browser.node` 固定一个或设置 `gateway.nodes.browser.mode`。

## 沙箱模式（工具容器）

如果智能体会话处于沙箱模式（`agents.defaults.sandbox.mode != "off"`），`browser` 工具可能受限：

- 默认情况下，沙箱会话通常使用**沙箱浏览器**（`target="sandbox"`），而不是宿主 Chrome
- Chrome 扩展中继接管需要控制**宿主**浏览器控制服务器

**选项：**

| 方案 | 操作 |
|------|------|
| 最简单 | 从**非沙箱**会话/智能体使用扩展 |
| 允许宿主控制 | 配置如下 |

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

然后确保工具未被工具策略拒绝，并（如需要）调用 `browser` 时设置 `target="host"`。

**调试**：`openclaw sandbox explain`

## 远程访问技巧

- 保持 Gateway 和节点主机在同一个 tailnet 上；避免将中继端口暴露给局域网或公网
- 有意识地配对节点；如果不想要远程控制，禁用浏览器代理路由（`gateway.nodes.browser.mode="off"`）

## 「extension path」工作原理

`openclaw browser extension path` 打印**已安装**的磁盘目录，其中包含扩展文件。

CLI 特意**不**打印 `node_modules` 路径。始终先运行 `openclaw browser extension install` 将扩展复制到 OpenClaw 状态目录下的稳定位置。

如果移动或删除该安装目录，Chrome 会将扩展标记为损坏，直到你从有效路径重新加载。

## 安全影响（重要）

这是强大但有风险的功能。把它当作给模型「浏览器的控制权」。

### 扩展能做什么

扩展使用 Chrome 的 debugger API（`chrome.debugger`）。附加后，模型可以：

- 在该标签页中点击/输入/导航
- 读取页面内容
- 访问该标签页已登录会话能访问的任何内容

### 风险说明

- **这不是隔离的**，不像专用的 OpenClaw 托管配置文件
- 如果你附加到日常使用的配置文件/标签页，你就授予了对该账户状态的访问权限

### 安全建议

| 建议 | 说明 |
|------|------|
| 使用专用配置文件 | 为扩展中继使用创建单独的 Chrome 配置文件（与个人浏览分开） |
| 保持 tailnet 限制 | Gateway 和节点主机仅限 tailnet；依赖 Gateway 认证 + 节点配对 |
| 避免局域网暴露 | 不要将中继端口暴露在 `0.0.0.0`，避免 Funnel（公网） |

## 相关文档

- [浏览器工具](/tools/browser) - 浏览器工具概述
- [安全审计](/gateway/security) - 安全最佳实践
- [Tailscale 设置](/gateway/tailscale) - Tailscale 配置指南
