---
summary: "在 OpenClaw 中使用 MiniMax M2.1"
read_when:
  - 你需要在 OpenClaw 中使用 MiniMax 模型
  - 你需要 MiniMax 设置指导
title: "MiniMax"
---

# MiniMax

MiniMax 是一家构建 **M2/M2.1** 模型系列的 AI 公司。当前专注于编码的版本是 **MiniMax M2.1**（2025 年 12 月 23 日发布），专用于现实世界的复杂任务。

来源：[MiniMax M2.1 发布说明](https://www.minimax.io/news/minimax-m21)

## 模型概述 (M2.1)

MiniMax 重点强调了 M2.1 的以下改进：

- 更强的**多语言编码**能力（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）。
- 更好的**Web/应用开发**和美学输出质量（包括原生移动端）。
- 改进的**复合指令**处理，用于办公风格的工作流程，基于交错思维和集成约束执行。
- **更简洁的响应**，token 使用量更低，迭代循环更快。
- 更强的**工具/代理框架**兼容性和上下文管理（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
- 更高质量的**对话和技术写作**输出。

## MiniMax M2.1 vs MiniMax M2.1 Lightning

- **速度：**Lightning 是 MiniMax 定价文档中的"快速"版本。
- **成本：**定价显示输入成本相同，但 Lightning 的输出成本更高。
- **编码计划路由：**Lightning 后端在 MiniMax 编码计划上不直接可用。MiniMax 会自动将大多数请求路由到 Lightning，但在流量高峰期间会回退到常规 M2.1 后端。

## 选择一种设置方式

### MiniMax OAuth（编码计划）——推荐

**最适合：**通过 OAuth 快速设置 MiniMax 编码计划，无需 API 密钥。

启用内置的 OAuth 插件并进行身份验证：

```bash
openclaw plugins enable minimax-portal-auth  # 如果已加载则跳过
openclaw gateway restart  # 如果 gateway 已在运行则重启
openclaw onboard --auth-choice minimax-portal
```

系统将提示您选择一个端点：

- **Global** - 国际用户 (`api.minimax.io`)
- **CN** - 中国用户 (`api.minimaxi.com`)

详情请参阅 [MiniMax OAuth 插件 README](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth)。

### MiniMax M2.1（API 密钥）

**最适合：**使用 Anthropic 兼容 API 的托管 MiniMax。

通过 CLI 配置：

- 运行 `openclaw configure`
- 选择 **Model/auth**
- 选择 **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.1 作为备用（Opus 为主）

**最适合：**保持 Opus 4.5 为主模型，故障转移到 MiniMax M2.1。

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

### 可选：通过 LM Studio 本地运行（手动）

**最适合：**使用 LM Studio 进行本地推理。
我们在使用 LM Studio 本地服务器的强大硬件（例如桌面/服务器）上测试 MiniMax M2.1 时获得了良好的效果。

通过 `openclaw.json` 手动配置：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 通过 `openclaw configure` 配置

使用交互式配置向导来设置 MiniMax，而无需编辑 JSON：

1. 运行 `openclaw configure`。
2. 选择 **Model/auth**。
3. 选择 **MiniMax M2.1**。
4. 在提示时选择您的默认模型。

## 配置选项

- `models.providers.minimax.baseUrl`：首选 `https://api.minimax.io/anthropic`（Anthropic 兼容）；`https://api.minimax.io/v1` 可用于 OpenAI 兼容负载。
- `models.providers.minimax.api`：首选 `anthropic-messages`；`openai-completions` 可用于 OpenAI 兼容负载。
- `models.providers.minimax.apiKey`：MiniMax API 密钥 (`MINIMAX_API_KEY`)。
- `models.providers.minimax.models`：定义 `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost`。
- `agents.defaults.models`：为您希望在允许列表中的模型设置别名。
- `models.mode`：如果您想将 MiniMax 与内置模型一起添加，请保持 `merge`。

## 注意事项

- 模型引用格式为 `minimax/<model>`。
- 编码计划使用量 API：`https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（需要编码计划密钥）。
- 如果需要精确的成本跟踪，请在 `models.json` 中更新定价值。
- MiniMax 编码计划的推荐链接（9 折优惠）：https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link
- 请参阅 [/concepts/model-providers](/zh-CN/concepts/model-providers) 了解提供商规则。
- 使用 `openclaw models list` 和 `openclaw models set minimax/MiniMax-M2.1` 进行切换。

## 故障排除

### "Unknown model: minimax/MiniMax-M2.1"

这通常意味着 **MiniMax 提供商未配置**（没有提供商条目，也未找到 MiniMax 身份验证配置文件/环境密钥）。此检测的修复已在 **2026.1.12** 中提供（撰写本文时尚未发布）。修复方法：

- 升级到 **2026.1.12**（或从源码 `main` 运行），然后重启 gateway。
- 运行 `openclaw configure` 并选择 **MiniMax M2.1**，或
- 手动添加 `models.providers.minimax` 代码块，或
- 设置 `MINIMAX_API_KEY`（或 MiniMax 身份验证配置文件）以便注入提供商。

请确保模型 ID **区分大小写**：

- `minimax/MiniMax-M2.1`
- `minimax/MiniMax-M2.1-lightning`

然后使用以下命令重新检查：

```bash
openclaw models list
```
