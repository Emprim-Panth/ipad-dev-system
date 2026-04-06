# tmux Setup

tmux is the foundation. It keeps terminal sessions alive on the Studio permanently — you detach, reconnect from any device, and everything is exactly where you left it.

---

## Install

```bash
brew install tmux
```

---

## Config

The full config lives at `~/.tmux.conf`. Copy from this repo:

```bash
cp config-templates/.tmux.conf ~/.tmux.conf
```

Key decisions in the config:
- **Prefix is `Ctrl+A`** — easier to hit than the default `Ctrl+B`
- **Status bar at top** — session name always visible
- **Mouse enabled** — scroll, click panes, resize on iPad touch
- **50,000 line scrollback** — never lose output
- **`Ctrl+A Q` detaches all clients** — useful when MacBook + Studio are both attached and you want to switch to iPad only

---

## The `ts` Command

`ts` is a wrapper script around tmux that makes session management fast.

**Install it:**
```bash
# Copy from this repo
cp scripts/ts ~/.local/bin/ts
chmod +x ~/.local/bin/ts
# Make sure ~/.local/bin is in your PATH
```

**Usage:**

```bash
ts                    # List all active sessions
ts studio             # Attach to (or create) "studio" session
ts new archon         # Create a new session named "archon"
ts kill old-session   # Kill a session
```

---

## Session Conventions

One session per project. Name them consistently:

| Session Name | Purpose |
|-------------|---------|
| `main` | General Studio shell, landing zone |
| `archon` | Archon-CAD development |
| `bim` | BIM Manager development |
| `worldtree` | World Tree development |
| `cortana` | Cortana AI stack management |
| `agents` | Active Claude Code agent sessions |

---

## tmux Key Reference

All shortcuts use the prefix `Ctrl+A`.

### Sessions
| Action | Keys |
|--------|------|
| **Session picker** | `Ctrl+A` then `s` (lowercase) |
| **Create new named session** | `Ctrl+A` then `S` (uppercase — prompts for name) |
| Next session | `Ctrl+A` then `N` |
| Previous session | `Ctrl+A` then `P` |
| Detach (leave session running) | `Ctrl+A` then `D` |
| Detach ALL clients | `Ctrl+A` then `Q` |

> **The `s` / `S` combo is the killer feature.** Lowercase `s` opens a tree-view picker of all sessions — navigate with arrows, enter to switch. Uppercase `S` prompts you to type a name and creates a brand new session. Between these two, you can manage any number of persistent project sessions without leaving tmux.

### Windows & Panes
| Action | Keys |
|--------|------|
| New window | `Ctrl+A` then `C` |
| Split vertical | `Ctrl+A` then `|` |
| Split horizontal | `Ctrl+A` then `-` |
| Move between panes | `Alt+Arrow keys` (no prefix needed) |
| Kill pane | `Ctrl+A` then `X` |
| Reload config | `Ctrl+A` then `R` |

### Scrollback
| Action | Keys |
|--------|------|
| Enter scroll mode | `Ctrl+A` then `[` |
| Exit scroll mode | `q` |
| Page up/down | `PgUp` / `PgDn` in scroll mode |

---

## Auto-Attach on SSH

Add this to `~/.zshrc` so that when you SSH in, it automatically attaches to the `main` session:

```bash
# Auto-attach to main tmux session on SSH
if [[ -n "$SSH_CONNECTION" && -z "$TMUX" ]]; then
    tmux new-session -A -s main
fi
```

This means `ssh studio-tmux` drops you straight into your workspace.

---

## Troubleshooting

**Sessions look blank / lost after reboot:**
tmux sessions don't survive a reboot by default. Use [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) if you need that, or just `ts new <name>` to recreate them. The work inside (code, git) is still there — only the running processes restart.

**Can't attach — "session already attached":**
```bash
tmux attach -d -t <name>   # Force detach other client and attach here
# OR use Ctrl+A then Q if you're already inside tmux
```

**Prefix not working on iPad:**
Make sure Termius's keyboard has `Ctrl` mapped. In Termius: Settings → Keyboard → Extended Keys, add `Ctrl`.
