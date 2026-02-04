---
summary: "打包脚本生成的 macOS 调试构建的签名步骤"
read_when:
  - 构建或签名 mac 调试构建
title: "macOS 签名"
---

# mac 签名（调试构建）

此应用通常从 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 构建，现在它：

- 设置稳定的调试 bundle 标识符：`ai.openclaw.mac.debug`
- 使用该 bundle id 写入 Info.plist（通过 `BUNDLE_ID=...` 覆盖）
- 调用 [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) 签名主二进制文件和应用包，以便 macOS 将每次重建视为相同的已签名包并保持 TCC 权限（通知、辅助功能、屏幕录制、麦克风、语音识别）。为了稳定的权限，使用真正的签名身份；临时签名是可选的且脆弱的（参见 [macOS 权限](/zh-cn/platforms/mac/permissions)）。
- 默认使用 `CODESIGN_TIMESTAMP=auto`；它为 Developer ID 签名启用可信时间戳。设置 `CODESIGN_TIMESTAMP=off` 跳过时间戳（离线调试构建）。
- 将构建元数据注入 Info.plist：`OpenClawBuildTimestamp`（UTC）和 `OpenClawGitCommit`（短哈希），以便关于面板可以显示构建、git 和调试/发布频道。
- **打包需要 Node 22+**：脚本运行 TS 构建和 Control UI 构建。
- 从环境读取 `SIGN_IDENTITY`。在你的 shell rc 中添加 `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（或你的 Developer ID Application 证书）以始终使用你的证书签名。临时签名需要通过 `ALLOW_ADHOC_SIGNING=1` 或 `SIGN_IDENTITY="-"` 显式选择加入（不推荐用于权限测试）。
- 签名后运行 Team ID 审计，如果应用包内任何 Mach-O 由不同的 Team ID 签名则失败。设置 `SKIP_TEAM_ID_CHECK=1` 跳过。

## 用法

```bash
# 从仓库根目录
scripts/package-mac-app.sh               # 自动选择身份；如果找不到则报错
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # 真正的证书
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # 临时（权限不会持久）
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # 显式临时（同样的警告）
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 仅开发的 Sparkle Team ID 不匹配解决方案
```

### 临时签名说明

使用 `SIGN_IDENTITY="-"`（临时）签名时，脚本会自动禁用**强化运行时**（`--options runtime`）。这是必要的，以防止应用在尝试加载不共享相同 Team ID 的嵌入式框架（如 Sparkle）时崩溃。临时签名也会破坏 TCC 权限持久性；参见 [macOS 权限](/zh-cn/platforms/mac/permissions) 了解恢复步骤。

## 关于页面的构建元数据

`package-mac-app.sh` 为包打上：

- `OpenClawBuildTimestamp`：打包时的 ISO8601 UTC
- `OpenClawGitCommit`：短 git 哈希（如果不可用则为 `unknown`）

关于选项卡读取这些键以显示版本、构建日期、git 提交，以及是否为调试构建（通过 `#if DEBUG`）。代码更改后运行打包器以刷新这些值。

## 为什么

TCC 权限与 bundle 标识符_和_代码签名绑定。带有变化 UUID 的未签名调试构建导致 macOS 在每次重建后忘记授权。签名二进制文件（默认临时）并保持固定的 bundle id/路径（`dist/OpenClaw.app`）在构建之间保持授权，与 VibeTunnel 方法匹配。
