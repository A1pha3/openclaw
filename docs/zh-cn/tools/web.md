---
summary: "Web 搜索 + 抓取工具（Brave Search API、Perplexity 直连/OpenRouter）"
read_when:
  - 想要启用 web_search 或 web_fetch
  - 需要设置 Brave Search API 密钥
  - 想要使用 Perplexity Sonar 进行网页搜索
title: "Web 工具"
---

# Web 工具

OpenClaw 提供两个轻量级 Web 工具：

| 工具 | 功能 |
|------|------|
| `web_search` | 通过 Brave Search API（默认）或 Perplexity Sonar 搜索网页 |
| `web_fetch` | HTTP 抓取 + 可读性提取（HTML → markdown/text） |

这些**不是**浏览器自动化。对于 JS 密集型站点或需要登录的场景，请使用 [浏览器工具](/zh-cn/tools/browser)。

## 工作原理

### web_search

- 调用配置的提供商并返回结果
- **Brave**（默认）：返回结构化结果（标题、URL、摘要）
- **Perplexity**：返回 AI 合成的答案，带有实时网页搜索的引用
- 结果按查询缓存 15 分钟（可配置）

### web_fetch

- 执行普通 HTTP GET 并提取可读内容（HTML → markdown/text）
- **不执行** JavaScript
- 默认启用（除非显式禁用）

## 选择搜索提供商

| 提供商 | 优点 | 缺点 | API 密钥 |
|--------|------|------|----------|
| **Brave**（默认） | 快速、结构化结果、免费层 | 传统搜索结果 | `BRAVE_API_KEY` |
| **Perplexity** | AI 合成答案、引用、实时性 | 需要 Perplexity 或 OpenRouter 访问 | `OPENROUTER_API_KEY` 或 `PERPLEXITY_API_KEY` |

在配置中设置提供商：

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave", // 或 "perplexity"
      },
    },
  },
}
```

## 获取 Brave API 密钥

1. 在 https://brave.com/search/api/ 创建 Brave Search API 账户
2. 在仪表板中，选择 **Data for Search** 计划（不是 "Data for AI"）并生成 API 密钥
3. 运行 `openclaw configure --section web` 将密钥存储在配置中（推荐），或在环境中设置 `BRAVE_API_KEY`

Brave 提供免费层和付费计划；查看 Brave API 门户了解当前限制和定价。

### 密钥存储位置

**推荐**：运行 `openclaw configure --section web`。将密钥存储在 `~/.openclaw/openclaw.json` 的 `tools.web.search.apiKey` 下。

**环境变量替代**：在网关进程环境中设置 `BRAVE_API_KEY`。对于网关安装，将其放在 `~/.openclaw/.env`（或服务环境）中。详见 [环境变量](/zh-cn/help/faq#how-does-openclaw-load-environment-variables)。

## 使用 Perplexity（直连或通过 OpenRouter）

Perplexity Sonar 模型具有内置的网页搜索功能，返回带引用的 AI 合成答案。可以通过 OpenRouter 使用（无需信用卡 - 支持加密货币/预付）。

### 获取 OpenRouter API 密钥

1. 在 https://openrouter.ai/ 创建账户
2. 添加余额（支持加密货币、预付或信用卡）
3. 在账户设置中生成 API 密钥

### 设置 Perplexity 搜索

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // API 密钥（如果设置了 OPENROUTER_API_KEY 或 PERPLEXITY_API_KEY 则可选）
          apiKey: "sk-or-v1-...",
          // 基础 URL（如果省略则根据密钥自动选择）
          baseUrl: "https://openrouter.ai/api/v1",
          // 模型（默认 perplexity/sonar-pro）
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

**环境变量替代**：在网关环境中设置 `OPENROUTER_API_KEY` 或 `PERPLEXITY_API_KEY`。

如果未设置基础 URL，OpenClaw 根据 API 密钥来源选择默认值：
- `PERPLEXITY_API_KEY` 或 `pplx-...` → `https://api.perplexity.ai`
- `OPENROUTER_API_KEY` 或 `sk-or-...` → `https://openrouter.ai/api/v1`
- 未知密钥格式 → OpenRouter（安全回退）

### 可用 Perplexity 模型

| 模型 | 描述 | 适用场景 |
|------|------|----------|
| `perplexity/sonar` | 带网页搜索的快速问答 | 快速查询 |
| `perplexity/sonar-pro`（默认） | 带网页搜索的多步推理 | 复杂问题 |
| `perplexity/sonar-reasoning-pro` | 思维链分析 | 深度研究 |

## web_search 配置

### 要求

- `tools.web.search.enabled` 不能为 `false`（默认：启用）
- 所选提供商的 API 密钥：
  - **Brave**：`BRAVE_API_KEY` 或 `tools.web.search.apiKey`
  - **Perplexity**：`OPENROUTER_API_KEY`、`PERPLEXITY_API_KEY` 或 `tools.web.search.perplexity.apiKey`

### 配置示例

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // 如果设置了 BRAVE_API_KEY 则可选
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### 工具参数

| 参数 | 说明 |
|------|------|
| `query` | **必需**，搜索查询 |
| `count` | 1-10；默认来自配置 |
| `country` | 可选，2 字母国家代码用于区域特定结果（如 "DE"、"US"、"ALL"） |
| `search_lang` | 可选，搜索结果的 ISO 语言代码（如 "de"、"en"、"fr"） |
| `ui_lang` | 可选，UI 元素的 ISO 语言代码 |
| `freshness` | 可选（仅 Brave），按发现时间过滤：`pd`（过去一天）、`pw`（过去一周）、`pm`（过去一月）、`py`（过去一年）或 `YYYY-MM-DDtoYYYY-MM-DD` |

### 使用示例

```javascript
// 德国特定搜索
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de",
})

// 法语搜索，法语 UI
await web_search({
  query: "actualites",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr",
})

// 最近结果（过去一周）
await web_search({
  query: "TMBG interview",
  freshness: "pw",
})
```

## web_fetch 配置

### 要求

- `tools.web.fetch.enabled` 不能为 `false`（默认：启用）
- 可选 Firecrawl 回退：设置 `tools.web.fetch.firecrawl.apiKey` 或 `FIRECRAWL_API_KEY`

### 配置示例

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // 如果设置了 FIRECRAWL_API_KEY 则可选
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // 毫秒（1 天）
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### 工具参数

| 参数 | 说明 |
|------|------|
| `url` | **必需**，仅 http/https |
| `extractMode` | `markdown` 或 `text` |
| `maxChars` | 截断长页面 |

### 注意事项

- `web_fetch` 先使用 Readability（主内容提取），然后使用 Firecrawl（如果配置）。如果两者都失败，工具返回错误
- Firecrawl 请求使用机器人规避模式并默认缓存结果
- `web_fetch` 默认发送类 Chrome 的 User-Agent 和 `Accept-Language`；如需可覆盖 `userAgent`
- `web_fetch` 阻止私有/内部主机名并重新检查重定向（通过 `maxRedirects` 限制）
- `web_fetch` 是尽力提取；某些站点需要使用浏览器工具
- 详见 [Firecrawl](/zh-cn/tools/firecrawl) 了解密钥设置和服务详情
- 响应被缓存（默认 15 分钟）以减少重复抓取
- 如果使用工具配置文件/白名单，添加 `web_search`/`web_fetch` 或 `group:web`
- 如果 Brave 密钥缺失，`web_search` 返回简短设置提示和文档链接

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| `web_search` 返回设置提示 | 检查 API 密钥是否正确配置 |
| Perplexity 搜索失败 | 确认 `OPENROUTER_API_KEY` 或 `PERPLEXITY_API_KEY` 已设置 |
| `web_fetch` 返回空内容 | 站点可能需要 JS 渲染，考虑使用浏览器工具 |
| 缓存导致旧结果 | 调整 `cacheTtlMinutes` 或等待缓存过期 |
