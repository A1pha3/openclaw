---
summary: "CLI status 命令 - 查看系统状态"
read_when:
  - 检查系统状态
  - 诊断问题
  - 查看运行信息
title: "status 命令"
---

# status 命令

查看 OpenClaw 系统状态。

---

## 用法

```bash
openclaw status [选项]
```

---

## 选项

| 选项 | 说明 |
|------|------|
| `--all` | 显示完整状态 |
| `--deep` | 深度检查 |

---

## 示例

### 快速状态

```bash
openclaw status
```

### 完整状态

```bash
openclaw status --all
```

### 深度检查

```bash
openclaw status --deep
```

---

## 相关文档

- [故障排除](/zh-CN/help/troubleshooting)
