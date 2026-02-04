---
summary: "修复 Linux 上 Chrome/Brave/Edge/Chromium CDP 启动问题"
read_when:
  - Linux 上浏览器控制失败
  - 特别是使用 snap 版 Chromium 时遇到问题
title: "浏览器故障排除（Linux）"
---

# 浏览器故障排除（Linux）

## 问题：「Failed to start Chrome CDP on port 18800」

OpenClaw 的浏览器控制服务器无法启动 Chrome/Brave/Edge/Chromium，报错：

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### 根本原因

在 Ubuntu（及许多 Linux 发行版）上，默认的 Chromium 安装是 **snap 包**。Snap 的 AppArmor 沙箱机制会干扰 OpenClaw 启动和监控浏览器进程的方式。

当你运行 `apt install chromium` 时，实际安装的是一个重定向到 snap 的存根包：

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

这**不是**真正的浏览器——只是一个包装器。

### 解决方案一：安装 Google Chrome（推荐）

安装官方 Google Chrome `.deb` 包，它不受 snap 沙箱限制：

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # 如果有依赖错误
```

然后更新 OpenClaw 配置（`~/.openclaw/openclaw.json`）：

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### 解决方案二：使用 Snap Chromium 的附加模式

如果必须使用 snap 版 Chromium，可以配置 OpenClaw 附加到手动启动的浏览器：

**第一步：更新配置**

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

**第二步：手动启动 Chromium**

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

**第三步（可选）：创建 systemd 用户服务自动启动 Chrome**

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

启用服务：

```bash
systemctl --user enable --now openclaw-browser.service
```

### 验证浏览器是否正常工作

**检查状态：**

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

**测试浏览功能：**

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

## 配置参考

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `browser.enabled` | 启用浏览器控制 | `true` |
| `browser.executablePath` | Chromium 系浏览器二进制路径（Chrome/Brave/Edge/Chromium） | 自动检测（优先使用默认浏览器） |
| `browser.headless` | 无界面模式运行 | `false` |
| `browser.noSandbox` | 添加 `--no-sandbox` 标志（某些 Linux 环境需要） | `false` |
| `browser.attachOnly` | 不启动浏览器，仅附加到现有实例 | `false` |
| `browser.cdpPort` | Chrome DevTools Protocol 端口 | `18800` |

## 问题：「Chrome extension relay is running, but no tab is connected」

你正在使用 `chrome` 配置文件（扩展程序中继）。它需要 OpenClaw 浏览器扩展附加到一个活动标签页。

**解决方法：**

1. **使用托管浏览器**：`openclaw browser start --browser-profile openclaw`（或设置 `browser.defaultProfile: "openclaw"`）
2. **使用扩展程序中继**：安装扩展程序，打开一个标签页，点击 OpenClaw 扩展图标进行附加

**注意：**

- `chrome` 配置文件会尽可能使用**系统默认的 Chromium 浏览器**
- 本地 `openclaw` 配置文件会自动分配 `cdpPort`/`cdpUrl`；仅在连接远程 CDP 时才需要手动设置这些参数

## 常见问题排查表

| 症状 | 可能原因 | 检查方法 | 解决方案 |
|------|----------|----------|----------|
| CDP 启动失败 | snap 沙箱限制 | `which chromium` 显示 snap 路径 | 安装 Google Chrome |
| 浏览器启动后立即退出 | 权限问题 | 查看系统日志 | 添加 `--no-sandbox` |
| 端口被占用 | 其他进程占用 | `lsof -i :18800` | 终止占用进程或更换端口 |
| 扩展无法连接 | 中继服务未运行 | 检查 Gateway 状态 | 确保 Gateway 在本机运行 |

## 相关文档

- [浏览器工具](/tools/browser) - 浏览器控制完整指南
- [Chrome 扩展](/tools/chrome-extension) - 扩展程序安装和使用
- [浏览器登录](/tools/browser-login) - 手动登录站点
