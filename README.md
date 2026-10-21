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

**Agent-Native Skill to control active Google Chrome tabs on macOS without restarting or losing user logins.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![macOS: Supported](https://img.shields.io/badge/macOS-Supported-brightgreen?style=flat-square)](https://apple.com/macos)
[![Agent Skill](https://img.shields.io/badge/Agent-Skill-blueviolet?style=flat-square)](https://github.com/aicode915/chrome-mac-agent-skill)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-success?style=flat-square)](https://github.com/aicode915/chrome-mac-agent-skill)

[English](./README.md) • [简体中文](./README_CN.md)

</div>

---

## ⚡ The Problem

When AI coding assistants and autonomous agents (like Claude Code, Cursor, Windsurf, or custom LLM frameworks) need to interact with web pages, they typically rely on **Playwright**, **Puppeteer**, or isolated browser wrappers.

This introduces severe workflow friction:
1. **Lost User Logins**: They launch a blank, isolated browser session with no cookies. Every protected page requires re-logging in, solving 2FA, or scanning QR codes.
2. **Forced Process Killing**: Attaching to a running Chrome normally forces you to close all your open windows and restart Chrome with `--remote-debugging-port=9222`.
3. **Heavy Dependency Footprint**: Multi-gigabyte browser installations and complex driver setups just to check a web page.

---

## 💡 The Solution: `chrome-mac-agent-skill`

This skill connects your AI Agent directly to your **currently open Google Chrome windows on macOS** via native AppleScript:

- 🚀 **Zero Restarts**: Never close your current work tabs.
- 🔑 **Preserves All Active Sessions**: Keeps your Google accounts, enterprise SSO portals, GitHub, and social logins intact.
- 🪶 **Zero External Dependencies**: Pure Markdown instruction set leveraging macOS built-in `osascript`. No `npm install`, no `pip install`.
- 🛡 **Native Sheet & Dialog Handling**: Built-in recipes to bypass `beforeunload` blockers and automatically dismiss native macOS dialogs.
- 📦 **Modular Progressive Disclosure**: Split into scenario-specific reference files to conserve AI context tokens.

---

## 📊 Feature Comparison

| Capability | Playwright / Puppeteer | Remote Debugging (CDP) | OS "Computer Use" (Vision + Mouse) | `chrome-mac-agent-skill` 🚀 |
| :--- | :---: | :---: | :---: | :---: |
| **Keeps Active Logins & Cookies** | ❌ No (Isolated) | ⚠️ Only if started with flag | ✅ Yes (Shares screen) | **✅ Yes (100% Live)** |
| **Silent Background Execution** | ⚠️ Headless only | ⚠️ If port opened | ❌ **No (Steals mouse & focus)** | **✅ Yes (Zero mouse hijack)** |
| **Action Speed** | ~1 - 3s | ~500ms | ❌ **Slow (5 - 10s per click)** | **⚡ Blazing Fast (<50ms)** |
| **Token & Cost Footprint** | Low (HTML) | Low (HTML) | ❌ **High (Thousands of Vision Tokens)** | **💎 Optimal (Zero Image Tokens)** |
| **System Permissions Required** | None | None | ❌ **High (Screen Recording & Accessibility)** | **🔒 Minimal (Chrome Menu Toggle Only)** |
| **Precision** | High | High | ⚠️ **Prone to Retina coordinate drift** | **🎯 100% Deterministic (DOM Level)** |
| **Requires Browser Restart** | ❌ Yes | ❌ Yes (Must restart Chrome) | ✅ No | **✅ No (Instant Attach)** |

---

## 🥊 Why not OS-level "Computer Use" (Screenshots + Mouse Clicks)?

While modern multimodal models (like Claude 3.5 / 3.7 Computer Use) can physically see the screen and click coordinates, using "Computer Use" for routine browser automation is often like using a sledgehammer to crack a nut:

1. **Mouse Hijacking vs. True Background Silence**:
   * *Computer Use*: Forces Chrome to stay frontmost. The agent moves your physical cursor around the screen, rendering your computer unusable while the task is executing.
   * *`chrome-mac-agent-skill`*: **100% background silent**. You can write code, watch videos, or minimize Chrome—the agent interacts behind the scenes without touching your mouse or stealing focus.
2. **Speed & Latency**:
   * *Computer Use*: Every single click requires: *Capture Screenshot -> Upload Image -> Vision Model Reasoning -> Coordinate Math -> Move Cursor -> Click -> Wait -> Verify Screenshot* (~5 to 10 seconds per step).
   * *`chrome-mac-agent-skill`*: Directly targets DOM nodes via native IPC in **under 50 milliseconds**.
3. **Token & Dollar Savings**:
   * *Computer Use*: Streams full-resolution 2K/4K screenshots every turn, consuming thousands of vision tokens per interaction.
   * *`chrome-mac-agent-skill`*: **Zero visual token overhead**. Executes pure, surgical JavaScript expressions.
4. **Least-Privilege Security**:
   * *Computer Use*: Demands high-risk macOS system permissions (**Screen Recording** to see everything on your display and **Accessibility** to control global input).
   * *`chrome-mac-agent-skill`*: **Never needs macOS Screen Recording permissions**. Operates strictly inside Google Chrome via native Apple Events.

---

## 🚀 How to Add to Your AI Agent

### Option 1: Drop-in Folder (Antigravity, Cursor, Windsurf)
Clone or copy this repository into your project's `.agents/skills/` or `.skills/` directory:
```bash
mkdir -p .agents/skills
git clone https://github.com/aicode915/chrome-mac-agent-skill.git .agents/skills/active-chrome-control
```

### Option 2: Claude Code
Copy `SKILL.md` and the `references/` directory into your project's `.claude/skills/chrome-mac-agent-skill/`.

---

## 🧭 Repository Structure

```text
chrome-mac-agent-skill/
├── SKILL.md                 # Entrypoint: Quick cheatsheet, prerequisites, and on-demand router
├── README.md                # Human documentation (English)
├── README_CN.md             # Human documentation (Chinese)
├── LICENSE                  # MIT License
└── references/              # Focused scenario playbooks (loaded by Agent on demand)
    ├── tab-management.md    # Tab querying, filtering, switching, creating, and closing
    ├── page-interaction.md  # Body text extraction, JS execution, reactive form inputs, Shadow DOM
    ├── dialog-handling.md   # Dismissing native macOS sheets and stripping beforeunload blockers
    └── background-alive.md  # Tab focus-loss spoofing, anti-pause hooks, and video inspection
```

---

## 🛠 Manual CLI Quickstart

You can test these commands directly in your macOS terminal:

### List All Active Chrome Tabs:
```bash
osascript -e 'tell application "Google Chrome" to repeat with w in windows
    repeat with t in tabs of w
        log (id of t as text) & " | " & title of t & " | " & URL of t
    end repeat
end repeat'
```

### Read Current Page Content:
```bash
osascript -e 'tell application "Google Chrome" to repeat with w in windows
    repeat with t in tabs of w
        if URL of t contains "github.com" then
            return execute t javascript "document.body.innerText.substring(0, 1000)"
        end if
    end repeat
end repeat'
```

---

## ⚠️ Prerequisite (One-Time Setup)

To allow AppleScript to run JavaScript in Google Chrome:
1. Open Google Chrome.
2. In the macOS menu bar, navigate to: **View (查看)** -> **Developer (开发者)**.
3. Check **Allow JavaScript from Apple Events (允许 Apple 事件中的 JavaScript)**.

---

## 📄 License

[MIT](./LICENSE) © 2026 chrome-mac-agent-skill contributors.
