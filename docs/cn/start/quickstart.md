---
read_when:
  - 你希望以最快的方式从安装到运行一个可用的 Gateway 网关
summary: 安装 OpenClaw，完成 Gateway 网关新手引导，并配对你的第一个渠道。
title: 快速开始
version: "2026.2.17"
last_updated: "2026-03-05"
x-i18n:
  generated_at: "2026-02-04T17:53:21Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 3c5da65996f89913cd115279ae21dcab794eadd14595951b676d8f7864fbbe2d
  source_path: start/quickstart.md
  workflow: 15
---

## 📋 前置条件

在开始之前，请确保满足以下要求：

<Checklist>
  <CheckItem checked={true}>
    **Node.js 22+**: 运行 `node --version` 检查，如未安装请访问 [Node.js 官网](https://nodejs.org/)
  </CheckItem>
  <CheckItem checked={true}>
    **操作系统**: macOS 12+ / Ubuntu 20.04+ / Windows 10+ (WSL2)
  </CheckItem>
  <CheckItem checked={true}>
    **内存**: 至少 512MB 可用内存（推荐 2GB+）
  </CheckItem>
  <CheckItem checked={true}>
    **磁盘空间**: 至少 500MB 可用空间（推荐 2GB+）
  </CheckItem>
  <CheckItem checked={false}>
    **手机号码**: 用于 WhatsApp 配对（推荐使用备用号码）
  </CheckItem>
</Checklist>

<Note type="warning">
OpenClaw 需要 Node.js 22 或更新版本。使用 `node --version` 检查你的版本。
</Note>

## 安装

<Tabs>
  <Tab title="npm">
    ```bash
    npm install -g openclaw@latest
    ```
  </Tab>
  <Tab title="pnpm">
    ```bash
    pnpm add -g openclaw@latest
    ```
  </Tab>
  <Tab title="yarn">
    ```bash
    yarn global add openclaw@latest
    ```
  </Tab>
</Tabs>

### ✅ 验证安装

安装完成后，验证 OpenClaw 是否正确安装：

```bash
# 检查版本
openclaw --version

# 检查 Node.js 版本
node --version

# 查看帮助信息
openclaw --help
```

期望输出：
- `openclaw --version` 应显示版本号（如 `2026.2.17`）
- `node --version` 应显示 `v22.x.x` 或更高版本

## 新手引导并运行 Gateway 网关

<Steps>
  <Step title="新手引导并安装服务">
    ```bash
    openclaw onboard --install-daemon
    ```
  </Step>
  <Step title="配对 WhatsApp">
    ```bash
    openclaw channels login
    ```
  </Step>
  <Step title="启动 Gateway 网关">
    ```bash
    openclaw gateway --port 18789
    ```
  </Step>
</Steps>

完成新手引导后，Gateway 网关将通过用户服务运行。你也可以使用 `openclaw gateway` 手动启动。

<Info>
之后在 npm 安装和 git 安装之间切换非常简单。安装另一种方式后，运行
`openclaw doctor` 即可更新 Gateway 网关服务入口点。
</Info>

### ✅ 验证新手引导

新手引导完成后，检查配置是否正确：

```bash
# 检查配置是否生成
openclaw config get

# 运行健康检查
openclaw health

# 检查渠道状态
openclaw channels status
```

期望输出：
- `openclaw config get` 应显示配置内容
- `openclaw health` 应显示 `OK` 或健康状态
- `openclaw channels status` 应显示已配置的渠道

## 从源码安装（开发）

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时会自动安装 UI 依赖
pnpm build
openclaw onboard --install-daemon
```

如果你还没有全局安装，可以在仓库目录中通过 `pnpm openclaw ...` 运行新手引导。

## 多实例快速开始（可选）

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

## 发送测试消息

需要一个正在运行的 Gateway 网关。

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

### ✅ 验证消息发送

```bash
# 查看消息队列状态
openclaw status

# 查看最近的日志
openclaw logs --tail 20
```

---

## 🐛 常见问题

### Node.js 版本过低

**问题**: 安装或运行时出现语法错误

**解决方案**:
```bash
# 使用 nvm 升级 Node.js
nvm install 22
nvm use 22
nvm alias default 22
```

### 权限错误 (EACCES)

**问题**: 安装时报 `EACCES` 错误

**解决方案**:
```bash
# 方案一：使用 nvm（推荐）
# nvm 自动处理权限

# 方案二：修复 npm 权限
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 端口被占用

**问题**: 启动时报 `EADDRINUSE` 错误

**解决方案**:
```bash
# 查找占用端口的进程
lsof -i :18789

# 终止占用进程
kill -9 <PID>

# 或使用其他端口
openclaw gateway --port 18790
```

### WhatsApp 配对失败

**问题**: 扫描二维码后配对失败

**解决方案**:
1. 确保使用**单独的手机号码**（推荐备用号码）
2. 检查手机网络连接
3. 重新运行 `openclaw channels login`
4. 查看详细日志：`openclaw logs --channel whatsapp`

### 服务无法启动

**问题**: Gateway 服务启动后立即退出

**解决方案**:
```bash
# 查看详细错误
openclaw gateway --verbose

# 运行诊断
openclaw doctor

# 检查日志
openclaw logs --level error --tail 50
```

---

## 📚 下一步

完成快速开始后，建议继续阅读：

- **[入门指南](/start/index)** - 详细的学习路径和文档导航
- **[WhatsApp 配置](/channels/whatsapp)** - WhatsApp 渠道详细配置
- **[Gateway 网关](/gateway/index)** - Gateway 运行手册
- **[故障排除](/operations/troubleshooting)** - 常见问题解决方案

---

## 📝 变更历史

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 2026.2.17 | 2026-03-05 | 添加前置条件、验证步骤、常见问题 |
| 2026.2.17 | 2026-02-04 | 初始版本 |
