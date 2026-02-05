---
summary: "Ollama 本地 LLM 运行时完全指南，包含安装配置、模型发现、性能优化和故障排查"
read_when:
  - 您想通过 Ollama 使用本地模型运行 OpenClaw
  - 您需要 Ollama 设置和配置指导
  - 您关注隐私保护和离线使用
title: "Ollama 本地部署完全指南"
---

# Ollama 本地部署完全指南

本文档详细介绍如何在 OpenClaw 中配置和使用 Ollama 本地大语言模型。完成本指南后，你将理解 Ollama 的工作原理，掌握本地部署的完整流程，并能够根据硬件配置优化性能。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）⭐

- [ ] 理解 Ollama 的核心概念和架构
- [ ] 能够完成 Ollama 的基础安装和配置
- [ ] 掌握模型发现机制的工作原理
- [ ] 能够拉取和使用基础模型

### 进阶目标（建议掌握）⭐⭐

- [ ] 配置远程 Ollama 实例
- [ ] 优化模型性能和资源使用
- [ ] 理解工具调用模型的要求
- [ ] 能够诊断连接问题

### 专家目标（挑战）⭐⭐⭐

- [ ] 设计高性能本地部署方案
- [ ] 实现多模型管理和切换
- [ ] 建立本地模型的监控系统

---

## 核心概念解析

### 为什么选择本地部署

**Ollama 的核心价值**：

| 优势 | 说明 | 适用场景 |
|------|------|----------|
| **隐私保护** | 数据不离开本地 | 敏感数据处理、企业内部使用 |
| **零延迟** | 无网络往返 | 实时对话、高频调用 |
| **零成本** | 无 API 费用 | 成本敏感场景、开发测试 |
| **离线可用** | 无需网络连接 | 离线环境、网络受限场景 |

**与云服务的对比**：

| 特性 | Ollama（本地） | 云服务（OpenAI/Anthropic） |
|------|----------------|---------------------------|
| 延迟 | < 10ms | 100-500ms |
| 成本 | 硬件成本 | 按 token 计费 |
| 隐私 | 完全本地 | 数据上传云端 |
| 可用性 | 依赖本地硬件 | 99.9% SLA |
| 模型选择 | 开源模型 | 闭源模型 |

### Ollama 工作原理

**架构设计**：

```
┌─────────────────────────────────────┐
│           OpenClaw                  │
│                                     │
│  ┌─────────────────────────────┐   │
│  │   Provider Abstraction      │   │
│  │   (统一接口，屏蔽差异)       │   │
│  └─────────────────────────────┘   │
│                 │                  │
│                 ▼                  │
│  ┌─────────────────────────────┐   │
│  │   Ollama Provider           │   │
│  │   - 模型发现                 │   │
│  │   - API 兼容层              │   │
│  │   - 工具调用转发            │   │
│  └─────────────────────────────┘   │
│                 │                  │
│                 ▼                  │
│  ┌─────────────────────────────┐   │
│  │   Ollama OpenAI API        │   │
│  │   (HTTP API)               │   │
│  └─────────────────────────────┘   │
│                 │                  │
│                 ▼                  │
│  ┌─────────────────────────────┐   │
│  │   Ollama Server            │   │
│  │   (本地推理引擎)            │   │
│  └─────────────────────────────┘   │
│                 │                  │
│                 ▼                  │
│  ┌─────────────────────────────┐   │
│  │   LLM Model                │   │
│  │   (Llama 3, Qwen, etc.)   │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

### OpenAI 兼容 API

Ollama 提供了 OpenAI 兼容 API，这意味着：

1. **无缝集成**：OpenClaw 可以使用标准 OpenAI 接口
2. **SDK 兼容**：可以使用 OpenAI 的任何 SDK
3. **工具调用**：支持工具调用功能的模型可被自动发现

**API 端点**：
- 默认地址：`http://localhost:11434/v1`
- 兼容格式：`/chat/completions`

---

## 快速开始

### 系统要求

**最低要求**：
- 操作系统：macOS / Linux / Windows (WSL2)
- 内存：8GB RAM
- 存储：5GB 可用空间

**推荐配置**：
- 内存：16GB+ RAM
- GPU：NVIDIA GPU（可选，大幅提升性能）
- 存储：20GB+ 可用空间

### 安装 Ollama

**macOS / Linux**：

```bash
# 安装 Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 启动服务
ollama serve
```

**Docker**：

```bash
# 使用 Docker 运行
docker run -d -p 11434:11434 \
  -v ollama:/root/.ollama \
  --name ollama \
  ollama/ollama
```

### 拉取模型

```bash
# 拉取 Llama 3.3（推荐基础模型）
ollama pull llama3.3

# 拉取代码专用模型
ollama pull qwen2.5-coder:32b

# 拉取推理模型
ollama pull deepseek-r1:32b

# 拉取轻量模型（低配置设备）
ollama pull llama3.2:3b

# 查看已安装模型
ollama list
```

### OpenClaw 基础配置

**方式一：环境变量（推荐）**

```bash
# 设置环境变量（任何值都可以，Ollama 不需要真实密钥）
export OLLAMA_API_KEY="ollama-local"
```

**方式二：配置文件**

```bash
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

**配置模型**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/llama3.3"
      }
    }
  }
}
```

---

## 模型发现机制

### 隐式发现原理

当满足以下条件时，OpenClaw 会自动发现 Ollama 模型：

1. 设置了 `OLLAMA_API_KEY` 环境变量
2. **未**定义显式的 `models.providers.ollama` 配置

**发现流程**：

```bash
# 1. 查询模型列表
GET http://localhost:11434/api/tags

# 2. 获取模型详情
GET http://localhost:11434/api/show?name=llama3.3

# 3. 筛选支持工具调用的模型
# 4. 读取上下文窗口配置
# 5. 生成模型目录
```

**自动发现的结果**：

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "apiKey": "ollama-local",
        "models": [
          {
            "id": "llama3.3",
            "name": "Llama 3.3",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0, "output": 0 },
            "contextWindow": 131072,
            "maxTokens": 1310720
          }
        ]
      }
    }
  }
}
```

### 发现规则

| 条件 | 处理方式 |
|------|----------|
| 模型报告支持 `tools` | 标记为工具可用 |
| 模型报告 `thinking` | 标记为推理模型 |
| 有 `context_length` | 使用报告的值作为上下文窗口 |
| 无上下文信息 | 默认为 8192 |
| 无成本信息 | 设置为 $0 |

### 查看可用模型

```bash
# Ollama 查看
ollama list

# OpenClaw 查看
openclaw models list
```

---

## 高级配置

### 显式配置（手动模型定义）

在以下情况下使用显式配置：

- Ollama 在其他主机/端口上运行
- 需要强制特定的上下文窗口
- 需要包含不支持工具的模型

**基本显式配置**：

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://ollama-host:11434/v1",
        "apiKey": "ollama-local",
        "api": "openai-completions",
        "models": [
          {
            "id": "llama3.3",
            "name": "Llama 3.3",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0, "output": 0 },
            "contextWindow": 8192,
            "maxTokens": 81920
          }
        ]
      }
    }
  }
}
```

### 远程 Ollama 配置

**场景**：Ollama 运行在远程服务器

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "apiKey": "ollama-local",
        "baseUrl": "http://192.168.1.100:11434/v1"
      }
    }
  }
}
```

**注意**：显式配置会禁用自动发现，需要手动定义所有模型。

### 模型选择配置

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/llama3.3",
        "fallback": [
          "ollama/qwen2.5-coder:32b",
          "ollama/deepseek-r1:32b"
        ]
      }
    }
  }
}
```

---

## 工具调用与推理模型

### 工具调用要求

Ollama 模型需要明确支持工具调用功能才能被 OpenClaw 使用。

**检查模型是否支持工具**：

```bash
# 查看模型详情
ollama show llama3.3

# 检查 capabilities
# 如果支持 tools，则会列出可用工具
```

**不支持工具的模型**：

如果模型不支持工具，需要在显式配置中标记：

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "apiKey": "ollama-local",
        "models": [
          {
            "id": "llama3.3",
            "tools": false,
            "reasoning": false
          }
        ]
      }
    }
  }
}
```

### 推理模型

当 Ollama 模型报告支持 `thinking` 时，OpenClaw 会将其标记为推理模型：

```bash
# 拉取推理模型
ollama pull deepseek-r1:32b
```

**配置推理模型**：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/deepseek-r1:32b"
      }
    }
  }
}
```

---

## 性能优化

### 内存优化

**查看内存使用**：

```bash
# 查看 Ollama 内存使用
ollama ps

# 查看系统内存
free -h  # Linux
htop     # macOS/Linux
```

**模型卸载**：

```bash
# 卸载不使用的模型
ollama rm llama3.3

# 卸载所有模型
ollama rm *
```

### GPU 加速

**NVIDIA GPU**：

```bash
# 检查 CUDA 是否可用
nvidia-smi

# 使用 GPU 运行模型
ollama run llama3.3 --gpu
```

**Apple Silicon**：

Ollama 自动使用 Metal 加速：

```bash
# 查看 GPU 使用情况
 Activity Monitor > GPU
```

### 批量处理优化

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "apiKey": "ollama-local",
        "batchSize": 32,  // 批量大小
        "timeout": 120000  // 超时时间（毫秒）
      }
    }
  }
}
```

---

## 故障排查指南

### 故障排查思维模型

```
1. 检查 Ollama 服务状态
   └── ollama serve 是否运行
   
2. 验证 API 连通性
   └── curl 测试本地 API
   
3. 检查模型是否安装
   └── ollama list
   
4. 验证 OpenClaw 配置
   └── 环境变量、配置文件
   
5. 查看详细日志
   └── OpenClaw 日志
```

### 常见问题与解决方案

#### 问题一：未检测到 Ollama

**症状**：
```
Error: Ollama provider not found
```

**排查步骤**：

```bash
# 1. 检查 Ollama 是否运行
ps aux | grep ollama

# 2. 手动启动服务
ollama serve

# 3. 验证 API 可访问
curl http://localhost:11434/api/tags

# 4. 检查环境变量
echo $OLLAMA_API_KEY
```

**解决方案**：

```bash
# 确保 Ollama 正在运行
ollama serve

# 设置环境变量
export OLLAMA_API_KEY="ollama-local"
```

#### 问题二：连接被拒绝

**症状**：
```
Error: Connection refused
Status: 000
```

**排查步骤**：

```bash
# 1. 检查 Ollama 端口
curl -v http://localhost:11434/api/tags

# 2. 检查防火墙设置
sudo ufw status  # Linux

# 3. 检查端口监听
netstat -tulpn | grep 11434
```

**解决方案**：

```bash
# 重启 Ollama
ollama serve

# 检查正确的端口
lsof -i :11434
```

#### 问题三：没有可用的模型

**症状**：
```
Error: No available models
```

**排查步骤**：

```bash
# 1. 查看已安装模型
ollama list

# 2. 检查模型是否支持工具
ollama show llama3.3 | grep tools

# 3. 查看 OpenClaw 模型列表
openclaw models list | grep ollama
```

**解决方案**：

```bash
# 拉取支持工具的模型
ollama pull llama3.3

# 拉取轻量模型
ollama pull llama3.2:3b

# 显式定义不支持工具的模型
# 见"显式配置"章节
```

#### 问题四：模型运行缓慢

**症状**：
```
Error: Model response timeout
```

**排查步骤**：

```bash
# 1. 检查系统资源
top -bn1 | head -20

# 2. 检查内存使用
free -m

# 3. 检查 GPU 使用
nvidia-smi
```

**解决方案**：

```bash
# 使用轻量模型
ollama pull llama3.2:3b

# 卸载不需要的模型
ollama rm llama3.3

# 增加系统内存或使用 GPU
```

#### 问题五：上下文窗口过小

**症状**：
```
Error: Context length exceeded
```

**解决方案**：

```json5
{
  "models": {
    "providers": {
      "ollama": {
        "apiKey": "ollama-local",
        "models": [
          {
            "id": "llama3.3",
            "contextWindow": 131072,
            "maxTokens": 1310720
          }
        ]
      }
    }
  }
}
```

---

## 适用场景分析

### 推荐使用 Ollama 的场景

| 场景 | 推荐模型 | 说明 |
|------|----------|------|
| 隐私敏感数据 | llama3.3 | 数据完全本地 |
| 开发测试 | llama3.2:3b | 快速迭代 |
| 代码辅助 | qwen2.5-coder:32b | 代码优化 |
| 复杂推理 | deepseek-r1:32b | 深度思考 |
| 资源受限环境 | llama3.2:1b | 最低配置 |

### 不适用场景

- 需要最强 AI 能力：使用 Claude/GPT-4
- 需要云端 API 集成：使用 OpenAI/Anthropic
- 需要全球可用性：使用云服务

### 混合使用策略

```json5
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "ollama": {
        "apiKey": "ollama-local"
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-20250514",
        "fallbacks": [
          "ollama/llama3.3"
        ]
      }
    }
  }
}
```

---

## 相关命令速查

| 命令 | 说明 |
|------|------|
| `ollama serve` | 启动 Ollama 服务 |
| `ollama pull <model>` | 拉取模型 |
| `ollama list` | 列出已安装模型 |
| `ollama rm <model>` | 卸载模型 |
| `ollama show <model>` | 查看模型详情 |
| `ollama run <model>` | 运行模型 |
| `openclaw models list` | 查看 OpenClaw 可用模型 |
| `openclaw health` | 检查认证状态 |

---

## 相关文档

- [AI 提供商概览](/zh-CN/providers/index) - 所有提供商对比
- [OpenAI 配置](/zh-CN/providers/openai) - 云服务配置
- [Anthropic 配置](/zh-CN/providers/anthropic) - Claude 配置
- [模型概念](/zh-CN/concepts/models) - 模型系统详解
- [配置参考](/zh-CN/config/reference) - 完整配置选项

---

## 实践任务

### 练习 1：完成基础配置 ⭐

**任务**：安装 Ollama 并运行第一个本地模型

**步骤**：
1. 安装 Ollama
2. 启动服务：`ollama serve`
3. 拉取模型：`ollama pull llama3.3`
4. 配置 OpenClaw
5. 测试运行

**成功标准**：能够在 OpenClaw 中使用本地 Llama 模型

### 练习 2：配置远程 Ollama ⭐⭐

**任务**：配置远程 Ollama 服务器

**步骤**：
1. 在服务器安装 Ollama
2. 配置网络访问
3. 在 OpenClaw 配置远程 URL
4. 测试连接

**成功标准**：能够使用远程服务器的模型

### 练习 3：性能优化实践 ⭐⭐⭐

**任务**：优化本地模型性能

**步骤**：
1. 分析当前性能瓶颈
2. 选择合适的模型大小
3. 配置资源限制
4. 建立性能基线

**成功标准**：延迟降低 50% 以上

---

## 自检清单

### 概念理解
- [ ] 理解 Ollama 的工作原理
- [ ] 知道模型发现的机制
- [ ] 理解工具调用模型的要求

### 动手能力
- [ ] 能够完成 Ollama 安装
- [ ] 能够拉取和使用模型
- [ ] 能够配置远程实例

### 问题解决
- [ ] 能够诊断连接问题
- [ ] 能够解决性能问题
- [ ] 能够优化资源使用
