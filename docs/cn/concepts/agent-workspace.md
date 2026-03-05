---
read_when:
  - 你需要解释智能体工作区或其文件布局
  - 你想备份或迁移智能体工作区
summary: "智能体工作区完整指南：位置、布局、文件映射、备份策略和最佳实践"
title: "智能体工作区"
---

# 智能体工作区

## 🎯 学习目标

完成本文档学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解工作区的概念和作用
- [ ] 掌握工作区文件的含义和用途
- [ ] 正确配置和初始化工作区
- [ ] 实施工作区备份策略

### 进阶目标（建议掌握）

- [ ] 管理多工作区环境
- [ ] 迁移工作区到新机器
- [ ] 理解沙箱隔离模式
- [ ] 优化工作区性能

---

## 💡 为什么需要工作区？

### 类比：工作区是智能体的"大脑"

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    工作区类比                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  人脑的结构：                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  长期记忆    │  存储重要事实和经验                               │   │
│  │  短期记忆    │  当前对话的上下文                                 │   │
│  │  个性        │  行为风格和反应模式                               │   │
│  │  技能        │  可以执行的操作                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenClaw 工作区结构：                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  AGENTS.md    │  操作指南和记忆（长期记忆）                         │   │
│  │  SOUL.md      │  人设和个性（个性）                               │   │
│  │  TOOLS.md     │  工具使用指南（技能）                               │   │
│  │  USER.md      │  用户信息（人际认知）                             │   │
│  │  skills/      │  专属技能（可执行操作）                           │   │
│  │  memory/      │  每日记忆日志（记忆存储）                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 核心价值

| 价值 | 说明 |
|------|------|
| **上下文连续性** | AI 记住之前的对话 |
| **个性化行为** | 定制 AI 的回复风格 |
| **技能扩展** | 添加自定义能力 |
| **持久记忆** | 长期信息存储 |

---

## 📂 默认位置

### 位置规则

| 配置 | 位置 |
|------|------|
| **基础工作区** | `~/.openclaw/workspace` |
| **Profile 工作区** | `~/.openclaw/workspace-<profile>` |
| **自定义工作区** | 配置文件指定 |

### 配置工作区

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
    },
  },
}
```

### 初始化工作区

```bash
# 自动创建工作区并填充引导文件
openclaw setup

# 指定工作区路径
openclaw setup --workspace ~/my-workspace
```

### 禁用引导文件

```json5
{
  agents: {
    skipBootstrap: true,
  },
}
```

---

## 📄 工作区文件映射

### 核心引导文件

| 文件 | 作用 | 加载时机 | 必需 |
|------|------|----------|------|
| **AGENTS.md** | 操作指南 + 记忆 | 每个会话 | ✅ 推荐 |
| **SOUL.md** | 人设、边界、语气 | 每个会话 | ✅ 推荐 |
| **TOOLS.md** | 工具使用说明 | 参考 | ✅ 推荐 |
| **IDENTITY.md** | 智能体名称/风格/表情 | 引导仪式 | ✅ 推荐 |
| **USER.md** | 用户档案 + 偏好 | 每个会话 | ✅ 推荐 |

### 可选文件

| 文件 | 作用 | 使用场景 |
|------|------|----------|
| **HEARTBEAT.md** | 心跳运行检查清单 | 定期健康检查 |
| **BOOT.md** | 启动检查清单 | 内部 hooks 启用时 |
| **BOOTSTRAP.md** | 一次性首次运行仪式 | 仅新工作区 |
| **MEMORY.md** | 精选长期记忆 | 主私聊会话 |

### 记忆系统

| 文件 | 说明 |
|------|------|
| `memory/YYYY-MM-DD.md` | 每日记忆日志 |
| `MEMORY.md` | 精选长期记忆 |

> **💡 提示**：参见 [记忆](/concepts/memory) 了解工作流程和自动记忆刷新。

### 扩展目录

| 目录 | 作用 |
|------|------|
| **skills/** | 工作区特定的 Skills |
| **canvas/** | Canvas UI 文件 |

---

## 🗂️ 工作区文件结构

### 完整目录树

```
~/.openclaw/workspace/
├── AGENTS.md              # 操作指南
├── SOUL.md                # 人设定义
├── TOOLS.md               # 工具说明
├── USER.md                # 用户信息
├── IDENTITY.md            # 智能体身份
├── HEARTBEAT.md           # 心跳检查（可选）
├── BOOT.md                # 启动检查（可选）
├── MEMORY.md              # 长期记忆（可选）
├── skills/                # 工作区 Skills
│   ├── my-skill/
│   └── ...
├── canvas/                # Canvas 文件
│   └── index.html
└── memory/                # 每日记忆
    ├── 2026-03-05.md
    └── 2026-03-04.md
```

---

## ⚠️ 重要注意事项

### 工作区 ≠ 配置目录

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    目录结构区分                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ~/.openclaw/                    ~/.openclaw/workspace/          │
│  ├── openclaw.json               ├── AGENTS.md                    │
│  ├── credentials/                ├── SOUL.md                      │
│  ├── agents/                    ├── TOOLS.md                     │
│  │   └── <id>/sessions/         ├── skills/                      │
│  ├── skills/                    └── memory/                      │
│  └── ...                                                         │
│                                                                         │
│  配置、凭证、会话                          智能体的"大脑"            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 沙箱隔离模式

| 模式 | 工作区位置 | 说明 |
|------|-----------|------|
| **off** | 主机工作区 | 无隔离，完全访问 |
| **non-main** | 沙箱工作区 | 非主会话隔离 |
| **all** | 沙箱工作区 | 所有会话隔离 |

### 沙箱配置

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",        // off | non-main | all
        workspaceAccess: "rw",   // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
      },
    },
  },
}
```

---

## 🔧 工作区管理

### 多工作区警告

**问题**：保留多个工作区可能导致认证或状态漂移。

**解决方案**：

```bash
# 检查额外工作区
openclaw doctor

# 归档旧工作区
trash ~/openclaw

# 或删除
rm -rf ~/openclaw
```

### 切换活动工作区

```bash
# 方法 1：更新配置
openclaw config set agents.defaults.workspace ~/new-workspace

# 方法 2：设置环境变量
export OPENCLAW_WORKSPACE=~/new-workspace
```

---

## 💾 Git 备份（推荐，私有）

### 为什么需要备份？

| 风险 | 后果 |
|------|------|
| 磁盘故障 | 丢失所有 AI 对话记忆 |
| 误删除 | 无法恢复个性化配置 |
| 迁移困难 | 无法在新机器恢复状态 |

### 初始化仓库

```bash
cd ~/.openclaw/workspace

# 初始化 Git 仓库
git init

# 添加核心文件
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/

# 提交
git commit -m "Add agent workspace"
```

### 添加私有远程

**选项 A：GitHub CLI**

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

**选项 B：手动配置**

```bash
# 创建私有仓库后
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 持续更新

```bash
cd ~/.openclaw/workspace

# 查看变更
git status

# 添加所有变更
git add .

# 提交
git commit -m "Update memory"

# 推送
git push
```

### .gitignore 配置

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

---

## 🚫 不要提交密钥

### 危险信号

| 内容 | 风险 | 处理 |
|------|------|------|
| API 密钥 | 泄露凭证 | 使用占位符 |
| OAuth token | 账户被盗 | 存储在 `~/.openclaw/credentials/` |
| 聊天记录 | 隐私泄露 | 不要提交 |
| 敏感附件 | 数据泄露 | 不要提交 |

### 安全建议

```bash
# 检查是否有密钥被提交
grep -r "sk-" ~/.openclaw/workspace/
grep -r "token" ~/.openclaw/workspace/

# 检查 git 历史
git log --all --full-history -- "*token*"
```

---

## 🔄 迁移到新机器

### 迁移步骤

```bash
# 1. 克隆工作区仓库
cd ~/.openclaw
git clone <your-repo-url> workspace

# 2. 配置工作区路径
openclaw config set agents.defaults.workspace ~/.openclaw/workspace

# 3. 填充缺失文件
openclaw setup --workspace ~/.openclaw/workspace

# 4. （可选）复制会话记录
# 从旧机器复制 ~/.openclaw/agents/<agentId>/sessions/
```

---

## 🎯 最佳实践

### 单工作区场景

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      sandbox: { mode: "off" },
    },
  },
}
```

### 多工作区场景

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
      },
      {
        id: "work",
        workspace: "~/.openclaw/workspace-work",
      },
    ],
  },
}
```

### 备份策略

| 策略 | 频率 | 保留期 |
|------|------|--------|
| Git 提交 | 每日 | 永久 |
| 磁盘快照 | 每周 | 1 个月 |
| 远程备份 | 每周 | 3 个月 |

---

## 📚 相关文档

| 文档 | 链接 |
|------|------|
| [智能体运行时](/concepts/agent) | 运行时架构 |
| [记忆系统](/concepts/memory) | 记忆工作流 |
| [Skills](/tools/skills) | Skills 系统 |
| [压缩](/concepts/compaction) | 上下文压缩 |

---

## 🎯 知识点回顾

| 技能 | 掌握程度 |
|------|----------|
| 配置工作区 | ⭐⭐⭐⭐⭐ |
| 管理引导文件 | ⭐⭐⭐⭐ |
| Git 备份 | ⭐⭐⭐⭐ |
| 迁移工作区 | ⭐⭐⭐ |

---

> **💡 专家提示**：使用 `openclaw setup` 可以重新创建缺失的引导文件而不覆盖现有内容！
