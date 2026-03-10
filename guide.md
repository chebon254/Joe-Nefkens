# Claude in Chrome ↔ Claude Desktop (Cowork) — Connection Fix Guide

> **Before our Zoom session:** Please read through this guide and complete the **Pre-Session Checklist** at the top. The more you do beforehand, the faster we resolve it live.

---

## Pre-Session Checklist (Do This Before We Meet)

- [ ] Update **Claude Desktop** to the latest version → open Claude Desktop → Help → Check for Updates
- [ ] Update **Google Chrome** to the latest version → `chrome://settings/help`
- [ ] Confirm you are using **Google Chrome** (not Brave, Arc, Opera, or any other browser)
- [ ] Make sure you are logged into Claude Desktop and Chrome with the **same Anthropic account email**
- [ ] Have access to open **Terminal** (Mac) or **Command Prompt** (Windows) — we will need it
- [ ] Close **all other screen-sharing or remote desktop apps** before our Zoom call

---

## Understanding What's Happening (Plain English)

Think of it like a telephone directory. When Claude Desktop installs, it writes a small configuration file (like a phone book entry) that tells Chrome: *"To reach me, call this program."*

When Chrome launches the extension, it looks up that directory entry. If the entry is **missing**, **broken**, or **pointing to the wrong app** — Chrome cannot make the call, and you see:

```
Claude in Chrome Extension — Disconnected
```

Our job is simply to find that broken entry and fix it.

---

## Section 1 — macOS Users

### Step 1: Open Terminal
Press **Cmd + Space**, type `Terminal`, press Enter.

### Step 2: Check the Native Messaging Hosts folder
Paste this command and press Enter:

```bash
ls ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/
```

**What you should see:**
```
com.anthropic.claude_browser_extension.json
```

**If you also see this file, that is the conflict:**
```
com.anthropic.claude_code_browser_extension.json
```

### Step 3: Read the Claude Desktop config file
Paste this and press Enter:

```bash
cat ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/com.anthropic.claude_browser_extension.json
```

You should see something like:

```json
{
  "name": "com.anthropic.claude_browser_extension",
  "description": "Claude Desktop Browser Integration",
  "path": "/Applications/Claude.app/Contents/Helpers/chrome-native-host",
  "type": "stdio",
  "allowed_origins": [
    "chrome-extension://fcoeoabgfenejglbffodgkkbkcdhcgfn/"
  ]
}
```

**Screenshot this output and share it with me before our call.**

### Step 4: Verify the helper binary exists
Copy the `path` value from above and run:

```bash
ls -la /Applications/Claude.app/Contents/Helpers/chrome-native-host
```

- ✅ If it shows file details → binary is fine
- ❌ If it says `No such file or directory` → Claude Desktop needs reinstalling

### Step 5: Check Claude Desktop logs
```bash
tail -50 ~/Library/Logs/Claude/chrome-native-host.log
```

Copy and paste the output — share it with me before the call.

---

## Section 2 — Windows Users

### Step 1: Open Command Prompt
Press **Win + R**, type `cmd`, press Enter.

### Step 2: Check the Registry for native messaging hosts
In Command Prompt, paste:

```cmd
reg query "HKCU\Software\Google\Chrome\NativeMessagingHosts" /s
```

Look for entries named:
- `com.anthropic.claude_browser_extension` ← should be here
- `com.anthropic.claude_code_browser_extension` ← if this is also here, that's the conflict

### Step 3: Check the JSON config path
The registry entry will contain a path to a `.json` file. Navigate to that folder in File Explorer and open the `.json` file with Notepad.

It should contain the same structure shown in the macOS section above. **Screenshot it and share with me.**

### Step 4: Check Claude Desktop logs
Open File Explorer and navigate to:
```
C:\Users\<YourUsername>\AppData\Roaming\Claude\Logs\
```

Open `chrome-native-host.log` with Notepad. Copy the last 50 lines and share them with me.

---

## Section 3 — Check the Chrome Extension Itself

1. Open Chrome and go to: `chrome://extensions`
2. Find **Claude in Chrome** in the list
3. Make sure the toggle is **ON** (enabled)
4. Click **"Service Worker"** or **"background page"** link under the extension
5. A DevTools window opens — click the **Console** tab
6. Look for any red error messages

**Common errors and what they mean:**

| Error Message | What It Means |
|---|---|
| `Specified native messaging host not found` | The JSON config file is missing |
| `Native host has exited` | The helper binary crashed or path is wrong |
| `Access to native messaging host is forbidden` | Extension ID mismatch in the config |
| `Failed to connect` | Claude Desktop is not running or Cowork is disabled |

**Screenshot the Console errors and share them with me.**

---

## Section 4 — Check Claude Desktop Settings

Inside Claude Desktop:

1. Click the **Settings** icon (gear/cog)
2. Look for a **"Connectors"** or **"Browser"** section
3. Make sure the **Chrome integration toggle is ON**
4. If there is an option that says **"Cowork mode"** — make sure that is also enabled

If the toggle is off, the helper binary never starts, and Chrome cannot connect regardless of anything else.

---

## Section 5 — Fixes We Will Apply Together on the Call

We will not do these alone — we'll walk through them together on Zoom. Listed here so you know what to expect:

### Fix A — Conflict between Claude Desktop and Claude Code (most common)
If both `com.anthropic.claude_browser_extension.json` AND `com.anthropic.claude_code_browser_extension.json` exist, we will temporarily rename the Code one:

```bash
# macOS — run in Terminal
mv ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json \
   ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json.backup
```

Then fully quit Chrome (Cmd+Q / Alt+F4) and relaunch.

### Fix B — Claude Desktop reinstall
If the helper binary is missing:
1. Quit Claude Desktop completely
2. Delete `/Applications/Claude.app` (Mac) or uninstall via Control Panel (Windows)
3. Download the latest version from [claude.ai/download](https://claude.ai/download)
4. Reinstall and relaunch

### Fix C — Broken JSON config
If the config file has the wrong path or wrong extension ID, we will edit it together using Terminal/Notepad. The correct extension ID is:
```
fcoeoabgfenejglbffodgkkbkcdhcgfn
```

### Fix D — Full restart sequence
After any fix, always do this in order:
1. Quit Chrome completely (**Cmd+Q** on Mac, **Alt+F4** on Windows — not just close the window)
2. Quit Claude Desktop completely
3. Reopen Claude Desktop first — wait 10 seconds
4. Reopen Chrome
5. Click the Claude in Chrome extension icon and check for the green connected status

---

## How to Confirm It's Working

After our fix, you should see:

1. The Claude in Chrome extension icon shows a **green connected indicator**
2. In Claude Desktop → Cowork mode, you can issue a browser command like *"take a screenshot of the current tab"*
3. Claude Desktop executes the command and returns a result from Chrome

If all three work — you're done! ✅


*Guide prepared by your freelancer for the Zoom troubleshooting session.*