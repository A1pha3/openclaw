---
summary: "Google Chat 应用支持状态、功能和配置"
read_when:
  - 正在处理 Google Chat 频道功能
title: "Google Chat"
---

# Google Chat (Chat API)

状态：已支持通过 Google Chat API webhook 进行私信和空间交互（仅限 HTTP）。

## 快速设置（新手）

1. 创建一个 Google Cloud 项目并启用 **Google Chat API**。
   - 访问：[Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - 如果尚未启用 API，请启用它。
2. 创建一个 **Service Account（服务账号）**：
   - 点击 **Create Credentials** > **Service Account**。
   - 命名（例如 `openclaw-chat`）。
   - 将权限留空（点击 **Continue**）。
   - 将访问主体留空（点击 **Done**）。
3. 创建并下载 **JSON Key（密钥）**：
   - 在服务账号列表中，点击您刚创建的账号。
   - 进入 **Keys** 标签页。
   - 点击 **Add Key** > **Create new key**。
   - 选择 **JSON** 并点击 **Create**。
4. 将下载的 JSON 文件存储在您的网关主机上（例如 `~/.openclaw/googlechat-service-account.json`）。
5. 在 [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) 中创建一个 Google Chat 应用：
   - 填写 **Application info**：
     - **App name**：（例如 `OpenClaw`）
     - **Avatar URL**：（例如 `https://openclaw.ai/logo.png`）
     - **Description**：（例如 `Personal AI Assistant`）
   - 启用 **Interactive features**。
   - 在 **Functionality** 下，勾选 **Join spaces and group conversations**。
   - 在 **Connection settings** 下，选择 **HTTP endpoint URL**。
   - 在 **Triggers** 下，选择 **Use a common HTTP endpoint URL for all triggers** 并将其设置为您的网关公共 URL 后跟 `/googlechat`。
     - _提示：运行 `openclaw status` 以查找您的网关公共 URL。_
   - 在 **Visibility** 下，勾选 **Make this Chat app available to specific people and groups in <Your Domain>**。
   - 在文本框中输入您的电子邮件地址（例如 `user@example.com`）。
   - 点击底部的 **Save**。
6. **启用应用状态**：
   - 保存后，**刷新页面**。
   - 查找 **App status** 部分（通常在保存后的顶部或底部）。
   - 将状态更改为 **Live - available to users**。
   - 再次点击 **Save**。
7. 使用服务账号路径和 webhook 受众配置 OpenClaw：
   - 环境变量：`GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - 或配置文件：`channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8. 设置 webhook 受众类型和值（与您的 Chat 应用配置匹配）。
9. 启动网关。Google Chat 将向您的 webhook 路径发送 POST 请求。

## 添加到 Google Chat

网关运行后，您的电子邮件地址已添加到可见性列表：

1. 访问 [Google Chat](https://chat.google.com/)。
2. 点击 **Direct Messages** 旁边的 **+**（加号）图标。
3. 在搜索栏（通常用于添加人员的地方）中，输入您在 Google Cloud Console 中配置的 **App name**。
   - **注意**：机器人不会出现在 "Marketplace" 浏览列表中，因为这是一个私有应用。您必须按名称搜索它。
4. 从结果中选择您的机器人。
5. 点击 **Add** 或 **Chat** 开始 1:1 对话。
6. 发送 "Hello" 来触发助手！

## 公共 URL（仅 Webhook）

Google Chat webhook 需要公共 HTTPS 端点。出于安全考虑，**仅将 `/googlechat` 路径暴露给互联网**。将 OpenClaw 控制面板和其他敏感端点保留在您的私有网络上。

### 选项 A：Tailscale Funnel（推荐）

使用 Tailscale Serve 作为私有控制面板，使用 Funnel 作为公共 webhook 路径。这使 `/` 保持私有，同时仅公开 `/googlechat`。

1. **检查您的网关绑定的地址：**

   ```bash
   ss -tlnp | grep 18789
   ```

   记下 IP 地址（例如 `127.0.0.1`、`0.0.0.0` 或您的 Tailscale IP，如 `100.x.x.x`）。

2. **仅将控制面板公开到 tailnet（端口 8443）：**

   ```bash
   # 如果绑定到 localhost（127.0.0.1 或 0.0.0.0）：
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # 如果仅绑定到 Tailscale IP（例如 100.106.161.80）：
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **仅公开 webhook 路径：**

   ```bash
   # 如果绑定到 localhost（127.0.0.1 或 0.0.0.0）：
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # 如果仅绑定到 Tailscale IP（例如 100.106.161.80）：
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **授权节点使用 Funnel：**
   如果提示，请访问输出中显示的授权 URL，以便在您的 tailnet 策略中为此节点启用 Funnel。

5. **验证配置：**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

 您的公共 webhook URL 将是：
 `https://<node-name>.<tailnet>.ts.net/googlechat`

 您的私有控制面板保持 tailnet 专用：
 `https://<node-name>.<tailnet>.ts.net:8443/`

 在 Google Chat 应用配置中使用公共 URL（不带 `:8443`）。

> 注意：此配置在重启后仍然有效。如需稍后删除，请运行 `tailscale funnel reset` 和 `tailscale serve reset`。

### 选项 B：反向代理（Caddy）

如果您使用 Caddy 等反向代理，请仅代理特定路径：

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

使用此配置，对 `your-domain.com/` 的任何请求都将被忽略或返回为 404，而 `your-domain.com/googlechat` 将被安全地路由到 OpenClaw。

### 选项 C：Cloudflare Tunnel

配置您的隧道入口规则以仅路由 webhook 路径：

- **Path**: `/googlechat` -> `http://localhost:18789/googlechat`
- **Default Rule**: HTTP 404（Not Found）

## 工作原理

1. Google Chat 向网关发送 webhook POST 请求。每个请求都包含一个 `Authorization: Bearer <token>` 头。
2. OpenClaw 根据配置的 `audienceType` + `audience` 验证令牌：
   - `audienceType: "app-url"` → 受众是您的 HTTPS webhook URL。
   - `audienceType: "project-number"` → 受众是 Cloud 项目编号。
3. 消息按空间路由：
   - 私信使用会话密钥 `agent:<agentId>:googlechat:dm:<spaceId>`。
   - 空间使用会话密钥 `agent:<agentId>:googlechat:group:<spaceId>`。
4. 私信访问默认为配对模式。未知发件人将收到配对码；通过以下方式批准：
   - `openclaw pairing approve googlechat <code>`
5. 群组空间默认需要 @提及。如果提及检测需要应用的用户名，请使用 `botUser`。

## 目标

使用这些标识符进行投递和允许列表：

- 私信：`users/<userId>` 或 `users/<email>`（接受电子邮件地址）。
- 空间：`spaces/<spaceId>`。

## 配置要点

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选；有助于提及检测
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

注意事项：

- 服务账号凭证也可以通过 `serviceAccount`（JSON 字符串）内联传递。
- 如果未设置 `webhookPath`，则默认 webhook 路径为 `/googlechat`。
- 当启用 `actions.reactions` 时，可通过 `reactions` 工具和 `channels action` 使用反应。
- `typingIndicator` 支持 `none`、`message`（默认）和 `reaction`（反应需要用户 OAuth）。
- 附件通过 Chat API 下载并存储在媒体管道中（大小由 `mediaMaxMb` 限制）。

## 故障排除

### 405 Method Not Allowed

如果 Google Cloud Logs Explorer 显示如下错误：

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

这意味着未注册 webhook 处理程序。常见原因：

1. **未配置频道**：您的配置中缺少 `channels.googlechat` 部分。通过以下方式验证：

   ```bash
   openclaw config get channels.googlechat
   ```

   如果返回 "Config path not found"，请添加配置（参见 [配置要点](#config-highlights)）。

2. **插件未启用**：检查插件状态：

   ```bash
   openclaw plugins list | grep googlechat
   ```

   如果显示 "disabled"，请在配置中添加 `plugins.entries.googlechat.enabled: true`。

3. **网关未重启**：添加配置后，重启网关：
   ```bash
   openclaw gateway restart
   ```

验证频道正在运行：

```bash
openclaw channels status
# 应显示：Google Chat default: enabled, configured, ...
```

### 其他问题

- 使用 `openclaw channels status --probe` 检查认证错误或缺少受众配置。
- 如果没有收到消息，请确认 Chat 应用的 webhook URL + 事件订阅。
- 如果提及限制阻止回复，请将 `botUser` 设置为应用的用户资源名称并验证 `requireMention`。
- 在发送测试消息时使用 `openclaw logs --follow`，以查看请求是否到达网关。

相关文档：

- [网关配置](/zh-CN/gateway/configuration)
- [安全](/zh-CN/gateway/security)
- [反应](/zh-CN/tools/reactions)
