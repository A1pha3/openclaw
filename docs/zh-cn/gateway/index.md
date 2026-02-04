---
summary: "网关服务、生命周期和操作的运行手册"
read_when:
  - "运行或调试网关进程"
title: "网关运行手册"
---

# 网关服务运行手册

最后更新：2025-12-09

## 什么是网关

- 拥有单个 Baileys/Telegram 连接和控制/事件平面的常驻进程。
- 替换传统的 `gateway` 命令。CLI 入口点：`openclaw gateway`。
- 持续运行直到停止；在致命错误时以非零退出，以便监督进程重新启动它。

## 如何运行（本地）

```bash
openclaw gateway --port 18789
# 要在 stdio 中获取完整的调试/跟踪日志：
openclaw gateway --port 18789 --verbose
# 如果端口繁忙，终止监听器然后启动：
openclaw gateway --force
# 开发循环（TS 更改时自动重新加载）：
pnpm gateway:watch
```

- 配置热重载监视 `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）。
  - 默认模式：`gateway.reload.mode="hybrid"`（安全更改热应用，关键更改时重启）。
  - 需要时通过 **SIGUSR1** 使用进程内重启进行热重载。
  - 使用 `gateway.reload.mode="off"` 禁用。
- 将 WebSocket 控制平面绑定到 `127.0.0.1:<port>`（默认 18789）。
- 同一端口也提供 HTTP 服务（控制 UI、hooks、A2UI）。单端口多路复用。
  - OpenAI Chat Completions (HTTP): [`/v1/chat/completions`](/gateway/openai-http-api)。
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api)。
  - Tools Invoke (HTTP): [`/tools/invoke`](/gateway/tools-invoke-http-api)。
- 默认在 `canvasHost.port`（默认 `18793`）上启动 Canvas 文件服务器，从 `~/.openclaw/workspace/canvas` 提供 `http://<gateway-host>:18793/__openclaw__/canvas/`。使用 `canvasHost.enabled=false` 或 `OPENCLAW_SKIP_CANVAS_HOST=1` 禁用。
- 输出到 stdout；使用 launchd/systemd 保持其运行并轮转日志。
- 传递 `--verbose` 以在排除故障时将调试日志（握手、请求/响应、事件）从日志文件镜像到 stdio。
- `--force` 使用 `lsof` 查找所选端口上的监听器，发送 SIGTERM，记录它终止的内容，然后启动网关（如果 `lsof` 缺失则快速失败）。
- 如果你在监督进程下运行（launchd/systemd/mac 应用子进程模式），停止/重启通常发送 **SIGTERM**；旧版本可能将此显示为 `pnpm` `ELIFECYCLE` 退出码 **143**（SIGTERM），这是正常关闭，不是崩溃。
- **SIGUSR1** 在获得授权时触发进程内重启（网关工具/配置应用/更新，或启用 `commands.restart` 手动重启）。
- 默认情况下需要网关认证：设置 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）或 `gateway.auth.password`。客户端必须发送 `connect.params.auth.token/password`，除非使用 Tailscale Serve 身份。
- 向导现在默认生成令牌，即使在环回上也是如此。
- 端口优先级：`--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > 默认 `18789`。

## 远程访问

- 首选 Tailscale/VPN；否则使用 SSH 隧道：
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
- 客户端然后通过隧道连接到 `ws://127.0.0.1:18789`。
- 如果配置了令牌，客户端必须在 `connect.params.auth.token` 中包含它，即使通过隧道也是如此。

## 多个网关（同一主机）

通常没有必要：一个网关可以为多个消息通道和代理服务。仅在冗余或严格隔离时使用多个网关（例如救援机器人）。

如果你隔离状态 + 配置并使用唯一端口，则支持。完整指南：[多个网关](/gateway/multiple-gateways)。

服务名称支持配置：

- macOS: `bot.molt.<profile>`（旧的 `com.openclaw.*` 可能仍然存在）
- Linux: `openclaw-gateway-<profile>.service`
- Windows: `OpenClaw Gateway (<profile>)`

安装元数据嵌入在服务配置中：

- `OPENCLAW_SERVICE_MARKER=openclaw`
- `OPENCLAW_SERVICE_KIND=gateway`
- `OPENCLAW_SERVICE_VERSION=<version>`

救援机器人模式：保持第二个网关隔离，有自己的配置、状态目录、工作区和基础端口间距。完整指南：[救援机器人指南](/gateway/multiple-gateways#rescue-bot-guide)。

### 开发配置（`--dev`）

快速路径：运行完全隔离的开发实例（配置/状态/工作区），而不影响你的主要设置。

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# 然后定位开发实例：
openclaw --dev status
openclaw --dev health
```

默认值（可以通过环境/标志/配置覆盖）：

- `OPENCLAW_STATE_DIR=~/.openclaw-dev`
- `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
- `OPENCLAW_GATEWAY_PORT=19001`（网关 WS + HTTP）
- 浏览器控制服务端口 = `19003`（派生：`gateway.port+2`，仅环回）
- `canvasHost.port=19005`（派生：`gateway.port+4`）
- 当你在 `--dev` 下运行 `setup`/`onboard` 时，`agents.defaults.workspace` 默认变为 `~/.openclaw/workspace-dev`

派生端口（经验法则）：

- 基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）
- 浏览器控制服务端口 = 基础 + 2（仅环回）
- `canvasHost.port = base + 4`（或 `OPENCLAW_CANVAS_HOST_PORT` / 配置覆盖）
- 浏览器配置文件 CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配（每个配置文件持久化）。

每个实例的清单：

- 唯一的 `gateway.port`
- 唯一的 `OPENCLAW_CONFIG_PATH`
- 唯一的 `OPENCLAW_STATE_DIR`
- 唯一的 `agents.defaults.workspace`
- 单独的 WhatsApp 号码（如果使用 WA）

每个配置的服务安装：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

示例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

## 协议（操作员视角）

- 完整文档：[网关协议](/gateway/protocol) 和 [桥接协议（旧版）](/gateway/bridge-protocol)。
- 来自客户端的强制第一帧：`req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`。
- 网关回复 `res {type:"res", id, ok:true, payload:hello-ok }`（或 `ok:false` 带有错误，然后关闭）。
- 握手后：
  - 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  - 事件：`{type:"event", event, payload, seq?, stateVersion?}`
- 结构化存在条目：对于 WS 客户端，`{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }`（`instanceId` 来自 `connect.client.instanceId`）。
- `agent` 响应是两阶段的：首先 `res` 确认 `{runId,status:"accepted"}`，然后在运行结束后最终 `res` `{runId,status:"ok"|"error",summary}`；流式输出作为 `event:"agent"` 到达。

## 方法（初始集）

- `health` — 完整健康快照（与 `openclaw health --json` 相同形状）。
- `status` — 简短摘要。
- `system-presence` — 当前存在列表。
- `system-event` — 发布存在/系统注释（结构化）。
- `send` — 通过活动通道发送消息。
- `agent` — 运行代理回合（在同一连接上流式传输事件）。
- `node.list` — 列出配对 + 当前连接的节点（包括 `caps`、`deviceFamily`、`modelIdentifier`、`paired`、`connected` 和广告的 `commands`）。
- `node.describe` — 描述节点（功能 + 支持的 `node.invoke` 命令；适用于配对节点和当前连接的未配对节点）。
- `node.invoke` — 在节点上调用命令（例如 `canvas.*`、`camera.*`）。
- `node.pair.*` — 配对生命周期（`request`、`list`、`approve`、`reject`、`verify`）。

另请参阅：[存在](/concepts/presence) 了解存在是如何生成/去重的，以及为什么稳定的 `client.instanceId` 很重要。

## 事件

- `agent` — 来自代理运行的流式工具/输出事件（带 seq 标记）。
- `presence` — 存在更新（带有 stateVersion 的增量）推送到所有连接的客户端。
- `tick` — 定期保持活跃/无操作以确认存活。
- `shutdown` — 网关正在退出；负载包括 `reason` 和可选的 `restartExpectedMs`。客户端应该重新连接。

## WebChat 集成

- WebChat 是一个原生 SwiftUI UI，直接与网关 WebSocket 通信以获取历史记录、发送、中止和事件。
- 远程使用通过相同的 SSH/Tailscale 隧道；如果配置了网关令牌，客户端在 `connect` 期间包含它。
- macOS 应用通过单个 WS 连接（共享连接）；它从初始快照中填充存在并监听 `presence` 事件以更新 UI。

## 类型和验证

- 服务器使用 AJV 针对协议定义发出的 JSON Schema 验证每个入站帧。
- 客户端（TS/Swift）消费生成的类型（TS 直接；Swift 通过仓库的生成器）。
- 协议定义是事实来源；使用以下命令重新生成架构/模型：
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`

## 连接快照

- `hello-ok` 包括带有 `presence`、`health`、`stateVersion` 和 `uptimeMs` 的 `snapshot`，加上 `policy {maxPayload,maxBufferedBytes,tickIntervalMs}`，以便客户端无需额外请求即可立即呈现。
- `health`/`system-presence` 仍然可用于手动刷新，但在连接时不需要。

## 错误代码（res.error 形状）

- 错误使用 `{ code, message, details?, retryable?, retryAfterMs? }`。
- 标准代码：
  - `NOT_LINKED` — WhatsApp 未认证。
  - `AGENT_TIMEOUT` — 代理未在配置的截止时间内响应。
  - `INVALID_REQUEST` — 架构/参数验证失败。
  - `UNAVAILABLE` — 网关正在关闭或依赖项不可用。

## 保持活跃行为

- 定期发出 `tick` 事件（或 WS ping/pong），以便客户端知道网关是存活的，即使没有流量也是如此。
- 发送/代理确认保持为独立响应；不要为发送重载 tick。

## 重播/间隔

- 事件不会重播。客户端检测序列间隔并应在继续之前刷新（`health` + `system-presence`）。WebChat 和 macOS 客户端现在会在间隔时自动刷新。

## 监督（macOS 示例）

- 使用 launchd 保持服务存活：
  - 程序：`openclaw` 的路径
  - 参数：`gateway`
  - KeepAlive: true
  - StandardOut/Err：文件路径或 `syslog`
- 失败时，launchd 会重新启动；致命错误配置应保持退出，以便操作员注意到。
- LaunchAgents 是每个用户的，需要登录会话；对于无头设置使用自定义 LaunchDaemon（不随附）。
  - `openclaw gateway install` 写入 `~/Library/LaunchAgents/bot.molt.gateway.plist`
    （或 `bot.molt.<profile>.plist`；旧的 `com.openclaw.*` 会被清理）。
  - `openclaw doctor` 审核 LaunchAgent 配置并可以将其更新为当前默认值。

## 网关服务管理（CLI）

使用网关 CLI 进行安装/启动/停止/重启/状态查询：

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

注意：

- `gateway status` 默认使用服务的解析端口/配置探测网关 RPC（使用 `--url` 覆盖）。
- `gateway status --deep` 添加系统级扫描（LaunchDaemons/系统单元）。
- `gateway status --no-probe` 跳过 RPC 探测（当网络关闭时有用）。
- `gateway status --json` 对脚本是稳定的。
- `gateway status` 单独报告**监督进程运行时**（launchd/systemd 运行）与**RPC 可达性**（WS 连接 + 状态 RPC）。
- `gateway status` 打印配置路径 + 探测目标以避免"localhost vs LAN 绑定"混淆和配置不匹配。
- `gateway status` 在服务看起来运行但端口关闭时包括最后一条网关错误行。
- `logs` 通过 RPC 尾随网关文件日志（不需要手动 `tail`/`grep`）。
- 如果检测到其他类似网关的服务，CLI 会发出警告，除非它们是 OpenClaw 配置服务。
  我们仍然建议大多数设置**每台机器一个网关**；对冗余或救援机器人使用隔离的配置/端口。请参阅[多个网关](/gateway/multiple-gateways)。
  - 清理：`openclaw gateway uninstall`（当前服务）和 `openclaw doctor`（旧版迁移）。
- `gateway install` 在已安装时是空操作；使用 `openclaw gateway install --force` 重新安装（配置/环境/路径更改）。

打包的 mac 应用：

- OpenClaw.app 可以捆绑基于 Node 的网关中继并安装标记为 `bot.molt.gateway` 的每个用户 LaunchAgent（或 `bot.molt.<profile>`；旧的 `com.openclaw.*` 标签仍然可以干净地卸载）。
- 要干净地停止它，使用 `openclaw gateway stop`（或 `launchctl bootout gui/$UID/bot.molt.gateway`）。
- 要重启，使用 `openclaw gateway restart`（或 `launchctl kickstart -k gui/$UID/bot.molt.gateway`）。
  - `launchctl` 仅在 LaunchAgent 安装时才有效；否则先使用 `openclaw gateway install`。
  - 在运行命名配置时，将标签替换为 `bot.molt.<profile>`。

## 监督（systemd 用户单元）

OpenClaw 在 Linux/WSL2 上默认安装 **systemd 用户服务**。我们建议对单用户机器使用用户服务（更简单的环境，每个用户的配置）。对多用户或始终在线的服务器使用 **系统服务**（不需要 lingering，共享监督）。

`openclaw gateway install` 写入用户单元。`openclaw doctor` 审核该单元并可以将其更新为当前推荐默认值。

创建 `~/.config/systemd/user/openclaw-gateway[-<profile>].service`：

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
Environment=OPENCLAW_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```

启用 lingering（需要以便用户服务在注销/空闲后存活）：

```
sudo loginctl enable-linger youruser
```

在 Linux/WSL2 上运行（可能会提示 sudo；写入 `/var/lib/systemd/linger`）。然后启用服务：

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

**替代方案（系统服务）** — 对于始终在线或多用户服务器，你可以安装 systemd **系统**单元而不是用户单元（不需要 lingering）。创建 `/etc/systemd/system/openclaw-gateway[-<profile>].service`（复制上面的单元，切换 `WantedBy=multi-user.target`，设置 `User=` + `WorkingDirectory=`），然后：

```
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

## Windows (WSL2)

Windows 安装应使用 **WSL2** 并遵循上面的 Linux systemd 部分。

## 运营检查

- 存活：打开 WS 并发送 `req:connect` → 期望 `res` 带有 `payload.type="hello-ok"`（带有快照）。
- 就绪：调用 `health` → 期望 `ok: true` 并且 `linkChannel` 中有链接的通道（如果适用）。
- 调试：订阅 `tick` 和 `presence` 事件；确保 `status` 显示链接/认证时间；存在条目显示网关主机和连接的客户端。

## 安全保证

- 默认假设每台主机一个网关；如果你运行多个配置，隔离端口/状态并定位正确的实例。
- 没有直接 Baileys 连接的回退；如果网关关闭，发送会快速失败。
- 非连接第一帧或格式错误的 JSON 会被拒绝，套接字关闭。
- 正常关闭：在关闭前发出 `shutdown` 事件；客户端必须处理关闭 + 重新连接。

## CLI 辅助工具

- `openclaw gateway health|status` — 通过网关 WS 请求健康/状态。
- `openclaw message send --target <num> --message "hi" [--media ...]` — 通过网关发送（WhatsApp 幂等）。
- `openclaw agent --message "hi" --to <num>` — 运行代理回合（默认等待最终结果）。
- `openclaw gateway call <method> --params '{"k":"v"}'` — 用于调试的原始方法调用器。
- `openclaw gateway stop|restart` — 停止/重启受监督的网关服务（launchd/systemd）。
- 网关辅助子命令假设在 `--url` 上运行一个网关；它们不再自动生成一个。

## 迁移指导

- 停止使用 `openclaw gateway` 和旧的 TCP 控制端口。
- 更新客户端以使用带有强制连接和结构化存在的 WS 协议。
