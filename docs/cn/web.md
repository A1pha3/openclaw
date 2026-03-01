---
summary: "OpenClaw Web 界面完全指南：浏览器控制界面、仪表盘、配置管理和远程访问"
read_when:
  - 你需要使用 Web 界面管理 OpenClaw
  - 你想了解 Web 界面的功能和配置
  - 你在配置远程访问或 Web 安全
  - 你在使用 WebChat 或控制 UI
title: "Web 界面"
---

# 🖥️ OpenClaw Web 界面完全指南

> **学习目标**：完成本章节学习后，你将能够理解 OpenClaw Web 界面的架构和功能，掌握从本地访问到远程配置的配置方法，理解 Web 安全机制，并能够高效使用 Web 界面进行日常管理和监控。

---

## 为什么要使用 Web 界面

在深入技术细节之前，我们需要先理解**Web 界面在 OpenClaw 系统中的定位**。虽然 CLI 提供了完整的控制能力，但 Web 界面为日常操作、监控和管理提供了更直观、更高效的方式。

### Web 界面的核心价值

```
┌─────────────────────────────────────────────────────────────────┐
│                    Web 界面的核心价值                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. 可视化管理                                                │
│      - 直观的配置编辑器                                         │
│      - 实时的状态监控                                           │
│      - 易于理解的数据展示                                        │
│                                                                  │
│   2. 快速访问                                                  │
│      - 无需安装额外应用                                         │
│      - 跨平台访问（任何浏览器）                                   │
│      - 移动设备友好                                             │
│                                                                  │
│   3. 协作支持                                                  │
│      - 多人可以同时访问                                         │
│      - 便于团队协作管理                                         │
│                                                                  │
│   4. 开发友好                                                  │
│      - 实时查看日志                                             │
│      - 调试工具集成                                             │
│      - 快速配置变更                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Web 界面架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                      Web 界面架构                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                    Gateway (控制平面)                     │  │
│   │   - 提供 Web 服务                                         │  │
│   │   - WebSocket 实时通信                                   │  │
│   │   - REST API 接口                                        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                    Web 表面                               │  │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │  │
│   │   │  控制 UI    │  │  仪表盘     │  │  WebChat   │   │  │
│   │   │ (管理界面)  │  │ (状态监控)  │  │ (聊天界面)  │   │  │
│   │   └─────────────┘  └─────────────┘  └─────────────┘   │  │
│   └─────────────────────────────────────────────────────────┘  │
│                              │                                   │
│              ┌───────────────┼───────────────┐                 │
│              ▼               ▼               ▼                  │
│        本地访问        远程访问        API 访问                │
│       (loopback)      (Tailscale)      (REST)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 认知负荷管理：学习路径设计

```
Web 界面技能金字塔：

                    ┌─────────────────────────┐
                    │   专家级：安全与扩展   │  自定义 API、插件集成
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   高级：远程访问       │  Tailscale、SSL 配置
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   中级：高级功能       │  API 集成、插件
                    └─────────────────────────┘
                         ▲
                    ┌─────────────────────────┐
                    │   初级：基础使用       │  访问、导航、基本操作
                    └─────────────────────────┘
```

**建议学习路径**：
- 新手：从本地访问开始，熟悉界面布局
- 用户：掌握配置管理和状态监控
- 管理员：深入远程访问和安全配置
- 开发者：学习 API 集成和扩展

---

## 第一部分：基础使用（⭐ 入门级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 访问 Web 界面
- [ ] 理解界面的基本布局
- [ ] 执行基本的操作
- [ ] 查看系统状态

### 1.1 访问 Web 界面

**本地访问**：

```
访问地址：http://127.0.0.1:18789/
```

**前提条件**：

```bash
# 确保 Gateway 正在运行
openclaw status

# 查看 Gateway 状态
# 应显示：Gateway running on port 18789
```

**打开仪表盘**：

```bash
# 使用 CLI 命令打开（推荐）
openclaw dashboard

# 或直接在浏览器中输入
http://127.0.0.1:18789/
```

### 1.2 界面布局

**主要功能区域**：

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw Web 界面布局                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  顶部导航栏                                               │  │
│   │  [Dashboard] [Chat] [Settings] [Logs] [Status]          │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   ┌──────────────────────────┬──────────────────────────────┐ │
│   │                          │                              │ │
│   │      主内容区域          │      侧边栏/面板              │ │
│   │                          │                              │ │
│   │   - 配置编辑器           │   - 快速操作                 │ │
│   │   - 聊天界面            │   - 系统状态                 │ │
│   │   - 日志查看            │   - 帮助信息                 │ │
│   │                          │                              │ │
│   │                          │                              │ │
│   └──────────────────────────┴──────────────────────────────┘ │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  底部状态栏                                               │  │
│   │   Gateway: 运行中 | 端口: 18789 | 版本: v3.0.0          │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 核心功能

**仪表盘（Dashboard）**：

| 功能 | 说明 | 使用频率 |
|------|------|----------|
| 系统概览 | 显示 CPU、内存、连接数 | 高 |
| 渠道状态 | 各渠道的连接状态 | 高 |
| 会话统计 | 活跃会话数、消息统计 | 中 |
| 资源监控 | 系统资源使用情况 | 中 |

**控制 UI（Control UI）**：

| 功能 | 说明 | 使用频率 |
|------|------|----------|
| 配置编辑器 | 可视化编辑配置 | 高 |
| 插件管理 | 启用/禁用插件 | 低 |
| 用户管理 | 用户权限配置 | 低 |

**WebChat**：

| 功能 | 说明 | 使用频率 |
|------|------|----------|
| 聊天界面 | 与 AI 助手对话 | 高 |
| 消息历史 | 查看历史对话 | 中 |
| 附件管理 | 上传/下载文件 | 中 |

---

## 第二部分：高级配置（⭐⭐ 进阶级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 配置 Web 界面的认证
- [ ] 理解 Web 界面的安全机制
- [ ] 配置访问控制
- [ ] 自定义界面选项

### 2.1 Web 配置详解

**基本配置**：

```json5
{
  gateway: {
    // Web 服务端口
    port: 18789,
    
    // 绑定地址（生产环境建议 loopback + 远程访问方案）
    bind: "loopback",
    
    // 认证配置
    auth: {
      type: "token",  // token、password、none
      token: "${GATEWAY_TOKEN}"
    }
  },
  
  // Web 界面特定配置
  web: {
    // 界面主题
    theme: "auto",  // auto、light、dark
    
    // 语言设置
    language: "zh-CN",
    
    // 自定义标题
    title: "OpenClaw Dashboard",
    
    // 快捷键
    shortcuts: {
      enabled: true
    }
  }
}
```

**认证类型配置**：

```json5
{
  gateway: {
    auth: {
      // 方式一：Token 认证（推荐）
      type: "token",
      token: "your-secure-token"
    }
  }
}
```

```json5
{
  gateway: {
    auth: {
      // 方式二：密码认证
      type: "password",
      username: "admin",
      passwordHash: "$2b$10$..."  // bcrypt 哈希
    }
  }
}
```

```json5
{
  gateway: {
    auth: {
      // 方式三：无认证（仅限本地开发）
      type: "none"
    }
  }
}
```

### 2.2 访问控制

**IP 白名单**：

```json5
{
  web: {
    accessControl: {
      // 启用 IP 白名单
      enabled: true,
      
      // 允许的 IP 列表
      allowList: [
        "127.0.0.1",
        "192.168.1.0/24",
        "10.0.0.0/8"
      ],
      
      // 拒绝的 IP 列表
      denyList: [
        "0.0.0.0"  // 拒绝所有公网 IP
      ]
    }
  }
}
```

**CORS 配置**：

```json5
{
  web: {
    cors: {
      enabled: true,
      
      // 允许的来源
      origins: [
        "http://localhost:3000",
        "https://myapp.example.com"
      ],
      
      // 允许的方法
      methods: ["GET", "POST", "PUT", "DELETE"],
      
      // 允许的头部
      headers: ["Content-Type", "Authorization"]
    }
  }
}
```

### 2.3 自定义选项

**界面定制**：

```json5
{
  web: {
    ui: {
      // 侧边栏配置
      sidebar: {
        collapsed: false,
        pinned: true
      },
      
      // 主题配置
      theme: {
        primaryColor: "#FF5A36",
        compactMode: false,
        animations: true
      },
      
      // 仪表盘布局
      dashboard: {
        widgets: [
          { id: "status", enabled: true, position: 0 },
          { id: "channels", enabled: true, position: 1 },
          { id: "logs", enabled: true, position: 2 },
          { id: "metrics", enabled: false, position: 3 }
        ]
      }
    }
  }
}
```

---

## 第三部分：远程访问（⭐⭐⭐ 高级）

### 学习目标

完成本节学习后，你将能够：
- [ ] 配置 Tailscale 远程访问
- [ ] 理解不同远程访问方案
- [ ] 实施安全的远程访问
- [ ] 解决远程访问问题

### 3.1 远程访问方案对比

| 方案 | 安全性 | 易用性 | 适用场景 |
|------|--------|--------|----------|
| **Tailscale Serve** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 长期远程访问 |
| **Tailscale Funnel** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 公开访问 |
| **SSH 隧道** | ⭐⭐⭐⭐ | ⭐⭐⭐ | 临时访问 |
| **VPN** | ⭐⭐⭐⭐⭐ | ⭐⭐ | 企业部署 |

### 3.2 Tailscale 集成

**Serve 模式配置**：

```json5
{
  gateway: {
    // 必须保持 loopback
    bind: "loopback",
    
    tailscale: {
      mode: "serve",
      resetOnExit: true
    },
    
    auth: {
      // Serve 模式可以使用 token 认证
      type: "token",
      token: "${GATEWAY_TOKEN}"
    }
  }
}
```

**Funnel 模式配置**：

```json5
{
  gateway: {
    bind: "loopback",
    
    tailscale: {
      mode: "funnel",
      resetOnExit: true
    },
    
    auth: {
      // Funnel 强制要求密码认证
      type: "password",
      username: "admin",
      passwordHash: "$2b$10$..."
    }
  }
}
```

**访问方式**：

```
# Serve 模式（tailnet 内）
https://your-machine.tailnet.ts.net/

# Funnel 模式（公网访问）
https://openclaw.your-domain.ts.net/
```

### 3.3 安全最佳实践

```
┌─────────────────────────────────────────────────────────────┐
│                  Web 界面安全 Checklist                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ✅ 基础安全                                                 │
│      □ 使用强认证（token 或 password）                       │
│      □ 启用 HTTPS（在 Tailscale 模式下自动启用）             │
│      □ 定期轮换访问令牌                                      │
│                                                              │
│   ✅ 访问控制                                                 │
│      □ 配置 IP 白名单（如果适用）                            │
│      □ 限制 API 访问频率                                     │
│      □ 启用审计日志                                          │
│                                                              │
│   ✅ 生产环境                                                 │
│      □ 避免使用 auth.type = "none"                          │
│      □ 使用 Tailscale 或 VPN 而非 公网暴露                   │
│      □ 配置适当的 CORS 策略                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 第四部分：故障排查（⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 诊断常见的 Web 界面问题
- [ ] 解决认证问题
- [ ] 修复加载问题
- [ ] 优化性能

### 4.1 常见问题与解决方案

**问题一：无法访问 Web 界面**

**排查步骤**：

```bash
# 1. 检查 Gateway 是否运行
openclaw status

# 2. 检查端口占用
ss -ltnp | grep 18789

# 3. 检查防火墙
sudo ufw status

# 4. 测试本地连接
curl http://127.0.0.1:18789/
```

**问题二：认证失败**

**排查步骤**：

```bash
# 1. 检查认证配置
openclaw config get gateway.auth

# 2. 重置 Token（如果使用 token 认证）
openclaw gateway token --regenerate

# 3. 查看认证日志
openclaw logs --follow | grep -i auth
```

**问题三：页面加载缓慢**

**排查步骤**：

```bash
# 1. 检查系统资源
htop

# 2. 查看 Gateway 日志
openclaw logs --follow | grep -i slow

# 3. 检查网络延迟（如果是远程访问）
curl -I https://your-machine.tailnet.ts.net/
```

### 4.2 性能优化

**优化建议**：

| 问题 | 解决方案 |
|------|----------|
| 页面加载慢 | 启用 gzip 压缩 |
| API 响应慢 | 减少实时数据更新频率 |
| 内存占用高 | 限制日志缓存大小 |
| 大量并发连接 | 增加 WebSocket 连接池 |

**配置示例**：

```json5
{
  web: {
    performance: {
      // 启用压缩
      compression: {
        enabled: true,
        level: 6
      },
      
      // API 缓存
      cache: {
        enabled: true,
        ttl: 30000  // 30 秒
      },
      
      // WebSocket 配置
      websocket: {
        pingInterval: 30000,
        maxConnections: 100
      }
    }
  }
}
```

---

## 第五部分：API 参考（⭐⭐⭐⭐）

### 学习目标

完成本节学习后，你将能够：
- [ ] 理解 Web API 的结构
- [ ] 使用 REST API 进行集成
- [ ] 实现自定义集成
- [ ] 自动化管理任务

### 5.1 REST API 概览

**API 端点**：

| 方法 | 端点 | 说明 | 认证 |
|------|------|------|------|
| GET | `/api/status` | 获取系统状态 | 可选 |
| GET | `/api/channels` | 列出渠道 | 必需 |
| POST | `/api/agent/send` | 发送消息 | 必需 |
| GET | `/api/logs` | 获取日志 | 必需 |
| GET | `/api/config` | 获取配置 | 必需 |
| PUT | `/api/config` | 更新配置 | 必需 |

**认证方式**：

```bash
# Token 认证
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://127.0.0.1:18789/api/status

# 或者通过查询参数
curl http://127.0.0.1:18789/api/status?token=YOUR_TOKEN
```

### 5.2 API 使用示例

**获取系统状态**：

```bash
curl http://127.0.0.1:18789/api/status

# 响应
{
  "status": "running",
  "version": "3.0.0",
  "uptime": 3600,
  "memory": {
    "heapUsed": 1024000,
    "heapTotal": 2048000
  },
  "connections": {
    "active": 5,
    "total": 150
  }
}
```

**发送消息**：

```bash
curl -X POST http://127.0.0.1:18789/api/agent/send \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Hello, OpenClaw!",
    "channel": "telegram"
  }'

# 响应
{
  "status": "queued",
  "messageId": "msg_123456"
}
```

**获取配置**：

```bash
curl http://127.0.0.1:18789/api/config \
  -H "Authorization: Bearer YOUR_TOKEN"

# 响应
{
  "gateway": {
    "port": 18789,
    "bind": "loopback"
  },
  "channels": {
    "telegram": {
      "enabled": true
    }
  }
}
```

---

## 练习与自检

### 基础技能自检

- [ ] 能够访问 Web 界面
- [ ] 理解界面基本布局
- [ ] 能够查看系统状态
- [ ] 掌握基本操作方法

### 进阶技能自检

- [ ] 能够配置认证方式
- [ ] 理解访问控制机制
- [ ] 能够配置 Tailscale 远程访问
- [ ] 掌握安全最佳实践

### 专家技能自检

- [ ] 能够使用 REST API
- [ ] 理解 API 集成方法
- [ ] 能够实现自动化任务
- [ ] 掌握性能优化技巧

---

## 参考资料

### 核心文档

- [网络配置](/zh-CN/network) - 网络架构详解
- [安全指南](/zh-CN/gateway/security) - 安全最佳实践
- [部署指南](/zh-CN/operations/deployment) - 部署配置

### 相关命令

```bash
# Web 界面命令
openclaw dashboard     # 打开仪表盘
openclaw status       # 查看状态
openclaw logs --follow  # 查看日志

# Gateway 管理
openclaw gateway start    # 启动
openclaw gateway stop     # 停止
openclaw gateway restart  # 重启
```

### 扩展阅读

- [RESTful API 设计](https://restfulapi.net/)
- [WebSocket 协议](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [Tailscale 文档](https://tailscale.com/docs/)
