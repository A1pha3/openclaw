---
summary: "入站渠道位置解析（Telegram + WhatsApp）和上下文字段"
read_when:
  - 添加或修改渠道位置解析
  - 在代理提示词或工具中使用位置上下文字段
title: "渠道位置解析"
---

# 渠道位置解析

OpenClaw 将聊天渠道中的共享位置规范化为：

- 附加到入站正文的可读文本，以及
- 自动回复上下文载荷中的结构化字段。

目前支持的渠道：

- **Telegram**（位置标记 + 场所 + 实时位置）
- **WhatsApp**（locationMessage + liveLocationMessage）
- **Matrix**（带 `geo_uri` 的 `m.location`）

## 文本格式化

位置呈现为友好的无括号行：

- 标记：
  - `📍 48.858844, 2.294351 ±12m`
- 命名地点：
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- 实时共享：
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

如果渠道包含标题/注释，它会附加到下一行：

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## 上下文字段

当存在位置时，这些字段会添加到 `ctx`：

- `LocationLat`（数字）
- `LocationLon`（数字）
- `LocationAccuracy`（数字，米；可选）
- `LocationName`（字符串；可选）
- `LocationAddress`（字符串；可选）
- `LocationSource`（`pin | place | live`）
- `LocationIsLive`（布尔值）

## 渠道说明

- **Telegram**：场所映射到 `LocationName/LocationAddress`；实时位置使用 `live_period`。
- **WhatsApp**：`locationMessage.comment` 和 `liveLocationMessage.caption` 作为标题行附加。
- **Matrix**：`geo_uri` 解析为标记位置；忽略海拔，且 `LocationIsLive` 始终为 false。
