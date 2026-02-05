---
summary: Node + tsx "__name is not a function" 崩溃问题的深度分析与解决方案
read_when:
  - 调试 Node 仅有的开发脚本或 watch 模式故障
  - 调查 OpenClaw 中的 tsx/esbuild 加载器崩溃
  - 理解 esbuild keepNames 辅助函数与 Node 25 的兼容性问题
title: "Node + tsx 崩溃问题诊断与修复"
---

# Node + tsx 运行时崩溃问题诊断与修复

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- 理解 `__name` 辅助函数的作用机制和产生原因
- 识别 Node 25 与 tsx/esbuild 组合时的特定崩溃模式
- 掌握至少三种绕过此问题的临时解决方案

### 进阶目标（建议掌握）

- 分析 esbuild 转换后的代码，理解 `keepNames` 功能的底层实现
- 在 Node LTS（22/24）上复现并验证问题的范围
- 为社区贡献最小复现用例，帮助上游修复此问题

### 专家目标（挑战）

- 为 tsx 或 esbuild 上游提交问题修复补丁
- 设计团队内部的 Node 版本兼容性测试策略
- 评估 Node 25 在生产环境中的采用风险

## 问题概述与影响范围

### 问题描述

通过 Node 运行 OpenClaw 时，使用 `tsx` 在启动过程中出现以下致命错误：

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

此错误导致 OpenClaw 完全无法启动，任何依赖开发脚本的操作都会失败。

### 影响范围分析

**受影响的配置组合：**

| 组件 | 版本 | 状态 |
|------|------|------|
| Node | v25.x（v25.3.0 确认） | ❌ 受影响 |
| Node | v22.x（Homebrew node@22） | ❌ 受影响 |
| tsx | 4.21.0 | ❌ 受影响 |
| 操作系统 | macOS | ❌ 受影响 |
| Bun | 任意版本 | ✅ 正常 |

**业务影响评估：**

- 开发环境：严重阻塞。所有需要 TypeScript 运行时的新开发者无法完成初始设置。
- CI/CD：中等影响。如果 CI 使用 Node + tsx，运行步骤会失败。
- 生产环境：通常不受影响，因为生产环境通常运行编译后的 JavaScript（`dist/` 目录）。

## 根本原因分析

### 技术原理：esbuild 的 keepNames 功能

要理解这个问题，我们需要深入了解 esbuild 的 `keepNames` 功能及其与 Node 25 的交互。

**为什么需要 `keepNames`？**

在现代 JavaScript/TypeScript 开发中，以下场景会丢失函数名称：

```typescript
// 原始代码
const myFunction = () => {
  console.log("Hello");
};

// 经过压缩后（terser/esbuild）
const a = () => { console.log("Hello"); };
// 函数名 'myFunction' 丢失

// 使用 keepNames 后
const a = (function() { return "myFunction"; })() || function() { 
  console.log("Hello"); 
};
// 保留原始函数名信息
```

**`__name` 辅助函数的实现机制：**

esbuild 的 `keepNames` 插件会发出一个 `__name` 辅助函数，并用 `__name(...)` 包装所有需要保留名称的函数定义：

```javascript
// esbuild 转换后的代码（简化示意）
function __name(fn, name) {
  Object.defineProperty(fn, 'name', { value: name, configurable: true });
  return fn;
}

// 使用 __name 包装函数
const myHandler = __name(function myHandler() {
  // 处理逻辑
}, "myHandler");
```

### 为什么会失败：Node 25 的加载器变更

**问题链路：**

1. tsx 使用 esbuild 转换 TypeScript/ESM 代码
2. esbuild 的 `keepNames` 选项会注入 `__name` 辅助函数
3. Node 25 引入新的模块加载器架构
4. 在特定条件下，`__name` 辅助函数在运行时缺失或被覆盖
5. 调用 `__name(...)` 时，它存在但不是函数，导致 TypeError

**技术假设（待验证）：**

- Node 25 的 ES Module 加载器处理顺序可能与之前版本不同
- `__name` 辅助函数可能在模块解析的某个阶段被错误地标记或过滤
- 类似的 `__name` 辅助函数问题在其他 esbuild 使用者中也有报告（GitHub Issue #1031）

## 环境信息与复现步骤

### 环境要求

| 组件 | 要求版本 | 说明 |
|------|----------|------|
| Node | v25.x 或 v22.x | v25.3.0 和 v22.22.0 已确认受影响 |
| tsx | 4.21.0 | 当前使用的版本 |
| pnpm | 任意兼容版本 | 用于依赖管理 |
| 操作系统 | macOS | 在其他平台上可能也能复现 |

### 复现步骤

**方法一：使用 OpenClaw 源码复现**

```bash
# 步骤 1：克隆并进入仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 步骤 2：安装依赖
pnpm install

# 步骤 3：使用 Node + tsx 运行（会失败）
node --import tsx src/entry.ts status

# 预期输出：
# [openclaw] Failed to start CLI: TypeError: __name is not a function
```

**方法二：运行最小复现用例**

```bash
# 使用仓库中提供的最小复现脚本
node --import tsx scripts/repro/tsx-name-repro.ts
```

### 版本兼容性矩阵

| Node 版本 | tsx 4.21.0 | Bun | 状态 |
|-----------|------------|-----|------|
| v25.3.0 | ❌ 失败 | ✅ 正常 | Node 25 回归 |
| v22.22.0 | ❌ 失败 | ✅ 正常 | Node 22 受影响 |
| v24.x | ⚠️ 未知 | ✅ 正常 | 待测试 |
| v20.x | ✅ 正常 | ✅ 正常 | 兼容 |

## 专家思维模型：回归分析框架

### 回归历史追踪

**关键提交：**

| 提交哈希 | 日期 | 变更内容 | 影响 |
|----------|------|----------|------|
| `2871657e` | 2026-01-06 | 脚本从 Bun 改为 tsx | 引入问题 |
| 之前 | 2026-01-06 之前 | Bun 作为唯一运行时 | 正常 |

**变更动机：**

将开发脚本从 Bun 切换到 tsx 是为了「使 Bun 成为可选项」，允许开发者使用自己偏好的运行时环境。这个决策本身是合理的，但意外地暴露了 Node + tsx 组合的兼容性问题。

### 决策分析：为什么这个问题没有提前发现？

**测试覆盖盲区：**

1. **运行时组合测试缺失**：CI 可能只测试 Bun 路径
2. **Node 版本测试滞后**：Node 25 较新，测试矩阵未及时更新
3. **最小复现环境不完整**：开发者的个人环境可能恰好绕过了问题

**教训：**

- 每次引入新的依赖组合时，应运行完整的兼容性测试
- 保持测试矩阵与最新稳定版本同步
- 对关键路径维护最小复现脚本

## 临时解决方案

根据你的环境和需求，选择以下解决方案之一：

### 方案一：回退到 Bun（推荐用于开发）

这是最简单的解决方案，不需要修改任何配置：

```bash
# 使用 Bun 运行开发脚本
bun run src/entry.ts status

# 或者使用 pnpm，它会自动调用正确的运行时
pnpm openclaw status
```

**优点：**
- 立即可用，无需配置更改
- Bun 通常性能更好
- 与 OpenClaw 团队测试环境一致

**缺点：**
- 需要安装 Bun

### 方案二：使用 tsc watch + Node 运行编译产物

此方案使用 TypeScript 编译器的 watch 模式：

```bash
# 终端 1：启动 tsc watch 模式
pnpm exec tsc --watch --preserveWatchOutput

# 终端 2：使用 Node 运行编译产物
node --watch openclaw.mjs status
```

**优点：**
- 不依赖 tsx
- 与生产环境运行方式一致

**缺点：**
- 需要两个终端
- 编译产物可能有轻微延迟

### 方案三：直接编译后运行（一次性验证）

如果你只需要运行一次而不是开发：

```bash
# 步骤 1：编译 TypeScript
pnpm exec tsc -p tsconfig.json

# 步骤 2：运行编译产物
node openclaw.mjs status
```

**优点：**
- 最接近生产环境的测试
- 验证编译过程本身是否正常

### 方案四：等待上游修复（长期方案）

如果你愿意等待：

1. 测试 Node LTS（22/24）以确认问题范围
2. 关注 tsx 和 esbuild 的更新
3. 当修复发布后，更新依赖版本

**跟踪资源：**
- [esbuild GitHub Issue #1031](https://github.com/evanw/esbuild/issues/1031)
- [tsx GitHub Issues](https://github.com/privatenumber/tsx/issues)
- [Node.js Issue Tracker](https://github.com/nodejs/node/issues)

## 故障排查清单

当你遇到此问题时，请按以下顺序检查：

### 检查清单

- [ ] 确认 Node 版本：`node --version`
- [ ] 确认 tsx 版本：`npx tsx --version`
- [ ] 尝试使用 Bun 运行：`bun run <script>`
- [ ] 查看完整的错误堆栈
- [ ] 检查是否有其他模块也使用了 `keepNames`
- [ ] 尝试清除缓存：`rm -rf node_modules/.cache`

### 诊断命令

```bash
# 1. 版本诊断
echo "Node: $(node --version)"
echo "tsx: $(npx tsx --version)"

# 2. 最小复现测试
node --import tsx scripts/repro/tsx-name-repro.ts

# 3. 完整构建测试
pnpm build && node dist/openclaw.mjs status

# 4. Bun 对比测试
bun run src/entry.ts status
```

## 适用场景分析

### 应该使用哪种方案？

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| 新开发者加入 | 方案一（Bun） | 最快开始开发 |
| CI/CD 环境 | 方案二（tsc watch） | 与构建流程一致 |
| 生产部署 | 方案三（预编译） | 验证构建产物 |
| 长期修复 | 方案四（等待上游） | 根本解决问题 |

### 未来迁移计划

当上游修复发布后：

1. 更新 tsx 和 esbuild 到修复版本
2. 在所有支持的 Node 版本上运行测试
3. 更新 CI 配置以覆盖 Node LTS
4. 更新文档说明新版本要求

## 进阶阅读

### 相关技术主题

- **esbuild API**：[esbuild 官方文档](https://esbuild.github.io/api/#keep_names)
- **Node.js 模块系统**：[Node.js 模块文档](https://nodejs.org/api/esm.html)
- **TypeScript 编译器选项**：[tsc 配置参考](https://www.typescriptlang.org/tsconfig)

### 类似问题的解决参考

- [OpenNext.js keepNames 说明](https://opennext.js.org/cloudflare/howtos/keep_names)
- [esbuild 社区讨论](https://github.com/evanw/esbuild/issues/1031)

## 下一步行动

### 立即行动（本周）

- [ ] 确定你的环境是否受影响
- [ ] 选择并应用临时解决方案
- [ ] 如果可能，测试 Node LTS（22/24）以帮助确认问题范围

### 中期计划（本月）

- [ ] 跟踪上游修复进展
- [ ] 在团队内部同步此问题及解决方案
- [ ] 考虑是否需要在 CI 中增加 Node + tsx 的测试覆盖

### 长期目标（下季度）

- [ ] 验证 Node 25 在生产环境中的兼容性
- [ ] 更新 OpenClaw 的 Node 版本要求文档
- [ ] 为团队建立 Node 版本兼容性测试流程

## 参考资料

### 外部链接

- [esbuild keepNames API](https://esbuild.github.io/api/#keep_names)
- [esbuild GitHub Issue #1031](https://github.com/evanw/esbuild/issues/1031)
- [OpenNext.js keepNames 指南](https://opennext.js.org/cloudflare/howtos/keep_names)
- [tsx GitHub 仓库](https://github.com/privatenumber/tsx)
- [Node.js 官方文档](https://nodejs.org)

### 相关文档

- [OpenClaw 开发环境配置](/platforms/mac/dev-setup)
- [从源码构建 OpenClaw](/install/from-source)
- [故障排查指南](/troubleshooting)
