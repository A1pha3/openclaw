---
summary: "Zalo Personal 插件：通过 zca-cli 进行 QR 登录 + 消息传递（插件安装 + 通道配置 + CLI + 工具）"
read_when:
  - 您想在 OpenClaw 中支持 Zalo Personal（unofficial）
  - 您正在配置或开发 zalouser 插件
title: "Zalo Personal 插件"
---

# Zalo Personal（插件）

通过插件为 OpenClaw 提供 Zalo Personal 支持，使用 `zca-cli` 来自动化正常的 Zalo 用户账户。

> **警告：** 非官方自动化可能导致账户暂停/封禁。使用风险自负。

## 命名

通道 ID 是 `zalouser`，以明确这是自动化**个人 Zalo 用户账户**（非官方）。我们将 `zalo` 保留给未来可能的官方 Zalo API 集成。

## 运行位置

此插件在**网关进程内部**运行。

如果您使用远程网关，请在**运行网关的机器上**安装/配置它，然后重启网关。

## 安装

### 选项 A：从 npm 安装

```bash
openclaw plugins install @openclaw/zalouser
```

之后重启网关。

### 选项 B：从本地文件夹安装（开发）

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

之后重启网关。

## 前置条件：zca-cli

网关机器必须在 `PATH` 上有 `zca`：

```bash
zca --version
```

## 配置

通道配置位于 `channels.zalouser`（而不是 `plugins.entries.*`）：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## 代理工具

工具名称：`zalouser`

操作：`send`、`image`、`link`、`friends`、`groups`、`me`、`status`
