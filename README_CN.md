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

| 功能特性 | Playwright / Puppeteer | 远程调试端口 (CDP) | 系统级 Computer Use (截屏+鼠标) | `chrome-mac-agent-skill` 🚀 |
| :--- | :---: | :---: | :---: | :---: |
| **保留用户已登录会话** | ❌ 否 (空白隔离环境) | ⚠️ 仅当启动时开启了端口 | ✅ 是 (直接看屏幕) | **✅ 是 (100% 完美保留)** |
| **后台完全静默执行** | ⚠️ 仅限无头模式 | ⚠️ 需配置端口 | ❌ **否 (强行霸占鼠标与屏幕)** | **✅ 是 (零鼠标劫持/无感后台运行)** |
| **动作执行响应速度** | ~1 - 3 秒 | ~500 毫秒 | ❌ **慢 (每次点击 5~10 秒)** | **⚡ 极致迅速 (<50 毫秒)** |
| **Token 与资金消耗** | 极低 | 极低 | ❌ **极贵 (每次消耗数千视觉 Token)** | **💎 极致省钱 (零视觉 Token 浪费)** |
| **系统权限要求** | 无特殊要求 | 无特殊要求 | ❌ **极高 (需屏幕录制 + 辅助功能权限)** | **🔒 极小 (仅需勾选 Chrome 菜单一次)** |
| **点击精准度** | 极高 | 极高 | ⚠️ **受屏幕缩放/Retina 漂移影响易点偏** | **🎯 100% 绝对精准 (DOM 级别命中)** |
| **需要关闭/重启 Chrome** | ❌ 需要 | ❌ 需要关闭已有 Chrome | ✅ 不需要 | **✅ 绝对不需要 (即连即用)** |

---

## 🥊 为什么不直接使用系统级 "Computer Use"（截屏 + 物理鼠标点击）？

虽然当前多模态大模型（如 Claude 3.5 / 3.7 Computer Use）具备直接“看屏幕像素并模拟鼠标点击”的能力，但在日常的浏览器操作中，Computer Use 往往是“高射炮打蚊子”：

1. **鼠标劫持 vs 真正的后台静默**：
   * *Computer Use 模式*：必须强行把 Chrome 窗口置顶在最前台，AI 会控制你的物理鼠标在屏幕上乱晃。在它操作期间，**你无法使用这台电脑**（你一动鼠标它就会点错位置）。
   * *`chrome-mac-agent-skill`*：**100% 后台静默**。无论 Chrome 是最小化还是被其他窗口遮挡，AI 都能在后台标签页直接读写数据。你完全可以一边让 AI 发帖/查资料，一边看视频或敲代码，互不干扰！
2. **执行速度与延迟**：
   * *Computer Use 模式*：每一个点击动作都需要经历完整的循环：*截屏 -> 上传大图 -> 多模态视觉模型理解 -> 计算像素坐标 -> 移动鼠标 -> 点击 -> 等待 -> 再截屏验证*（单次操作耗时 **5~10 秒**）。
   * *`chrome-mac-agent-skill`*：通过 macOS 原生 AppleScript IPC 直接定位 DOM 节点，响应时间在 **50 毫秒以内**。
3. **Token 与账单成本**：
   * *Computer Use 模式*：每一轮交互都需要传输 2K/4K 全屏高清截图，跑完一个任务会消耗大量 Vision Tokens。
   * *`chrome-mac-agent-skill`*：**零视觉 Token 开销**，只传输精炼的文本指令，成本降低 95% 以上。
4. **最小权限与安全隐私**：
   * *Computer Use 模式*：必须向 AI 授予 macOS 极高风险的系统级权限：**「屏幕录制 (Screen Recording)」**（允许窥视你屏幕上的全部软件与隐私）和**「辅助功能 (Accessibility)」**（允许全局接管鼠标键盘）。
   * *`chrome-mac-agent-skill`*：**完全不需要 macOS 屏幕录制权限**，操作严格被隔离在 Google Chrome 内部，安全感拉满。

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
