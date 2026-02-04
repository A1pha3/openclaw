---
summary: "OpenClaw macOS 菜单栏图标状态和动画"
read_when:
  - 更改菜单栏图标行为
title: "菜单栏图标"
---

# 菜单栏图标状态

作者：steipete · 更新：2025-12-06 · 范围：macOS 应用（`apps/macos`）

- **空闲**：正常图标动画（眨眼、偶尔摆动）。
- **暂停**：状态项使用 `appearsDisabled`；无动作。
- **语音触发（大耳朵）**：当听到唤醒词时，语音唤醒检测器调用 `AppState.triggerVoiceEars(ttl: nil)`，在捕获话语期间保持 `earBoostActive=true`。耳朵放大（1.9x），获得圆形耳孔以提高可读性，然后在 1s 静音后通过 `stopVoiceEars()` 缩小。仅从应用内语音管道触发。
- **工作中（助手运行）**：`AppState.isWorking=true` 驱动"尾巴/腿部跑动"微动作：工作进行时更快的腿部摆动和轻微偏移。目前在 WebChat 助手运行期间切换；在连接其他长任务时添加相同的切换。

连接点

- 语音唤醒：运行时/测试器在触发时调用 `AppState.triggerVoiceEars(ttl: nil)`，在 1s 静音后调用 `stopVoiceEars()` 以匹配捕获窗口。
- 助手活动：在工作跨度周围设置 `AppStateStore.shared.setWorking(true/false)`（在 WebChat 助手调用中已完成）。保持跨度短并在 `defer` 块中重置以避免动画卡住。

形状和大小

- 基础图标在 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` 中绘制。
- 耳朵缩放默认为 `1.0`；语音增强设置 `earScale=1.9` 并切换 `earHoles=true`，不改变整体框架（18×18 pt 模板图像渲染到 36×36 px Retina 后备存储）。
- 跑动使用腿部摆动最高约 1.0，带有小的水平抖动；它是在任何现有空闲摆动之上的附加。

行为说明

- 没有用于耳朵/工作的外部 CLI/代理切换；保持它仅限于应用自己的信号以避免意外抖动。
- 保持 TTL 短（<10s），以便如果任务挂起，图标能快速返回基线。
