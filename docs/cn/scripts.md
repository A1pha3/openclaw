---
summary: "仓库脚本使用指南：scripts/ 目录的目的、约定与自定义开发"
read_when:
  - 需要运行仓库中的脚本时
  - 理解 scripts/ 目录的结构和用途时
  - 在 scripts/ 目录下添加或修改脚本时
title: "脚本（Scripts）"
---

# 仓库脚本使用指南

## 学习目标

完成本章节学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解 `scripts/` 目录的定位和核心用途
- [ ] 掌握脚本约定的基本原则
- [ ] 能够运行仓库提供的标准脚本
- [ ] 理解脚本与 CLI 命令的关系

### 进阶目标（建议掌握）

- [ ] 理解脚本的安全边界和执行要求
- [ ] 能够创建符合规范的自定义脚本
- [ ] 掌握脚本调试和故障排查方法

### 专家目标（挑战）

- [ ] 能够设计高效的脚本工作流
- [ ] 理解脚本与 CI/CD 系统的集成

---

## 第一部分：scripts/ 目录概述

### 1.1 什么是 scripts/ 目录？

**核心定义**：`scripts/` 目录包含用于**本地工作流程**和**运维任务**的辅助脚本。这些脚本是**可选的**，仅在特定场景下使用。

**设计哲学**：脚本是对 CLI 命令的补充。当任务**明确与脚本关联**（如一次性迁移、复杂初始化）时使用脚本；当任务需要**重复执行**或需要**标准化接口**时优先使用 CLI。

```
工具选择决策树：

    需要执行任务？
            │
            ▼
    ┌───────────────────────┐
    │ 任务是一次性的吗？    │
    └───────────┬───────────┘
                │
        ┌───────┴───────┐
        ▼               ▼
       是              否
        │               │
        ▼               ▼
    ┌───────────┐   ┌───────────────────────┐
    │ 使用脚本   │   │ CLI 命令存在吗？      │
    └───────────┘   └───────────┬───────────┘
                                │
                        ┌───────┴───────┐
                        ▼               ▼
                       是              否
                        │               │
                        ▼               ▼
                    ┌───────────┐   ┌───────────────────┐
                    │ 使用 CLI   │   │ 优先添加 CLI 命令  │
                    └───────────┘   └───────────────────┘
```

### 1.2 脚本的定位

| 维度 | 脚本 | CLI 命令 |
|------|------|----------|
| **使用频率** | 通常一次性或低频 | 日常使用 |
| **用户接口** | 直接执行脚本文件 | `openclaw <command>` |
| **灵活性** | 高度可定制 | 标准化接口 |
| **维护性** | 可能缺乏文档 | 通常有完整文档 |
| **依赖** | 可能依赖特定环境 | 通常独立运行 |

**核心原则**：当 CLI 表面存在时，优先使用 CLI 命令。例如，身份认证监控应使用 `openclaw models status --check` 而不是手动执行监控脚本。

### 1.3 目录结构

```
scripts/ 目录结构：

scripts/
├── setup-git-hooks.js      # Git hooks 设置脚本
├── format-staged.js        # 预提交格式化脚本
├── committer               # 提交消息生成脚本
├── update-clawtributors.ts # 更新贡献者脚本
├── bundle-a2ui.sh          # A2UI 打包脚本
├── package-mac-app.sh      # macOS 应用打包脚本
├── restart-mac.sh          # macOS 重启脚本
├── clawlog.sh              # macOS 日志查询脚本
├── ...
└── README.md               # 脚本目录说明（本文件）
```

---

## 第二部分：核心脚本详解

### 2.1 Git Hooks 脚本

#### 2.1.1 setup-git-hooks.js

**功能**：设置 Git hooks，将仓库的 hooks 链接到 `.git/hooks/`。

**使用方式**：

```bash
# 执行脚本
node scripts/setup-git-hooks.js

# 或直接运行
chmod +x scripts/setup-git-hooks.js
./scripts/setup-git-hooks.js
```

**工作原理**：

```
Git Hooks 设置流程：

    ┌─────────────────────────────────────────┐
    │  检测 .git/hooks/ 目录                   │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  链接 scripts/* 到 .git/hooks/*          │
    │  pre-commit → scripts/format-staged.js   │
    │  commit-msg → scripts/committer         │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  输出设置结果                             │
    └─────────────────────────────────────────┘
```

**常见问题**：

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 链接失败 | 权限不足 | 确保在仓库根目录执行 |
| Hooks 不执行 | Git 配置禁用 hooks | 检查 `core.hooksPath` 设置 |
| 重复链接 | 已存在同名 hook | 手动删除后重新执行 |

#### 2.1.2 format-staged.js

**功能**：对暂存的 `src/` 和 `test/` 文件执行预提交格式化。

**使用方式**：

```bash
# 自动作为 pre-commit hook 执行
git commit -m "message"

# 手动执行
node scripts/format-staged.js
```

**执行流程**：

```
预提交格式化流程：

    ┌─────────────────────────────────────────┐
    │  获取暂存文件列表                         │
    │  git diff --cached --name-only          │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  过滤 src/ 和 test/ 目录                 │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  对每个文件执行格式化                     │
    │  pnpm oxfmt <file>                       │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  重新暂存格式化后的文件                   │
    │  git add <files>                         │
    └─────────────────────────────────────────┘
```

### 2.2 提交脚本

#### 2.2.1 committer

**功能**：生成规范化的提交消息，支持自动关联文件和任务。

**使用方式**：

```bash
# 基本使用
./scripts/committer "提交消息" <file...>

# 带文件
./scripts/committer "fix: 修复登录bug" src/auth/login.ts

# 多文件
./scripts/committer "feat: 添加用户配置功能" src/user/*.ts
```

**输出示例**：

```
$ ./scripts/committer "feat: 添加用户配置功能" src/user/config.ts

✓ 暂存文件：src/user/config.ts
✓ 生成提交消息：
feat: 添加用户配置功能

提交命令：
git commit -m "feat: 添加用户配置功能"
```

**约定格式**：

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: 添加用户配置功能` |
| `fix` | Bug 修复 | `fix: 修复登录bug` |
| `docs` | 文档更新 | `docs: 更新安装指南` |
| `style` | 格式调整 | `style: 代码格式化` |
| `refactor` | 重构 | `refactor: 重构认证模块` |
| `test` | 测试 | `test: 添加登录测试用例` |
| `chore` | 维护 | `chore: 更新依赖版本` |

### 2.3 macOS 专用脚本

#### 2.3.1 clawlog.sh

**功能**：查询 macOS 统一日志系统中的 OpenClaw 相关日志。

**使用方式**：

```bash
# 基本查询（最近 100 条）
./scripts/clawlog.sh

# 实时追踪
./scripts/clawlog.sh --follow

# 按类别筛选
./scripts/clawlog.sh --category Gateway
./scripts/clawlog.sh --category Agent
./scripts/clawlog.sh --category Channel

# 指定时间范围
./scripts/clawlog.sh --since "1 hour ago"
./scripts/clawlog.sh --since "today"

# 查看错误日志
./scripts/clawlog.sh --error

# 组合使用
./scripts/clawlog.sh --follow --category Gateway --error
```

**参数说明**：

| 参数 | 说明 | 示例 |
|------|------|------|
| 无参数 | 显示最近 100 条日志 | `./scripts/clawlog.sh` |
| `--follow`, `-f` | 实时追踪日志 | `./scripts/clawlog.sh -f` |
| `--category`, `-c` | 按类别筛选 | `./scripts/clawlog.sh -c Gateway` |
| `--since` | 指定时间范围 | `./scripts/clawlog.sh --since "today"` |
| `--error`, `-e` | 只显示错误 | `./scripts/clawlog.sh -e` |
| `--help`, `-h` | 显示帮助 | `./scripts/clawlog.sh -h` |

**使用要求**：

- 操作系统：macOS
- 权限：需要 `sudo` 权限执行 `/usr/bin/log`
- 密码：可能需要输入 sudo 密码

**输出示例**：

```
$ ./scripts/clawlog.sh --category Gateway --error

Password:
timestamp                       [类型]    消息
────────────────────────────────────────────────────
2024-01-15 10:30:15  [Error]   Gateway: Failed to connect to Telegram
2024-01-15 10:28:03  [Error]   Gateway: Configuration validation failed
2024-01-15 10:15:22  [Error]   Gateway: Port 18789 already in use
```

#### 2.3.2 package-mac-app.sh

**功能**：打包 OpenClaw macOS 应用。

**使用方式**：

```bash
# 基本打包（当前架构）
./scripts/package-mac-app.sh

# 指定架构
./scripts/package-mac-app.sh --arch arm64    # Apple Silicon
./scripts/package-mac-app.sh --arch x64     # Intel

# 输出目录
./scripts/package-mac-app.sh --output /path/to/output
```

**输出文件**：

```
dist/
└── OpenClaw-<version>-<arch>.dmg
```

**使用要求**：

- macOS 操作系统
- Xcode 命令行工具
- 有效的代码签名证书（用于发布版本）
- Apple 开发者账号（用于公证）

**故障排除**：

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 签名失败 | 证书问题 | 检查钥匙串访问中的证书 |
| 公证失败 | 权限问题 | 检查 Apple Developer 账号 |
| 权限错误 | 文件权限不足 | 确保脚本可执行：`chmod +x` |

#### 2.3.3 restart-mac.sh

**功能**：重启 OpenClaw macOS 应用及其相关服务。

**使用方式**：

```bash
# 基本重启
./scripts/restart-mac.sh

# 强制重启（杀掉旧进程）
./scripts/restart-mac.sh --force

# 只重启 Gateway
./scripts/restart-mac.sh --gateway-only
```

### 2.4 其他脚本

#### 2.4.1 update-clawtributors.ts

**功能**：更新 README 中的贡献者头像列表。

**使用方式**：

```bash
# 基本执行
pnpm update-clawtributors

# 或直接运行
bun scripts/update-clawtributors.ts
```

**工作原理**：

```
贡献者更新流程：

    ┌─────────────────────────────────────────┐
    │  从 GitHub API 获取贡献者列表             │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  获取每个贡献者的头像 URL                 │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  更新 README.md 中的贡献者头像区域       │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  提交更改（如果有）                       │
    └─────────────────────────────────────────┘
```

#### 2.4.2 bundle-a2ui.sh

**功能**：打包 A2UI（Canvas 用户界面组件）。

**使用方式**：

```bash
# 基本打包
./scripts/bundle-a2ui.sh

# 开发模式（不压缩）
./scripts/bundle-a2ui.sh --dev

# 指定输出
./scripts/bundle-a2ui.sh --output /path/to/output
```

---

## 第三部分：脚本约定与规范

### 3.1 脚本命名约定

| 约定 | 说明 | 示例 |
|------|------|------|
| 扩展名 | 根据语言选择 | `.js`、`.ts`、`.sh` |
| 可执行权限 | Shell 脚本需要 | `chmod +x script.sh` |
| 命名风格 | 描述性强 | `setup-git-hooks.js` |
| 国际化 | 支持中文参数说明 | 脚本内使用中文输出 |

### 3.2 脚本结构规范

**推荐结构**：

```typescript
// 1. Shebang（如果是 Shell 脚本）
#!/usr/bin/env bash

// 2. 脚本说明
/**
 * 脚本名称
 *
 * 功能描述...
 *
 * 使用方式：
 *   ./script.ts [--option] <argument>
 *
 * 示例：
 *   ./script.ts --verbose
 */

// 3. 依赖检查
function check_dependencies() {
  // 检查必要命令是否存在
}

// 4. 参数解析
function parse_args() {
  // 解析命令行参数
}

// 5. 主逻辑
function main() {
  check_dependencies
  parse_args "$@"
  // 执行主要逻辑
}

// 6. 入口
main "$@"
```

### 3.3 输出规范

脚本应该提供清晰的输出：

| 输出类型 | 格式 | 场景 |
|----------|------|------|
| 成功 | 绿色 ✓ 或简洁消息 | 操作完成 |
| 错误 | 红色 ✗ + 错误信息 | 操作失败 |
| 警告 | 黄色 ⚠️ + 警告信息 | 需要注意 |
| 进度 | 进度条或状态消息 | 长时间操作 |
| 调试 | 详细输出（可选） | 排错时使用 |

### 3.4 错误处理

```
错误处理最佳实践：

    ┌─────────────────────────────────────────┐
    │  脚本开始                                │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  错误检查点                              │
    │  ├─ 依赖是否安装？                        │
    │  ├─ 参数是否正确？                        │
    │  ├─ 文件是否存在？                        │
    │  └─ 权限是否足够？                        │
    └─────────────────┬───────────────────────┘
                      ▼
    ┌─────────────────────────────────────────┐
    │  错误时退出并显示信息                     │
    │  echo "✗ 错误描述" >&2                  │
    │  exit 1                                  │
    └─────────────────────────────────────────┘
```

---

## 第四部分：添加自定义脚本

### 4.1 添加脚本的时机

**应该添加脚本的情况**：

- 一次性数据迁移任务
- 复杂的初始化流程
- 与特定工具链的集成
- 团队特定的运维任务
- 文档中没有记录的内部流程

**不应该添加脚本的情况**：

- 日常使用的命令 → 应该添加到 CLI
- 简单的单行命令 → 直接运行
- 临时调试任务 → 使用临时脚本
- 复杂功能 → 考虑作为独立项目

### 4.2 创建脚本步骤

**步骤一：确定脚本类型**

| 类型 | 场景 | 示例 |
|------|------|------|
| Shell 脚本 | 系统操作、调用外部命令 | `restart-mac.sh` |
| TypeScript | 与项目代码交互 | `update-clawtributors.ts` |
| JavaScript | 简单工具脚本 | `setup-git-hooks.js` |

**步骤二：创建脚本文件**

```bash
# 创建脚本目录（如果需要）
mkdir -p scripts/my-script

# 创建脚本文件
touch scripts/my-script.ts

# 添加可执行权限（如果是 Shell 脚本）
chmod +x scripts/my-script.sh
```

**步骤三：编写脚本内容**

```typescript
/**
 * 我的自定义脚本
 *
 * 功能描述...
 *
 * 使用方式：
 *   pnpm script:my-script [--option] <argument>
 *
 * 或直接运行：
 *   bun scripts/my-script.ts --option value
 */

import { execSync } from "node:child_process";
import { existsSync, readFileSync } from "node:fs";

// 颜色输出
const colors = {
  green: "\x1b[32m",
  red: "\x1b[31m",
  yellow: "\x1b[33m",
  reset: "\x1b[0m",
};

function log(message: string, type: "info" | "success" | "error" | "warn" = "info") {
  const prefix = {
    info: "ℹ",
    success: "✓",
    error: "✗",
    warn: "⚠",
  };
  const color = {
    info: colors.green,
    success: colors.green,
    error: colors.red,
    warn: colors.yellow,
  };
  console.log(`${color[type]}${prefix[type]} ${colors.reset}${message}`);
}

function checkDependencies() {
  // 检查必要依赖
  const required = ["bun", "git"];
  for (const dep of required) {
    try {
      execSync(`which ${dep}`, { stdio: "ignore" });
    } catch {
      log(`缺少依赖: ${dep}`, "error");
      process.exit(1);
    }
  }
}

function main() {
  log("开始执行自定义脚本...", "info");
  checkDependencies();

  // 主逻辑
  log("脚本执行完成", "success");
}

main();
```

**步骤四：在 package.json 中注册**

```json
{
  "scripts": {
    "script:my-script": "bun scripts/my-script.ts",
    "script:my-script:help": "bun scripts/my-script.ts --help"
  }
}
```

### 4.3 脚本文档

**在相关文档中添加说明**：

```markdown
## 自定义脚本

### 我的自定义脚本

用于...

**使用方式**：

```bash
# 基本使用
pnpm script:my-script

# 带参数
pnpm script:my-script --verbose
```

**功能**：
- 功能一...
- 功能二...

**依赖**：
- 依赖一...
- 依赖二...
```

---

## 第五部分：故障排查

### 5.1 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 脚本无法执行 | 缺少执行权限 | `chmod +x <script>` |
| 找不到命令 | PATH 问题 | 使用绝对路径或 `./` 前缀 |
| 依赖缺失 | 环境未配置 | 先安装依赖 |
| 参数错误 | 用法不正确 | 查看帮助：`<script> --help` |

### 5.2 调试技巧

```bash
# 查看脚本帮助
./script.sh --help

# 使用详细输出
./script.sh --verbose

# 直接查看源码
cat scripts/script-name.ts

# 检查脚本语法
bash -n scripts/script.sh  # Shell 脚本语法检查
```

### 5.3 权限问题

macOS 上的脚本可能遇到权限问题：

```bash
# 赋予执行权限
chmod +x scripts/*.sh
chmod +x scripts/*.js

# 如果仍有问题，检查父母录权限
ls -la scripts/

# 修复权限
chown $USER:staff scripts/
```

---

## 总结与进阶

### 知识点回顾

本章节的核心知识点包括：

1. **脚本定位**：理解 scripts/ 目录的用途和与 CLI 的关系
2. **核心脚本**：掌握 Git hooks、提交脚本、macOS 脚本的使用
3. **约定规范**：了解脚本命名、结构和输出规范
4. **自定义开发**：能够创建符合规范的自定义脚本

### 学习成果检验

完成本章节学习后，你应该能够：

- [ ] 说出 scripts/ 目录的核心用途
- [ ] 描述脚本和 CLI 命令的选择依据
- [ ] 解释主要脚本的工作原理
- [ ] 创建简单的自定义脚本

### 下一步学习

- **实践项目**：创建一个解决日常问题的自定义脚本
- **参考资源**：[CLI 命令参考](/cli/) — 了解更多命令
- **Git 操作**：[git-master 技能](/skills/git-master) — Git 相关最佳实践

---

## 附录

### 脚本速查表

| 脚本 | 用途 | 使用频率 |
|------|------|----------|
| `setup-git-hooks.js` | 设置 Git hooks | 首次设置 |
| `format-staged.js` | 预提交格式化 | 每次提交 |
| `committer` | 生成提交消息 | 每次提交 |
| `clawlog.sh` | macOS 日志查询 | 故障排查 |
| `package-mac-app.sh` | macOS 打包 | 发布时 |
| `restart-mac.sh` | macOS 重启 | 维护时 |
| `update-clawtributors.ts` | 更新贡献者 | 发布时 |
| `bundle-a2ui.sh` | A2UI 打包 | 开发时 |

### 相关链接

- [开发环境搭建](/platforms/mac/dev-setup)
- [macOS 权限配置](/platforms/mac/permissions)
- [日志系统](/logging)

---

_文档版本：v2.0_
_最后更新：2026-02-05_
