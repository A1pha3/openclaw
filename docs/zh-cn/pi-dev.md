---
title: "Pi 开发工作流程"
---

# Pi 开发工作流程

本指南总结了 OpenClaw 中 pi 集成工作的合理工作流程。

## 类型检查和 lint

- 类型检查和构建：`pnpm build`
- Lint：`pnpm lint`
- 格式检查：`pnpm format`
- 推送前的完整门禁：`pnpm lint && pnpm build && pnpm test`

## 运行 Pi 测试

对 pi 集成测试集使用专用脚本：

```bash
scripts/pi/run-tests.sh
```

要包括锻炼真实提供商行为的实时测试：

```bash
scripts/pi/run-tests.sh --live
```

该脚本通过这些 glob 运行所有 pi 相关的单元测试：

- `src/agents/pi-*.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-tool-definition-adapter.test.ts`
- `src/agents/pi-extensions/*.test.ts`

## 手动测试

推荐流程：

- 在开发模式下运行网关：
  - `pnpm gateway:dev`
- 直接触发代理：
  - `pnpm openclaw agent --message "Hello" --thinking low`
- 使用 TUI 进行交互式调试：
  - `pnpm tui`

对于工具调用行为，提示 `read` 或 `exec` 操作，以便你可以看到工具流式传输和负载处理。

## 干净状态重置

状态位于 OpenClaw 状态目录下。默认是 `~/.openclaw`。如果设置了 `OPENCLAW_STATE_DIR`，请改用该目录。

要重置所有内容：

- `openclaw.json` 用于配置
- `credentials/` 用于身份验证配置文件和令牌
- `agents/<agentId>/sessions/` 用于代理会话历史
- `agents/<agentId>/sessions.json` 用于会话索引
- `sessions/` 如果存在遗留路径
- `workspace/` 如果你想要空白工作区

如果你只想重置会话，删除该代理的 `agents/<agentId>/sessions/` 和 `agents/<agentId>/sessions.json`。如果你不想重新身份验证，请保留 `credentials/`。

## 参考

- https://docs.openclaw.ai/testing
- https://docs.openclaw.ai/start/getting-started
