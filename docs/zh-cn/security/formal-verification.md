---
summary: OpenClaw 形式化验证安全模型——使用 TLA+/TLC 进行机器检查的安全回归测试
read_when:
  - 理解 OpenClaw 的安全模型和验证方法
  - 复现形式化验证结果
  - 为 OpenClaw 贡献新的安全声明
title: "OpenClaw 形式化验证完全指南"
---

# OpenClaw 形式化验证完全指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解形式化验证在 OpenClaw 安全模型中的角色
- 掌握运行 TLC 模型检查器的基本方法
- 理解安全声明和负模型的概念

### 进阶目标（建议掌握）

- 理解 TLA+ 规范语言的基本语法
- 能够阅读和理解现有的安全模型
- 评估模型检查结果的有效性

### 专家目标（挑战）

- 为新的安全功能编写 TLA+ 规范
- 设计负模型以验证错误检测能力
- 优化模型以提高检查效率

## 核心概念与设计理念

### 什么是形式化验证？

形式化验证是使用数学方法验证系统正确性的技术。与传统测试不同，形式化验证可以系统地检查所有可能的系统状态，从而发现边缘情况和潜在漏洞。

**对比传统测试与形式化验证：**

| 对比维度 | 传统测试 | 形式化验证 |
|----------|----------|------------|
| 覆盖范围 | 有限样本 | 所有可达状态 |
| 发现的bug | 已执行的路径 | 所有可能的路径 |
| 证明力度 | 可能遗漏 | 有穷状态空间上的穷举 |
| 适用范围 | 集成测试 | 核心安全逻辑 |

### OpenClaw 的验证策略

OpenClaw 采用「可执行的、有攻击者驱动的安全回归套件」作为其形式化验证策略：

**核心思想：**

> 不是证明「系统在所有方面都是安全的」，而是验证「系统强制执行其预期的安全策略」。

**验证层次：**

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw 验证层次                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   声明层（人类可读）                                             │
│   └── 「Nodes.run 需要节点命令允许列表」                         │
│                                                                 │
│   模型层（TLA+ 规范）                                           │
│   └── 定义状态机、转换规则、不变式                              │
│                                                                 │
│   检查层（TLC 执行）                                           │
│   └── 穷举状态空间，验证不变式                                 │
│                                                                 │
│   证据层（反例跟踪）                                           │
│   └── 失败时的详细执行路径                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 声明的构成要素

每个安全声明包含以下部分：

| 要素 | 说明 | 示例 |
|------|------|------|
| **声明描述** | 用自然语言描述安全策略 | Nodes.run 需要声明的命令 |
| **前置条件** | 声明成立的前提 | 配置存在、命令被允许 |
| **后置条件** | 期望的状态 | 命令执行并记录 |
| **模型假设** | 明确的限制 | 仅检查有限状态空间 |

## 安全模型架构

### 模型仓库结构

所有 TLA+ 模型保存在独立的仓库中：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)

**仓库结构：**

```
openclaw-formal-models/
├── docs/
│   └── gateway-exposure-matrix.md
├── models/
│   ├── gateway-exposure-v2/
│   │   ├── gateway-exposure-v2.tla
│   │   └── MC.cfg
│   ├── nodes-pipeline/
│   │   ├── nodes-pipeline.tla
│   │   └── MC.cfg
│   └── pairing/
│       ├── pairing.tla
│       └── MC.cfg
├── bin/
│   └── tlc
├── Makefile
└── README.md
```

### 验证领域

OpenClaw 的形式化验证覆盖以下安全领域：

**1. 网关暴露与认证**

| 声明 | 描述 | 风险等级 |
|------|------|----------|
| gateway-exposure-v2 | 网关绑定到非 loopback 时的安全风险 | 高 |
| gateway-exposure-v2-protected | 令牌/密码认证的保护效果 | 高 |

**2. 节点命令管道**

| 声明 | 描述 | 风险等级 |
|------|------|----------|
| nodes-pipeline | 节点命令的允许列表机制 | 极高 |
| approvals-token | 批准令牌化防止重放攻击 | 高 |

**3. 配对存储**

| 声明 | 描述 | 风险等级 |
|------|------|----------|
| pairing | 配对请求的 TTL 和并发控制 | 中 |
| pairing-cap | 待处理请求上限 | 中 |

**4. 入口门控**

| 声明 | 描述 | 风险等级 |
|------|------|----------|
| ingress-gating | 控制命令无法绕过提及门控 | 高 |

**5. 路由隔离**

| 声明 | 描述 | 风险等级 |
|------|------|----------|
| routing-isolation | DM 会话隔离 | 高 |

## 运行环境配置

### 系统要求

| 要求 | 详情 |
|------|------|
| Java | 11 或更高版本（TLC 运行在 JVM 上） |
| 内存 | 至少 4GB RAM（复杂模型可能需要更多） |
| 磁盘 | 至少 500MB 可用空间 |

### 克隆与构建

**克隆仓库：**

```bash
# 克隆形式化模型仓库
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 查看可用目标
make
```

**TLC 工具安装：**

仓库已经内置了固定版本的 `tla2tools.jar`：

```bash
# 直接使用仓库提供的脚本
./bin/tlc --help
```

### 运行模型检查

**运行所有模型：**

```bash
make all
```

**运行单个模型：**

```bash
make gateway-exposure-v2
make nodes-pipeline
make pairing
```

## 模型详解

### 网关暴露模型

**声明：** 在没有认证的情况下绑定到 loopback 以外的范围可能会导致远程妥协的可能性增加。

**模型假设：**

- 攻击者可以从网络访问网关
- 网关可能绑定到公开地址
- 可能存在或不存在认证层

**运行验证：**

```bash
# 基础验证（应通过）
make gateway-exposure-v2

# 带认证保护（应通过）
make gateway-exposure-v2-protected

# 负模型（预期失败，用于验证测试框架）
make gateway-exposure-v2-negative
```

**预期结果：**

| 模型 | 预期状态 | 说明 |
|------|----------|------|
| gateway-exposure-v2 | 绿色（通过） | 声明在模型中成立 |
| gateway-exposure-v2-protected | 绿色（通过） | 认证提供保护 |
| gateway-exposure-v2-negative | 红色（预期） | 验证反例生成能力 |

### 节点命令管道模型

**声明：** `nodes.run` 需要（a）节点命令允许列表加上声明的命令，以及（b）配置时的实时批准。

**模型结构：**

```
┌─────────────────────────────────────────────────────────────────┐
│                    节点命令管道模型                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   输入                                                         │
│   ├── 节点命令请求                                              │
│   ├── 允许列表配置                                             │
│   └── 批准令牌                                                 │
│                                                                 │
│   处理阶段                                                     │
│   ├── 阶段 1：命令检查                                         │
│   │      └── 命令是否在允许列表中？                            │
│   │                                                             │
│   ├── 阶段 2：批准验证                                         │
│   │      └── 是否有有效批准令牌？                              │
│   │                                                             │
│   └── 阶段 3：执行                                             │
│          └── 执行已批准的命令                                   │
│                                                                 │
│   不变式                                                         │
│   └── 只有通过所有检查的命令才会执行                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**运行验证：**

```bash
# 完整管道验证
make nodes-pipeline

# 批准令牌机制验证
make approvals-token

# 负模型验证
make nodes-pipeline-negative
make approvals-token-negative
```

### 配对存储模型

**声明：** 配对请求遵守 TTL 和待处理请求上限。

**并发控制：**

| 机制 | 说明 | 验证目标 |
|------|------|----------|
| MaxPending | 待处理请求上限 | 不会超过限制 |
| TTL | 请求有效期 | 自动过期 |
| 幂等性 | 重复请求不创建重复 | 相同请求处理一次 |

**运行验证：**

```bash
# 基础配对模型
make pairing

# 配对上限
make pairing-cap

# 并发安全
make pairing-race

# 幂等性
make pairing-idempotency

# 负模型
make pairing-negative
make pairing-race-negative
```

## 专家思维模型

### 状态空间分析框架

```
┌─────────────────────────────────────────────────────────────────┐
│                    状态空间分析方法                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   步骤 1：定义状态                                               │
│   ├── 识别系统中的所有实体                                      │
│   ├── 定义每个实体的可能状态                                     │
│   └── 列出所有状态组合                                          │
│                                                                 │
│   步骤 2：定义转换                                               │
│   ├── 识别所有可能的操作                                        │
│   ├── 定义每个操作的前置条件和后置条件                           │
│   └── 列出所有合法的转换                                        │
│                                                                 │
│   步骤 3：识别不变式                                             │
│   ├── 定义必须始终为真的属性                                     │
│   └── 识别安全关键的不变式                                       │
│                                                                 │
│   步骤 4：执行检查                                               │
│   ├── 使用 TLC 穷举所有状态                                      │
│   ├── 验证所有不变式                                            │
│   └── 分析反例（如有）                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 负模型设计

**负模型的目的是验证测试框架的有效性：**

```
正模型（应通过）：验证声明在正常条件下成立
负模型（应失败）：验证错误条件会产生反例

┌─────────────────────────────────────────────────────────────────┐
│                    正负模型对比                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   正模型                                                       │
│   ────────────────────                                         │
│   输入：合法的配置和请求                                         │
│   期望：所有不变式保持                                          │
│   结果：TLC 报告无反例                                          │
│                                                                 │
│   负模型                                                       │
│   ────────────────────                                         │
│   输入：经过设计的问题场景（故意违规）                           │
│   期望：应该发现不变式违反                                      │
│   结果：TLC 报告反例（符合预期）                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 不变式类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **安全不变式** | 必须始终成立 | 「未经授权的命令不能执行」 |
| **活性不变式** | 最终必须成立 | 「每个请求最终都会得到响应」 |
| **事务不变式** | 事务边界保持 | 「转账前后总额不变」 |

## v1++ 扩展模型

### 并发与重试模型

**关注点：**

- 非原子更新的处理
- 重试策略的正确性
- 消息扇出的可靠性

**入口跟踪关联模型：**

```markdown
声明：摄取应该在扇出时保持跟踪关联，并且在提供者重试时是幂等的。

含义：
├── 事件分解后保持相同的跟踪 ID
├── 重试不会导致双重处理
└── 事件 ID 缺失时有安全的回退策略
```

**运行验证：**

```bash
# 跟踪关联
make ingress-trace
make ingress-trace2

# 幂等性
make ingress-idempotency

# 重复数据删除回退
make ingress-dedupe-fallback

# 负模型
make ingress-trace-negative
make ingress-idempotency-negative
```

### 路由优先级模型

**声明：** 路由必须默认保持 DM 会话隔离，并且只有在明确配置时才合并会话。

**关键不变式：**

| 不变式 | 描述 |
|--------|------|
| dmScope 优先级 | 通道特定配置优先于全局默认 |
| identityLinks | 只在链接组内合并会话 |

**运行验证：**

```bash
# 优先级验证
make routing-precedence
make routing-identitylinks

# 负模型
make routing-precedence-negative
make routing-identitylinks-negative
```

## 结果解读

### TLC 输出分析

**成功输出：**

```
TLC2 Version 2.19 of 18 July 2023
...
Model checking completed. No errors found.
Total duration: 00:02:34
States generated: 1,234,567
```

**失败输出：**

```
TLC2 Version 2.19 of 18 July 2023
...
Error: Invariant SecurityInvariant is violated.
State 123:
  variable1 = "value1"
  variable2 = "value2"
...
```

### 局限性说明

**重要提醒：**

1. **模型不是实现**：TLA+ 模型是对实现的抽象，可能存在差异
2. **状态空间限制**：TLC 探索的状态空间受限于配置，可能不完整
3. **假设依赖**：结果依赖于明确的模型假设

**不在验证范围内：**

- 完整的 TypeScript 实现正确性证明
- 性能相关的验证
- 未建模的外部依赖行为

## 贡献指南

### 添加新模型

**步骤一：定义声明**

用自然语言描述要验证的安全策略。

**步骤二：编写 TLA+ 规范**

```tla
---------------------------- MODULE MyModule ----------------------------
EXTENDS Naturals, Sequences

VARIABLES
    state, \* 系统状态
    approved  \* 批准状态

Init ==
    /\ state = "idle"
    /\ approved = FALSE

Transition ==
    /\ state = "idle"
    /\ state' = "executing"
    /\ approved' = approved

SecurityInvariant ==
    state /= "executing" \/ approved = TRUE
=============================================================================
```

**步骤三：创建 MC.cfg 配置**

```
\* MC.cfg
INIT Init
NEXT Next
INVARIANT SecurityInvariant
CONSTRAINT
```

**步骤四：添加到 Makefile**

```makefile
my-module:
    ./bin/tlc -model models/my-module/my-module.tla \
              -config models/my-module/MC.cfg
```

### 验证清单

**提交前的检查清单：**

- [ ] 正模型运行通过
- [ ] 负模型产生预期反例
- [ ] 文档描述清晰
- [ ] 变量命名一致
- [ ] 注释说明关键设计决策

## 常见问题

### 问题一：TLC 内存不足

**症状：**

```
java.lang.OutOfMemoryError: Java heap space
```

**解决方案：**

```bash
# 增加 JVM 堆内存
export JAVA_OPTS="-Xmx8g"
./bin/tlc -model model.tla -config MC.cfg
```

### 问题二：模型检查时间过长

**症状：**

```
Running for several hours without completion...
```

**优化策略：**

1. 缩小状态空间：
   ```
   减少 CONSTANT 值的范围
   ```

2. 并行化：
   ```
   ./bin/tlc -workers 4 -model model.tla
   ```

3. 使用看护条件：
   ```
   只检查reachable states
   ```

### 问题三：反例无法重现

**症状：**

```
TLC reports violation but we cannot reproduce it
```

**排查步骤：**

1. 检查 TLC 版本是否一致
2. 验证随机种子
3. 审查反例跟踪

## 相关文档

### 内部资源

- [网关安全配置](/gateway/security)
- [通道安全指南](/channels/security)
- [安全最佳实践](/security/best-practices)

### 外部资源

- [TLA+ 入门教程](https://lamport.azurewebsites.net/tla/tutorial/home.html)
- [TLC 模型检查器](https://lamport.azurewebsites.net/tla/model-value.html)
- [Learn TLA+](https://learntla.com/)

### 相关项目

- [OpenClaw 形式化模型仓库](https://github.com/vignesh07/openclaw-formal-models)
- [TLA+ 工具箱](https://lamport.azurewebsites.net/tla/toolbox.html)
