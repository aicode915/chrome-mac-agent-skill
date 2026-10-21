# Reference: Page Interaction & DOM Manipulation

This reference provides JavaScript payloads designed to be injected via AppleScript (`execute t javascript ...`) to reliably read page state, interact with elements, fill reactive forms, and traverse Shadow DOM trees.

---

## 1. Reading Page Content

### A. Extract Clean Visible Body Text (with length limit)
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    return document.body.innerText.substring(0, 4000);
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

### B. Extract Structured Data or Links
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    const links = Array.from(document.querySelectorAll('a[href]'))
                        .map(a => ({ text: a.innerText.trim(), href: a.href }))
                        .filter(l => l.text.length > 0);
                    return JSON.stringify(links.slice(0, 20));
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 2. Interactive Clicking & Shadow DOM Penetration

Standard `document.querySelector` fails when elements live inside Web Components or Shadow Roots. Use this recursive traversal helper:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    function deepFind(root, targetText) {
                        if (!root) return null;
                        const candidates = root.querySelectorAll('button, a, input[type=\"button\"], input[type=\"submit\"], [role=\"button\"], label, span');
                        for (const el of candidates) {
                            if (el.innerText && el.innerText.trim().includes(targetText) && el.offsetParent !== null) {
                                return el;
                            }
                        }
                        for (const el of root.querySelectorAll('*')) {
                            if (el.shadowRoot) {
                                const found = deepFind(el.shadowRoot, targetText);
                                if (found) return found;
                            }
                        }
                        return null;
                    }

                    const target = deepFind(document, 'Submit');
                    if (target) {
                        target.scrollIntoView({ behavior: 'smooth', block: 'center' });
                        // Dispatch mouse sequence for high-compatibility frameworks
                        target.dispatchEvent(new MouseEvent('mousedown', { bubbles: true }));
                        target.dispatchEvent(new MouseEvent('mouseup', { bubbles: true }));
                        target.click();
                        return 'CLICKED';
                    }
                    return 'ELEMENT_NOT_FOUND';
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 3. Reactive Form Filling (React, Vue, Angular)

Simply setting `input.value = "..."` frequently fails to update framework state (e.g., Vue `v-model` or React synthetic event bindings). Always dispatch synthetic input and change events:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    const input = document.querySelector('input[name=\"search\"], #username, [placeholder*=\"Search\"]');
                    if (!input) return 'INPUT_NOT_FOUND';

                    input.focus();
                    input.value = 'My Query';
                    input.dispatchEvent(new Event('input', { bubbles: true }));
                    input.dispatchEvent(new Event('change', { bubbles: true }));
                    return 'INPUT_FILLED';
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```

---

## 4. Extracting Captcha Image Data

Extracts Base64 image data strings so multimodal agents can visually solve human verification challenges:

```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                return execute t javascript "(function() {
                    const img = document.querySelector('.captcha-image, img[src*=\"captcha\"], img[src*=\"code\"], img[src^=\"data:image\"]');
                    if (!img) return 'NO_CAPTCHA_IMAGE';
                    
                    // If image is already data URI, return directly
                    if (img.src.startsWith('data:image')) return img.src;

                    // Otherwise draw to offscreen canvas to obtain Base64
                    try {
                        const canvas = document.createElement('canvas');
                        canvas.width = img.naturalWidth || img.width;
                        canvas.height = img.naturalHeight || img.height;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(img, 0, 0);
                        return canvas.toDataURL('image/png');
                    } catch(e) {
                        return img.src;
                    }
                })()"
            end if
        end repeat
    end repeat
    return "TAB_NOT_FOUND"
end tell
EOF
```
