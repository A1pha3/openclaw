---
summary: "用于 web_search 的 Brave Search API 设置"
read_when:
  - "你想将 Brave Search 用于 web_search"
  - "你需要 BRAVE_API_KEY 或计划详情"
title: "Brave Search"
---

# Brave Search API

OpenClaw 将 Brave Search 作为 `web_search` 的默认提供商。

## 获取 API 密钥

1. 在 https://brave.com/search/api/ 创建一个 Brave Search API 账户
2. 在仪表板中，选择 **Data for Search** 计划并生成一个 API 密钥
3. 将密钥存储在配置中（推荐）或在网关环境中设置 `BRAVE_API_KEY`

## 配置示例

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## 注意

- Data for AI 计划与 `web_search` **不**兼容。
- Brave 提供免费层以及付费计划；请查看 Brave API 门户了解当前限制。

有关完整的 web_search 配置，请参阅 [Web 工具](/tools/web)。
