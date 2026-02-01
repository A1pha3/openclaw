---
summary: "部署平台 - macOS、Linux、Windows、Docker 等"
read_when:
  - 选择部署平台
  - 平台特定配置
  - 了解支持的平台
title: "部署平台"
---

# 💻 部署平台

OpenClaw 支持多种平台和部署方式。

---

## 🎯 支持的平台

### 操作系统

| 平台 | 支持度 | 说明 |
|------|--------|------|
| **macOS** | ⭐⭐⭐⭐⭐ | 原生支持，功能最全 |
| **Linux** | ⭐⭐⭐⭐⭐ | 服务器首选 |
| **Windows (WSL2)** | ⭐⭐⭐ | 需要 WSL2 |
| **iOS** | ⭐⭐⭐ | 作为节点 |
| **Android** | ⭐⭐⭐ | 作为节点 |

### 云服务

| 平台 | 特点 |
|------|------|
| **Docker** | 容器化，易于部署 |
| **Fly.io** | 边缘部署 |
| **Hetzner** | 性价比高 |
| **DigitalOcean** | 简单易用 |

---

## 🚀 快速开始

### macOS

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Linux

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Docker

```bash
docker run -p 18789:18789 openclaw/openclaw
```

---

## 🔧 平台特定说明

### macOS
- 支持 iMessage
- 推荐 macOS 14+

### Linux
- 支持 systemd
- 推荐 Ubuntu 22.04+

### Windows
- 必须使用 WSL2
- 不支持原生 Windows

---

## 📖 相关文档

- [安装指南](/zh-CN/install)
- [部署指南](/zh-CN/operations/deployment)
- [Docker 安装](/zh-CN/install/docker)
