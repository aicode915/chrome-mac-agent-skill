# Reference: Background Keep-Alive & Anti-Pause Hooks

Many web applications (such as corporate training portals, exam engines, or video dashboards) pause execution or flag automated tests when their browser tab loses active focus (`blur`) or when users switch tabs (`visibilitychange`).

This reference provides techniques to keep active tabs running in the background and prevent pause events.

---

## 1. Universal Anti-Pause & Focus Freeze Hook

Inject this payload into the target tab immediately after opening or navigation:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    try {
                        // 1. Block focus-loss and tab-switch events
                        window.addEventListener('blur', function(e) { e.stopImmediatePropagation(); }, true);
                        window.addEventListener('mouseleave', function(e) { e.stopImmediatePropagation(); }, true);
                        document.addEventListener('visibilitychange', function(e) { e.stopImmediatePropagation(); }, true);

                        // 2. Freeze visibility state to always active
                        Object.defineProperty(document, 'hidden', { value: false, writable: false });
                        Object.defineProperty(document, 'visibilityState', { value: 'visible', writable: false });

                        return 'ANTI_PAUSE_HOOKS_ACTIVE';
                    } catch(err) {
                        return 'ERROR: ' + err.message;
                    }
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 2. Inspect HTML5 Video Playback State

Inspect video progress without needing to activate the tab:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    const v = document.querySelector('video');
                    if (!v) return 'NO_VIDEO_FOUND';
                    return JSON.stringify({
                        paused: v.paused,
                        currentTime: Math.floor(v.currentTime),
                        duration: Math.floor(v.duration),
                        progress: v.duration ? ((v.currentTime / v.duration) * 100).toFixed(1) + '%' : '0%'
                    });
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 3. Human-like Sequential Actions with Jitter

When interacting with platforms sensitive to automated batch operations, execute sequential steps with natural delays rather than synchronous loops:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(async function() {
                    const items = Array.from(document.querySelectorAll('.clickable-option, .list-item'));
                    for (let i = 0; i < Math.min(items.length, 5); i++) {
                        const el = items[i];
                        el.scrollIntoView({ behavior: 'smooth', block: 'center' });
                        
                        // Randomized human delay: 800ms - 2200ms
                        const delay = Math.floor(Math.random() * 1400) + 800;
                        await new Promise(r => setTimeout(r, delay));

                        el.dispatchEvent(new MouseEvent('mousedown', { bubbles: true }));
                        el.dispatchEvent(new MouseEvent('mouseup', { bubbles: true }));
                        el.click();
                    }
                    return 'COMPLETED_NATURALLY';
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```
