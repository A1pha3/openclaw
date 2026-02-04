---
title: Fly.io
description: 在 Fly.io 上部署 OpenClaw
---

# Fly.io 部署

**目标：** 在 [Fly.io](https://fly.io) 机器上运行 OpenClaw Gateway，具有持久存储、自动 HTTPS 和 Discord/频道访问功能。

## 所需内容

- 已安装 [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/)
- Fly.io 账户（免费套餐即可）
- 模型认证：Anthropic API 密钥（或其他提供商密钥）
- 频道凭证：Discord bot token、Telegram token 等

## 初学者快速路径

1. 克隆仓库 → 自定义 `fly.toml`
2. 创建应用 + 卷 → 设置密钥
3. 使用 `fly deploy` 部署
4. SSH 进入创建配置或使用 Control UI

## 1) 创建 Fly 应用

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 创建新的 Fly 应用（选择您自己的名称）
fly apps create my-openclaw

# 创建持久卷（1GB 通常足够）
fly volumes create openclaw_data --size 1 --region iad
```

**提示：** 选择靠近您的区域。常见选项：`lhr`（伦敦）、`iad`（弗吉尼亚）、`sjc`（圣何塞）。

## 2) 配置 fly.toml

编辑 `fly.toml` 以匹配您的应用名称和要求。

**安全说明：** 默认配置会暴露公共 URL。对于没有公共 IP 的加固部署，请参阅[私有部署（加固）](#私有部署加固)或使用 `fly.private.toml`。

```toml
app = "my-openclaw"  # 您的应用名称
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**关键设置：**

| 设置                        | 原因                                                                 |
| --------------------------- | ------------------------------------------------------------------- |
| `--bind lan`                | 绑定到 `0.0.0.0` 以便 Fly 代理可以访问 gateway                      |
| `--allow-unconfigured`      | 在没有配置文件的情况下启动（您将在之后创建）                          |
| `internal_port = 3000`       | 必须匹配 `--port 3000`（或 `OPENCLAW_GATEWAY_PORT`）以便进行 Fly 健康检查 |
| `memory = "2048mb"`          | 512MB 太小；建议 2GB                                                |
| `OPENCLAW_STATE_DIR = "/data"` | 在卷上持久化状态                                                   |

## 3) 设置密钥

```bash
# 必需：Gateway token（用于非环回绑定）
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# 模型提供商 API 密钥
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# 可选：其他提供商
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# 频道 token
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**说明：**

- 非环回绑定（`--bind lan`）需要 `OPENCLAW_GATEWAY_TOKEN` 以确保安全。
- 将这些 token 视为密码。
- **对于所有 API 密钥和 token，首选环境变量而非配置文件。**这可以防止密钥进入 `openclaw.json`，避免意外泄露或记录。

## 4) 部署

```bash
fly deploy
```

首次部署会构建 Docker 镜像（约 2-3 分钟）。后续部署会更快。

部署后，验证：

```bash
fly status
fly logs
```

您应该看到：

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) 创建配置文件

SSH 进入机器以创建适当的配置：

```bash
fly ssh console
```

创建配置目录和文件：

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**说明：** 使用 `OPENCLAW_STATE_DIR=/data` 时，配置路径为 `/data/openclaw.json`。

**说明：** Discord token 可以来自以下任一来源：

- 环境变量：`DISCORD_BOT_TOKEN`（推荐用于密钥）
- 配置文件：`channels.discord.token`

如果使用环境变量，无需在配置中添加 token。Gateway 会自动读取 `DISCORD_BOT_TOKEN`。

重启以应用更改：

```bash
exit
fly machine restart <machine-id>
```

## 6) 访问 Gateway

### Control UI

在浏览器中打开：

```bash
fly open
```

或访问 `https://my-openclaw.fly.dev/`

粘贴您的 gateway token（来自 `OPENCLAW_GATEWAY_TOKEN`）以进行身份验证。

### 日志

```bash
fly logs              # 实时日志
fly logs --no-tail    # 最近日志
```

### SSH 控制台

```bash
fly ssh console
```

## 故障排除

### "App is not listening on expected address"

Gateway 绑定到 `127.0.0.1` 而不是 `0.0.0.0`。

**修复：** 在 `fly.toml` 的进程命令中添加 `--bind lan`。

### 健康检查失败 / 连接被拒绝

Fly 无法在配置的端口上访问 gateway。

**修复：** 确保 `internal_port` 与 gateway 端口匹配（设置 `--port 3000` 或 `OPENCLAW_GATEWAY_PORT=3000`）。

### OOM / 内存问题

容器持续重启或被杀死。迹象：`SIGABRT`、`v8::internal::Runtime_AllocateInYoungGeneration` 或静默重启。

**修复：** 在 `fly.toml` 中增加内存：

```toml
[[vm]]
  memory = "2048mb"
```

或更新现有机器：

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**说明：** 512MB 太小。1GB 可能可以工作，但在负载下或使用详细日志时可能会 OOM。**建议使用 2GB。**

### Gateway 锁问题

Gateway 拒绝启动，出现"already running"错误。

当容器重启但 PID 锁文件仍保留在卷上时会发生这种情况。

**修复：** 删除锁文件：

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

锁文件位于 `/data/gateway.*.lock`（不在子目录中）。

### 配置未读取

如果使用 `--allow-unconfigured`，gateway 会创建最小配置。您的自定义配置 `/data/openclaw.json` 应在重启时被读取。

验证配置是否存在：

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### 通过 SSH 写入配置

`fly ssh console -C` 命令不支持 shell 重定向。要写入配置文件：

```bash
# 使用 echo + tee（从本地管道到远程）
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# 或使用 sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**说明：** 如果文件已存在，`fly sftp` 可能会失败。首先删除：

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### 状态未持久化

如果在重启后丢失凭证或会话，状态目录正在写入容器文件系统。

**修复：** 确保在 `fly.toml` 中设置了 `OPENCLAW_STATE_DIR=/data` 并重新部署。

## 更新

```bash
# 拉取最新更改
git pull

# 重新部署
fly deploy

# 检查健康状态
fly status
fly logs
```

### 更新机器命令

如果需要在不进行完整重新部署的情况下更改启动命令：

```bash
# 获取机器 ID
fly machines list

# 更新命令
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# 或增加内存
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**说明：** 在 `fly deploy` 之后，机器命令可能会重置为 `fly.toml` 中的命令。如果您进行了手动更改，请在部署后重新应用。

## 私有部署（加固）

默认情况下，Fly 会分配公共 IP，使您的 gateway 可在 `https://your-app.fly.dev` 访问。这很方便，但意味着您的部署可被互联网扫描器（Shodan、Censys 等）发现。

对于**没有公共暴露**的加固部署，请使用私有模板。

### 何时使用私有部署

- 您只进行**出站**调用/消息（没有入站 webhooks）
- 您对任何 webhook 回调使用 **ngrok 或 Tailscale** 隧道
- 您通过 **SSH、代理或 WireGuard** 而非浏览器访问 gateway
- 您希望部署**对互联网扫描器隐藏**

### 设置

使用 `fly.private.toml` 而不是标准配置：

```bash
# 使用私有配置部署
fly deploy -c fly.private.toml
```

或转换现有部署：

```bash
# 列出当前 IP
fly ips list -a my-openclaw

# 释放公共 IP
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# 切换到私有配置，以便将来的部署不会重新分配公共 IP
#（删除 [http_service] 或使用私有模板部署）
fly deploy -c fly.private.toml

# 分配仅私有 IPv6
fly ips allocate-v6 --private -a my-openclaw
```

此后，`fly ips list` 应该只显示一个 `private` 类型的 IP：

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### 访问私有部署

由于没有公共 URL，请使用以下方法之一：

**选项 1：本地代理（最简单）**

```bash
# 将本地端口 3000 转发到应用
fly proxy 3000:3000 -a my-openclaw

# 然后在浏览器中打开 http://localhost:3000
```

**选项 2：WireGuard VPN**

```bash
# 创建 WireGuard 配置（一次性）
fly wireguard create

# 导入 WireGuard 客户端，然后通过内部 IPv6 访问
# 示例：http://[fdaa:x:x:x:x::x]:3000
```

**选项 3：仅 SSH**

```bash
fly ssh console -a my-openclaw
```

### 使用私有部署的 Webhooks

如果需要 webhook 回调（Twilio、Telnyx 等）而没有公共暴露：

1. **ngrok 隧道** - 在容器内运行 ngrok 或作为 sidecar
2. **Tailscale Funnel** - 通过 Tailscale 暴露特定路径
3. **仅出站** - 某些提供商（Twilio）在没有 webhooks 的情况下可以正常进行出站调用

使用 ngrok 的语音通话配置示例：

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

ngrok 隧道在容器内运行，提供公共 webhook URL 而不暴露 Fly 应用本身。

### 安全优势

| 方面            | 公共       | 私有    |
| --------------- | ---------- | ------- |
| 互联网扫描器    | 可被发现   | 隐藏    |
| 直接攻击        | 可能       | 被阻止  |
| Control UI 访问 | 浏览器     | 代理/VPN |
| Webhook 交付    | 直接       | 通过隧道 |

## 说明

- Fly.io 使用 **x86 架构**（不是 ARM）
- Dockerfile 兼容两种架构
- 对于 WhatsApp/Telegram 入门，使用 `fly ssh console`
- 持久数据位于卷上的 `/data`
- Signal 需要 Java + signal-cli；使用自定义镜像并保持内存在 2GB 以上。

## 成本

使用推荐配置（`shared-cpu-2x`、2GB RAM）：

- 约 $10-15/月，具体取决于使用情况
- 免费套餐包含一定的配额

详情请参阅 [Fly.io 定价](https://fly.io/docs/about/pricing/)。
