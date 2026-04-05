# Telegram Bot — Remote Command Interface

The Telegram bot lets you control the Studio from anywhere — your phone, iPad, Apple Watch, doesn't matter. If you have Telegram, you have access.

---

## Setup

### 1. Create the bot

1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. `/newbot` → give it a name → get the token
3. Save the token to `~/.cortana/secrets/telegram-bot-token`

```bash
mkdir -p ~/.cortana/secrets
echo "YOUR_BOT_TOKEN_HERE" > ~/.cortana/secrets/telegram-bot-token
chmod 600 ~/.cortana/secrets/telegram-bot-token
```

### 2. Get your Telegram user ID

Message [@userinfobot](https://t.me/userinfobot) — it replies with your user ID.

### 3. Configure the bot

Copy and edit the config template:

```bash
cp config-templates/telegram-config.json ~/.claude/telegram/config.json
```

Edit `config.json`:
```json
{
  "allowed_users": [YOUR_TELEGRAM_USER_ID]
}
```

> **Security note:** Only your user ID can use the bot. Anyone else's messages are silently ignored.

### 4. Install Python dependencies

```bash
python3 -m venv ~/telegram-venv
source ~/telegram-venv/bin/activate
pip install python-telegram-bot requests
```

### 5. Start the bot

```bash
cd ~/.claude/telegram
source ~/telegram-venv/bin/activate
python bot_v2.py &
echo $! > bot.pid
```

---

## All Commands

### Status & Health

| Command | Does |
|---------|------|
| `/health` | Full status: services, active sessions, project state |
| `/patrol` | Check all tmux sessions for errors/stalls right now |
| `/windows` | List open macOS windows |
| `/screenshot` | Screenshot the Studio desktop |

### Session Management

| Command | Does |
|---------|------|
| `/sessions` | List all tmux sessions with last activity |
| `/sessions create <name> [project]` | Create a named session |
| `/sessions kill <name>` | Kill a session (with confirmation) |
| `/sessions resume <name>` | Show recent output from a session |
| `/terminal <name>` | Last 30 lines of output from a session |
| `/terminal <name> <command>` | Send a command to a session |
| `/terminal <name> watch` | Stream output for 60 seconds |

### Project Operations

| Command | Does |
|---------|------|
| `/project <name>` | Project card: git status, tickets, recent commits |
| `/project <name> build` | Trigger build |
| `/project <name> test` | Run test suite |
| `/project <name> git` | Git status + recent log |
| `/project <name> tickets` | Open tickets from Compass |

### Compass (Project State)

| Command | Does |
|---------|------|
| `/compass` | All projects overview |
| `/compass <project>` | Current goal, phase, blockers |
| `/compass tickets` | All open tickets, sorted by priority |

### World Tree Dashboard

| Command | Does |
|---------|------|
| `/wt` | Screenshot full World Tree window |
| `/wt projects` | Screenshot projects panel |
| `/wt brain` | Screenshot Central Brain panel |
| `/wt status` | Text summary: projects, git state, decisions |

### Mac Control

| Command | Does |
|---------|------|
| `/do <instruction>` | Full autopilot task (uses local LLM to plan + execute) |
| `/click <x,y>` | Click at coordinates |
| `/type <text>` | Type text |
| `/key <key>` | Send a key press |
| `/open <AppName>` | Open an application |
| `/menu App > Menu > Item` | Navigate a menu |
| `/scroll up\|down [amount]` | Scroll |
| `/readui` | Read UI state of focused window |

---

## Example Workflows

**Check on a build from your phone:**
```
/project archon build
→ "Build started"
→ (2 mins later) "Build complete: 885 tests pass"
```

**See what's happening right now:**
```
/health
→ Services: 🟢 telegram | 🟢 ollama (3 models) | 🟢 cortex
   Sessions (4 active):
     main — idle 5m
     archon — building (2m)
     bim — attached
     cortana — idle 12m
   Projects: Archon-CAD: clean | BIM Manager: TASK-088 open
```

**Restart a stuck session:**
```
/sessions kill archon
/sessions create archon Archon-CAD
/terminal archon cd ~/Development/Archon-CAD && ts archon
```

---

## Keep-Alive / Auto-Start on Boot

Set the bot to start automatically with a launchd plist:

```bash
cp config-templates/com.cortana.telegram-bot.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.cortana.telegram-bot.plist
```

---

## Logs

```bash
tail -f ~/.claude/logs/telegram-bot.log
```

Logs rotate at 500KB, keeps 3 backups.
