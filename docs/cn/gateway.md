---
summary: "Gateway 网关完整指南——OpenClaw 核心控制中心的架构、配置和运维，包括 WebSocket 服务、认证管理、远程访问和高可用部署"
read_when:
  - 理解 Gateway 网关的核心作用和架构
  - 配置 Gateway 服务和网络设置
  - 设置远程访问和安全认证
  - 实现 Gateway 的高可用部署
title: "网关概述"
---

# 🌐 Gateway 网关

Gateway（网关）是 OpenClaw 系统的**核心控制中枢**，所有消息渠道、AI 代理、工具执行和客户端连接都通过 Gateway 进行协调和管理。理解 Gateway 的工作原理对于正确部署、配置和维护 OpenClaw 系统至关重要。本文将深入介绍 Gateway 的架构设计、配置选项、运维实践和故障排查，帮助你建立对这一核心组件的全面认知。

> **Gateway 的核心定位**
>
> 如果把 OpenClaw 比作一个交响乐团，那么 Gateway 就是乐团的指挥台。各种消息渠道（WhatsApp、Telegram、Slack 等）是不同的乐器组，AI 代理是演奏乐章的音乐家，工具系统是乐谱架上的各种乐谱——而 Gateway 负责协调所有的输入输出，确保每个部分在正确的时间发出正确的声音。没有 Gateway，就无法实现多渠道的协调工作，也无法实现复杂的任务调度。

---

## 🎯 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 Gateway 的架构设计和工作流程
- 完成 Gateway 的基础安装和配置
- 启动和停止 Gateway 服务
- 理解认证和安全模型

### 进阶目标（建议掌握）

- 配置高级网络选项和远程访问
- 实现多实例部署和负载均衡
- 配置监控和日志系统
- 优化性能和资源使用

### 专家目标（挑战）

- 开发自定义 Gateway 扩展
- 实现分布式架构设计
- 设计企业级安全策略
- 构建全球化部署方案

---

## 🗺️ 学习路径

根据你的经验水平和需求，选择最适合的学习路径：

| 学习路径 | 适合人群 | 预计时间 | 核心内容 |
|----------|----------|----------|----------|
| **快速上手** | 首次部署 Gateway | 30 分钟 | 启动服务，完成基本配置 |
| **日常运维** | 普通用户日常管理 | 1 小时 | 状态查看、日志分析 |
| **高级配置** | 需要深度定制 | 2 小时 | 远程访问、安全配置 |
| **生产部署** | 生产环境运维 | 4 小时 | 高可用、监控、备份 |

---

## 🧠 原理解释：Gateway 架构

### Gateway 的核心职责

Gateway 在 OpenClaw 系统中承担以下核心职责：

**消息路由中枢**

Gateway 接收来自所有渠道的消息，根据配置的路由规则将消息分发到对应的 AI 代理。代理处理完成后，Gateway 负责将响应发送回正确的渠道。这种集中式的消息路由使得多渠道管理变得简单统一。

**连接管理器**

Gateway 维护与所有渠道服务和客户端的持久连接：与消息渠道（WhatsApp、Telegram 等）的 WebSocket 连接；与 AI 提供商的 API 连接；与工具执行器的进程通信；与 CLI 客户端和 Web 界面的连接。连接管理包括心跳检测、断线重连、负载均衡等功能。

**认证授权中心**

Gateway 是所有访问的入口点，负责：验证客户端身份和权限；管理 API 令牌和会话；执行访问控制策略；记录审计日志。

**状态协调器**

Gateway 维护系统状态的全局视图：会话状态和上下文管理；任务队列和调度；资源配置和限制；健康状态监控。

### Gateway 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         外部系统                                  │
│    WhatsApp、Telegram、Slack、Discord、API 调用、Webhook 等       │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Gateway 服务                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   消息入口   │  │   认证中心   │  │   路由引擎   │           │
│  │              │  │              │  │              │           │
│  │  • WebSocket │  │  • Token    │  │  • 消息路由  │           │
│  │  • Webhook   │  │  • OAuth    │  │  • 会话分配  │           │
│  │  • HTTP API  │  │  • 权限验证  │  │  • 优先级    │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   状态管理   │  │   工具执行   │  │   监控日志   │           │
│  │              │  │              │  │              │           │
│  │  • 会话状态  │  │  • 工具调度  │  │  • 指标收集  │           │
│  │  • 上下文    │  │  • 结果返回  │  │  • 日志记录  │           │
│  │  • 队列管理  │  │  • 错误处理  │  │  • 告警      │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                                 │
            ┌──────────────────┼──────────────────┐
            ▼                  ▼                  ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│     AI 代理      │ │     工具系统     │ │     客户端      │
│                  │ │                  │ │                  │
│  • Claude        │ │  • 浏览器工具    │ │  • CLI          │
│  • GPT           │ │  • 文件工具     │ │  • Web 界面     │
│  • 本地模型      │ │  • 执行工具     │ │  • API 调用     │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### 消息流转详解

当用户通过任意渠道发送消息时，整个消息处理流程如下：

**步骤一：消息接收**

用户通过某个渠道发送消息。消息首先到达渠道适配器（如 Telegram Bot API），然后通过 WebSocket 或 HTTP POST 发送到 Gateway。Gateway 接收到原始消息后，进行初步的格式转换和验证。

**步骤二：认证和路由**

Gateway 首先验证消息来源的合法性：检查渠道连接的有效性；验证发送者的身份；确认发送者有权限发送消息。通过验证后，路由引擎根据配置规则确定这条消息应该由哪个 AI 代理处理：匹配发送者的会话历史；应用配置的路由规则；确定代理类型和会话标识。

**步骤三：代理处理**

消息被传递到对应的 AI 代理：代理加载相关的对话上下文；构建包含系统提示词、历史消息、工具定义的消息；调用配置的 AI 模型生成响应。

**步骤四：工具调用（如有需要）**

如果响应中包含工具调用指令：Gateway 解析工具调用请求；调度工具执行器执行工具调用；收集工具执行结果并返回给代理；代理根据结果生成最终响应。

**步骤五：响应发送**

代理生成最终响应后：Gateway 根据消息来源渠道进行格式转换；将响应发送到对应的渠道；更新会话上下文和状态。

整个流程涉及多个组件的协调，但这些复杂性对用户是透明的——用户只需要通过熟悉的聊天界面与 AI 交互即可。

---

## 🚀 Gateway 快速开始

### 安装 Gateway

**方法一：全局安装（推荐）**

```bash
# 使用 npm/pnpm/bun 安装
npm install -g openclaw@latest

# 验证安装
openclaw version

# 启动引导程序
openclaw onboard --install-daemon
```

**方法二：从源码运行（开发者）**

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖
pnpm install

# 构建项目
pnpm build

# 运行 Gateway
pnpm openclaw gateway run --verbose
```

### 启动 Gateway

**前台运行（调试模式）**

```bash
# 前台运行，输出详细日志
openclaw gateway run --verbose

# 指定端口
openclaw gateway run --port 18789

# 绑定地址
openclaw gateway run --bind 0.0.0.0
```

**后台服务（生产模式）**

```bash
# 安装为系统服务
openclaw gateway install-daemon

# 启动服务
openclaw gateway start

# 停止服务
openclaw gateway stop

# 重启服务
openclaw gateway restart

# 查看服务状态
openclaw gateway status
```

### 验证安装

```bash
# 检查 Gateway 健康状态
openclaw health

# 查看 Gateway 版本
openclaw version

# 查看连接状态
openclaw channels status

# 测试发送消息
openclaw message send --to "test" --message "Hello from OpenClaw"
```

---

## ⚙️ Gateway 配置

### 配置文件结构

Gateway 的主要配置文件是 `~/.openclaw/openclaw.json`：

```json5
{
  // Gateway 配置
  gateway: {
    // 网络配置
    network: {
      // 绑定地址
      host: "127.0.0.1",
      // 监听端口
      port: 18789,
      // 启用 HTTPS
      https: {
        enabled: false,
        // 证书路径
        cert: "/path/to/cert.pem",
        key: "/path/to/key.pem"
      }
    },
    
    // 认证配置
    auth: {
      // 认证类型
      type: "token",  // token / oauth / disabled
      // API 令牌
      token: "${GATEWAY_TOKEN}",
      // 令牌过期时间
      tokenExpiry: "7d",
      // CORS 设置
      cors: {
        enabled: true,
        origins: ["*"]
      }
    },
    
    // 会话配置
    sessions: {
      // 会话超时时间
      timeout: "30m",
      // 最大并发会话数
      maxConcurrent: 100,
      // 会话历史保留
      historyRetention: "7d"
    },
    
    // 日志配置
    logging: {
      // 日志级别
      level: "info",  // debug / info / warn / error
      // 日志格式
      format: "json",
      // 日志文件路径
      file: {
        enabled: true,
        path: "/var/log/openclaw/gateway.log",
        maxSize: "100m",
        maxFiles: 5
      }
    },
    
    // 监控配置
    monitoring: {
      // 启用健康检查
      healthCheck: true,
      // 指标收集
      metrics: {
        enabled: true,
        endpoint: "/metrics"
      }
    }
  },
  
  // 代理配置
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-20250514"
    }
  },
  
  // 渠道配置
  channels: {
    // 各渠道配置...
  }
}
```

### 网络配置详解

**基础网络配置**：

```json5
{
  gateway: {
    network: {
      host: "0.0.0.0",      // 监听所有地址
      port: 18789,           // 默认端口
      
      // 连接选项
      connections: {
        // WebSocket 最大连接数
        maxWebSocketConnections: 1000,
        // WebSocket 超时时间
        webSocketTimeout: "5m",
        // HTTP 最大并发连接
        maxHttpConnections: 100,
        // 请求超时时间
        requestTimeout: "30s"
      },
      
      // Rate Limiting
      rateLimit: {
        enabled: true,
        // 窗口大小（毫秒）
        windowMs: 60000,
        // 每个窗口最大请求数
        maxRequests: 100,
        // 超出限制的响应码
        statusCode: 429
      }
    }
  }
}
```

**HTTPS 配置**：

```json5
{
  gateway: {
    network: {
      https: {
        enabled: true,
        
        // 证书配置
        cert: {
          // 证书文件路径
          certFile: "/etc/ssl/certs/openclaw.crt",
          // 私钥文件路径
          keyFile: "/etc/ssl/private/openclaw.key",
          // CA 证书（可选）
          caFile: "/etc/ssl/certs/ca.crt"
        },
        
        // 自动续期（使用 Let's Encrypt）
        letsEncrypt: {
          enabled: false,
          domain: "openclaw.example.com",
          email: "admin@example.com"
        },
        
        // TLS 版本
        minVersion: "1.2",
        // 密码套件
        cipherSuites: [
          "TLS_AES_128_GCM_SHA256",
          "TLS_AES_256_GCM_SHA384"
        ]
      }
    }
  }
}
```

### 认证配置详解

**Token 认证**：

```json5
{
  gateway: {
    auth: {
      type: "token",
      
      token: {
        // 令牌长度（字节）
        length: 32,
        // 令牌过期时间
        expiry: "7d",
        // 是否允许刷新令牌
        refreshable: true,
        // 刷新令牌过期时间
        refreshExpiry: "30d"
      },
      
      // 访问控制列表
      acl: {
        // 默认策略
        defaultPolicy: "deny",
        // 规则列表
        rules: [
          {
            // 资源路径
            path: "/api/*",
            // 允许的角色
            allow: ["admin", "user"],
            // HTTP 方法
            methods: ["GET", "POST"]
          },
          {
            path: "/admin/*",
            allow: ["admin"]
          }
        ]
      }
    }
  }
}
```

**OAuth 2.0 认证**：

```json5
{
  gateway: {
    auth: {
      type: "oauth",
      
      oauth: {
        // 提供商
        provider: "google",
        // 客户端 ID
        clientId: "${GOOGLE_CLIENT_ID}",
        // 客户端密钥
        clientSecret: "${GOOGLE_CLIENT_SECRET}",
        // 重定向 URI
        redirectUri: "https://openclaw.example.com/auth/callback",
        // 授权范围
        scopes: ["openid", "profile", "email"],
        
        // JWT 配置
        jwt: {
          // JWT 密钥
          secret: "${JWT_SECRET}",
          // JWT 过期时间
          expiry: "1h"
        }
      }
    }
  }
}
```

---

## 🔐 安全配置

### 安全最佳实践

**网络隔离**：

```json5
{
  gateway: {
    security: {
      // 网络隔离
      networkIsolation: {
        // 启用
        enabled: true,
        // 允许的入站地址
        allowedInbound: ["10.0.0.0/8", "192.168.0.0/16"],
        // 禁止的出站地址
        blockedOutbound: ["0.0.0.0/0"]
      },
      
      // 防火墙集成
      firewall: {
        enabled: true,
        // 防火墙类型
        type: "iptables",
        // 规则
        rules: [
          {
            action: "allow",
            source: "10.0.0.0/8",
            port: 18789,
            protocol: "tcp"
          }
        ]
      }
    }
  }
}
```

**DDoS 防护**：

```json5
{
  gateway: {
    security: {
      ddos: {
        enabled: true,
        
        // 连接限制
        connections: {
          // 每个 IP 最大连接数
          maxPerIp: 10,
          // 全局最大连接数
          maxTotal: 1000
        },
        
        // 请求速率限制
        rateLimit: {
          // 窗口大小
          window: "1m",
          // 每个 IP 最大请求数
          maxPerIp: 60,
          // 超出限制的惩罚时间
          banDuration: "15m"
        },
        
        // 异常检测
        anomalyDetection: {
          enabled: true,
          // 检测阈值
          threshold: 0.8,
          // 响应动作
          response: "rate-limit"  // rate-limit / block / captcha
        }
      }
    }
  }
}
```

### 审计日志

```json5
{
  gateway: {
    logging: {
      audit: {
        enabled: true,
        
        // 记录的事件类型
        events: [
          "authentication",
          "authorization",
          "data_access",
          "configuration_change",
          "admin_action"
        ],
        
        // 敏感数据脱敏
        masking: {
          enabled: true,
          // 脱敏字段
          fields: [
            "password",
            "apiKey",
            "token",
            "creditCard"
          ],
          // 脱敏字符数
          charsToMask: 4
        },
        
        // 导出配置
        export: {
          // 导出格式
          format: "json",
          // 导出目标
          destination: "/var/log/openclaw/audit.log"
        }
      }
    }
  }
}
```

---

## 📡 远程访问

### Tailscale 集成

Tailscale 提供了一种简单安全的方式来实现远程访问：

**配置步骤**：

```bash
# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 登录 Tailscale
sudo tailscale up --auth-key=${TAILSCALE_AUTH_KEY}

# 配置 OpenClaw 使用 Tailscale
openclaw config set gateway.tailscale.enabled true
openclaw config set gateway.tailscale.hostname "openclaw-gateway"

# 重启 Gateway
openclaw gateway restart
```

**Tailscale 配置**：

```json5
{
  gateway: {
    tailscale: {
      enabled: true,
      
      // Tailscale 主机名
      hostname: "openclaw-gateway",
      
      // 访问控制
      acl: {
        // 是否允许子网路由
        subnetRoutes: false,
        // 是否接受出口流量
        exitNode: false
      },
      
      // Funnel（公网访问）
      funnel: {
        enabled: false,
        // 需要 HTTPS 认证
        https: true,
        // 认证方式
        auth: "password"  // password / none
      }
    }
  }
}
```

### SSH 隧道

对于没有 Tailscale 的环境，可以使用 SSH 隧道：

**本地端口转发**：

```bash
# 从本地机器执行
ssh -L 18789:localhost:18789 user@gateway-host
```

**远程端口转发**：

```bash
# 在 Gateway 主机上执行
ssh -R 18789:localhost:18789 user@local-machine
```

### 反向代理

**Nginx 配置**：

```nginx
server {
    listen 443 ssl http2;
    server_name openclaw.example.com;

    ssl_certificate /etc/letsencrypt/live/openclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket 支持
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }
}
```

---

## 📊 监控和运维

### 健康检查

```bash
# 检查 Gateway 健康状态
openclaw health

# 检查各个组件
openclaw health --detailed

# 使用 HTTP 端点
curl http://localhost:18789/health
```

**健康检查端点响应**：

```json
{
  "status": "healthy",
  "timestamp": "2026-02-05T12:00:00Z",
  "components": {
    "gateway": {
      "status": "healthy",
      "uptime": "7d 12h 30m",
      "version": "1.2.3"
    },
    "channels": {
      "status": "healthy",
      "connected": 3,
      "total": 5
    },
    "agents": {
      "status": "healthy",
      "active": 2,
      "idle": 5
    },
    "memory": {
      "status": "healthy",
      "used": "512MB",
      "limit": "2GB",
      "percentage": 25.6
    }
  }
}
```

### 日志管理

**查看日志**：

```bash
# 查看最近 100 行日志
openclaw logs --tail 100

# 实时查看日志
openclaw logs --follow

# 查看错误日志
openclaw logs --level error

# 搜索特定内容
openclaw logs --grep "channel" --follow

# 导出日志
openclaw logs --export /path/to/logs.txt
```

**日志级别**：

| 级别 | 说明 | 使用场景 |
|------|------|----------|
| **debug** | 详细的调试信息 | 开发和问题诊断 |
| **info** | 一般信息 | 日常监控 |
| **warn** | 警告信息 | 潜在问题 |
| **error** | 错误信息 | 问题排查 |
| **fatal** | 严重错误 | 紧急问题 |

### 性能监控

```json5
{
  gateway: {
    monitoring: {
      metrics: {
        enabled: true,
        
        // 指标端点
        endpoint: "/metrics",
        
        // 收集的指标
        collect: {
          // 系统指标
          system: {
            cpu: true,
            memory: true,
            disk: true,
            network: true
          },
          
          // 应用指标
          application: {
            requests: true,
            latency: true,
            errors: true,
            channels: true,
            agents: true
          }
        },
        
        // 导出到监控系统
        export: {
          enabled: true,
          type: "prometheus",  // prometheus / influxdb / datadog
          endpoint: "http://prometheus:9090/metrics"
        }
      }
    }
  }
}
```

---

## 🚨 故障排除

### 常见问题

**问题一：Gateway 无法启动**

```
症状：运行 `openclaw gateway run` 后提示端口被占用
```

**解决方案**：

```bash
# 检查端口占用
lsof -i :18789

# 查看 Gateway 进程
ps aux | grep openclaw

# 检查配置文件
openclaw doctor
```

**问题二：连接被拒绝**

```
症状：客户端无法连接到 Gateway
```

**解决方案**：

```bash
# 确认 Gateway 在运行
openclaw gateway status

# 检查防火墙设置
sudo ufw status

# 测试本地连接
curl http://127.0.0.1:18789/health
```

**问题三：认证失败**

```
症状：使用 API Token 连接失败
```

**解决方案**：

```bash
# 验证 Token 配置
openclaw config get gateway.auth.token

# 重新生成 Token
openclaw gateway token --regenerate

# 检查时间同步
ntpq -p
```

**问题四：内存使用过高**

```
症状：Gateway 占用大量内存
```

**解决方案**：

```bash
# 查看内存使用
openclaw status --memory

# 减少会话历史保留时间
openclaw config set gateway.sessions.historyRetention "1d"

# 限制并发会话数
openclaw config set gateway.sessions.maxConcurrent 50

# 重启 Gateway
openclaw gateway restart
```

### 诊断工具

```bash
# 运行全面诊断
openclaw doctor --verbose

# 网络诊断
openclaw doctor --network

# 配置诊断
openclaw doctor --config

# 安全诊断
openclaw doctor --security
```

---

## 📖 相关文档

- [系统架构](/concepts/architecture)——整体架构详解
- [配置参考](/config/reference)——完整配置选项
- [安全指南](/gateway/security)——安全最佳实践
- [远程访问](/gateway/remote)——远程访问配置
- [故障排除](/help/troubleshooting)——问题解决指南
