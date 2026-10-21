# Reference: Tab Management Operations

This reference provides production-ready AppleScript snippets for discovering, targeting, creating, activating, and closing Google Chrome tabs on macOS.

---

## 1. List and Filter Tabs

### A. List All Open Tabs (with Window & Tab Index)
```bash
osascript -e 'tell application "Google Chrome"
    set out to ""
    set wIndex to 1
    repeat with w in windows
        set tIndex to 1
        repeat with t in tabs of w
            set out to out & (id of t as text) & " | Window " & wIndex & " Tab " & tIndex & " | " & (title of t) & " | " & (URL of t) & linefeed
            set tIndex to tIndex + 1
        end repeat
        set wIndex to wIndex + 1
    end repeat
    return out
end tell'
```

### B. Find Tab by URL Substring or Title
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if (URL of t contains "github.com") or (title of t contains "Pull Requests") then
                return (id of t as text) & " | " & (title of t) & " | " & (URL of t)
            end if
        end repeat
    end repeat
    return "NOT_FOUND"
end tell
EOF
```

---

## 2. Navigate and Open Tabs

### A. Navigate Existing Tab to a New URL
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN" then
                set URL of t to "https://app.example.com/dashboard"
                return "NAVIGATED"
            end if
        end repeat
    end repeat
    return "NOT_FOUND"
end tell
EOF
```

### B. Open a New Tab in the Frontmost Window
```bash
osascript -e 'tell application "Google Chrome"
    tell front window
        make new tab with properties {URL:"https://app.example.com"}
    end tell
    return "TAB_OPENED"
end tell'
```

### C. Open a New Background Tab (Without Stealing Focus)
```bash
osascript -e 'tell application "Google Chrome"
    tell front window
        set newTab to make new tab with properties {URL:"https://app.example.com"}
    end tell
    return "BACKGROUND_TAB_OPENED"
end tell'
```

---

## 3. Focus and Activation

### Bring Tab and Chrome Window to Front
```bash
osascript << 'EOF'
tell application "Google Chrome"
    activate
    repeat with w in windows
        repeat with i from 1 to count of tabs of w
            set t to tab i of w
            if URL of t contains "TARGET_DOMAIN" then
                set active tab index of w to i
                set index of w to 1
                return "ACTIVATED"
            end if
        end repeat
    end repeat
    return "NOT_FOUND"
end tell
EOF
```

---

## 4. Close Tabs

### Close Tab Matching Specific URL or Title
```bash
osascript << 'EOF'
tell application "Google Chrome"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains "TARGET_DOMAIN_TO_CLOSE" then
                close t
                return "CLOSED"
            end if
        end repeat
    end repeat
    return "NOT_FOUND"
end tell
EOF
```
