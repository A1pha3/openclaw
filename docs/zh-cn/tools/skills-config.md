---
summary: "技能配置 schema 和示例"
read_when:
  - 添加或修改技能配置
  - 调整内置技能白名单或安装行为
title: "技能配置"
---

# 技能配置

所有技能相关配置都在 `~/.openclaw/openclaw.json` 的 `skills` 字段下。

## 完整配置示例

```json5
{
  skills: {
    // 内置技能白名单（可选）
    allowBundled: ["gemini", "peekaboo"],

    // 加载配置
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },

    // 安装配置
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun（网关运行时仍需 Node；不推荐 bun）
    },

    // 单个技能配置
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## 配置字段说明

### 顶层字段

| 字段 | 说明 |
|------|------|
| `allowBundled` | 内置技能白名单（可选）。设置后只有列表中的内置技能可用；不影响托管/工作区技能 |
| `load.extraDirs` | 额外的技能扫描目录（优先级最低） |
| `load.watch` | 是否监视技能文件夹并自动刷新（默认：true） |
| `load.watchDebounceMs` | 文件监视防抖时间，毫秒（默认：250） |
| `install.preferBrew` | 优先使用 brew 安装器（默认：true） |
| `install.nodeManager` | Node 包管理器偏好：`npm` / `pnpm` / `yarn` / `bun`（默认：npm） |
| `entries.<skillKey>` | 单个技能的配置覆盖 |

### 单个技能字段 (`entries.<skillKey>`)

| 字段 | 说明 |
|------|------|
| `enabled` | 设为 `false` 可禁用技能（即使已安装/内置） |
| `env` | 注入的环境变量（仅当变量未设置时生效） |
| `apiKey` | 便捷字段，用于声明了 `primaryEnv` 的技能 |

## 注意事项

### 技能键名

- `entries` 下的键默认对应技能名称
- 如果技能定义了 `metadata.openclaw.skillKey`，则使用该键

### 变更生效时机

- 启用 watcher 时，技能变更会在下一个 agent 轮次生效

### 沙箱环境与环境变量

当会话处于**沙箱模式**时，技能进程在 Docker 容器内运行。沙箱**不会**继承宿主机的 `process.env`。

解决方案：

1. 使用 `agents.defaults.sandbox.docker.env`（或 agent 级别的 `agents.list[].sandbox.docker.env`）
2. 将环境变量烘焙到自定义沙箱镜像中

全局 `env` 和 `skills.entries.<skill>.env/apiKey` 仅对**宿主机**运行生效。

## 相关文档

- [技能系统概述](/zh-cn/tools/skills) - 技能位置、优先级、门控规则
- [ClawHub](/zh-cn/tools/clawhub) - 公共技能注册表
