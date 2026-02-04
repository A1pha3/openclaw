---
summary: "`openclaw memory` 命令参考（状态/索引/搜索）"
read_when:
  - 想要索引或搜索语义记忆
  - 调试记忆可用性或索引问题
title: "memory"
---

# `openclaw memory`

管理语义记忆的索引和搜索。由活动的记忆插件提供（默认：`memory-core`；设置 `plugins.slots.memory = "none"` 可禁用）。

## 为什么需要语义记忆

语义记忆让代理能够：

- **记住上下文**：跨会话保留重要信息
- **知识检索**：从工作区文档中提取相关内容
- **个性化响应**：基于历史交互提供更准确的答案
- **减少重复**：不需要每次都重新解释背景

## 相关链接

- 记忆概念：[Memory](/zh-cn/concepts/memory)
- 插件系统：[Plugins](/zh-cn/plugins)

## 基本命令

```bash
# 查看记忆状态
openclaw memory status

# 深度状态检查
openclaw memory status --deep

# 深度检查 + 自动索引
openclaw memory status --deep --index

# 详细输出
openclaw memory status --deep --index --verbose

# 手动触发索引
openclaw memory index
openclaw memory index --verbose

# 搜索记忆
openclaw memory search "发布检查清单"

# 指定代理
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## 选项说明

| 选项 | 说明 |
|------|------|
| `--agent <id>` | 指定单个代理（默认：所有已配置代理） |
| `--verbose` | 在探测和索引时输出详细日志 |
| `--deep` | 探测向量库 + 嵌入模型可用性 |
| `--index` | 如果存储是脏的则运行重新索引 |

## 记忆状态

```bash
openclaw memory status
```

输出示例：

```
Memory Status
─────────────
Agent: main
  Vector store: ready (chromadb)
  Embedding model: text-embedding-3-small
  Documents indexed: 156
  Last indexed: 2024-01-15 14:30:00
  Extra paths: ~/notes, ~/projects/docs
```

## 索引工作流

### 手动索引

```bash
openclaw memory index --verbose
```

输出示例：

```
Indexing memory for agent: main
  Provider: openai
  Model: text-embedding-3-small
  Sources:
    - ~/.openclaw/workspace (42 files)
    - ~/notes (15 files)
  Batch 1/3: processing 20 documents...
  Batch 2/3: processing 20 documents...
  Batch 3/3: processing 17 documents...
  Total: 57 documents indexed
```

### 自动索引

在状态检查时自动索引：

```bash
openclaw memory status --deep --index
```

## 搜索记忆

```bash
openclaw memory search "如何配置 webhook"
```

输出示例：

```
Search results for: "如何配置 webhook"

1. [0.92] ~/.openclaw/workspace/AGENTS.md
   "Webhook 配置在 automation.webhooks 下..."

2. [0.87] ~/notes/openclaw-setup.md
   "要启用 webhook，首先在网关配置中..."

3. [0.81] ~/.openclaw/workspace/memory/2024-01-10-webhook-setup.md
   "今天设置了 Gmail webhook..."
```

## 配置

### 基本配置

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // 或 "none" 禁用
    },
  },
}
```

### 额外路径

```json5
{
  memorySearch: {
    extraPaths: [
      "~/notes",
      "~/projects/docs",
    ],
  },
}
```

## 故障排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 向量库不可用 | 依赖未安装 | 检查 chromadb 或其他向量库安装 |
| 嵌入模型失败 | API 密钥问题 | 检查 OpenAI 或其他嵌入提供商配置 |
| 索引为空 | 工作区为空 | 添加文档到工作区或配置 extraPaths |
| 搜索无结果 | 索引过期 | 运行 `openclaw memory index` |
| 内存不足 | 文档过多 | 减少索引文件数量或增加系统内存 |

## 最佳实践

1. **定期索引**：在添加重要文档后手动索引
2. **组织工作区**：保持文档结构清晰
3. **使用 extraPaths**：将常用参考文档纳入索引
4. **监控状态**：定期运行 `memory status --deep` 检查健康状态
