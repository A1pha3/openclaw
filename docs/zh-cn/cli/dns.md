---
summary: "`openclaw dns` 命令参考（广域发现辅助）"
read_when:
  - 想要通过 Tailscale + CoreDNS 实现广域发现（DNS-SD）
  - 正在为自定义发现域名设置分裂 DNS（例如：openclaw.internal）
title: "dns"
---

# `openclaw dns`

广域发现的 DNS 辅助工具（Tailscale + CoreDNS）。目前专注于 macOS + Homebrew CoreDNS。

## 为什么需要 DNS 辅助

广域发现允许你的设备跨网络找到网关：

- **跨网络发现**：不在同一局域网也能发现网关
- **自定义域名**：使用类似 `gateway.openclaw.internal` 的友好名称
- **Tailscale 集成**：利用 Tailscale 网络进行安全发现

## 相关链接

- 网关发现：[Discovery](/zh-cn/gateway/discovery)
- 广域发现配置：[Configuration](/zh-cn/gateway/configuration)

## 设置

```bash
# 预览设置（不应用更改）
openclaw dns setup

# 应用设置
openclaw dns setup --apply
```

## 工作原理

`dns setup` 命令会：

1. 检测你的 Tailscale 网络配置
2. 生成 CoreDNS 配置
3. 配置分裂 DNS 以解析自定义发现域名
4. 设置必要的系统配置（如果使用 `--apply`）

## 前置条件

- **macOS**：当前仅支持 macOS
- **Homebrew CoreDNS**：`brew install coredns`
- **Tailscale**：已安装并配置 Tailscale

## 配置示例

设置后，你可以使用自定义域名访问网关：

```bash
# 使用友好名称而非 IP
openclaw tui --url ws://gateway.openclaw.internal:18789
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| CoreDNS 未找到 | 未安装 | `brew install coredns` |
| Tailscale 未检测到 | 未运行或未配置 | 检查 Tailscale 状态 |
| 域名无法解析 | DNS 缓存 | 刷新 DNS 缓存 |
| 设置失败 | 权限不足 | 检查系统权限 |

## 高级配置

如果需要自定义发现域名，可以在配置文件中设置：

```json5
{
  gateway: {
    discovery: {
      domain: "myclaw.internal",
    },
  },
}
```

然后重新运行 `openclaw dns setup --apply`。
