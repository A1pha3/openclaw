---
summary: "语音唤醒与按键通话重叠时的语音浮层生命周期"
read_when:
  - 调整语音浮层行为
title: "语音浮层"
---

# 语音浮层生命周期 (macOS)

受众：macOS 应用贡献者。目标：在语音唤醒和按键通话重叠时保持语音浮层的可预测行为。

### 当前设计意图

- 如果浮层已因语音唤醒而可见，用户按下热键时，热键会话会**接管**现有文本而不是重置。在按住热键期间浮层保持显示。当用户释放时：如果有修剪后的文本则发送，否则关闭。
- 仅语音唤醒仍会在静音时自动发送；按键通话在释放时立即发送。

### 已实现（2025年12月9日）

- 浮层会话现在为每次捕获（语音唤醒或按键通话）携带一个令牌。当令牌不匹配时，部分/最终/发送/关闭/音量更新会被丢弃，避免过时的回调。
- 按键通话会将任何可见的浮层文本作为前缀接管（因此在语音浮层显示时按下热键会保留文本并追加新语音）。它最多等待1.5秒获取最终转录，然后回退到当前文本。
- 提示音/浮层日志以 `info` 级别输出到类别 `voicewake.overlay`、`voicewake.ptt` 和 `voicewake.chime`（会话开始、部分、最终、发送、关闭、提示音原因）。

### 后续步骤

1. **VoiceSessionCoordinator (actor)**
   - 同一时间只拥有一个 `VoiceSession`。
   - API（基于令牌）：`beginWakeCapture`、`beginPushToTalk`、`updatePartial`、`endCapture`、`cancel`、`applyCooldown`。
   - 丢弃携带过时令牌的回调（防止旧的识别器重新打开浮层）。

2. **VoiceSession (model)**
   - 字段：`token`、`source`（wakeWord|pushToTalk）、已提交/易失文本、提示音标志、计时器（自动发送、空闲）、`overlayMode`（display|editing|sending）、冷却截止时间。

3. **浮层绑定**
   - `VoiceSessionPublisher`（`ObservableObject`）将活动会话镜像到 SwiftUI。
   - `VoiceWakeOverlayView` 仅通过发布者渲染；它从不直接修改全局单例。
   - 浮层用户操作（`sendNow`、`dismiss`、`edit`）使用会话令牌回调协调器。

4. **统一发送路径**
   - 在 `endCapture` 时：如果修剪后文本为空 → 关闭；否则 `performSend(session:)`（播放一次发送提示音、转发、关闭）。
   - 按键通话：无延迟；语音唤醒：可选的自动发送延迟。
   - 在按键通话完成后对唤醒运行时应用短暂冷却，以防止语音唤醒立即重新触发。

5. **日志记录**
   - 协调器在子系统 `bot.molt`、类别 `voicewake.overlay` 和 `voicewake.chime` 中输出 `.info` 日志。
   - 关键事件：`session_started`、`adopted_by_push_to_talk`、`partial`、`finalized`、`send`、`dismiss`、`cancel`、`cooldown`。

### 调试清单

- 在复现卡住的浮层时流式传输日志：

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- 验证只有一个活动会话令牌；过时的回调应该被协调器丢弃。
- 确保按键通话释放总是使用活动令牌调用 `endCapture`；如果文本为空，预期 `dismiss` 而没有提示音或发送。

### 迁移步骤（建议）

1. 添加 `VoiceSessionCoordinator`、`VoiceSession` 和 `VoiceSessionPublisher`。
2. 重构 `VoiceWakeRuntime` 以创建/更新/结束会话，而不是直接操作 `VoiceWakeOverlayController`。
3. 重构 `VoicePushToTalk` 以接管现有会话并在释放时调用 `endCapture`；应用运行时冷却。
4. 将 `VoiceWakeOverlayController` 连接到发布者；移除来自运行时/PTT 的直接调用。
5. 添加会话接管、冷却和空文本关闭的集成测试。

## 为什么需要令牌机制

| 场景 | 无令牌的问题 | 令牌解决方案 |
|------|-------------|--------------|
| 快速切换模式 | 旧回调更新错误的会话 | 令牌不匹配时丢弃 |
| 并发识别器 | 多个来源竞争更新 | 只有活动令牌能更新 |
| 延迟的最终结果 | 会话已结束但收到旧结果 | 验证令牌后丢弃 |
