---
summary: "通过 WKWebView + 自定义 URL 方案嵌入的助手控制 Canvas 面板架构设计与开发指南"
read_when:
  - 实现 macOS Canvas 面板
  - 为可视化工作区添加助手控制
  - 调试 WKWebView canvas 加载
  - 集成 A2UI 界面框架
title: "Canvas 画布面板"
---

# Canvas 画布面板架构与开发指南

本章节深入解析 OpenClaw macOS 应用中的 Canvas 画布面板系统，涵盖其基于 WKWebView 的技术架构、本地文件服务机制、A2UI 界面框架集成以及与网关的通信协议。通过学习本章节，你将掌握 Canvas 面板的设计原理、配置方法与自定义开发技能，能够为助手构建丰富的可视化交互界面。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 **Canvas 画布面板**的定位与核心功能场景
- [ ] 掌握 Canvas 文件存储结构与自定义 URL 方案映射机制
- [ ] 理解 WKWebView 嵌入模式与面板行为特性
- [ ] 能够通过 CLI 命令控制 Canvas 面板的显示、导航与内容更新

### 进阶目标（建议掌握）

- [ ] 深入理解 **A2UI 框架**的组件模型与消息协议（v0.8）
- [ ] 设计与实现自定义 Canvas 内容（HTML/CSS/JS 组件）
- [ ] 配置 A2UI 主机环境，实现助手驱动的动态界面渲染
- [ ] 优化 Canvas 性能，处理大文件加载与实时更新场景

### 专家目标（挑战）

- [ ] 扩展 A2UI 协议支持，实现自定义组件与消息类型
- [ ] 构建复杂的 Canvas 应用架构（多会话、微前端集成）
- [ ] 实现 Canvas 安全沙盒，处理用户输入与外部内容隔离

---

## 第一部分：架构设计原理

### 为什么需要 Canvas 画布面板

OpenClaw 作为个人 AI 助手，需要一种方式来呈现丰富的可视化交互界面。传统的文本对话虽然高效，但在以下场景中存在局限：

**场景一：信息密度展示**

当助手需要展示代码差异、数据可视化、文档预览等内容时，静态文本难以承载。此时 Canvas 面板提供了一个可编程的 WebView 容器，助手可以推送 HTML/CSS/JavaScript 内容，实现任意复杂度的界面渲染。

**场景二：实时协作界面**

A2UI 框架允许助手实时推送界面更新，实现类似现代 Web 应用的用户体验。助手可以构建可交互的表单、动态列表、实时数据看板等 UI 组件。

**场景三：自定义工具集成**

开发者可以将自定义的 Web 工具（如 API 测试工具、数据库客户端）封装为 Canvas 面板，通过助手命令调用，实现工具与 AI 能力的无缝集成。

### 技术选型：WKWebView + 自定义 URL 方案

Canvas 面板采用 WKWebView 作为渲染引擎，辅以自定义 URL 方案实现本地文件服务。这种架构设计有以下考量：

| 技术组件 | 选型理由 | 替代方案比较 |
|---------|---------|-------------|
| WKWebView | macOS 原生高性能 Web 渲染引擎，支持丰富的 JavaScript 互操作 API | WebView2（Windows）、Electron（跨平台但重量级） |
| 自定义 URL 方案 | 无需本地 HTTP 服务器即可加载本地文件，减少攻击面 | `file://` 协议（存在安全限制） |
| 本地文件服务 | 文件变更自动热重载，开发体验友好 | 打包到应用 bundle（更新需重新发布） |

**与 file:// 协议的对比**：

```
file:// 协议限制：
├── 跨域请求（CORS）完全阻止
├── 部分 Web API 不可用（navigator、fetch 限制）
├── 本地文件访问受浏览器安全策略限制

自定义 URL 方案优势：
├── 通过应用层实现虚拟文件系统映射
├── 精确控制文件访问权限
├── 支持运行时文件热重载
└── 与 macOS 应用沙盒兼容
```

### 文件存储架构

Canvas 状态存储在应用支持目录下的结构化布局中：

```
~/Library/Application Support/OpenClaw/canvas/
├── <session>/
│   ├── index.html              # 默认入口页面
│   ├── assets/
│   │   ├── app.css            # 样式文件
│   │   ├── app.js             # 脚本文件
│   │   └── images/            # 图片资源
│   ├── widgets/
│   │   └── todo/
│   │       ├── index.html     # 小组件入口
│   │       └── widget.js      # 组件逻辑
│   └── components/            # 可复用组件库
```

**目录命名规范**：

- `<session>` 通常为会话标识符（如 `main`、`dev`、`test`）
- 支持多会话隔离，每个会话拥有独立的 Canvas 根目录
- 应用自动创建默认 `main` 会话目录

### URL 方案映射机制

Canvas 面板通过**自定义 URL 方案**提供本地文件，映射关系如下：

```
URL 格式：openclaw-canvas://<session>/<path>

映射规则：
├── <session>           → ~/Library/Application Support/OpenClaw/canvas/<session>/
├── <path>              → 相对路径的文件
├── index.html 缺失      → 显示内置脚手架页面
└── 文件不存在           → 404 错误页面
```

**示例映射**：

| 自定义 URL | 映射文件路径 |
|-----------|-------------|
| `openclaw-canvas://main/` | `~/Library/Application Support/OpenClaw/canvas/main/index.html` |
| `openclaw-canvas://main/assets/app.css` | `~/Library/Application Support/OpenClaw/canvas/main/assets/app.css` |
| `openclaw-canvas://main/widgets/todo/` | `~/Library/Application Support/OpenClaw/canvas/main/widgets/todo/index.html` |

---

## 第二部分：面板行为与配置

### 面板布局与交互行为

Canvas 面板作为 macOS 应用的一部分，遵循以下行为规范：

**窗口管理**：

- **锚定位置**：面板默认锚定在菜单栏下方或鼠标光标附近
- **可调整大小**：用户可自由调整面板尺寸，位置与大小被记忆
- **单实例模式**：同一时间只显示一个 Canvas 面板（多会话时自动切换）

**生命周期**：

- **热重载**：本地 Canvas 文件变更时自动重新加载（无需重启应用）
- **状态持久化**：面板关闭后重新打开时恢复上次位置与内容
- **会话切换**：切换会话时自动切换对应的 Canvas 目录

**启用控制**：

Canvas 功能可在应用设置中禁用：

```
设置 → 通用 → "允许 Canvas"
```

禁用后：
- Canvas 菜单项隐藏
- `canvas.*` 节点命令返回 `CANVAS_DISABLED` 错误
- 正在显示的 Canvas 面板关闭

### 开发工作流

**快速开始：内置脚手架**

如果 Canvas 根目录不存在 `index.html`，应用显示**内置脚手架页面**：

```html
<!-- 内置脚手架页面内容 -->
<!DOCTYPE html>
<html>
<head>
    <title>OpenClaw Canvas</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, sans-serif;
            padding: 20px;
            color: #333;
        }
        .hint {
            color: #666;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <h1>Canvas 面板</h1>
    <p class="hint">在此目录添加 index.html 即可显示自定义内容</p>
    <p>路径：~/Library/Application Support/OpenClaw/canvas/main/</p>
</body>
</html>
```

**创建自定义 Canvas**：

```bash
# 1. 创建 Canvas 目录
mkdir -p ~/Library/Application\ Support/OpenClaw/canvas/main/widgets/demo

# 2. 创建入口页面
cat > ~/Library/Application\ Support/OpenClaw/canvas/main/index.html <<'EOF'
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的 Canvas 应用</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background: #f5f5f5;
        }
        .card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="card">
        <h1>Hello from Canvas!</h1>
        <p>这是一个自定义 Canvas 页面。</p>
    </div>
    <script>
        // 监听助手消息
        window.addEventListener('message', (event) => {
            console.log('收到消息:', event.data);
        });
    </script>
</body>
</html>
EOF

# 3. 打开 Canvas 面板验证
openclaw nodes canvas present --node main
```

**热重载验证**：

修改 `index.html` 后保存，Canvas 面板自动刷新显示新内容（无需手动触发）。

---

## 第三部分：助手 API 接口

### 网关 WebSocket 通信架构

Canvas 面板通过 **网关 WebSocket** 与助手进行双向通信。这种设计使得助手可以：

- 推送界面更新到 Canvas
- 接收 Canvas 中的用户交互事件
- 执行任意 JavaScript 代码
- 捕获 Canvas 快照用于日志或调试

**通信架构图**：

```
┌─────────────────────────────────────────────────────────────────┐
│                        OpenClaw 网关                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    WebSocket 控制平面                      │  │
│  │              ws://gateway-host:18789                       │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│              ┌───────────────┼───────────────┐                  │
│              ▼               ▼               ▼                  │
│        ┌─────────┐    ┌─────────┐    ┌─────────┐               │
│        │  macOS  │    │  iOS    │    │ Android │               │
│        │  应用   │    │  节点   │    │  节点   │               │
│        └────┬────┘    └────┬────┘    └────┬────┘               │
│             │               │               │                    │
│             └───────────────┼───────────────┘                    │
│                             ▼                                    │
│                    ┌─────────────────┐                            │
│                    │   Canvas 面板   │                            │
│                    │   (WKWebView)   │                            │
│                    └─────────────────┘                            │
└─────────────────────────────────────────────────────────────────┘
```

### Canvas CLI 命令集

OpenClaw 提供以下 CLI 命令控制 Canvas：

| 命令 | 功能 | 示例 |
|------|------|------|
| `nodes canvas present` | 显示 Canvas 面板 | `openclaw nodes canvas present --node main` |
| `nodes canvas navigate` | 导航到指定路径或 URL | `openclaw nodes canvas navigate --node main --url "/"` |
| `nodes canvas eval` | 执行 JavaScript | `openclaw nodes canvas eval --node main --js "document.title"` |
| `nodes canvas snapshot` | 捕获 Canvas 快照 | `openclaw nodes canvas snapshot --node main` |
| `nodes canvas a2ui push` | 推送 A2UI 界面 | `openclaw nodes canvas a2ui push --node main --text "Hello"` |

**命令详细用法**：

```bash
# 显示 Canvas 面板（指定节点）
openclaw nodes canvas present --node main

# 导航到本地路径（/ 指向默认 index.html）
openclaw nodes canvas navigate --node main --url "/"

# 导航到 HTTP(S) URL
openclaw nodes canvas navigate --node main --url "https://example.com"

# 执行 JavaScript 并获取返回值
openclaw nodes canvas eval --node main --js "document.title"

# 捕获可视化快照（返回 base64 编码图像）
openclaw nodes canvas snapshot --node main
```

**路径格式说明**：

```
navigate 命令接受的 URL 格式：

├── 本地 canvas 路径
│   ├── "/"           → 渲染 index.html 或脚手架
│   ├── "/widgets/"   → 渲染 widgets/index.html
│   └── "/assets/*"    → 渲染静态资源
│
├── 外部 HTTP URL
│   ├── "http://..."  → 直接导航（需用户确认）
│   └── "https://..." → 直接导航（需用户确认）
│
└── 本地文件 URL
    └── "file://..."  → 直接加载本地文件
```

---

## 第四部分：A2UI 界面框架

### A2UI 框架概述

**A2UI** 是一个轻量级的声明式 UI 框架，专为 AI 助手驱动的界面设计。A2UI 允许助手通过 JSON 消息描述界面结构，框架负责渲染与交互处理。

**核心设计理念**：

- **声明式**：助手描述"想要什么"，框架处理"如何渲染"
- **实时更新**：支持增量界面更新，无需重新渲染整个页面
- **事件驱动**：用户交互自动回传到助手处理

### A2UI 主机部署

A2UI 由网关 canvas 主机服务托管，macOS 应用在首次打开 Canvas 时自动导航到 A2UI 主机页面。

**默认 A2UI 主机 URL**：`http://<gateway-host>:18793/__openclaw__/a2ui/`

**架构组件**：

```
┌─────────────────────────────────────────────────────────────────┐
│                     OpenClaw 网关                                │
│  ┌─────────────────────┐  ┌─────────────────────┐              │
│  │   Gateway WS 服务    │  │   Canvas 主机服务    │              │
│  │   ws://:18789       │  │   http://:18793     │              │
│  └──────────┬──────────┘  └──────────▲──────────┘              │
│             │                        │                           │
│             │ WebSocket              │ HTTP                      │
│             │ 消息                   │ A2UI 页面                 │
│             ▼                        │                           │
│  ┌─────────────────────────┐        │                           │
│  │      macOS 应用         │─────────┘                           │
│  │    (Canvas 面板)        │    A2UI 渲染                       │
│  └─────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

### A2UI 协议版本与兼容性

当前实现的 **A2UI v0.8** 支持以下服务器→客户端消息类型：

| 消息类型 | 功能 | 支持状态 |
|---------|------|---------|
| `beginRendering` | 开始渲染界面 | ✅ 支持 |
| `surfaceUpdate` | 更新界面组件 | ✅ 支持 |
| `dataModelUpdate` | 更新数据模型 | ✅ 支持 |
| `deleteSurface` | 删除界面 | ✅ 支持 |
| `createSurface` | 创建新界面 | ❌ 不支持（v0.9） |

### A2UI 消息格式详解

**surfaceUpdate 消息结构**：

```json
{
  "surfaceUpdate": {
    "surfaceId": "main",
    "components": [
      {
        "id": "root",
        "component": {
          "Column": {
            "children": {
              "explicitList": ["title", "content"]
            }
          }
        }
      },
      {
        "id": "title",
        "component": {
          "Text": {
            "text": {
              "literalString": "Canvas (A2UI v0.8)"
            },
            "usageHint": "h1"
          }
        }
      },
      {
        "id": "content",
        "component": {
          "Text": {
            "text": {
              "literalString": "A2UI 推送测试成功"
            },
            "usageHint": "body"
          }
        }
      }
    ]
  }
}
```

**完整渲染流程**：

```jsonl
{"surfaceUpdate":{"surfaceId":"main","components":[...]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
```

### A2UI 快速入门

**方式一：通过文件推送（测试 A2UI 功能）**：

```bash
# 1. 创建 A2UI 消息文件
cat > /tmp/a2ui-test.jsonl <<'EOF'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"A2UI 推送测试——如果能看到这条消息，说明 A2UI 集成正常"},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOF

# 2. 推送到 Canvas
openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-test.jsonl --node main
```

**方式二：使用 --text 参数（极简测试）**：

```bash
# 推送简单文本消息
openclaw nodes canvas a2ui push --node main --text "Hello from A2UI"
```

**预期效果**：Canvas 面板显示标题 "Hello from A2UI"（使用默认样式渲染）。

---

## 第五部分：深度集成与扩展

### 从 Canvas 触发助手运行

Canvas 页面可以通过**深度链接（Deep Link）**机制触发新的助手运行，实现用户交互到 AI 响应的闭环。

**深度链接格式**：`openclaw://agent?...`

**参数说明**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `message` | 字符串 | 发送给助手的消息内容 |
| `channel` | 字符串 | 指定消息发送渠道（可选） |
| `thinking` | 字符串 | 思考级别（off/minimal/low/medium/high/xhigh） |

**JavaScript 调用示例**：

```javascript
// 触发助手运行
window.location.href = "openclaw://agent?message=Review%20this%20design";

// 指定思考级别
window.location.href = "openclaw://agent?message=分析这段代码&thinking=high";
```

**确认机制**：

除非提供有效的授权密钥，否则应用会弹出确认对话框，防止恶意网页未经授权触发助手。

### JavaScript 与助手通信

Canvas 页面可以与 OpenClaw 网关进行双向通信：

**发送消息到助手**：

```javascript
// 通过深度链接触发
window.location.href = "openclaw://agent?message=" + encodeURIComponent("处理这个请求");
```

**接收助手消息**：

```javascript
// 监听助手推送的消息
window.addEventListener('message', function(event) {
    // event.data 包含助手推送的数据
    console.log('收到助手消息:', event.data);
    
    // 处理 A2UI 消息
    if (event.data.type === 'surfaceUpdate') {
        // 更新界面
    }
});
```

### 自定义 Canvas 应用开发

**完整示例：TODO 小组件**：

```html
<!-- ~/Library/Application Support/OpenClaw/canvas/main/widgets/todo/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>TODO 小组件</title>
    <style>
        * { box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, sans-serif;
            padding: 12px;
            background: #1e1e1e;
            color: #eee;
            min-width: 280px;
        }
        .input-row { display: flex; gap: 8px; margin-bottom: 12px; }
        input {
            flex: 1;
            padding: 8px 12px;
            border-radius: 6px;
            border: 1px solid #333;
            background: #2d2d2d;
            color: #eee;
        }
        button {
            padding: 8px 16px;
            border-radius: 6px;
            border: none;
            background: #007aff;
            color: white;
            cursor: pointer;
        }
        button:hover { background: #0056b3; }
        ul { list-style: none; padding: 0; margin: 0; }
        li {
            display: flex;
            align-items: center;
            padding: 8px 0;
            border-bottom: 1px solid #333;
        }
        li.completed span { text-decoration: line-through; opacity: 0.5; }
        .delete {
            margin-left: auto;
            color: #ff3b30;
            cursor: pointer;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <h3>TODO 列表</h3>
    <div class="input-row">
        <input type="text" id="todoInput" placeholder="添加新任务...">
        <button onclick="addTodo()">添加</button>
    </div>
    <ul id="todoList"></ul>

    <script>
        // 本地状态
        let todos = JSON.parse(localStorage.getItem('todos') || '[]');
        const todoList = document.getElementById('todoList');
        const todoInput = document.getElementById('todoInput');

        // 渲染列表
        function render() {
            todoList.innerHTML = todos.map((t, i) => `
                <li class="${t.done ? 'completed' : ''}" onclick="toggle(${i})">
                    <span>${escapeHtml(t.text)}</span>
                    <span class="delete" onclick="deleteTodo(${i}); event.stopPropagation();">删除</span>
                </li>
            `).join('');
        }

        // 添加任务
        function addTodo() {
            const text = todoInput.value.trim();
            if (text) {
                todos.push({ text, done: false });
                saveAndRender();
                todoInput.value = '';
            }
        }

        // 切换完成状态
        function toggle(index) {
            todos[index].done = !todos[index].done;
            saveAndRender();
        }

        // 删除任务
        function deleteTodo(index) {
            todos.splice(index, 1);
            saveAndRender();
        }

        // 保存并渲染
        function saveAndRender() {
            localStorage.setItem('todos', JSON.stringify(todos));
            render();
        }

        // HTML 转义
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }

        // 初始化
        render();

        // 回车添加
        todoInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') addTodo();
        });
    </script>
</body>
</html>
```

---

## 第六部分：安全机制

### 安全设计原则

Canvas 系统采用多层安全机制保护用户数据与应用稳定性：

**安全层级一：路径遍历防护**

自定义 URL 方案实现了完整的路径遍历防护：

```
✅ 允许的路径模式：
├── openclaw-canvas://main/index.html
├── openclaw-canvas://main/assets/app.css
└── openclaw-canvas://main/widgets/demo/index.html

❌ 拒绝的路径模式：
├── openclaw-canvas://main/../credentials/   (目录遍历攻击)
├── openclaw-canvas://main/../../etc/passwd  (敏感文件访问)
└── openclaw-canvas://../../../tmp/exploit   (任意位置写入)
```

实现机制：所有文件路径在应用层规范化，验证路径前缀是否在 Canvas 根目录下。

**安全层级二：本地文件隔离**

Canvas 使用自定义 URL 方案而非 `file://` 协议，提供以下优势：

- 精确控制可访问的文件范围
- 阻止跨目录的 JavaScript 访问
- 与 macOS 应用沙盒机制兼容

**安全层级三：外部 URL 确认**

当 Canvas 导航到外部 `http(s)` URL 时：

- 用户需要明确确认导航
- 可在设置中禁用外部 URL 导航
- 钓鱼网站防护依赖用户判断

### 权限与敏感数据处理

**敏感数据隔离原则**：

1. **凭证不暴露**：Canvas 无法访问网关凭证存储
2. **会话隔离**：每个 Canvas 会话独立，无法访问其他会话数据
3. **网络限制**：Canvas 中的 JavaScript 遵循 CORS 策略，无法任意请求内部 API

**调试与日志**：

Canvas 快照功能用于调试，但：

- 快照仅保存在本地
- 不自动上传到云端
- 用户可随时清除快照缓存

---

## 第七部分：故障排查指南

### 问题分类与诊断流程

```
Canvas 问题判定树：

├── Canvas 面板无法显示
│   ├── 设置中 Canvas 被禁用 → 启用 "允许 Canvas"
│   ├── Canvas 文件不存在 → 检查 ~/Library/Application Support/OpenClaw/canvas/
│   └── WKWebView 初始化失败 → 查看应用日志
│
├── 显示空白页面
│   ├── index.html 语法错误 → 检查浏览器开发者工具
│   ├── JavaScript 执行错误 → 查看控制台错误
│   └── 资源文件 404 → 验证路径映射
│
├── A2UI 推送无反应
│   ├── A2UI 服务未运行 → 检查网关状态
│   ├── 协议版本不兼容 → 确认使用 v0.8 消息格式
│   └── surfaceId 不匹配 → 验证 surfaceId 一致性
│
└── 文件热重载不生效
    ├── 文件路径错误 → 验证 openclaw-canvas:// URL 映射
    ├── 权限问题 → 检查文件权限
    └── 缓存问题 → 重启 Canvas 面板
```

### 常见故障与解决方案

**故障一：Canvas 面板显示"无法加载"**

**症状**：打开 Canvas 面板后显示错误页面

**诊断**：

```bash
# 检查 Canvas 根目录是否存在
ls -la ~/Library/Application\ Support/OpenClaw/canvas/

# 检查文件权限
ls -la ~/Library/Application\ Support/OpenClaw/canvas/main/
```

**解决方案**：

1. 创建默认 `main` 目录：`mkdir -p ~/Library/Application\ Support/OpenClaw/canvas/main`
2. 添加 `index.html` 或等待系统显示内置脚手架
3. 重启应用或重新打开 Canvas 面板

**故障二：A2UI 推送后界面无变化**

**症状**：运行 `openclaw nodes canvas a2ui push` 成功但 Canvas 内容不变

**诊断**：

```bash
# 验证网关 WebSocket 连接健康
openclaw status --deep

# 检查 A2UI 服务状态
openclaw health --json | grep a2ui
```

**解决方案**：

1. 确认 Canvas 面板已打开（A2UI 仅渲染到已打开的面板）
2. 验证 `surfaceId` 与 `beginRendering` 中的 `root` 组件 ID 匹配
3. 查看网关日志：`tail -f /tmp/openclaw/openclaw-gateway.log | grep a2ui`

**故障三：自定义 URL 方案不工作**

**症状**：在浏览器或外部应用中无法打开 `openclaw-canvas://` URL

**诊断**：

```bash
# 检查 macOS 应用是否注册了 URL scheme
ls ~/Library/Application\ Support/OpenClaw/canvas/

# 验证 URL 格式
openclaw-canvas://main/index.html
```

**说明**：`openclaw-canvas://` 是 macOS 应用内部使用的 URL 方案，外部应用无法调用。这是设计决策，而非故障。

**故障四：JavaScript 跨域请求被阻止**

**症状**：Canvas 中的 fetch/XMLHttpRequest 请求失败

**原因**：WKWebView 默认启用 CORS 策略

**解决方案**：

```javascript
// 方案一：使用 JSONP（仅 GET 请求）
function jsonp(url, callback) {
    const script = document.createElement('script');
    script.src = url + (url.includes('?') ? '&' : '?') + 'callback=' + callback;
    document.body.appendChild(script);
}

// 方案二：请求服务器设置 CORS 头
// Access-Control-Allow-Origin: *
```

---

## 适用场景与最佳实践

### 推荐使用 Canvas 的场景

**场景一：AI 驱动的可视化响应**

当助手需要展示代码差异、数据分析图表、文档预览等内容时，Canvas 提供无限可能的渲染能力：

```
示例：代码审查助手
├── 助手分析代码变更
├── 通过 A2UI 推送代码对比视图
├── 用户在 Canvas 中标注评论
└── 评论内容回传到助手继续讨论
```

**场景二：交互式工具封装**

将常用工具封装为 Canvas 小组件：

```
示例：API 测试工具
├── Canvas 页面包含请求配置表单
├── JavaScript 发送请求到目标 API
├── 实时显示响应结果
└── 一键将响应数据发送给助手分析
```

**场景三：富媒体消息展示**

展示助手生成的可视化内容：

- Markdown 渲染
- 数据可视化图表
- 交互式流程图
- 富文本编辑器

### 不适用 Canvas 的场景

**场景一：简单文本响应**

如果响应内容以文本为主，直接使用消息通道更高效，无需打开 Canvas 面板。

**场景二：需要原生功能**

如果需要访问摄像头、文件选择器等原生功能，应使用 iOS/Android 节点而非 Canvas。

**场景三：实时音视频**

Canvas 无法承载实时音视频流，此类需求应使用专用节点。

---

## 章节总结

本章节全面介绍了 OpenClaw Canvas 画布面板系统的架构设计与开发实践。通过学习，你应该已经掌握：

1. **架构原理**：WKWebView + 自定义 URL 方案的技术选型与安全考量
2. **文件管理**：Canvas 存储结构与 URL 映射机制
3. **CLI 控制**：通过网关命令控制 Canvas 的完整操作集
4. **A2UI 集成**：声明式 UI 框架的使用与扩展方法
5. **安全机制**：多层防护确保用户数据与应用安全
6. **故障排查**：系统化的诊断流程与常见问题解决方案

后续建议结合 [语音唤醒](/zh-cn/platforms/mac/voicewake) 与 [远程访问](/zh-cn/platforms/mac/remote) 章节，深入了解 macOS 应用的其他核心功能模块。
