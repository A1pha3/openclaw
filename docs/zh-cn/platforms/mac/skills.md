---
summary: "macOS Skills 设置界面及网关驱动的状态"
read_when:
  - 更新 macOS Skills 设置界面
  - 修改技能门控或安装行为
title: "技能"
---

# 技能 (macOS)

macOS 应用通过网关显示 OpenClaw 技能；它不会在本地解析技能。

## 数据来源

- `skills.status`（网关）返回所有技能及其资格和缺失的要求
  （包括内置技能的白名单限制）。
- 要求从每个 `SKILL.md` 中的 `metadata.openclaw.requires` 派生。

## 安装操作

- `metadata.openclaw.install` 定义安装选项（brew/node/go/uv）。
- 应用调用 `skills.install` 在网关主机上运行安装程序。
- 当提供多个安装器时，网关仅显示一个首选安装器
  （如果可用则使用 brew，否则使用来自 `skills.install` 的节点管理器，默认为 npm）。

## 环境变量/API 密钥

- 应用将密钥存储在 `~/.openclaw/openclaw.json` 的 `skills.entries.<skillKey>` 下。
- `skills.update` 更新 `enabled`、`apiKey` 和 `env`。

## 远程模式

- 安装 + 配置更新发生在网关主机上（而非本地 Mac）。

## 为什么这样设计

| 设计决策 | 原因 |
|----------|------|
| 网关集中管理 | 确保所有客户端看到一致的技能状态 |
| 远程安装 | 支持在无头服务器上运行网关 |
| 单一首选安装器 | 简化用户选择，避免混淆 |
| 密钥集中存储 | 方便备份和迁移，避免分散管理 |

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 技能显示"不可用" | 检查 `skills.status` 返回的 `missing` 字段 |
| 安装失败 | 查看网关日志，确认依赖项是否满足 |
| API 密钥不生效 | 确认密钥存储在正确的配置路径下 |
| 远程模式下安装位置错误 | 记住安装发生在网关主机上，而非 Mac |
