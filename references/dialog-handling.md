# Reference: Native Dialog Handling & BeforeUnload Bypass

This reference guides how to handle and prevent macOS system-level sheets and browser dialogs (e.g., *"Reload site? Changes you made may not be saved"* with `[Cancel]` and `[Reload]` buttons).

---

## 1. Why Native Dialogs Block Automation

When a tab has active form fields, unsaved drafts, or dirty state (such as Twitter/X tweet compositing or form editors), reloading or navigating away triggers the browser's native `beforeunload` event.

Because this alert is a **native macOS sheet** rather than an HTML DOM element, standard web queries (`document.querySelector`) cannot detect or interact with it.

---

## 2. Tier 1: Proactive Prevention (Recommended • 0 Permissions Required)

The cleanest and most reliable strategy is to **strip beforeunload listeners before initiating reloads or navigation**:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                execute t javascript "(function() {
                    window.onbeforeunload = null;
                    window.addEventListener('beforeunload', function(e) {
                        e.stopImmediatePropagation();
                        delete e['returnValue'];
                    }, true);
                })()"
                reload t
                return "RELOADED_WITHOUT_DIALOG"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 3. Tier 2: Reactive Native Sheet Dismissal (macOS System Events)

If a confirmation sheet has **already appeared on screen**, use macOS `System Events` to automatically click the confirm button:

```bash
osascript << 'EOF'
try
    tell application "System Events"
        tell process "Google Chrome"
            repeat with w in windows
                if exists (sheet 1 of w) then
                    try
                        -- Match Chinese ("重新加载" / "离开") and English ("Reload" / "Leave")
                        click (first button of sheet 1 of w whose name is "重新加载" or name is "Reload" or name is "离开" or name is "Leave")
                        return "DISMISSED_NATIVE_SHEET"
                    on error
                        -- Fallback: press Return (default highlighted action button)
                        keystroke return
                        return "DISMISSED_VIA_ENTER"
                    end try
                end if
            end repeat
            return "NO_SHEET_DETECTED"
        end tell
    end tell
on error errMsg
    return "ACCESSIBILITY_PERMISSION_NOTE: " & errMsg
end try
EOF
```

> [!NOTE]
> If `System Events` outputs error `-25211` ("osascript is not allowed assistive access"), macOS Accessibility permission has not been granted to Terminal. Use **Tier 1 (Proactive Prevention)** instead, as it requires zero extra macOS system permissions.

---

## 4. One-Click Safe Reload Snippet

Combine both strategies into a unified safe reload operation:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                -- 1. Strip beforeunload handlers
                execute t javascript "window.onbeforeunload = null;"
                -- 2. Trigger reload
                reload t
                return "SAFE_RELOAD_SUCCESS"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```
