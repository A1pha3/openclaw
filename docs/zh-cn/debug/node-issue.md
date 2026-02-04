---
summary: Node + tsx "__name is not a function" 崩溃问题说明和解决方法
read_when:
  - 调试 Node 仅有的开发脚本或 watch 模式故障
  - 调查 OpenClaw 中的 tsx/esbuild 加载器崩溃
title: "Node + tsx 崩溃问题"
---

# Node + tsx "__name is not a function" 崩溃

## 问题概述

通过 Node 运行 OpenClaw 时，使用 `tsx` 在启动过程中出现以下错误：

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

此问题始于将开发脚本从 Bun 切换到 `tsx`（提交 `2871657e`，2026-01-06）。相同的运行路径在 Bun 下可以正常工作。

## 环境信息

- Node：v25.x（在 v25.3.0 上观察到）
- tsx：4.21.0
- 操作系统：macOS（在其他运行 Node 25 的平台上可能也能复现）

## 复现步骤（Node 仅）

```bash
# 在仓库根目录下
node --version
pnpm install
node --import tsx src/entry.ts status
```

## 仓库中的最小复现

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```

## Node 版本检查

- Node 25.3.0：失败
- Node 22.22.0（Homebrew `node@22`）：失败
- Node 24：尚未安装，需要验证

## 笔记 / 假设

- `tsx` 使用 esbuild 转换 TS/ESM。esbuild 的 `keepNames` 会发出一个 `__name` 辅助函数，并用 `__name(...)` 包装函数定义。
- 崩溃表明 `__name` 存在但在运行时不是函数，这意味着该辅助函数在 Node 25 加载器路径中缺失或被覆盖。
- 类似的 `__name` 辅助函数问题在其他 esbuild 使用者中也有报告，当辅助函数缺失或被重写时会出现。

## 回归历史

- `2871657e`（2026-01-06）：脚本从 Bun 改为 tsx，以使 Bun 成为可选项。
- 之前（Bun 路径），`openclaw status` 和 `gateway:watch` 正常工作。

## 临时解决方案

- 使用 Bun 运行开发脚本（当前的临时回滚）。
- 使用 Node + tsc watch，然后运行编译后的输出：
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- 本地确认：`pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` 在 Node 25 上可以正常工作。
- 如果可能，在 TS 加载器中禁用 esbuild keepNames（防止 `__name` 辅助函数插入）；tsx 当前不暴露此选项。
- 使用 `tsx` 测试 Node LTS（22/24）以确认问题是否特定于 Node 25。

## 参考资料

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep_names
- https://github.com/evanw/esbuild/issues/1031

## 下一步

- 在 Node 22/24 上复现以确认 Node 25 回归问题。
- 测试 `tsx` nightly 或固定到早期版本（如果存在已知的回归问题）。
- 如果在 Node LTS 上可以复现，使用 `__name` 堆栈跟踪向上游提交最小复现。
