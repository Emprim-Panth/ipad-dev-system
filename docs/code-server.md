# Code-Server (VS Code in Browser)

Full VS Code IDE running on the Studio, accessible from any browser. iPad, laptop, phone — if it has a browser and Tailscale, you're in.

---

## Why This Exists

SSH terminals work, but conversations with Claude Code need screen real estate. code-server gives you:
- Full VS Code editor with syntax highlighting, IntelliSense, extensions
- Integrated terminal (full height, not crammed at the bottom)
- File explorer sidebar — browse, create, rename, delete files on the Studio
- Works on iPad Safari, any laptop browser, anywhere

---

## Install

```bash
brew install code-server
```

---

## Configuration

**Config file:** `~/.config/code-server/config.yaml`

```yaml
bind-addr: 0.0.0.0:8080
auth: password
password: YOUR_PASSWORD_HERE
cert: false
```

- `0.0.0.0` makes it reachable from other devices (not just localhost)
- `auth: password` requires a password on the login page
- Port `8080` is the default — change if you need to

**Copy from this repo:**
```bash
cp config-templates/code-server-config.yaml ~/.config/code-server/config.yaml
# Then edit the password
```

---

## VS Code Settings

**Settings file:** `~/.local/share/code-server/User/settings.json`

Our settings are optimized for terminal-heavy remote dev work:

```json
{
  // Terminal takes priority — opens as editor tab, not bottom panel
  "terminal.integrated.defaultLocation": "editor",
  "terminal.integrated.fontSize": 16,
  "terminal.integrated.lineHeight": 1.3,
  "terminal.integrated.fontFamily": "MesloLGS NF, Menlo, Monaco, monospace",
  "terminal.integrated.cursorStyle": "line",
  "terminal.integrated.cursorBlinking": true,
  "terminal.integrated.scrollback": 10000,
  "terminal.integrated.defaultProfile.osx": "zsh",

  // Editor
  "editor.fontSize": 15,
  "editor.lineHeight": 1.5,
  "editor.wordWrap": "on",
  "editor.minimap.enabled": false,

  // Clean UI — less chrome, more workspace
  "workbench.activityBar.location": "top",
  "workbench.sideBar.location": "left",
  "breadcrumbs.enabled": false,
  "workbench.statusBar.visible": true,
  "workbench.tips.enabled": false,
  "editor.folding": false,
  "workbench.tree.indent": 16,

  // Dark theme
  "workbench.colorTheme": "Visual Studio Dark",
  "window.menuBarVisibility": "classic"
}
```

**Copy from this repo:**
```bash
mkdir -p ~/.local/share/code-server/User
cp config-templates/code-server-settings.json ~/.local/share/code-server/User/settings.json
```

---

## Start / Stop / Restart

```bash
# Start as background service (auto-starts on login)
brew services start code-server

# Stop
brew services stop code-server

# Restart (after config changes)
brew services restart code-server

# Run manually (foreground, for debugging)
code-server
```

**Logs:** `/opt/homebrew/var/log/code-server.log`

---

## Accessing It

### From your local network
```
http://YOUR_STUDIO_IP:8080
```

### From anywhere (via Tailscale)
```
http://your-studio.tailnet.ts.net:8080
```

Bookmark this URL. It works from any network as long as your device has Tailscale connected.

See [Tailscale Setup](tailscale.md) for details.

---

## Layout Tips

The default layout opens with the file explorer on the left and a clean editor area. For terminal-heavy work:

### Get a Terminal Open
- **Menu → Terminal → New Terminal** (or hamburger menu ☰ → Terminal → New Terminal)
- Terminal opens as an editor tab — full height, not a tiny bottom panel

### Useful Layouts

| Layout | How | Good For |
|--------|-----|----------|
| Terminal only | Close sidebar (`Ctrl+B`), open terminal tab | Claude Code conversations |
| Side-by-side | Terminal tab + file tab, drag to split | Editing while running agents |
| Zen mode | `Cmd+K Z` | Maximum screen real estate |
| Multiple terminals | Click `+` in terminal tab bar | Parallel sessions |

### Keyboard Shortcuts

| Action | Keys |
|--------|------|
| New terminal | `` Ctrl+` `` |
| Toggle sidebar | `Ctrl+B` |
| Zen mode (full screen, no chrome) | `Cmd+K Z` |
| Split editor | `Cmd+\` |
| Quick file open | `Cmd+P` |
| Command palette | `Cmd+Shift+P` |

---

## Running tmux Inside code-server

Open a terminal in code-server, then:

```bash
# Attach to existing session
tmux attach -t main

# Or use the ts wrapper
ts main

# Or use the session picker
# (once inside tmux: Ctrl+A then s)
```

This gives you the best of both worlds — VS Code's file explorer and editor alongside your persistent tmux sessions.

---

## Troubleshooting

**Can't connect from iPad:**
1. Is code-server running? `brew services list | grep code-server`
2. Is Tailscale connected on both devices? Check the Tailscale app
3. Try the IP directly: `http://100.x.x.x:8080`

**Terminal too small / font issues:**
Adjust in Settings (`Cmd+,`):
- `terminal.integrated.fontSize` — bump to 18 or 20 for iPad
- `terminal.integrated.lineHeight` — try 1.4 or 1.5

**Password not working:**
Check `~/.config/code-server/config.yaml` — the password is stored in plain text there.

**Extensions not loading:**
code-server uses the Open VSX registry, not Microsoft's marketplace. Most popular extensions are available, but some Microsoft-specific ones aren't. Search Open VSX: https://open-vsx.org/
