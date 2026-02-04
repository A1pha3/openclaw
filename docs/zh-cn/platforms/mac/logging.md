---
summary: "OpenClaw 日志系统：滚动诊断文件日志 + 统一日志隐私标志"
read_when:
  - 捕获 macOS 日志或调查私有数据日志问题
  - 调试语音唤醒/会话生命周期问题
title: "macOS 日志"
---

# 日志系统 (macOS)

## 滚动诊断文件日志（调试面板）

OpenClaw 通过 swift-log 路由 macOS 应用日志（默认使用统一日志），并可在需要持久化捕获时将滚动文件日志写入磁盘。

- **详细程度**: 调试面板 → Logs → App logging → Verbosity
- **启用**: 调试面板 → Logs → App logging → "Write rolling diagnostics log (JSONL)"
- **位置**: `~/Library/Logs/OpenClaw/diagnostics.jsonl`（自动轮转；旧文件后缀为 `.1`、`.2` 等）
- **清除**: 调试面板 → Logs → App logging → "Clear"

注意事项：

- 此功能**默认关闭**。仅在主动调试时启用。
- 将此文件视为敏感信息；分享前请先审查内容。

## macOS 统一日志的私有数据

统一日志会隐藏大多数有效负载，除非子系统选择 `privacy -off`。根据 Peter 关于 macOS [日志隐私机制](https://steipete.me/posts/2025/logging-privacy-shenanigans)（2025）的文章，这是通过 `/Library/Preferences/Logging/Subsystems/` 中按子系统名称命名的 plist 文件控制的。只有新的日志条目才会采用该标志，因此请在复现问题之前启用它。

## 为 OpenClaw 启用（`bot.molt`）

先将 plist 写入临时文件，然后以 root 身份原子安装：

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

- 无需重启；logd 会很快检测到该文件，但只有新的日志行才会包含私有有效负载。
- 使用现有的辅助脚本查看更详细的输出，例如 `./scripts/clawlog.sh --category WebChat --last 5m`。

## 调试后禁用

- 移除覆盖配置：`sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`
- 可选：运行 `sudo log config --reload` 强制 logd 立即丢弃覆盖配置。
- 请记住此表面可能包含电话号码和消息正文；仅在确实需要额外详细信息时保留该 plist 文件。

## 为什么需要这样做

| 场景 | 说明 |
|------|------|
| 调试语音唤醒问题 | 需要查看完整的语音识别结果和触发事件 |
| 追踪消息流 | 需要查看消息内容以定位丢失或延迟 |
| WebChat 问题 | 需要查看 WebSocket 通信的完整载荷 |
| 会话生命周期问题 | 需要查看会话创建/销毁的详细上下文 |

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 日志仍显示 `<private>` | 确保 plist 安装正确后，重新触发要调试的事件 |
| 日志文件增长过快 | 完成调试后及时禁用滚动日志 |
| 找不到 clawlog.sh | 脚本位于仓库根目录的 `scripts/` 文件夹 |
| 权限被拒绝 | 使用 `sudo` 安装 plist 文件 |
