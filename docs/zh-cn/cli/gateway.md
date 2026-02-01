---
summary: "CLI gateway 命令 - 网关控制"
read_when:
  - 启动/停止网关
  - 管理网关服务
  - 网关运维
title: "gateway 命令"
---

# gateway 命令

控制 OpenClaw 网关服务。

---

## 用法

```bash
openclaw gateway <子命令>
```

---

## 子命令

| 子命令 | 说明 |
|--------|------|
| `run` | 前台运行 |
| `start` | 启动服务 |
| `stop` | 停止服务 |
| `restart` | 重启服务 |
| `status` | 查看状态 |

---

## 示例

### 前台运行（调试）

```bash
openclaw gateway run --verbose
```

### 服务管理

```bash
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway status
```

---

## 相关文档

- [网关概述](/zh-CN/gateway)
- [故障排除](/zh-CN/help/troubleshooting)
