---
summary: "CLI 参考 - `openclaw security` (审计和修复常见安全问题)"
read_when:
  - 对配置/状态进行快速安全审计
  - 应用安全"修复"建议（chmod、收紧默认值）
title: "security"
---

# `openclaw security`

安全工具（审计 + 可选修复）。

## 为什么需要这个命令

安全是头等大事：

- **自动审计**：扫描常见安全配置问题
- **建议修复**：提供具体的修复建议
- **风险评估**：识别潜在的安全风险

## 相关文档

- 安全指南：[安全](/zh-cn/gateway/security)

## 审计

```bash
# 基础安全审计
openclaw security audit

# 深度审计
openclaw security audit --deep

# 审计并应用修复
openclaw security audit --fix
```

## 检查项目

### DM 会话隔离

审计会在以下情况发出警告：

- 多个 DM 发送者共享主会话

**建议**：

| 场景 | 推荐设置 |
|------|----------|
| 共享收件箱 | `session.dmScope="per-channel-peer"` |
| 多账号渠道的共享收件箱 | `session.dmScope="per-account-channel-peer"` |

### 小模型安全

审计会在以下情况发出警告：

- 使用小模型（≤300B 参数）
- 未启用沙箱
- 启用了 web/browser 工具

**风险**：小模型更容易受到提示注入攻击，结合网页浏览能力可能导致安全问题。

## 使用场景

### 定期安全检查

```bash
# 每周运行安全审计
openclaw security audit --deep
```

### 配置更改后

```bash
# 更改配置后检查安全影响
openclaw security audit
```

### 修复发现的问题

```bash
# 审计并应用建议的修复
openclaw security audit --fix
```

## 安全最佳实践

### 1. DM 会话隔离

```jsonc
{
  "session": {
    "dmScope": "per-channel-peer"
  }
}
```

### 2. 沙箱配置

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main"  // 或 "all" 以获得最大安全性
      }
    }
  }
}
```

### 3. 工具策略

限制敏感工具的访问：

```jsonc
{
  "tools": {
    "browser": {
      "enabled": false  // 除非必要否则禁用
    }
  }
}
```

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 审计警告太多 | 逐个解决，从高风险开始 |
| 修复后仍有警告 | 某些警告可能需要手动配置 |
| 不确定是否应用修复 | 先运行不带 `--fix` 查看建议 |

## 相关命令

| 命令 | 用途 |
|------|------|
| `openclaw security audit` | 安全审计 |
| `openclaw doctor` | 通用诊断检查 |
| `openclaw sandbox explain` | 解释当前沙箱配置 |
