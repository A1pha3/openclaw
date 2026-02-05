---
summary: "OpenClaw托管浏览器完整指南——自动化操作、使用场景与故障排查"
read_when:
  - 配置代理控制的浏览器自动化
  - 理解为什么OpenClaw使用独立浏览器而非个人Chrome
  - 在macOS应用中实现浏览器设置和生命周期管理
title: "浏览器（OpenClaw托管）"
---

# 浏览器工具（OpenClaw托管）

本指南全面介绍OpenClaw的托管浏览器系统，帮助您理解如何通过代理控制独立的浏览器实例，实现安全的Web自动化。完成本章节学习后，您将能够配置和使用OpenClaw浏览器，理解其架构设计，并掌握常见问题的排查方法。

## 学习目标

完成本章节学习后，您将能够：

### 基础目标（必掌握）

- 理解OpenClaw托管浏览器的核心概念和设计理念
- 掌握浏览器配置文件的管理（创建、切换、删除）
- 执行基础的浏览器操作（导航、点击、输入、截图）
- 理解openclaw托管模式与Chrome扩展中继的区别

### 进阶目标（建议掌握）

- 配置多浏览器配置文件（工作、远程等场景）
- 实现远程CDP控制
- 集成Browserless等远程浏览器服务
- 配置浏览器安全策略

### 专家目标（挑战）

- 优化浏览器自动化工作流程
- 设计和实现定制化的浏览器控制策略
- 排查复杂的浏览器兼容性问题

---

## 第一部分：核心概念解析

### 为什么需要独立的托管浏览器

在理解具体用法之前，我们需要先理解OpenClaw为什么采用独立的托管浏览器设计。

**设计背景**：

传统的AI助手集成浏览器通常采用两种方式：直接控制用户已安装的浏览器，或使用无头浏览器进行后台操作。然而，这两种方式都存在问题：

| 方案 | 问题 |
|------|------|
| 直接控制用户浏览器 | 安全风险——AI可能访问用户的敏感数据（密码、会话等） |
| 独立无头浏览器 | 隔离性好但功能有限——缺乏完整的浏览器环境 |

**OpenClaw的解决方案**：

```
OpenClaw托管浏览器 = 独立的浏览器实例 + 专用的用户数据目录 + CDP控制接口

优势：
├── 隔离性——永不触及用户的个人浏览器配置
├── 确定性——按targetId定位元素，而非依赖视觉位置
├── 可控性——支持多配置文件，每个配置文件独立运行
└── 安全性——浏览器控制仅限loopback，通过网关认证
```

### 托管模式与Chrome扩展中继的区别

OpenClaw提供两种浏览器控制模式，理解它们的区别对于正确配置至关重要：

| 特性 | openclaw托管模式 | Chrome扩展中继 |
|------|------------------|----------------|
| 浏览器实例 | 独立的浏览器进程 | 使用系统Chrome标签页 |
| 隔离性 | 完全隔离 | 共享Chrome实例 |
| 配置要求 | 无需安装扩展 | 需要安装OpenClaw扩展 |
| CDP端口 | 18800-18899范围 | 固定18792端口 |
| 适用场景 | 自动化测试、爬虫 | 快速集成现有浏览器 |

**选择建议**：
- 自动化任务首选openclaw托管模式
- 需要使用已登录的浏览器配置时选择扩展中继
- 生产环境建议使用openclaw托管模式

### 浏览器架构设计

OpenClaw浏览器采用分层架构设计：

```
┌─────────────────────────────────────────────────────────────┐
│                      代理（Agent）                           │
├─────────────────────────────────────────────────────────────┤
│                      网关（Gateway）                         │
│  ┌─────────────────┐  ┌───────────────────────────────────┐  │
│  │   控制服务器     │  │          CDP中继                   │  │
│  │  (loopback)     │  │  (loopback:18792)                │  │
│  └────────┬────────┘  └───────────────────────────────────┘  │
├───────────┼──────────────────────────────────────────────────┤
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────────────────────────────────────────────────┐
│  │              Chromium-based Browser                        │
│  │  (Chrome/Brave/Edge/Chromium)                             │
│  │  ├── 专用用户数据目录                                      │
│  │  ├── CDP端口监听                                          │
│  │  └── Playwright集成（如已安装）                           │
│  └─────────────────────────────────────────────────────────────┘
```

**核心组件说明**：

1. **控制服务器**：小型HTTP服务，接受浏览器控制请求
2. **CDP（Chrome DevTools Protocol）**：Chrome DevTools协议，用于与浏览器通信
3. **Playwright**：高级浏览器自动化库，在CDP之上提供便捷API
4. **用户数据目录**：独立的浏览器配置文件存储位置

---

## 第二部分：快速开始

### 2.1 基础操作流程

以下是使用OpenClaw浏览器的标准流程：

```bash
# 第一步：检查浏览器状态
openclaw browser --browser-profile openclaw status

# 第二步：启动浏览器（如未运行）
openclaw browser --browser-profile openclaw start

# 第三步：打开网页
openclaw browser --browser-profile openclaw open https://example.com

# 第四步：获取页面快照
openclaw browser --browser-profile openclaw snapshot

# 第五步：执行交互操作
openclaw browser --browser-profile openclaw click 12
openclaw browser --browser-profile openclaw type 23 "hello"
```

**术语说明**：
- `browser-profile`：浏览器配置文件的名称，默认为openclaw
- `snapshot`：获取当前页面的可访问性树快照
- 数字引用（12、23等）：来自snapshot的元素标识符

### 2.2 代理工具调用

通过代理使用时，浏览器工具的调用方式：

```json
{
  "tool": "browser",
  "action": "snapshot",
  "profile": "openclaw"
}
```

```json
{
  "tool": "browser",
  "action": "act",
  "kind": "click",
  "ref": "12"
}
```

---

## 第三部分：配置文件管理

### 3.1 配置文件类型

OpenClaw支持三种类型的浏览器配置文件：

| 类型 | 说明 | 配置方式 |
|------|------|----------|
| **openclaw托管** | 专用的浏览器实例，有独立的用户数据目录 | cdpPort自动分配 |
| **远程CDP** | 连接到远程运行的浏览器实例 | 显式指定cdpUrl |
| **扩展中继** | 通过Chrome扩展控制本地Chrome | 使用chrome配置文件 |

### 3.2 创建新配置文件

```bash
# 创建openclaw托管模式的配置文件
openclaw browser create-profile \
  --name work \
  --driver openclaw \
  --cdp-port 18801 \
  --color "#0066CC"

# 创建扩展中继模式的配置文件
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### 3.3 配置文件配置示例

完整配置结构：

```json5
{
  browser: {
    enabled: true,
    remoteCdpTimeoutMs: 1500,
    remoteCdpHandshakeTimeoutMs: 3000,
    defaultProfile: "openclaw",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: {
        cdpPort: 18800,
        color: "#FF4500"
      },
      work: {
        cdpPort: 18801,
        color: "#0066CC"
      },
      remote: {
        cdpUrl: "http://10.0.0.42:9222",
        color: "#00AA00"
      },
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

**配置项说明**：

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| enabled | 是否启用浏览器功能 | true |
| defaultProfile | 默认使用的配置文件 | "chrome" |
| executablePath | 浏览器可执行文件路径 | 自动检测 |
| headless | 是否以无头模式运行 | false |
| noSandbox | 是否禁用沙箱（Linux用） | false |
| cdpPort | CDP监听端口（openclaw模式） | 自动分配 |
| cdpUrl | CDP地址（远程/CDP模式） | 无 |

---

## 第四部分：详细使用说明

### 4.1 标签页管理

```bash
# 列出所有标签页
openclaw browser tabs

# 查看特定标签页信息
openclaw browser tab 2

# 新建标签页
openclaw browser tab new

# 切换到指定标签页（序号）
openclaw browser tab select 2

# 关闭指定标签页
openclaw browser tab close 2

# 打开URL
openclaw browser open https://example.com

# 聚焦标签页
openclaw browser focus abcd1234

# 关闭标签页
openclaw browser close abcd1234
```

### 4.2 页面导航与操作

```bash
# 导航到URL
openclaw browser navigate https://example.com

# 调整窗口大小
openclaw browser resize 1280 720

# 点击元素（数字引用）
openclaw browser click 12

# 双击元素
openclaw browser click 12 --double

# 输入文本
openclaw browser type 23 "hello world"

# 按回车键
openclaw browser press Enter

# 悬停元素
openclaw browser hover 44

# 拖拽元素
openclaw browser drag 10 11

# 下拉选择
openclaw browser select 9 OptionA OptionB

# 滚动到可见
openclaw browser scrollintoview e12

# 填写表单
openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'
```

### 4.3 页面检查与调试

```bash
# 获取页面快照（AI模式，默认）
openclaw browser snapshot

# 获取ARIA快照（可访问性树）
openclaw browser snapshot --format aria

# 获取交互式快照（最常用）
openclaw browser snapshot --interactive

# 高效快照（紧凑模式）
openclaw browser snapshot --efficient

# 带截图标签的快照
openclaw browser snapshot --labels

# 限定范围的快照
openclaw browser snapshot --selector "#main"
openclaw browser snapshot --frame "iframe#main"

# 查看控制台错误
openclaw browser console --level error

# 清除错误日志
openclaw browser errors --clear

# 查看网络请求
openclaw browser requests --filter api --clear

# 高亮元素
openclaw browser highlight e12

# 开始性能追踪
openclaw browser trace start

# 停止性能追踪
openclaw browser trace stop
```

### 4.4 截图与导出

```bash
# 截图（视口范围）
openclaw browser screenshot

# 整页截图
openclaw browser screenshot --full-page

# 指定元素截图
openclaw browser screenshot --ref 12

# 导出PDF
openclaw browser pdf

# 获取响应体
openclaw browser responsebody "**/api" --max-chars 5000
```

### 4.5 等待机制

```bash
# 等待元素出现
openclaw browser wait "#main"

# 等待URL匹配
openclaw browser wait --url "**/dash"

# 等待加载完成
openclaw browser wait --load networkidle

# 等待JS条件满足
openclaw browser wait --fn "window.ready===true"

# 组合等待条件
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

### 4.6 等待增强说明

OpenClaw的wait命令支持多种等待条件，可以灵活组合：

**URL模式匹配语法**：
- `*` 匹配任意字符
- `**` 匹配任意路径
- `?` 匹配单个字符

**加载状态选项**：
- `domcontentloaded`：DOM加载完成
- `load`：所有资源加载完成
- `networkidle`：网络空闲

### 4.7 文件上传与对话框

```bash
# 准备文件上传（点击上传按钮前）
openclaw browser upload /tmp/file.pdf

# 或直接设置文件输入
openclaw browser upload /tmp/file.pdf --input-ref 12

# 处理对话框
openclaw browser dialog --accept
```

**注意**：`upload`和`dialog`是预备调用，需要在触发选择器或对话框的点击/按压之前执行。

---

## 第五部分：快照与引用系统

### 5.1 两种快照模式

OpenClaw提供两种快照模式，理解它们的区别对于高效使用浏览器工具至关重要：

#### AI快照（数字引用）

```bash
# 默认模式
openclaw browser snapshot

# 等价于
openclaw browser snapshot --format ai
```

**特点**：
- 输出包含数字引用的文本快照
- 引用格式：`aria-ref="12"`
- 操作示例：`openclaw browser click 12`
- 内部通过Playwright的aria-ref解析

**适用场景**：快速原型开发、AI驱动的自动化

#### 角色快照（element引用）

```bash
# 交互式快照
openclaw browser snapshot --interactive

# 紧凑模式
openclaw browser snapshot --compact

# 带深度的树状结构
openclaw browser snapshot --depth 6
```

**特点**：
- 基于ARIA角色的元素定位
- 引用格式：`[ref=e12]`
- 操作示例：`openclaw browser click e12`
- 内部通过`getByRole()`解析

**适用场景**：稳定的自动化测试、需要精确元素定位的场景

### 5.2 引用使用指南

**引用稳定性**：
- 引用在导航之间**不稳定**——页面变化后引用可能失效
- 如果操作失败，重新运行`snapshot`获取新引用

**选择器优先级**：
1. 优先使用角色引用（e12格式）——更稳定
2. 数字引用（12格式）适合AI模式——更灵活
3. 不建议使用CSS选择器——违背设计原则

**iframe处理**：
- 角色快照使用`--frame`限定作用域后，引用仅在该iframe内有效
- AI快照不支持iframe限定

---

## 第六部分：高级配置

### 6.1 使用Brave或其他浏览器

OpenClaw支持多种Chromium内核浏览器：

```bash
# macOS
openclaw config set browser.executablePath "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"

# Linux
openclaw config set browser.executablePath "/usr/bin/brave-browser"

# Windows
openclaw config set browser.executablePath "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
```

**自动检测顺序**：
1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

### 6.2 远程CDP配置

连接到远程运行的浏览器实例：

```json5
{
  browser: {
    profiles: {
      remote: {
        cdpUrl: "http://10.0.0.42:9222",
        color: "#00AA00"
      }
    }
  }
}
```

**认证支持**：
- 查询令牌：`https://provider.example?token=<token>`
- HTTP Basic认证：`https://user:pass@provider.example`

### 6.3 Browserless集成

Browserless是托管的Chromium服务：

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

**注意事项**：
- 使用真实的API密钥替换`<BROWSERLESS_API_KEY>`
- 选择与账户匹配的区域端点

### 6.4 节点浏览器代理

对于远程网关，OpenClaw支持自动将浏览器操作路由到有浏览器的节点：

```bash
# 在节点上禁用代理
openclaw config set nodeHost.browserProxy.enabled false

# 在网关上禁用
openclaw config set gateway.nodes.browser.mode "off"
```

---

## 第七部分：使用场景

### 场景一：Web表单自动填写

**需求**：自动化填写长表单并提交

**步骤**：
```bash
# 1. 导航到表单页面
openclaw browser navigate https://example.com/form

# 2. 获取页面快照
openclaw browser snapshot --interactive

# 3. 填写各字段
openclaw browser type <ref> "值1"
openclaw browser type <ref> "值2"

# 4. 提交表单
openclaw browser click <submit-ref>
```

### 场景二：网页内容抓取

**需求**：从动态网页提取结构化数据

**步骤**：
```bash
# 1. 导航到目标页面
openclaw browser navigate https://example.com/data

# 2. 等待内容加载
openclaw browser wait --load networkidle

# 3. 获取快照
openclaw browser snapshot --format aria

# 4. 使用evaluate执行提取逻辑
openclaw browser evaluate --fn "(...) => {...}" --ref <table-ref>
```

### 场景三：端到端测试验证

**需求**：验证Web应用的交互流程

**步骤**：
```bash
# 1. 开始追踪
openclaw browser trace start

# 2. 执行测试流程
openclaw browser navigate https://app.example.com
openclaw browser click <login-ref>
openclaw browser type <email-ref> "test@example.com"
openclaw browser type <password-ref> "password"
openclaw browser click <submit-ref>

# 3. 验证结果
openclaw browser wait --fn "document.getElementById('success') !== null"

# 4. 停止追踪
openclaw browser trace stop
```

### 场景四：页面截图存档

**需求**：生成网页的完整截图

**步骤**：
```bash
# 1. 导航并等待加载
openclaw browser navigate https://example.com/page
openclaw browser wait --load networkidle

# 2. 截图
openclaw browser screenshot --full-page --ref <page-ref>

# 3. 或导出PDF
openclaw browser pdf
```

### 场景五：多配置文件管理

**需求**：隔离不同用途的浏览器配置

**配置**：
```json5
{
  browser: {
    defaultProfile: "openclaw",
    profiles: {
      openclaw: {
        cdpPort: 18800,
        color: "#FF4500"
      },
      work: {
        cdpPort: 18801,
        color: "#0066CC"
      },
      remote: {
        cdpUrl: "http://10.0.0.42:9222",
        color: "#00AA00"
      }
    }
  }
}
```

**使用**：
```bash
# 使用工作配置
openclaw browser --browser-profile work navigate https://work.example.com

# 使用远程配置
openclaw browser --browser-profile remote navigate https://remote.example.com
```

---

## 第八部分：安全与隐私

### 8.1 核心安全原则

OpenClaw浏览器设计遵循以下安全原则：

| 原则 | 说明 |
|------|------|
| **隔离性** | 专用用户数据目录，永不触及个人浏览器配置 |
| **本地性** | 浏览器控制仅限loopback，远程访问需认证 |
| **最小权限** | 默认禁用高风险操作，需要时显式启用 |
| **可追溯性** | 所有操作可记录和审计 |

### 8.2 安全配置建议

```json5
{
  browser: {
    // 仅允许本地连接
    allowedHosts: ["127.0.0.1", "localhost"],

    // 禁用危险功能（如不需要）
    evaluateEnabled: false,

    // 启用下载确认
    downloadPrompt: true
  }
}
```

### 8.3 敏感数据保护

**注意事项**：
- 浏览器配置文件中可能包含已登录的会话——视为敏感数据
- `evaluate`和`wait --fn`在页面上下文中执行任意JS
- 提示注入可能影响这些操作

**建议**：
- 不需要evaluate时设置`browser.evaluateEnabled=false`
- 对敏感网站使用专用的临时配置文件
- 定期清理不再使用的配置文件

### 8.4 远程访问安全

**网关暴露**：
- 浏览器控制服务绑定到loopback
- 通过网关认证或节点配对控制访问
- 建议保持网关在专用网络（Tailscale）上

**远程CDP**：
- 将CDP URL和令牌视为机密
- 优先使用HTTPS端点和短命令牌
- 避免将令牌直接嵌入配置文件

---

## 第九部分：故障排除

### 9.1 常见问题分类

| 问题类型 | 典型症状 | 排查方向 |
|----------|----------|----------|
| **启动失败** | 浏览器无法启动 | 检查可执行路径、系统依赖 |
| **连接失败** | CDP连接超时 | 检查端口、防火墙 |
| **操作失败** | 点击/输入无效 | 检查元素引用、页面状态 |
| **渲染问题** | 页面显示异常 | 检查JS执行、CDP版本 |

### 9.2 浏览器无法启动

**症状**：运行`openclaw browser start`失败

**排查步骤**：

```bash
# 1. 检查浏览器是否安装
which google-chrome
which chromium
which brave

# 2. 检查可执行路径配置
openclaw config get browser.executablePath

# 3. 尝试手动启动浏览器测试
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version

# 4. 查看详细错误日志
openclaw logs --category browser
```

**常见原因**：

| 原因 | 解决方案 |
|------|----------|
| 浏览器未安装 | 安装Chrome/Brave/Edge/Chromium |
| 可执行路径错误 | 设置正确的`executablePath` |
| 权限不足 | 确保有执行权限 |
| 端口被占用 | 关闭占用端口的程序 |

### 9.3 CDP连接失败

**症状**：浏览器启动但无法连接

**排查步骤**：

```bash
# 1. 检查CDP端口是否监听
netstat -an | grep 18800

# 2. 测试本地连接
curl http://127.0.0.1:18800/json

# 3. 检查防火墙设置
sudo ufw status

# 4. 尝试远程CDP模式
openclaw config set browser.profiles.openclaw.attachOnly true
```

### 9.4 操作失败——不可见元素

**症状**：`click`操作返回"元素不可见"

**排查步骤**：

```bash
# 1. 重新获取快照
openclaw browser snapshot --interactive

# 2. 检查元素是否在视口内
openclaw browser scrollintoview <ref>

# 3. 等待元素可见
openclaw browser wait "#selector"

# 4. 高亮元素查看实际位置
openclaw browser highlight <ref>
```

### 9.5 操作失败——严格模式违规

**症状**：Playwright报告"strict mode violation"

**原因**：Playwright的严格模式要求元素定位唯一

**解决方案**：

```bash
# 1. 使用更精确的选择器
openclaw browser snapshot --selector "#specific-form input.name"

# 2. 使用角色快照获取唯一引用
openclaw browser snapshot --interactive
# 使用新的ref重试

# 3. 或临时禁用严格模式（如果适用）
```

### 9.6 截图失败

**症状**：`screenshot`命令失败

**排查步骤**：

```bash
# 1. 检查页面是否完全加载
openclaw browser wait --load networkidle

# 2. 尝试截取特定元素而非整页
openclaw browser screenshot --ref <element-ref>

# 3. 检查是否有覆盖层阻止截图
openclaw browser evaluate --fn "() => document.querySelector('.modal')?.remove()"

# 4. 尝试使用无头模式
openclaw config set browser.headless true
```

### 9.7 调试工作流

当操作失败时，使用以下系统化的调试流程：

```bash
# 第一步：获取交互式快照
openclaw browser snapshot --interactive

# 第二步：使用快照中的ref执行操作
openclaw browser click <new-ref>

# 第三步：如果仍然失败，高亮元素
openclaw browser highlight <ref>

# 第四步：检查控制台错误
openclaw browser console --level error
openclaw browser errors --clear

# 第五步：查看网络请求
openclaw browser requests --filter api --clear

# 第六步：启用详细追踪
openclaw browser trace start
# 复现问题
openclaw browser trace stop
```

### 9.8 Linux特定问题

对于Linux系统，特别是使用snap安装的Chromium，可能遇到兼容性问题。

**常见问题**：

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 浏览器崩溃 | snap Chromium沙箱问题 | 使用`noSandbox: true` |
| 字体缺失 | 容器环境缺少字体 | 安装必要的字体包 |
| GPU加速失败 | 虚拟环境不支持 | 禁用GPU加速 |

**配置示例**：

```json5
{
  browser: {
    noSandbox: true,
    executablePath: "/usr/bin/chromium",
    headless: true
  }
}
```

详细Linux排查指南请参阅[浏览器故障排除（Linux）](/tools/browser-linux-troubleshooting)。

---

## 第十部分：专家思维模型

### 10.1 浏览器操作决策树

```
开始浏览器操作
    │
    ▼
需要与页面交互吗？
    │
    ├─否→ 仅需读取内容
    │     │
    │     ▼
    │     snapshot --format aria
    │     │
    │     ▼
    │     使用grep/evaluate提取数据
    │
    └─是→ 需要用户交互
          │
          ▼
      获取交互式快照
      snapshot --interactive
          │
          ▼
      引用稳定吗？
          │
          ├─否→ 重新获取快照，使用新ref
          │
          └─是→ 执行操作
                │
                ▼
            操作成功吗？
                │
                ├─否→ 排查：highlight + console
                │
                └─是→ 继续下一步操作
```

### 10.2 性能优化策略

| 场景 | 优化建议 |
|------|----------|
| 大量截图 | 使用`--ref`截取元素而非整页 |
| 频繁导航 | 复用浏览器实例而非每次新建 |
| 等待加载 | 使用`--load networkidle`而非固定延迟 |
| 大页面 | 使用`--efficient`快照模式 |
| 多步骤流程 | 使用`background` + `process`轮询 |

### 10.3 稳定性最佳实践

1. **每次交互前获取快照**——引用可能在导航后失效
2. **使用角色快照进行精确操作**——比AI快照更稳定
3. **添加明确的等待条件**——避免竞态条件
4. **记录操作序列**——便于复现问题
5. **使用唯一配置文件**——避免状态污染

---

## 适用场景速查

| 场景 | 推荐配置 | 关键命令 |
|------|----------|----------|
| 快速内容提取 | openclaw模式 | snapshot --format aria |
| 自动化表单填写 | openclaw模式 | snapshot --interactive + click/type |
| 端到端测试 | openclaw模式 + trace | trace start/stop |
| 页面截图存档 | openclaw模式 | screenshot --full-page |
| 使用已登录配置 | extension模式 | chrome配置文件 |
| 远程服务器执行 | remote CDP模式 | remote配置文件 |
| 云端浏览器服务 | Browserless集成 | browserless配置文件 |

---

## 相关文档

- [Chrome扩展中继](/tools/chrome-extension)——使用现有Chrome标签页
- [浏览器登录与X/Twitter发布](/tools/browser-login)——登录和反机器人处理
- [浏览器故障排除（Linux）](/tools/browser-linux-troubleshooting)——Linux特定问题
- [Playwright文档](https://playwright.dev)——高级浏览器自动化

---

**重要提示**：OpenClaw浏览器是代理自动化的安全隔离表面，而非日常使用的浏览器。建议为不同的自动化任务使用独立的配置文件，避免与个人浏览器数据混淆。配置敏感操作时，始终考虑安全影响。