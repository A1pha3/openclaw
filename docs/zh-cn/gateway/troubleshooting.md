---
summary: "常见 OpenClaw 故障的快速故障排除指南"
read_when:
  - 调查运行时问题或故障
title: "故障排除"
---

# 故障排除 🔧

当 OpenClaw 出问题时，这里是如何修复它的方法。

如果你只想要一个快速的分类配方，请从 FAQ 的[前 60 秒](/help/faq#first-60-seconds-if-somethings-broken)开始。本页更深入地讨论运行时故障和诊断。

提供程序特定快捷方式：[/channels/troubleshooting](/channels/troubleshooting)

## 状态和诊断

快速分类命令（按顺序）：

| 命令                            | 它告诉你的内容                                                                                      | 何时使用它                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| `openclaw status`                | 本地摘要：OS + 更新、网关可达性/模式、服务、代理/会话、提供程序配置状态                               | 第一次检查，快速概览                            |
| `openclaw status --all`          | 完整本地诊断（只读、可粘贴、较安全）包括日志尾迹                                                      | 当你需要分享调试报告时                         |
| `openclaw status --deep`         | 运行网关健康检查（包括提供程序探测；需要可达的网关）                                                  | 当"已配置"不等于"工作"时                       |
| `openclaw gateway probe`         | 网关发现 + 可达性（本地 + 远程目标）                                                                  | 当你怀疑探测的是错误的网关时                   |
| `openclaw channels status --probe` | 请求运行中的网关的频道状态（并可选探测）                                                              | 当网关可达但频道行为异常时                     |
| `openclaw gateway status`        | 监督器状态（launchd/systemd/schtasks）、运行时 PID/退出、最后一个网关错误                              | 当服务"看起来已加载"但什么都没运行时           |
| `openclaw logs --follow`         | 实时日志（最佳信号用于运行时问题）                                                                    | 当你需要实际失败原因时                         |

**分享输出：** 优先使用 `openclaw status --all`（它会编辑令牌）。如果你粘贴 `openclaw status`，考虑先设置 `OPENCLAW_SHOW_SECRETS=0`（令牌预览）。

另见：[健康检查](/gateway/health) 和 [日志](/logging)。

## 常见问题

### 未找到提供程序 "anthropic" 的 API 密钥

这意味着**代理的认证存储为空**或缺少 Anthropic 凭证。
认证是**每个代理的**，因此新代理不会继承主代理的密钥。

修复选项：

- 重新运行向导并为该代理选择 **Anthropic**。
- 或者在**网关主机**上粘贴设置令牌：
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
- 或者将 `auth-profiles.json` 从主代理目录复制到新代理目录。

验证：

```bash
openclaw models status
```

### OAuth 令牌刷新失败（Anthropic Claude 订阅）

这意味着存储的 Anthropic OAuth 令牌已过期，刷新失败。
如果你使用 Claude 订阅（没有 API key），最可靠的修复是切换到 **Claude Code 设置令牌**并粘贴到**网关主机**。

**推荐（设置令牌）：**

```bash
# 在网关主机上运行（粘贴设置令牌）
openclaw models auth setup-token --provider anthropic
openclaw models status
```

如果你在其他地方生成了令牌：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

更多详情：[Anthropic](/providers/anthropic) 和 [OAuth](/concepts/oauth)。

### 控制 UI 在 HTTP 上失败（"需要设备身份" / "连接失败"）

如果你通过纯 HTTP 打开控制面板（例如 `http://<lan-ip>:18789/` 或 `http://<tailscale-ip>:18789/`），浏览器处于**非安全上下文**并阻止 WebCrypto，因此无法生成设备身份。

**修复：**

- 优先使用 [Tailscale Serve](/gateway/tailscale) 的 HTTPS。
- 或者在网关上本地打开：`http://127.0.0.1:18789/`。
- 如果你必须停留在 HTTP，启用 `gateway.controlUi.allowInsecureAuth: true` 并使用网关令牌（仅令牌；无设备身份/配对）。参见[控制 UI](/web/control-ui#insecure-http)。

### CI 机密扫描失败

这意味着 `detect-secrets` 找到了不在基线中的新候选。
遵循[机密扫描](/gateway/security#secret-scanning-detect-secrets)。

### 服务已安装但没有任何运行

如果网关服务已安装但进程立即退出，服务可能看起来"已加载"而没有任何东西在运行。

**检查：**

```bash
openclaw gateway status
openclaw doctor
```

医生/服务将显示运行时状态（PID/最后退出）和日志提示。

**日志：**

- 首选：`openclaw logs --follow`
- 文件日志（始终）：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`（或你配置的 `logging.file`）
- macOS LaunchAgent（如果安装）：`$OPENCLAW_STATE_DIR/logs/gateway.log` 和 `gateway.err.log`
- Linux systemd（如果安装）：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**启用更多日志：**

- 提升文件日志详细程度（持久化 JSONL）：
  ```json
  { "logging": { "level": "debug" } }
  ```
- 提升控制台详细程度（仅 TTY 输出）：
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
- 快速提示：`--verbose` 仅影响**控制台**输出。文件日志仍由 `logging.level` 控制。

参见 [/logging](/logging) 了解格式、配置和访问的完整概览。

### "网关启动被阻止：设置 gateway.mode=local"

这意味着配置存在但 `gateway.mode` 未设置（或不是 `local`），因此网关拒绝启动。

**修复（推荐）：**

- 运行向导并将网关运行模式设置为 **Local**：
  ```bash
  openclaw configure
  ```
- 或直接设置：
  ```bash
  openclaw config set gateway.mode local
  ```

**如果你本来想运行远程网关：**

- 设置远程 URL 并保持 `gateway.mode=remote`：
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**临时/开发用：** 传递 `--allow-unconfigured` 以在没有 `gateway.mode=local` 的情况下启动网关。

**还没有配置文件？** 运行 `openclaw setup` 创建启动器配置，然后重新运行网关。

### 服务环境（PATH + 运行时）

网关服务以**最小 PATH** 运行以避免 shell/管理器混乱：

- macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
- Linux：`/usr/local/bin`、`/usr/bin`、`/bin`

这故意排除了版本管理器（nvm/fnm/volta/asdf）和包管理器（pnpm/npm），因为服务不加载你的 shell 初始化。像 `DISPLAY` 这样的运行时变量应该放在 `~/.openclaw/.env` 中（由网关早期加载）。
Exec 在 `host=gateway` 运行将你的登录 shell `PATH` 合并到 exec 环境中，因此缺少的工具通常意味着你的 shell 初始化没有导出它们（或设置 `tools.exec.pathPrepend`）。参见 [/tools/exec](/tools/exec)。

WhatsApp + Telegram 频道需要 **Node**；Bun 不支持。如果你的服务是用 Bun 或版本管理的 Node 路径安装的，运行 `openclaw doctor` 迁移到系统 Node 安装。

### 技能在沙箱中缺少 API 密钥

**症状：** 技能在主机上工作但在沙箱中失败并显示缺少 API 密钥。

**为什么：** 沙箱化 exec 在 Docker 内部运行，**不**继承主机 `process.env`。

**修复：**

- 设置 `agents.defaults.sandbox.docker.env`（或每个代理 `agents.list[].sandbox.docker.env`）
- 或将密钥烘焙到你的自定义沙箱镜像中
- 然后运行 `openclaw sandbox recreate --agent <id>`（或 `--all`）

### 服务运行但端口未监听

如果服务报告**运行**但没有任何东西在网关端口上监听，网关可能拒绝绑定。

**这里"运行"意味着什么**

- `Runtime: running` 意味着你的监督器（launchd/systemd/schtasks）认为进程是活的。
- `RPC probe` 意味着 CLI 实际上可以连接到网关 WebSocket 并调用 `status`。
- 始终信任 `Probe target:` + `Config (service):` 作为"我们实际上尝试了什么？"行。

**检查：**

- `gateway.mode` 必须是 `local` 才能用于 `openclaw gateway` 和服务。
- 如果你设置了 `gateway.mode=remote`，**CLI 默认**使用远程 URL。服务仍然可以在本地运行，但你的 CLI 可能探测的是错误的地方。使用 `openclaw gateway status` 查看服务的解析端口 + 探测目标（或传递 `--url`）。
- `openclaw gateway status` 和 `openclaw doctor` 显示**最后一个网关错误**当服务看起来在运行但端口关闭时。
- 非 loopback 绑定（`lan`/`tailnet`/`custom`，或 `auto` 当 loopback 不可用时）需要认证：
  `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- `gateway.remote.token` 仅用于远程 CLI 调用；它**不**启用本地认证。
- `gateway.token` 被忽略；使用 `gateway.auth.token`。

**如果 `openclaw gateway status` 显示配置不匹配**

- `Config (cli): ...` 和 `Config (service): ...` 通常应该匹配。
- 如果不匹配，你几乎肯定是在编辑一个配置而服务正在运行另一个。
- 修复：从你希望服务使用的相同 `--profile` / `OPENCLAW_STATE_DIR` 重新运行 `openclaw gateway install --force`。

**如果 `openclaw gateway status` 报告服务配置问题**

- 监督器配置（launchd/systemd/schtasks）缺少当前默认值。
- 修复：运行 `openclaw doctor` 更新它（或 `openclaw gateway install --force` 进行完整重写）。

**如果 `Last gateway error:` 提到"拒绝绑定...没有认证"**

- 你将 `gateway.bind` 设置为非 loopback 模式（`lan`/`tailnet`/`custom`，或 `auto` 当 loopback 不可用时）但未配置认证。
- 修复：设置 `gateway.auth.mode` + `gateway.auth.token`（或导出 `OPENCLAW_GATEWAY_TOKEN`）并重启服务。

**如果 `openclaw gateway status` 说 `bind=tailnet` 但未找到 tailnet 接口**

- 网关尝试绑定到 Tailscale IP（100.64.0.0/10）但主机上未检测到任何。
- 修复：在那台机器上启动 Tailscale（或将 `gateway.bind` 改为 `loopback`/`lan`）。

**如果 `Probe note:` 说探测使用 loopback**

- 这对于 `bind=lan` 是预期的：网关监听 `0.0.0.0`（所有接口），loopback 应该仍然在本地连接。
- 对于远程客户端，使用真实的 LAN IP（不是 `0.0.0.0`）加上端口，并确保已配置认证。

### 地址已被使用（端口 18789）

这意味着某个东西已经在网关端口上监听。

**检查：**

```bash
openclaw gateway status
```

它将显示监听器以及可能的原因（网关已在运行、SSH 隧道）。
如需要，停止服务或选择不同的端口。

### 检测到额外的工作区文件夹

如果你从旧版本升级，你可能磁盘上仍有 `~/openclaw`。
多个工作区目录可能导致混乱的认证或状态漂移，因为只有一个工作区是活动的。

**修复：** 保持单个活动工作区并归档/删除其余的。参见[代理工作区](/concepts/agent-workspace#extra-workspace-folders)。

### 主聊天在沙箱工作区中运行

症状：`pwd` 或文件工具显示 `~/.openclaw/sandboxes/...`，即使你期望主机工作区。

**为什么：** `agents.defaults.sandbox.mode: "non-main"` 基于 `session.mainKey`（默认 `"main"`）。
群组/频道会话使用自己的密钥，因此它们被视为非 main 并获得沙箱工作区。

**修复选项：**

- 如果你希望代理使用主机工作区：设置 `agents.list[].sandbox.mode: "off"`。
- 如果你希望在沙箱内访问主机工作区：为该代理设置 `workspaceAccess: "rw"`。

### "代理已中止"

代理在响应中途被中断。

**原因：**

- 用户发送了 `stop`、`abort`、`esc`、`wait` 或 `exit`
- 超时已超出
- 进程崩溃

**修复：** 只需发送另一条消息。会话继续。

### "代理在回复前失败：未知模型：anthropic/claude-haiku-3-5"

OpenClaw 故意拒绝**旧版/不安全模型**（特别是那些更容易受到提示注入影响的模型）。如果你看到此错误，模型名称不再受支持。

**修复：**

- 为提供程序选择**最新**模型并更新你的配置或模型别名。
- 如果你不确定哪些模型可用，运行 `openclaw models list` 或 `openclaw models scan` 并选择受支持的模型。
- 检查网关日志以获取详细的失败原因。

另见：[Models CLI](/cli/models) 和 [模型提供程序](/concepts/model-providers)。

### 消息未触发

**检查 1：** 发送者在允许列表中吗？

```bash
openclaw status
```

在输出中查找 `AllowFrom: ...`。

**检查 2：** 对于群组聊天，需要提及吗？

```bash
# 消息必须匹配 mentionPatterns 或明确提及；默认值存在于频道群组/服务器中。
# 多代理：`agents.list[].groupChat.mentionPatterns` 覆盖全局模式。
grep -n "agents\|groupChat\|mentionPatterns\|channels\.whatsapp\.groups\|channels\.telegram\.groups\|channels\.imessage\.groups\|channels\.discord\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**检查 3：** 检查日志

```bash
openclaw logs --follow
# 或者如果你想要快速过滤器：
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\|skip\|unauthorized"
```

### 配对码未到达

如果 `dmPolicy` 是 `pairing`，未知发送者应该收到一个代码，他们的消息在被批准之前被忽略。

**检查 1：** 是否有待处理请求在等待？

```bash
openclaw pairing list <channel>
```

待处理的 DM 配对请求默认为**每个频道 3 个**。如果列表已满，新请求不会生成代码，直到一个被批准或过期。

**检查 2：** 请求是否已创建但没有回复发送？

```bash
openclaw logs --follow | grep "pairing request"
```

**检查 3：** 确认该频道的 `dmPolicy` 不是 `open`/`allowlist`。

### 图片 + 提及不工作

已知问题：当你只发送带有提及的图片（没有其他文本）时，WhatsApp 有时不会包含提及元数据。

**解决方法：** 在提及时添加一些文本：

- ❌ `@openclaw` + 图片
- ✅ `@openclaw 查看这个` + 图片

### 会话未恢复

**检查 1：** 会话文件在那里吗？

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**检查 2：** 重置窗口是否太短？

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080 // 7 天
    }
  }
}
```

**检查 3：** 是否有人发送了 `/new`、`/reset` 或重置触发器？

### 代理超时

默认超时是 30 分钟。对于长任务：

```json
{
  "reply": {
    "timeoutSeconds": 3600 // 1 小时
  }
}
```

或使用 `process` 工具将长命令后台运行。

### WhatsApp 断开连接

```bash
# 检查本地状态（凭证、会话、排队的事件）
openclaw status
# 探测运行中的网关 + 频道（WA 连接 + Telegram + Discord API）
openclaw status --deep

# 查看最近的连接事件
openclaw logs --limit 200 | grep "connection\|disconnect\|logout"
```

**修复：** 通常只要网关运行就会自动重新连接。如果你卡住了，重启网关进程（无论你如何监督它），或用详细输出手动运行它：

```bash
openclaw gateway --verbose
```

如果你已登录/取消链接：

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # 如果 logout 无法干净地删除所有内容
openclaw channels login --verbose       # 重新扫描 QR
```

### 媒体发送失败

**检查 1：** 文件路径有效吗？

```bash
ls -la /path/to/your/image.jpg
```

**检查 2：** 太大了吗？

- 图片：最大 6MB
- 音频/视频：最大 16MB
- 文档：最大 100MB

**检查 3：** 检查媒体日志

```bash
grep "media\|fetch\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

### 高内存使用

OpenClaw 将对话历史保存在内存中。

**修复：** 定期重启或设置会话限制：

```json
{
  "session": {
    "historyLimit": 100 // 保留的最大消息数
  }
}
```

## 常见故障排除

### "网关无法启动——配置无效"

OpenClaw 现在在配置包含未知键、格式错误的值或无效类型时拒绝启动。
这是出于安全性的有意为之。

用 Doctor 修复：

```bash
openclaw doctor
openclaw doctor --fix
```

注意：

- `openclaw doctor` 报告每个无效条目。
- `openclaw doctor --fix` 应用迁移/修复并重写配置。
- 即使配置无效，诊断命令如 `openclaw logs`、`openclaw health`、`openclaw status`、`openclaw gateway status` 和 `openclaw gateway probe` 仍会运行。

### "所有模型失败"——我应该首先检查什么？

- **凭证**：提供程序的凭证是否存在（auth 配置文件 + 环境变量）。
- **模型路由**：确认 `agents.defaults.model.primary` 和回退是你可以访问的模型。
- **网关日志**在 `/tmp/openclaw/…` 中查看确切的提供程序错误。
- **模型状态**：使用 `/model status`（聊天）或 `openclaw models status`（CLI）。

### 我在我的个人 WhatsApp 号码上运行——为什么自聊很奇怪？

启用自聊模式并允许你自己的号码：

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"],
    },
  },
}
```

参见 [WhatsApp 设置](/channels/whatsapp)。

### WhatsApp 让我登录了。我如何重新认证？

再次运行登录命令并扫描 QR 码：

```bash
openclaw channels login
```

### `main` 上的构建错误——标准修复路径是什么？

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. 检查 GitHub issues 或 Discord
4. 临时解决方法：检出旧提交

### npm install 失败（allow-build-scripts / 缺少 tar 或 yargs）。现在怎么办？

如果你从源头运行，使用仓库的包管理器：**pnpm**（首选）。
仓库声明了 `packageManager: "pnpm@…"`。

典型恢复：

```bash
git status   # 确保你在仓库根目录
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

为什么：pnpm 是这个仓库配置的包管理器。

### 我如何在 git 安装和 npm 安装之间切换？

使用**网站安装程序**并用标志选择安装方法。它会原地升级并重写网关服务以指向新的安装。

切换**到 git 安装**：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
```

切换**到 npm 全局**：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

注意：

- git 流只有在仓库干净时才进行变基。首先提交或隐藏更改。
- 切换后，运行：
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

### Telegram 块流未在工具调用之间拆分文本。为什么会这样？

块流只发送**完成的文本块**。你看到单条消息的常见原因：

- `agents.defaults.blockStreamingDefault` 仍然是 `"off"`。
- `channels.telegram.blockStreaming` 设置为 `false`。
- `channels.telegram.streamMode` 是 `partial` 或 `block`**并且草稿流正在活动**
  （私人聊天 + 话题）。草稿流在这种情况下禁用块流。
- 你的 `minChars` / 合并设置太高，导致块被合并。
- 模型发出一个大的文本块（没有中间回复刷新点）。

修复清单：

1. 将块流设置放在 `agents.defaults` 下，而不是根目录。
2. 如果你想要真正的多消息块回复，设置 `channels.telegram.streamMode: "off"`。
3. 在调试时使用更小的块/合并阈值。

参见 [流式传输](/concepts/streaming)。

### Discord 即使 `requireMention: false` 也不在我的服务器中回复。为什么？

`requireMention` 只控制**之后**频道通过允许列表的提及限制。
默认情况下 `channels.discord.groupPolicy` 是 **allowlist**，因此服务器必须明确启用。
如果你设置了 `channels.discord.guilds.<guildId>.channels`，只有列出的频道被允许；省略它以允许服务器中的所有频道。

修复清单：

1. 设置 `channels.discord.groupPolicy: "open"` **或** 添加服务器允许列表条目（可选地还有频道允许列表）。
2. 在 `channels.discord.guilds.<guildId>.channels` 中使用**数字频道 ID**。
3. 将 `requireMention: false` 放在 `channels.discord.guilds` 下（全局或每个频道）。
   顶级 `channels.discord.requireMention` 不是支持的键。
4. 确保机器人有**消息内容意向**和频道权限。
5. 运行 `openclaw channels status --probe` 获取审核提示。

文档：[Discord](/channels/discord)、[频道故障排除](/channels/troubleshooting)。

### Cloud Code Assist API 错误：无效工具 schema (400)。现在怎么办？

这几乎总是**工具 schema 兼容性问题**。Cloud Code Assist 端点接受 JSON Schema 的严格子集。
OpenClaw 在当前 `main` 中清理/规范化工具 schema，但修复尚未在最新发布中（截至 2026 年 1 月 13 日）。

修复清单：

1. **更新 OpenClaw**：
   - 如果你可以从源头运行，拉取 `main` 并重启网关。
   - 否则，等待包含 schema 清理器的下一个发布。
2. 避免不支持的关键字如 `anyOf/oneOf/allOf`、`patternProperties`、
   `additionalProperties`、`minLength`、`maxLength`、`format` 等。
3. 如果你定义自定义工具，保持顶级 schema 为 `type: "object"` 加上 `properties` 和简单枚举。

参见 [工具](/tools) 和 [TypeBox schema](/concepts/typebox)。

## macOS 特定问题

### 应用在授予权限（语音/麦克风）时崩溃

如果应用在点击隐私提示上的"允许"时消失或显示"中止陷阱 6"：

**修复 1：重置 TCC 缓存**

```bash
tccutil reset All bot.molt.mac.debug
```

**修复 2：强制新的捆绑 ID**
如果重置不起作用，在 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 中更改 `BUNDLE_ID`（例如添加 `.test` 后缀）并重新构建。这会让 macOS 将其视为新应用。

### 网关卡在"启动中..."

应用连接到本地网关端口 `18789`。如果它卡住了：

**修复 1：停止监督器（首选）**
如果网关由 launchd 监督，杀死 PID 只会重新生成它。首先停止监督器：

```bash
openclaw gateway status
openclaw gateway stop
# 或者：launchctl bootout gui/$UID/bot.molt.gateway（替换为 bot.molt.<profile>；遗留的 com.openclaw.* 仍然工作）
```

**修复 2：端口繁忙（找到监听器）**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

如果它是一个无人监督的进程，尝试优雅地停止，然后升级：

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # 最后手段
```

**修复 3：检查 CLI 安装**
确保全局 `openclaw` CLI 已安装并与应用版本匹配：

```bash
openclaw --version
npm install -g openclaw@<version>
```

## 调试模式

获取详细日志：

```bash
# 在配置中打开跟踪日志：
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# 然后运行详细命令以将调试输出镜像到 stdout：
openclaw gateway --verbose
openclaw channels login --verbose
```

## 日志位置

| 日志                               | 位置                                                                                                                                                                                                                                                                                                                    |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 网关文件日志（结构化）              | `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（或 `logging.file`）                                                                                                                                                                                                                                                                 |
| 网关服务日志（监督器）              | macOS：`$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log`（默认：`~/.openclaw/logs/...`；配置文件使用 `~/.openclaw-<profile>/logs/...`）<br />Linux：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| 会话文件                           | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`                                                                                                                                                                                                                                                                            |
| 媒体缓存                           | `$OPENCLAW_STATE_DIR/media/`                                                                                                                                                                                                                                                                                                |
| 凭证                               | `$OPENCLAW_STATE_DIR/credentials/`                                                                                                                                                                                                                                                                                          |

## 健康检查

```bash
# 监督器 + 探测目标 + 配置路径
openclaw gateway status
# 包括系统级扫描（遗留/额外服务、端口监听器）
openclaw gateway status --deep

# 网关可达吗？
openclaw health --json
# 如果失败，使用连接详细信息重新运行：
openclaw health --verbose

# 有什么在默认端口上监听吗？
lsof -nP -iTCP:18789 -sTCP:LISTEN

# 最近的活动（RPC 日志尾）
openclaw logs --follow
# 如果 RPC 关闭的回退
tail -20 /tmp/openclaw/openclaw-*.log
```

## 重置所有内容

核选项：

```bash
openclaw gateway stop
# 如果你安装了服务并想要干净安装：
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # 重新配对 WhatsApp
openclaw gateway restart           # 或：openclaw gateway
```

⚠️ 这会丢失所有会话并需要重新配对 WhatsApp。

## 获取帮助

1. 首先检查日志：`/tmp/openclaw/`（默认：`openclaw-YYYY-MM-DD.log`，或你配置的 `logging.file`）
2. 在 GitHub 上搜索现有 issues
3. 用以下内容打开新 issue：
   - OpenClaw 版本
   - 相关的日志片段
   - 重现步骤
   - 你的配置（编辑机密信息！）

---

_"你试过重启它吗？"_ —— 每个 IT 人

🦞🔧

### 浏览器未启动（Linux）

如果你看到 `"Failed to start Chrome CDP on port 18800"`：

**最可能的原因：** Ubuntu 上的 Snap 打包 Chromium。

**快速修复：** 改为安装 Google Chrome：

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

然后在配置中设置：

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**完整指南：** 参见 [browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
