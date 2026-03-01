---
summary: SOUL Evil Hook 完全指南——实现人格切换、随机机会与清除窗口的高级注入机制
read_when:
  - 你想要启用或调整 SOUL Evil hook
  - 你想要实现窗口期人格切换或随机机会人格变异
  - 你想要理解 hook 系统的注入机制和生命周期
title: "SOUL Evil Hook 完全指南"
---

# SOUL Evil Hook 完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 SOUL Evil hook 的设计目的和核心功能
- 掌握启用和配置 SOUL Evil hook 的完整流程
- 能够创建并使用备用的 SOUL 文件实现人格切换

### 进阶目标（建议掌握）

- 理解 hook 的执行时机和注入机制
- 掌握清除窗口和随机机会的组合使用技巧
- 能够设计复杂的人格切换策略

### 专家目标（挑战）

- 为 OpenClaw 开发自定义 hook
- 设计针对特定场景的人格变异算法
- 优化 hook 性能并解决边界情况

## 核心概念与设计理念

### 什么是 SOUL Evil Hook？

SOUL Evil hook 是 OpenClaw 钩子系统中的一个特殊组件，它能够在特定条件下将注入的 `SOUL.md` 内容替换为 `SOUL_EVIL.md` 内容。这种机制让你能够在不修改磁盘文件的情况下，实现人格的动态切换。

**核心特性：**

- **内存注入**：不修改磁盘文件，所有替换在内存中完成
- **条件触发**：支持清除窗口和随机机会两种触发机制
- **灵活配置**：可自定义备用文件名、触发概率和窗口时间

**与普通 SOUL 的关系：**

```
┌─────────────────────────────────────────────────────────────────┐
│                        引导过程                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐                                              │
│   │ SOUL.md      │  ← 正常人格配置（基础）                      │
│   └──────┬───────┘                                              │
│          │                                                       │
│          │ (hook 检查)                                           │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │ SOUL_EVIL.md │  ← 替换人格配置（条件触发）                   │
│   └──────────────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 应用场景

**场景一：测试环境的人格压力测试**

在测试新功能时，切换到「Evil」人格来探索边缘情况：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 1.0,
          "purge": {
            "at": "21:00",
            "duration": "15m"
          }
        }
      }
    }
  }
}
```

**场景二：随机人格注入（模拟用户多样性）**

在测试聊天机器人响应时，引入随机的人格变异：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1
        }
      }
    }
  }
}
```

**场景三：限时特殊模式**

在特定时间段内启用特殊人格（例如夜间模式）：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_NIGHT.md",
          "chance": 0.0,
          "purge": {
            "at": "22:00",
            "duration": "8h"
          }
        }
      }
    }
  }
}
```

## 工作原理深度解析

### 执行时机与生命周期

SOUL Evil hook 的执行发生在系统提示组装的关键阶段：

```
┌─────────────────────────────────────────────────────────────────┐
│                    agent:bootstrap 生命周期                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. 配置加载                                                   │
│      └── 读取 hooks.internal.entries.soul-evil                  │
│                                                                 │
│   2. HOOK 注册阶段                                              │
│      └── soul-evil hook 注册到内部 hook 调度器                   │
│                                                                 │
│   3. SOUL 文件读取阶段                                          │
│      └── 尝试读取 SOUL.md（内存中）                             │
│                                                                 │
│   4. HOOK 执行阶段  ←───  SOUL Evil hook 介入点                 │
│      ├── 检查 purge 窗口                                         │
│      ├── 检查随机机会                                            │
│      └── 替换为 SOUL_EVIL.md（如条件满足）                      │
│                                                                 │
│   5. 提示组装阶段                                               │
│      └── 使用最终的 SOUL 内容组装系统提示                       │
│                                                                 │
│   6. 代理运行阶段                                               │
│      └── 子代理启动（不受 hook 影响）                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 替换机制详解

**文件读取流程：**

```typescript
// 简化伪代码
async function loadSoulFile(hookContext: HookContext): Promise<string> {
  // 1. 读取正常 SOUL 文件
  let soulContent = await readFile('SOUL.md');

  // 2. 执行 soul-evil hook
  const hookResult = await hooks.execute('soul-evil', {
    originalContent: soulContent,
    context: hookContext
  });

  // 3. 如果 hook 返回替换内容，使用它
  if (hookResult.replacement) {
    soulContent = hookResult.replacement;
  }

  return soulContent;
}
```

**条件判断逻辑：**

```typescript
function shouldReplace(
  config: SoulEvilConfig,
  currentTime: Date
): { shouldReplace: boolean; reason: 'purge' | 'chance' | 'none' } {
  // 1. 检查 purge 窗口（优先级更高）
  if (isInPurgeWindow(config.purge, currentTime)) {
    return { shouldReplace: true, reason: 'purge' };
  }

  // 2. 检查随机机会
  if (Math.random() < config.chance) {
    return { shouldReplace: true, reason: 'chance' };
  }

  // 3. 不替换
  return { shouldReplace: false, reason: 'none' };
}
```

### 清除窗口算法

**时间解析：**

```
输入: { at: "21:00", duration: "15m" }

解析结果:
- startTime: 当天 21:00:00
- endTime: 当天 21:15:00
- duration: 15 分钟 = 900 秒
```

**窗口判断：**

```typescript
function isInPurgeWindow(
  purge: PurgeConfig,
  current: Date
): boolean {
  const now = current.getTime();
  const [hours, minutes] = purge.at.split(':').map(Number);
  const durationMs = parseDuration(purge.duration);

  // 获取今天的窗口边界
  const startOfDay = new Date(current);
  startOfDay.setHours(0, 0, 0, 0);
  const windowStart = startOfDay.getTime() + hours * 3600000 + minutes * 60000;
  const windowEnd = windowStart + durationMs;

  return now >= windowStart && now <= windowEnd;
}
```

**时区处理：**

- 使用 `agents.defaults.userTimezone`（如果已设置）
- 否则使用主机系统时区
- 确保所有时间计算基于同一时区

### 随机机会机制

**概率分布：**

| chance 值 | 预期行为 | 实际频率 |
|-----------|----------|----------|
| 0.0 | 从不触发 | 0% |
| 0.1 | 10% 概率 | 约 1/10 次 |
| 0.5 | 50% 概率 | 约 1/2 次 |
| 1.0 | 总是触发 | 100% |

**注意事项：**

- 每次 `agent:bootstrap` 运行时独立计算概率
- 不累积，不保存状态
- 在长时间运行的会话中，大数定律会使实际频率趋近于 chance 值

## 完整配置指南

### 启用步骤

**步骤一：启用 hook 系统**

```bash
openclaw hooks enable soul-evil
```

**步骤二：配置参数**

在 `~/.openclaw/openclaw.json` 中添加配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": {
            "at": "21:00",
            "duration": "15m"
          }
        }
      }
    }
  }
}
```

**步骤三：创建备用 SOUL 文件**

在工作区根目录（`SOUL.md` 旁边）创建 `SOUL_EVIL.md`：

```markdown
# Evil SOUL

你是一个恶作剧助手，喜欢用幽默的方式回应用户的请求。

## 核心特征

- 使用轻松幽默的语气
- 偶尔提出俏皮的建议
- 保持友好但有点调皮的态度

## 行为准则

- 永远不要伤害用户
- 保持积极向上的态度
- 在适当的时候加入幽默元素
```

### 配置选项详解

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | 布尔值 | `true` | 是否启用此 hook |
| `file` | 字符串 | `SOUL_EVIL.md` | 备用 SOUL 文件名 |
| `chance` | 数字（0-1） | `0.1` | 随机触发概率 |
| `purge.at` | 字符串（HH:mm） | 必填 | 清除窗口开始时间 |
| `purge.duration` | 持续时间 | 必填 | 窗口持续时长 |

### 持续时间格式

| 格式 | 示例 | 等价值 |
|------|------|--------|
| 秒 | `30s` | 30 秒 |
| 分钟 | `10m` | 10 分钟 |
| 小时 | `1h` | 1 小时 |
| 组合 | `1h30m` | 1 小时 30 分钟 |

### 优先级规则

当清除窗口和随机机会同时配置时：

```
优先级：purge（清除窗口） > chance（随机机会）

示例：
- purge 窗口内：无论 chance 值如何，都触发替换
- purge 窗口外：按 chance 值决定是否触发
```

## 专家思维模型

### 场景设计框架

设计人格切换策略时，考虑以下维度：

```
人格切换策略 = (触发条件) + (切换目标) + (持续时长) + (回退机制)

┌─────────────────────────────────────────────────────────────────┐
│                    策略设计四象限                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   时间驱动               │   事件驱动                           │
│   ─────────────────────┼────────────────────                   │
│   │                     │                                      │
│   │  定时窗口           │   随机概率                           │
│   │  例：22:00-08:00    │   例：10% 概率                       │
│   │                     │                                      │
│   ─────────────────────┼────────────────────                   │
│                                                                 │
│   状态驱动               │   混合模式                           │
│   ─────────────────────┼────────────────────                   │
│   │                     │                                      │
│   │  用户触发           │   条件组合                           │
│   │  例：用户命令       │   例：窗口+概率                      │
│   │                     │                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 性能影响分析

**Hook 执行开销：**

| 阶段 | 开销 | 优化建议 |
|------|------|----------|
| 配置文件解析 | 纳秒级 | 缓存解析结果 |
| 条件判断 | 纳秒级 | 简化逻辑 |
| 文件读取 | 毫秒级 | 预加载 |
| 内容替换 | 纳秒级 | 使用引用 |

**最佳实践：**

- 避免高频配置变更
- 预加载常用 SOUL 文件
- 限制 purge 窗口的频率

## 故障排查

### 常见问题与解决方案

**问题一：hook 未触发**

**排查步骤：**

```bash
# 1. 检查 hook 是否启用
openclaw hooks list

# 2. 检查配置是否正确加载
openclaw config get hooks.internal.entries.soul-evil

# 3. 查看日志确认执行情况
openclaw logs | rg "soul-evil"
```

**可能原因：**

- `enabled` 设置为 `false`
- `SOUL.md` 不在引导列表中
- 配置未重新加载（需要重启网关）

**问题二：purge 窗口不生效**

**排查步骤：**

```bash
# 1. 检查时区设置
date +%Z

# 2. 验证时间格式
openclaw config get hooks.internal.entries.soul-evil.purge

# 3. 手动触发测试
# 将 chance 设置为 1.0 测试基本功能
```

**问题三：SOUL_EVIL.md 文件不存在**

**解决方案：**

```bash
# 检查文件是否存在
ls -la SOUL_EVIL.md

# 如果不存在，创建它
touch SOUL_EVIL.md

# 或者复制 SOUL.md 作为基础
cp SOUL.md SOUL_EVIL.md
```

**日志警告：**

如果 `SOUL_EVIL.md` 缺失或为空，OpenClaw 会记录警告并保留正常的 `SOUL.md`：

```
[warn] soul-evil: SOUL_EVIL.md not found or empty, falling back to SOUL.md
```

### 调试技巧

**启用完整日志：**

```json
{
  "diagnostics": {
    "flags": ["hooks.*"]
  }
}
```

**验证 hook 执行：**

```
[debug] hooks: soul-evil executing
[debug] hooks: soul-evil checking purge window
[debug] hooks: soul-evil purge window active (21:00 - 21:15)
[debug] hooks: soul-evil replacing SOUL.md with SOUL_EVIL.md
[debug] hooks: soul-evil completed
```

## 注意事项与限制

### 重要限制

1. **子代理不受影响**：子代理运行时不包含 `SOUL.md`，hook 对其无效
2. **磁盘文件不变更**：所有替换在内存中完成，不修改磁盘文件
3. **配置需要重启**：更改配置后需要重启网关才能生效
4. **单次引导生效**：hook 只在 `agent:bootstrap` 时执行一次

### 性能注意事项

- 避免过短的 purge 窗口（频繁的状态切换）
- 限制随机机会的频率
- 监控 SOUL 文件的大小（大型文件影响加载时间）

### 安全性考虑

- 确保 `SOUL_EVIL.md` 的内容符合安全策略
- 避免在生产环境使用过高的触发概率
- 定期审查备用 SOUL 文件的内容

## 相关文档

### 内部链接

- [Hook 系统概述](/hooks)
- [系统提示配置](/concepts/system-prompt)
- [AGENTS.md 模板](/reference/templates/AGENTS)
- [SOUL.md 模板](/reference/templates/SOUL)

### 相关配置

- [agents.defaults 配置](/gateway/configuration#agentsdefaultsobject)
- [hooks 配置](/gateway/configuration#hooksobject)
- [userTimezone 设置](/gateway/configuration#agentsdefaultsobject)
