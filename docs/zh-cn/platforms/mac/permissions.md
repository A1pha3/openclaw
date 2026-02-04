---
summary: "macOS 权限持久化（TCC）和签名要求"
read_when:
  - 调试缺失或卡住的 macOS 权限提示
  - 打包或签名 macOS 应用
  - 更改 bundle ID 或应用安装路径
title: "macOS 权限"
---

# macOS 权限（TCC）

macOS 权限授予是脆弱的。TCC 将权限授予与应用的代码签名、bundle 标识符和磁盘路径关联。如果其中任何一个改变，macOS 会将应用视为新应用，可能会删除或隐藏提示。

## 稳定权限的要求

- 相同路径：从固定位置运行应用（对于 OpenClaw，是 `dist/OpenClaw.app`）。
- 相同 bundle 标识符：更改 bundle ID 会创建新的权限身份。
- 已签名应用：未签名或临时签名的构建不会持久化权限。
- 一致的签名：使用真正的 Apple Development 或 Developer ID 证书，以便签名在重建之间保持稳定。

临时签名每次构建都会生成新身份。macOS 会忘记之前的授权，提示可能完全消失，直到清除过期条目。

## 提示消失时的恢复清单

1. 退出应用。
2. 在系统设置 -> 隐私与安全中移除应用条目。
3. 从相同路径重新启动应用并重新授予权限。
4. 如果提示仍然不出现，使用 `tccutil` 重置 TCC 条目并重试。
5. 某些权限只有在完全重启 macOS 后才会重新出现。

重置示例（根据需要替换 bundle ID）：

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

如果你在测试权限，始终使用真正的证书签名。临时构建仅适用于不关心权限的快速本地运行。

## 常见权限问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 权限提示不出现 | TCC 缓存过期条目 | 使用 tccutil 重置 |
| 每次重建后权限丢失 | 临时签名 | 使用开发者证书签名 |
| 应用路径更改后权限失效 | TCC 绑定路径 | 保持固定安装位置 |
| 签名后权限仍不持久 | 证书问题 | 检查证书有效性 |
