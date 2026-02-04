---
summary: "使用 apply_patch 工具批量应用多文件补丁"
read_when:
  - 需要跨多个文件进行结构化编辑
  - 想要记录或调试基于补丁的编辑操作
title: "apply_patch 工具"
---

# apply_patch 工具

使用结构化补丁格式批量应用文件变更。当需要跨多个文件或多个代码块进行编辑时，这比单个 `edit` 调用更加稳定可靠。

## 为什么需要这个工具

在复杂的代码重构场景中，你可能需要同时修改多个文件的多处位置。传统的逐个编辑方式不仅效率低下，而且容易出错。`apply_patch` 工具让你可以：

- **批量操作**：一次调用完成多个文件的修改
- **原子性**：要么全部成功，要么全部失败
- **可审计**：补丁格式清晰，便于代码审查

## 补丁格式

工具接受单个 `input` 字符串参数，包含一个或多个文件操作：

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## 参数说明

| 参数 | 必填 | 说明 |
|------|------|------|
| `input` | 是 | 完整的补丁内容，必须包含 `*** Begin Patch` 和 `*** End Patch` |

## 支持的操作

| 操作 | 语法 | 说明 |
|------|------|------|
| 新增文件 | `*** Add File: <path>` | 创建新文件并添加内容 |
| 更新文件 | `*** Update File: <path>` | 修改现有文件的内容 |
| 删除文件 | `*** Delete File: <path>` | 删除指定文件 |
| 重命名/移动 | `*** Move to: <new-path>` | 在 `Update File` 块内使用，重命名文件 |
| 文件末尾标记 | `*** End of File` | 标记仅在文件末尾插入的位置 |

## 启用配置

此工具默认禁用，需要手动启用：

```json5
{
  tools: {
    exec: {
      applyPatch: {
        enabled: true,
        // 可选：限制特定模型使用
        allowModels: ["openai-codex/gpt-5.2"]
      }
    }
  }
}
```

**注意**：

- 此功能为实验性功能
- 目前仅支持 OpenAI 系列模型（包括 OpenAI Codex）
- 配置项位于 `tools.exec` 下

## 使用示例

### 基础示例

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

### 多文件操作示例

```
*** Begin Patch
*** Add File: src/utils/helper.ts
+export function helper() {
+  return "hello";
+}
*** Update File: src/index.ts
@@
-import { old } from './old';
+import { helper } from './utils/helper';
@@
-const result = old();
+const result = helper();
*** Delete File: src/old.ts
*** End Patch
```

### 文件重命名示例

```
*** Begin Patch
*** Update File: src/oldName.ts
*** Move to: src/newName.ts
@@
-// old comment
+// renamed file
*** End Patch
```

## 路径解析

- 所有路径相对于工作区根目录解析
- 支持嵌套目录结构
- 自动创建不存在的父目录

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 工具不可用 | 未启用配置 | 设置 `tools.exec.applyPatch.enabled: true` |
| 模型不支持 | 模型不在允许列表中 | 检查 `allowModels` 配置或使用支持的模型 |
| 补丁应用失败 | 格式错误或上下文不匹配 | 检查补丁格式，确保源文件内容匹配 |
| 权限错误 | 文件系统权限不足 | 检查工作区目录的写入权限 |

## 最佳实践

1. **小步快走**：将大型重构拆分为多个小补丁
2. **保持上下文**：在 `@@` 块中包含足够的上下文行
3. **先测试后应用**：在重要操作前备份或使用版本控制
4. **避免歧义**：确保替换模式在文件中唯一匹配

## 相关文档

- [编辑工具](/tools/edit) - 单文件编辑
- [执行工具](/tools/exec) - 命令行操作
