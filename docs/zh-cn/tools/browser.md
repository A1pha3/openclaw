---
summary: "集成浏览器控制服务 + 操作命令"
read_when:
  - 添加代理控制的浏览器自动化
  - 调试为什么 openclaw 干扰你自己的 Chrome
  - 在 macOS 应用中实现浏览器设置 + 生命周期
title: "浏览器（OpenClaw 管理的）"
---

# 浏览器（openclaw 管理的）

OpenClaw 可以运行一个**专用的 Chrome/Brave/Edge/Chromium 配置文件**，由代理控制。它与你的个人浏览器隔离，并通过网关内的小型本地控制服务管理（仅限 loopback）。

初学者视图：

- 将其视为**单独的、仅代理的浏览器**。
- `openclaw` 配置文件**不会**触及你的个人浏览器配置文件。
- 代理可以在安全车道中**打开标签页、阅读页面、点击和输入**。
- 默认的 `chrome` 配置文件通过扩展中继使用**系统默认 Chromium 浏览器**；切换到 `openclaw` 以使用隔离的托管浏览器。

## 你得到什么

- 一个名为 **openclaw** 的独立浏览器配置文件（默认橙色强调）。
- 确定性标签控制（列表/打开/焦点/关闭）。
- 代理操作（点击/输入/拖动/选择）、快照、截图、PDF。
- 可选的多配置文件支持（`openclaw`、`work`、`remote`、...）。

这个浏览器**不是**你的日常驱动程序。它是代理自动化和验证的安全、隔离表面。

## 快速开始

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

如果你得到"浏览器禁用"，在配置中启用它（见下文）并重启网关。

## 配置文件：`openclaw` vs `chrome`

- `openclaw`：托管、隔离浏览器（不需要扩展）。
- `chrome`：扩展中继到你的**系统浏览器**（需要 OpenClaw 扩展附加到标签页）。

如果你想要默认的托管模式，设置 `browser.defaultProfile: "openclaw"`。

## 配置

浏览器设置位于 `~/.openclaw/openclaw.json`。

```json5
{
  browser: {
    enabled: true, // 默认：true
    // cdpUrl: "http://127.0.0.1:18792", // 旧版单配置文件覆盖
    remoteCdpTimeoutMs: 1500, // 远程 CDP HTTP 超时（毫秒）
    remoteCdpHandshakeTimeoutMs: 3000, // 远程 CDP WebSocket 握手超时（毫秒）
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

注意：

- 浏览器控制服务绑定到从 `gateway.port` 派生的端口上的 loopback（默认：`18791`，即 gateway + 2）。中继使用下一个端口（`18792`）。
- 如果你覆盖网关端口（`gateway.port` 或 `OPENCLAW_GATEWAY_PORT`），派生的浏览器端口会偏移以保持在同一"系列"中。
- `cdpUrl` 未设置时默认为中继端口。
- `remoteCdpTimeoutMs` 适用于远程（非 loopback）CDP 可达性检查。
- `remoteCdpHandshakeTimeoutMs` 适用于远程 CDP WebSocket 可达性检查。
- `attachOnly: true` 表示"永不启动本地浏览器；仅当它已经在运行时附加"。
- `color` + 每个配置文件的 `color` 为浏览器 UI 着色，以便你可以看到哪个配置文件是活动的。
- 默认配置文件是 `chrome`（扩展中继）。对于托管浏览器，使用 `defaultProfile: "openclaw"`。
- 自动检测顺序：如果是基于 Chromium 的浏览器，则使用系统默认浏览器；否则 Chrome → Brave → Edge → Chromium → Chrome Canary。
- 本地 `openclaw` 配置文件自动分配 `cdpPort`/`cdpUrl` —— 仅对远程 CDP 设置这些。

## 使用 Brave（或另一个基于 Chromium 的浏览器）

如果你的**系统默认**浏览器是基于 Chromium 的（Chrome/Brave/Edge/etc），OpenClaw 自动使用它。设置 `browser.executablePath` 以覆盖自动检测：

CLI 示例：

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## 本地 vs 远程控制

- **本地控制（默认）：** 网关启动 loopback 控制服务，可以启动本地浏览器。
- **远程控制（节点主机）：** 在有浏览器的机器上运行节点主机；网关将浏览器操作代理到它。
- **远程 CDP：** 设置 `browser.profiles.<name>.cdpUrl`（或 `browser.cdpUrl`）以附加到远程基于 Chromium 的浏览器。在这种情况下，OpenClaw 不会启动本地浏览器。

远程 CDP URL 可以包括认证：

- 查询令牌（例如，`https://provider.example?token=<token>`）
- HTTP Basic 认证（例如，`https://user:pass@provider.example`）

OpenClaw 在调用 `/json/*` 端点和连接 CDP WebSocket 时保留认证。优先使用环境变量或机密管理器获取令牌，而不是将它们提交到配置文件。

## 节点浏览器代理（零配置默认）

如果你在有浏览器的机器上运行**节点主机**，OpenClaw 可以自动将浏览器工具调用路由到该节点，而无需任何额外的浏览器配置。这是远程网关的默认路径。

注意：

- 节点主机通过**代理命令**暴露其本地浏览器控制服务器。
- 配置文件来自节点自己的 `browser.profiles` 配置（与本地相同）。
- 如果你不想要它，请禁用：
  - 在节点上：`nodeHost.browserProxy.enabled=false`
  - 在网关上：`gateway.nodes.browser.mode="off"`

## Browserless（托管远程 CDP）

[Browserless](https://browserless.io) 是一个托管的 Chromium 服务，通过 HTTPS 暴露 CDP 端点。你可以将 OpenClaw 浏览器配置文件指向 Browserless 区域端点并使用你的 API 密钥进行认证。

示例：

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
        color: "#00AA00",
      },
    },
  },
}
```

注意：

- 用你的真实 Browserless 令牌替换 `<BROWSERLESS_API_KEY>`。
- 选择与你 Browserless 账户匹配的区域端点（见他们的文档）。

## 安全

关键想法：

- 浏览器控制仅限 loopback；访问通过网关的认证或节点配对流动。
- 保持网关和任何节点主机在专用网络（Tailscale）上；避免公开暴露。
- 将远程 CDP URL/令牌视为机密；优先使用环境变量或机密管理器。

远程 CDP 提示：

- 优先使用 HTTPS 端点和短命令牌（如果可能）。
- 避免直接将长命令牌嵌入配置文件。

## 配置文件（多浏览器）

OpenClaw 支持多个命名配置文件（路由配置）。配置文件可以是：

- **openclaw 管理的**：专用的基于 Chromium 的浏览器实例，有自己的用户数据目录 + CDP 端口
- **远程**：显式 CDP URL（在其他地方运行的基于 Chromium 的浏览器）
- **扩展中继**：通过本地中继 + Chrome 扩展使用你现有的 Chrome 标签页

默认值：

- 如果缺失，自动创建 `openclaw` 配置文件。
- `chrome` 配置文件是内置的 Chrome 扩展中继（默认指向 `http://127.0.0.1:18792`）。
- 本地 CDP 端口默认从 **18800–18899** 分配。
- 删除配置文件将其本地数据目录移动到废纸篓。

所有控制端点接受 `?profile=<name>`；CLI 使用 `--browser-profile`。

## Chrome 扩展中继（使用你现有的 Chrome）

OpenClaw 还可以通过本地 CDP 中继 + Chrome 扩展驱动**你现有的 Chrome 标签页**（没有单独的"openclaw" Chrome 实例）。

完整指南：[Chrome 扩展](/tools/chrome-extension)

流程：

- 网关在本地运行（同一机器）或节点主机在浏览器机器上运行。
- 本地**中继服务器**在 loopback `cdpUrl`（默认：`http://127.0.0.1:18792`）上监听。
- 你点击 **OpenClaw Browser Relay** 扩展图标在一个标签页上以附加（它不会自动附加）。
- 代理通过选择正确的配置文件，使用正常的 `browser` 工具控制该标签页。

如果网关在其他地方运行，在浏览器机器上运行节点主机，以便网关可以代理浏览器操作。

### 沙箱化会话

如果代理会话是沙箱化的，`browser` 工具可能默认为 `target="sandbox"`（沙箱浏览器）。Chrome 扩展中继接管需要主机浏览器控制，因此要么：

- 运行会话不沙箱化，或
- 设置 `agents.defaults.sandbox.browser.allowHostControl: true` 并在调用工具时使用 `target="host"`

### 设置

1. 加载扩展（开发/未打包）：

```bash
openclaw browser extension install
```

- Chrome → `chrome://extensions` → 启用"开发者模式"
- "加载已解压的" → 选择 `openclaw browser extension path` 打印的目录
- 固定扩展，然后在你想要控制的标签页上点击它（徽章显示 `ON`）。

2. 使用它：

- CLI：`openclaw browser --browser-profile chrome tabs`
- 代理工具：`browser` 带有 `profile="chrome"`

可选：如果你想要不同的名称或中继端口，创建你自己的配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

注意：

- 此模式依赖于大多数操作的 Playwright-on-CDP（截图/快照/操作）。
- 再次点击扩展图标以分离。

## 隔离保证

- **专用用户数据目录**：永不使用你的个人浏览器配置文件。
- **专用端口**：避免 `9222` 以防止与开发工作流碰撞。
- **确定性标签控制**：按 `targetId` 而不是"最后一个标签"定位标签。

## 浏览器选择

在本地启动时，OpenClaw 按顺序选择第一个可用的：

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

你可以用 `browser.executablePath` 覆盖。

平台：

- macOS：检查 `/Applications` 和 `~/Applications`。
- Linux：查找 `google-chrome`、`brave`、`microsoft-edge`、`chromium` 等。
- Windows：检查常见安装位置。

## 控制 API（可选）

仅用于本地集成，网关暴露一个小 loopback HTTP API：

- 状态/启动/停止：`GET /`、`POST /start`、`POST /stop`
- 标签页：`GET /tabs`、`POST /tabs/open`、`POST /tabs/focus`、`DELETE /tabs/:targetId`
- 快照/截图：`GET /snapshot`、`POST /screenshot`
- 操作：`POST /navigate`、`POST /act`
- 钩子：`POST /hooks/file-chooser`、`POST /hooks/dialog`
- 下载：`POST /download`、`POST /wait/download`
- 调试：`GET /console`、`POST /pdf`
- 调试：`GET /errors`、`GET /requests`、`POST /trace/start`、`POST /trace/stop`、`POST /highlight`
- 网络：`POST /response/body`
- 状态：`GET /cookies`、`POST /cookies/set`、`POST /cookies/clear`
- 状态：`GET /storage/:kind`、`POST /storage/:kind/set`、`POST /storage/:kind/clear`
- 设置：`POST /set/offline`、`POST /set/headers`、`POST /set/credentials`、`POST /set/geolocation`、`POST /set/media`、`POST /set/timezone`、`POST /set/locale`、`POST /set/device`

所有端点接受 `?profile=<name>`。

### Playwright 要求

某些功能（导航/操作/AI 快照/角色快照、元素截图、PDF）需要 Playwright。如果 Playwright 未安装，这些端点返回清晰的 501 错误。ARIA 快照和基本截图仍然适用于 openclaw 管理的 Chrome。对于 Chrome 扩展中继驱动程序，ARIA 快照和截图需要 Playwright。

如果你看到 `Playwright is not available in this gateway build`，安装完整的 Playwright 包（不是 `playwright-core`）并重启网关，或使用浏览器支持重新安装 OpenClaw。

## 它如何工作（内部）

高级流程：

- 一个小的**控制服务器**接受 HTTP 请求。
- 它通过 **CDP** 连接到基于 Chromium 的浏览器（Chrome/Brave/Edge/Chromium）。
- 对于高级操作（点击/输入/快照/PDF），它在 CDP 之上使用 **Playwright**。
- 当 Playwright 缺失时，只有非 Playwright 操作可用。

这个设计使代理保持在稳定、确定的接口上，同时让你交换本地/远程浏览器和配置文件。

## CLI 快速参考

所有命令接受 `--browser-profile <name>` 以定位特定配置文件。所有命令也接受 `--json` 以获得机器可读的输出（稳定负载）。

基础：

- `openclaw browser status`
- `openclaw browser start`
- `openclaw browser stop`
- `openclaw browser tabs`
- `openclaw browser tab`
- `openclaw browser tab new`
- `openclaw browser tab select 2`
- `openclaw browser tab close 2`
- `openclaw browser open https://example.com`
- `openclaw browser focus abcd1234`
- `openclaw browser close abcd1234`

检查：

- `openclaw browser screenshot`
- `openclaw browser screenshot --full-page`
- `openclaw browser screenshot --ref 12`
- `openclaw browser screenshot --ref e12`
- `openclaw browser snapshot`
- `openclaw browser snapshot --format aria --limit 200`
- `openclaw browser snapshot --interactive --compact --depth 6`
- `openclaw browser snapshot --efficient`
- `openclaw browser snapshot --labels`
- `openclaw browser snapshot --selector "#main" --interactive`
- `openclaw browser snapshot --frame "iframe#main" --interactive`
- `openclaw browser console --level error`
- `openclaw browser errors --clear`
- `openclaw browser requests --filter api --clear`
- `openclaw browser pdf`
- `openclaw browser responsebody "**/api" --max-chars 5000`

操作：

- `openclaw browser navigate https://example.com`
- `openclaw browser resize 1280 720`
- `openclaw browser click 12 --double`
- `openclaw browser click e12 --double`
- `openclaw browser type 23 "hello" --submit`
- `openclaw browser press Enter`
- `openclaw browser hover 44`
- `openclaw browser scrollintoview e12`
- `openclaw browser drag 10 11`
- `openclaw browser select 9 OptionA OptionB`
- `openclaw browser download e12 /tmp/report.pdf`
- `openclaw browser waitfordownload /tmp/report.pdf`
- `openclaw browser upload /tmp/file.pdf`
- `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
- `openclaw browser dialog --accept`
- `openclaw browser wait --text "Done"`
- `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
- `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
- `openclaw browser highlight e12`
- `openclaw browser trace start`
- `openclaw browser trace stop`

状态：

- `openclaw browser cookies`
- `openclaw browser cookies set session abc123 --url "https://example.com"`
- `openclaw browser cookies clear`
- `openclaw browser storage local get`
- `openclaw browser storage local set theme dark`
- `openclaw browser storage session clear`
- `openclaw browser set offline on`
- `openclaw browser set headers --json '{"X-Debug":"1"}'`
- `openclaw browser set credentials user pass`
- `openclaw browser set credentials --clear`
- `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
- `openclaw browser set geo --clear`
- `openclaw browser set media dark`
- `openclaw browser set timezone America/New_York`
- `openclaw browser set locale en-US`
- `openclaw browser set device "iPhone 14"`

注意：

- `upload` 和 `dialog` 是**预备**调用；在触发选择器/对话框的点击/按压之前运行它们。
- `upload` 也可以通过 `--input-ref` 或 `--element` 直接设置文件输入。
- `snapshot`：
  - `--format ai`（Playwright 安装时的默认设置）：返回带有数字引用的 AI 快照（`aria-ref="<n>"`）。
  - `--format aria`：返回可访问性树（无引用；仅检查）。
  - `--efficient`（或 `--mode efficient`）：紧凑的角色快照预设（交互 + 紧凑 + 深度 + 更低的 maxChars）。
  - 配置默认（工具/CLI 仅）：当调用者不传递模式时，设置 `browser.snapshotDefaults.mode: "efficient"` 以使用高效快照（参见 [Gateway configuration](/gateway/configuration#browser-openclaw-managed-browser)）。
  - 角色快照选项（`--interactive`、`--compact`、`--depth`、`--selector`）强制基于角色的快照，带有像 `ref=e12` 的引用。
  - `--frame "<iframe selector>` 将角色快照作用域限定到 iframe（与像 `e12` 的角色引用配对）。
  - `--interactive` 输出一个扁平的、易于选择的交互元素列表（最适合驱动操作）。
  - `--labels` 添加带有覆盖 `e12` 标签的视口截图（打印 `MEDIA:<path>`）。
- `click`/`type`/etc 需要来自 `snapshot` 的 `ref`（数字 `12` 或角色引用 `e12`）。
  - CSS 选择器有意不支持用于操作。

## 快照和引用

OpenClaw 支持两种"快照"样式：

- **AI 快照（数字引用）**：`openclaw browser snapshot`（默认；`--format ai`）
  - 输出：包含数字引用的文本快照。
  - 操作：`openclaw browser click 12`、`openclaw browser type 23 "hello"`。
  - 内部，引用通过 Playwright 的 `aria-ref` 解析。

- **角色快照（像 `e12` 的角色引用）**：`openclaw browser snapshot --interactive`（或 `--compact`、`--depth`、`--selector`、`--frame`）
  - 输出：基于角色的列表/树，带有 `[ref=e12]`（和可选的 `[nth=1]`）。
  - 操作：`openclaw browser click e12`、`openclaw browser highlight e12`。
  - 内部，引用通过 `getByRole(...)` 解析（加上重复的 `nth()`）。
  - 添加 `--labels` 以包括带有覆盖 `e12` 标签的视口截图。

引用行为：

- 引用**在导航之间不稳定**；如果某物失败，重新运行 `snapshot` 并使用新的引用。
- 如果角色快照是用 `--frame` 拍摄的，角色引用作用域限定到该 iframe 直到下一个角色快照。

## 等待增强

你可以等待的不仅仅是时间/文本：

- 等待 URL（Playwright 支持 glob）：
  - `openclaw browser wait --url "**/dash"`
- 等待加载状态：
  - `openclaw browser wait --load networkidle`
- 等待 JS 谓词：
  - `openclaw browser wait --fn "window.ready===true"`
- 等待选择器变得可见：
  - `openclaw browser wait "#main"`

这些可以组合：

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## 调试工作流

当操作失败时（例如"不可见"、"严格模式违规"、"覆盖"）：

1. `openclaw browser snapshot --interactive`
2. 使用 `click <ref>` / `type <ref>`（在交互模式下优先使用角色引用）
3. 如果仍然失败：`openclaw browser highlight <ref>` 以查看 Playwright 正在定位什么
4. 如果页面行为奇怪：
   - `openclaw browser errors --clear`
   - `openclaw browser requests --filter api --clear`
5. 深度调试：记录跟踪：
   - `openclaw browser trace start`
   - 重现问题
   - `openclaw browser trace stop`（打印 `TRACE:<path>`）

## JSON 输出

`--json` 用于脚本和结构化工具。

示例：

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

JSON 中的角色快照包括 `refs` 加一个小的 `stats` 块（行/字符/引用/交互），以便工具可以推理负载大小和密度。

## 状态和环境旋钮

这些对于"让站点表现得像 X"工作流程很有用：

- Cookie：`cookies`、`cookies set`、`cookies clear`
- 存储：`storage local|session get|set|clear`
- 离线：`set offline on|off`
- 标头：`set headers --json '{"X-Debug":"1"}'`（或 `--clear`）
- HTTP basic 认证：`set credentials user pass`（或 `--clear`）
- 地理位置：`set geo <lat> <lon> --origin "https://example.com"`（或 `--clear`）
- 媒体：`set media dark|light|no-preference|none`
- 时区/语言环境：`set timezone ...`、`set locale ...`
- 设备/视口：
  - `set device "iPhone 14"`（Playwright 设备预设）
  - `set viewport 1280 720`

## 安全和隐私

- openclaw 浏览器配置文件可能包含已登录的会话；将其视为敏感的。
- `browser act kind=evaluate` / `openclaw browser evaluate` 和 `wait --fn`
  在页面上下文中执行任意 JavaScript。提示注入可以引导它。如果不需要，用 `browser.evaluateEnabled=false` 禁用它。
- 对于登录和反机器人注释（X/Twitter 等），参见 [Browser login + X/Twitter posting](/tools/browser-login)。
- 保持网关/节点主机私有（仅限 loopback 或 tailnet）。
- 远程 CDP 端点很强大；隧道和保护它们。

## 故障排除

对于 Linux 特定问题（特别是 snap Chromium），参见
[Browser troubleshooting](/tools/browser-linux-troubleshooting)。

## 代理工具 + 控制如何工作

代理获得**一个工具**用于浏览器自动化：

- `browser` — 状态/启动/停止/标签页/打开/焦点/关闭/快照/截图/导航/操作

它如何映射：

- `browser snapshot` 返回稳定的 UI 树（AI 或 ARIA）。
- `browser act` 使用快照 `ref` ID 来点击/输入/拖动/选择。
- `browser screenshot` 捕获像素（全页或元素）。
- `browser` 接受：
  - `profile` 选择命名的浏览器配置文件（openclaw、chrome 或远程 CDP）。
  - `target`（`sandbox` | `host` | `node`）选择浏览器所在的位置。
  - 在沙箱化会话中，`target: "host"` 需要 `agents.defaults.sandbox.browser.allowHostControl=true`。
  - 如果省略 `target`：沙箱化会话默认为 `sandbox`，非沙箱化会话默认为 `host`。
  - 如果连接了支持浏览器的节点，工具可能会自动路由到它，除非你固定 `target="host"` 或 `target="node"`。

这使代理保持确定性并避免脆弱的选择器。
