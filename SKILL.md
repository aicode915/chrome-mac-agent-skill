---
name: active-chrome-mac-skill
description: "Controls and inspects currently active Google Chrome tabs on macOS via native AppleScript without restarting Chrome, launching separate browser profiles, or losing active user logins and cookies. Use when an AI Agent needs to interact with the user's live browser sessions."
---

# Active Chrome macOS Controller Skill

Control and inspect the user's **currently active Google Chrome tabs on macOS** directly. Unlike Playwright or Puppeteer which spawn isolated, cookie-less browser instances, this skill communicates with the running Chrome process via native macOS AppleScript, **preserving all active logins, enterprise SSO sessions, cookies, and local state with zero restarts**.

---

## ⚡ Prerequisites & Self-Healing

Google Chrome on macOS requires a one-time permission toggle:
> **Chrome Menu Bar** -> **View (查看)** -> **Developer (开发者)** -> check **Allow JavaScript from Apple Events (允许 Apple 事件中的 JavaScript)**.

### Fast Permission Health Check
Run this single command to verify access:
```bash
osascript -e 'tell application "Google Chrome" to execute front window'\''s active tab javascript "1+1"' 2>&1
```
* If it returns `2`, permissions are active.
* If it returns an error mentioning `Allow JavaScript from Apple Events` (Error -1753 or code 12), guide the user:
  > *"Please enable 'Allow JavaScript from Apple Events' in Chrome: View > Developer > Allow JavaScript from Apple Events."*

---

## 🧭 Progressive Reference Routing (On-Demand Loading)

To save context tokens, this skill is split into focused scenario modules. **Read the corresponding reference file only when your task requires it**:

| Scenario | When to Read | Reference File |
| :--- | :--- | :--- |
| **Tab Management** | Need to find tabs, switch tabs, open new URLs, or close tabs | [`references/tab-management.md`](./references/tab-management.md) |
| **DOM & Elements** | Need to extract page text/HTML, fill forms, or click buttons | [`references/page-interaction.md`](./references/page-interaction.md) |
| **Dialogs & Alerts** | Dealing with "Leave site?" / "Reload site?" sheets or dialogs | [`references/dialog-handling.md`](./references/dialog-handling.md) |
| **Background Keep-Alive** | Page pauses when switching tabs or monitors focus loss | [`references/background-alive.md`](./references/background-alive.md) |

---

## 🚀 Quick Execution Cheatsheet (High-Frequency One-Liners)

### 1. List Open Tabs (Titles, URLs & Tab IDs)
```bash
osascript -e 'tell application "Google Chrome"
    set out to ""
    repeat with w in windows
        repeat with t in tabs of w
            set out to out & (id of t as text) & " | " & (title of t) & " | " & (URL of t) & linefeed
        end repeat
    end repeat
    return out
end tell'
```

### 2. Extract Visible Text from Matching Tab
```bash
osascript -e 'tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "document.body.innerText.substring(0, 3000)"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell'
```

### 3. Execute Arbitrary JavaScript on Matching Tab
For complex multi-line JavaScript, always use **Heredoc (`<< 'EOF'`)** to eliminate string escaping bugs:
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "document.title + ' | ' + location.href"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

### 4. Click Button/Link by Visible Text (Shadow DOM Penetrating)
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    function deepFind(root) {
                        if (!root) return null;
                        const els = root.querySelectorAll('button, a, input[type=\"button\"], input[type=\"submit\"], [role=\"button\"], label');
                        for (const el of els) {
                            if (el.innerText && el.innerText.trim().includes('BUTTON_TEXT') && el.offsetParent !== null) return el;
                        }
                        for (const el of root.querySelectorAll('*')) {
                            if (el.shadowRoot) {
                                const found = deepFind(el.shadowRoot);
                                if (found) return found;
                            }
                        }
                        return null;
                    }
                    const target = deepFind(document);
                    if (target) {
                        target.scrollIntoView({ behavior: 'smooth', block: 'center' });
                        target.click();
                        return 'CLICKED';
                    }
                    return 'NOT_FOUND';
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 🛡 Security & Best Practices

1. **Target by URL Substring or Title**: Avoid hardcoding window indexes; always query by `URL of t contains ...` or `title of t contains ...`.
2. **Prevent Navigation Blockers**: When reloading or leaving pages with active forms, proactively inject `window.onbeforeunload = null;` to prevent native browser blocking dialogs (see [`references/dialog-handling.md`](./references/dialog-handling.md)).
3. **No External Dependencies**: Never install pip/npm packages for browser automation if native AppleScript suffices. Keep workflows zero-config and lightweight.
