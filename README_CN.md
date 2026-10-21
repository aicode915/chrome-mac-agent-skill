<div align="center">

```
   _____      __  _             ________                            
  /  _  \____/  |_(_)_ _____  __\______ \_______  ____  _____   ____
 /  /_\  \_  __   __\  |\__  \/  ||    |  \_  __ \/  _ \/     \_/ __ \
/    |    \  |  |  |  | | / __ \_ |    `   \  | \(  <_> )  Y Y  \  ___/
\____|__  /__|  |__|__|_|(____  / /_______  /__|   \____/|__|_|  /\___  >
        \/                    \/          \/                   \/     \/
```

# chrome-mac-agent-skill

**专为 AI Agent 打造的 macOS Google Chrome 原生会话控制技能包。免重启 Chrome，保留全部用户登录态。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![macOS: Supported](https://img.shields.io/badge/macOS-Supported-brightgreen?style=flat-square)](https://apple.com/macos)
[![Agent Skill](https://img.shields.io/badge/Agent-Skill-blueviolet?style=flat-square)](https://github.com/aicode915/chrome-mac-agent-skill)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-success?style=flat-square)](https://github.com/aicode915/chrome-mac-agent-skill)

[English](./README.md) • [简体中文](./README_CN.md)

</div>

---

## ⚡ 痛点背景

当 AI 编程助手或智能体（如 Claude Code、Cursor、Windsurf、Antigravity）需要帮我们操作浏览器网页时，主流方案通常依赖 **Playwright** 或 **Puppeteer**。

这会带来极其痛苦的工作流割裂：
1. **丢失用户登录态**：它们默认启动一个全新的空白无痕浏览器，没有 Cookie。每次打开内部系统、邮箱或后台，都必须重新扫码、输密码或找手机短信 2FA。
2. **强迫重启浏览器**：如果想连接日常使用的 Chrome，通常必须杀掉所有窗口，带 `--remote-debugging-port` 参数重启。开发者日常几十个未保存的标签页直接丢失。
3. **环境庞大繁重**：为了看个网页，需要下载动辄数百兆的 Chromium 和繁杂的 Node/Python 驱动依赖。

---

## 💡 解决方案：`chrome-mac-agent-skill`

本技能包直接利用 macOS 原生系统级 AppleScript 与后台正在运行的真实 Chrome 进程通信：

- 🚀 **零重启、即插即用**：完全不需要关闭当前打开的工作标签页。
- 🔑 **无缝继承所有登录态**：Google 账号、企业 SSO 门户、GitHub、内网管理后台等所有 Cookie 完好无损。
- 🪶 **纯 Markdown 零依赖**：无需安装任何 `pip` 或 `npm` 依赖包，利用 macOS 内置的 `osascript` 毫秒级执行。
- 🛡 **原生系统弹窗与离开拦截破解**：内置自动剥离 `beforeunload` 锁，并支持自动点击 macOS 系统级确认弹窗（“重新加载此网站？”）。
- 📦 **模块化渐进式加载（Progressive Disclosure）**：主干仅保留核心指令，深度疑难场景按需读取，极致节省大模型 Context Token。

---

## 📊 与现有方案对比

| 功能特性 | Playwright / Puppeteer | 远程调试端口 (CDP) | `chrome-mac-agent-skill` 🚀 |
| :--- | :---: | :---: | :---: |
| **保留用户已登录会话** | ❌ 否 (空白隔离环境) | ⚠️ 仅当启动时开启了端口 | **✅ 是 (100% 完美保留)** |
| **需要关闭/重启 Chrome** | ❌ 需要 | ❌ 需要关闭已有 Chrome | **✅ 绝对不需要** |
| **外部运行依赖** | ❌ 庞大 (Node/Chromium) | ⚠️ 需要 WebSocket 库 | **✅ 零依赖 (macOS 自带)** |
| **Agent 导入即用性** | ⚠️ 需要部署 MCP 服务 | ⚠️ 需要配置端口与服务 | **✅ 纯 Markdown 技能包即插即用** |
| **离开/重载防误触弹窗** | ⚠️ 复杂事件拦截 | ⚠️ 需要 Page.handleDialog | **✅ 单行直接静默绕过** |

---

## 🚀 如何导入到你的 AI Agent

### 方式 1：直接放入工作区（Antigravity, Cursor, Windsurf）
将本仓库克隆或复制到项目的技能目录下：
```bash
mkdir -p .agents/skills
git clone https://github.com/aicode915/chrome-mac-agent-skill.git .agents/skills/active-chrome-control
```

### 方式 2：Claude Code
将 `SKILL.md` 和 `references/` 复制到项目的 `.claude/skills/chrome-mac-agent-skill/`。

---

## 🧭 仓库架构

```text
chrome-mac-agent-skill/
├── SKILL.md                 # 核心入口：快速指令速查、前置检查与按需路由索引
├── README.md                # 英文说明文档
├── README_CN.md             # 中文说明文档
├── LICENSE                  # MIT 开源协议
└── references/              # 专项深度指南（Agent 按需读取）
    ├── tab-management.md    # 标签页发现、精准匹配、新建、切换与关闭
    ├── page-interaction.md  # 正文提取、JS 求值、穿透 Shadow DOM 点击、响应式表单
    ├── dialog-handling.md   # 系统原生弹窗点击与 beforeunload 静默解除
    └── background-alive.md  # 页面防切屏、防失焦挂起与后台保活注入
```

---

## ⚠️ 前置要求（仅需开启一次）

为允许 AppleScript 在 Chrome 中执行 JavaScript：
1. 打开 Google Chrome；
2. 在 macOS 顶部菜单栏选择：**查看 (View)** -> **开发者 (Developer)**；
3. 勾选 **允许 Apple 事件中的 JavaScript (Allow JavaScript from Apple Events)**。

---

## 📄 开源许可证

[MIT](./LICENSE) © 2026 chrome-mac-agent-skill contributors.
