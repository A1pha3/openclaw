---
summary: "OpenClaw macOS 发布清单（Sparkle 订阅源、打包、签名）"
read_when:
  - 发布或验证 OpenClaw macOS 版本
  - 更新 Sparkle appcast 或订阅源资产
title: "macOS 发布"
---

# OpenClaw macOS 发布 (Sparkle)

此应用现已支持 Sparkle 自动更新。发布版本必须使用 Developer ID 签名、压缩，并发布带有签名的 appcast 条目。

## 前提条件

- 已安装 Developer ID Application 证书（示例：`Developer ID Application: <开发者名称> (<TEAMID>)`）。
- Sparkle 私钥路径已通过环境变量 `SPARKLE_PRIVATE_KEY_FILE` 设置（指向您的 Sparkle ed25519 私钥；公钥已嵌入 Info.plist）。如果缺失，请检查 `~/.profile`。
- 公证凭据（钥匙串配置文件或 API 密钥）用于 `xcrun notarytool`，如果您想要 Gatekeeper 安全的 DMG/zip 分发。
  - 我们使用名为 `openclaw-notary` 的钥匙串配置文件，通过 shell 配置文件中的 App Store Connect API 密钥环境变量创建：
    - `APP_STORE_CONNECT_API_KEY_P8`、`APP_STORE_CONNECT_KEY_ID`、`APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- 已安装 `pnpm` 依赖项（`pnpm install --config.node-linker=hoisted`）。
- Sparkle 工具通过 SwiftPM 自动获取，位于 `apps/macos/.build/artifacts/sparkle/Sparkle/bin/`（`sign_update`、`generate_appcast` 等）。

## 构建与打包

注意事项：

- `APP_BUILD` 映射到 `CFBundleVersion`/`sparkle:version`；保持数字 + 单调递增（不带 `-beta`），否则 Sparkle 会将其比较为相等。
- 默认为当前架构（`$(uname -m)`）。对于发布/通用构建，设置 `BUILD_ARCHS="arm64 x86_64"`（或 `BUILD_ARCHS=all`）。
- 使用 `scripts/package-mac-dist.sh` 生成发布产物（zip + DMG + 公证）。使用 `scripts/package-mac-app.sh` 进行本地/开发打包。

```bash
# 从仓库根目录；设置发布 ID 以启用 Sparkle 订阅源。
# APP_BUILD 必须是数字 + 单调递增以供 Sparkle 比较。
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <开发者名称> (<TEAMID>)" \
scripts/package-mac-app.sh

# 压缩用于分发（包含资源分叉以支持 Sparkle 增量更新）
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.zip

# 可选：同时构建人性化的 DMG（拖拽到 /Applications）
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.dmg

# 推荐：构建 + 公证/盖章 zip + DMG
# 首先，创建一次钥匙串配置文件：
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <开发者名称> (<TEAMID>)" \
scripts/package-mac-dist.sh

# 可选：随发布一起提供 dSYM
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.1.27-beta.1.dSYM.zip
```

## Appcast 条目

使用发布说明生成器，以便 Sparkle 渲染格式化的 HTML 说明：

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

从 `CHANGELOG.md` 生成 HTML 发布说明（通过 [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)）并将其嵌入 appcast 条目。
发布时将更新的 `appcast.xml` 与发布资产（zip + dSYM）一起提交。

## 发布与验证

- 将 `OpenClaw-2026.1.27-beta.1.zip`（和 `OpenClaw-2026.1.27-beta.1.dSYM.zip`）上传到标签 `v2026.1.27-beta.1` 的 GitHub 发布。
- 确保原始 appcast URL 与嵌入的订阅源匹配：`https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`。
- 健全性检查：
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` 返回 200。
  - `curl -I <enclosure url>` 在资产上传后返回 200。
  - 在之前的公开版本上，从"关于"标签运行"检查更新..."并验证 Sparkle 能顺利安装新版本。

完成定义：已签名的应用 + appcast 已发布，从旧版本安装的更新流程正常工作，发布资产已附加到 GitHub 发布。

## 发布清单速查

| 步骤 | 命令/操作 |
|------|-----------|
| 1. 构建应用 | `scripts/package-mac-dist.sh` |
| 2. 生成 appcast | `scripts/make_appcast.sh` |
| 3. 上传资产 | GitHub Release 页面 |
| 4. 验证 appcast | `curl -I <appcast-url>` |
| 5. 测试更新 | 从旧版本检查更新 |
