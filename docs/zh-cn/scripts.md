---
summary: "仓库脚本：目的、范围和安全注意事项"
read_when:
  - "运行仓库中的脚本"
  - "在 ./scripts 下添加或更改脚本"
title: "脚本"
---

# 脚本

`scripts/` 目录包含用于本地工作流程和运维任务的辅助脚本。当任务明确与脚本关联时使用这些；否则优先使用 CLI。

## 约定

- 脚本是**可选的**，除非在文档或发布清单中引用。
- 当 CLI 表面存在时优先使用（例如：身份验证监控使用 `openclaw models status --check`）。
- 假设脚本是特定于主机的；在新机器上运行之前阅读它们。

## Git hooks

- `scripts/setup-git-hooks.js`：在 git 仓库内时对 `core.hooksPath` 进行尽力而为的设置。
- `scripts/format-staged.js`：对暂存的 `src/` 和 `test/` 文件进行预提交格式化。

## 身份验证监控脚本

身份验证监控脚本记录在此处：
[/automation/auth-monitoring](/automation/auth-monitoring)

## 添加脚本时

- 保持脚本专注并有文档。
- 在相关文档中添加简短条目（如果缺失则创建一个）。
