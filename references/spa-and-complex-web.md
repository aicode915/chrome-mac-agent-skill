# Reference: Modern SPAs & Complex Web Applications

This reference provides battle-tested patterns for handling modern Single Page Applications (React, Vue, Next.js), complex rich-text frameworks (Draft.js, Lexical, ProseMirror), virtualized lists, client-side routing, and AppleScript IPC return value constraints.

---

## 1. AppleScript Safe IIFE & JSON Serialization

### Problem: The `missing value` Trap
AppleScript's `execute tab javascript ...` bridge only supports primitive types (`string`, `integer`, `boolean`). If your JavaScript returns `undefined`, `null`, a DOM Node (`HTMLDivElement`), a function, or throws an unhandled error, AppleScript silently returns `missing value` or throws an error.

### Production Solution: Safe Defensive IIFE Wrapper
Always wrap injected scripts in a self-contained IIFE with `try...catch` and explicit `JSON.stringify`:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(() => {
                    try {
                        const items = Array.from(document.querySelectorAll('.target-item'))
                            .map(el => el.innerText.trim())
                            .filter(Boolean);
                        return JSON.stringify(items.slice(0, 50));
                    } catch (e) {
                        return 'ERROR: ' + e.message;
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

## 2. Virtualized Lists & Long Feeds (Twitter, Slack, Gmail, Feishu)

### Problem: DOM Destruction in Virtual Windows
Modern web apps use virtualization (`react-window`, `react-virtualized`, `[data-testid="cellInnerDiv"]`, `role="feed"`). DOM nodes outside the current viewport are recycled or unmounted. Traditional `document.querySelectorAll()` returns only what fits on screen.

### Production Solution: Incremental Scroll & Collect
Dispatch smooth scroll increments with micro-delays to allow the virtualizer to mount upcoming nodes:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(() => {
                    return (async () => {
                        const collected = new Set();
                        for (let step = 0; step < 5; step++) {
                            const cells = document.querySelectorAll('[data-testid=\"cellInnerDiv\"], [role=\"row\"], article');
                            cells.forEach(c => {
                                const txt = c.innerText ? c.innerText.trim() : '';
                                if (txt.length > 0) collected.add(txt);
                            });
                            window.scrollBy({ top: 400, behavior: 'smooth' });
                            await new Promise(r => setTimeout(r, 600));
                        }
                        return JSON.stringify(Array.from(collected).slice(0, 30));
                    })();
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 3. SPA Client-Side Route Synchronization (`history.pushState`)

### Problem: Asynchronous Navigation Without Page Reload
When clicking links or buttons in modern SPAs, navigation occurs via `history.pushState()`. Reading `URL of t` immediately after `.click()` often returns the *old* URL because the transition hasn't completed in the browser process.

### Production Solution: Polling with Timeout

```applescript
tell application "Google Chrome"
    -- 1. Trigger the SPA client-side navigation
    execute targetTab javascript "document.querySelector('a.route-link').click()"
    
    -- 2. Wait for route transition (poll up to 5 seconds)
    set hasNavigated to false
    repeat 10 times
        set curUrl to URL of targetTab
        if curUrl contains "expected-subpath" then
            set hasNavigated to true
            exit repeat
        end if
        delay 0.5
    end repeat
end tell
```

---

## 4. Complex Rich-Text Editors (Draft.js, Lexical, ProseMirror)

### Problem: Immutable React State Disconnect
Setting `.innerText`, `.innerHTML`, or `.value` on a `[contenteditable="true"]` element (used in Twitter/X, Slack, Notion, Jira) does **not** update the underlying framework's immutable state. The "Submit" or "Tweet" button remains disabled (`disabled: true`).

### Solution A: Web Intent Routing (Recommended for Social/Sharing Platforms)
For platforms that support Web Intents (Twitter/X, Reddit, Mastodon, LinkedIn), bypass the UI editor entirely. Open a new tab with the intent URL:

```applescript
tell application "Google Chrome"
    tell front window
        -- Text and status ID are pre-parsed into React state upon page load
        make new tab with properties {URL:"https://x.com/intent/tweet?in_reply_to=123456789&text=Hello%20World"}
    end tell
end tell
```
Once loaded, the submit button is natively active. A simple `.click()` completes the action.

### Solution B: Synthetic InputEvent Dispatch
For apps without Web Intents, focus the element and use `execCommand('insertText')` combined with synthetic `InputEvent` dispatching:

```javascript
(() => {
    const editor = document.querySelector('[contenteditable="true"]');
    if (!editor) return 'EDITOR_NOT_FOUND';
    
    editor.focus();
    document.execCommand('selectAll', false, null);
    document.execCommand('insertText', false, 'Your text content here');
    editor.dispatchEvent(new Event('input', { bubbles: true }));
    return 'INSERTED';
})()
```

---

## 5. Platform Notification Feed Disambiguation

### Problem: Algorithmic Recommendations Mixed with Direct Interactions
Platforms like X, LinkedIn, and Weibo mix algorithmic suggestions (e.g. sparkle ✨ "Highlight for you" or "Recommended topic") directly into primary notification feeds (`/notifications`). An AI Agent must not confuse these with actionable user replies or mentions.

### Production Guidelines:
1. **Target Specific Sub-Routes**: Navigate directly to `/notifications/mentions` or `/notifications/unread` instead of the broad `/notifications` feed.
2. **Inspect Badge/Icon Semantics**: In JavaScript, check the SVG icon's `aria-label` or path data to distinguish direct interactions (replies, mentions, quotes) from algorithmic highlights before taking action.
