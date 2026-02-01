---
summary: "CLI onboard 命令 - 引导配置向导"
read_when:
  - 首次配置 OpenClaw
  - 运行引导向导
  - 理解 onboard 流程
title: "onboard 命令"
---

# onboard 命令

运行交互式引导向导，完成 OpenClaw 的初始配置。

---

## 用法

```bash
openclaw onboard [选项]
```

---

## 选项

| 选项 | 说明 |
|------|------|
| `--install-daemon` | 安装后台服务 |
| `--reset` | 重置配置 |
| `--non-interactive` | 非交互模式 |
| `--workspace <路径>` | 指定工作区 |

---

## 示例

### 完整引导（推荐）

```bash
openclaw onboard --install-daemon
```

### 仅配置

```bash
openclaw onboard
```

### 非交互模式

```bash
openclaw onboard --non-interactive --workspace ~/workspace
```

---

## 引导流程

1. **模型/认证** - 选择 AI 提供商
2. **工作区** - 设置工作目录
3. **网关** - 配置端口和认证
4. **渠道** - 配置消息平台
5. **守护进程** - 安装后台服务
6. **健康检查** - 验证配置

---

## 相关文档

- [新手上路](/zh-CN/start/getting-started)
- [向导模式详解](/zh-CN/start/wizard)
