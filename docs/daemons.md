# Daemon Infrastructure

The Studio runs several background services that keep the system alive 24/7. All managed via macOS launchd.

---

## Active Services

| Service | PID File / Label | Purpose |
|---------|-----------------|---------|
| **Ollama** | `com.cortana.ollama` | Local LLM inference server |
| **Cortana Daemon** | `com.cortana.daemon` | Main daemon — polling, health checks, notifications |
| **Cortana Harness** | `com.cortana.harness` | Session dispatcher — launches and manages agent sessions |
| **Session Guardian** | `com.cortana.session-guardian` | Monitors tmux orchestrator, restarts on death |
| **Telegram Bot** | `com.cortana.telegram` | Telegram bridge for remote control |
| **Hivemind** | `com.cortana.hivemind` | cortana-core coordinator |
| **Mnemos** | `com.cortana.mnemos` | Memory daemon |
| **Caffeinate** | `com.cortana.caffeinate` | Prevents Mac from sleeping |
| **code-server** | `homebrew.mxcl.code-server` | VS Code in browser |

---

## Managing Services

```bash
# List all Cortana services
launchctl list | grep cortana

# Start a service
launchctl kickstart gui/$(id -u)/com.cortana.harness

# Stop a service
launchctl kill SIGTERM gui/$(id -u)/com.cortana.harness

# Restart a service
launchctl kickstart -k gui/$(id -u)/com.cortana.harness

# Check service status
launchctl print gui/$(id -u)/com.cortana.harness

# For brew-managed services
brew services list
brew services restart code-server
```

---

## Key Daemon Details

### Cortana Daemon (`cortana-daemon.py`)
- **Polls:** Every 5s (general), 15s (gateway), 30s (health)
- **Session timeout:** 30 days
- **Max concurrent tasks:** 3
- **Auto-cleanup:** Every 300s
- **Notifications:** On start and completion

### Cortana Harness (`cortana-harness.ts`)
- **Runtime:** Bun
- **Purpose:** Dispatches Claude Code sessions to tmux
- **Recovery:** Persists pool state to JSON after every mutation
- **MCP config for leads:** `~/.cortana/harness/mcp-configs/lead-mcp.json`
- **Socket:** `~/.cortana/harness/harness.sock`

### Session Guardian
- **Check interval:** Every 15 seconds
- **Monitors:** tmux orchestrator session liveness
- **On death:** Restarts with full context injection (via Ollama compression)
- **Escalation:** Notifies Evan via Herald if restarted 5+ times/hour
- **Context restoration:** Reads last 40 messages, compresses via Ollama, injects into new session

---

## Launchd Plist Location

All plists: `~/Library/LaunchAgents/`

```bash
ls ~/Library/LaunchAgents/com.cortana.*.plist
```

### Plist Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.cortana.service-name</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/bun</string>
        <string>/path/to/script.ts</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/Users/you/Development/cortana-core</string>
    <key>KeepAlive</key>
    <dict>
        <key>Crashed</key><true/>
        <key>SuccessfulExit</key><false/>
    </dict>
    <key>ThrottleInterval</key>
    <integer>30</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/you/.cortana/logs/service-stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/you/.cortana/logs/service-stderr.log</string>
</dict>
</plist>
```

---

## Logs

All daemon logs: `~/.cortana/logs/`

```bash
# Tail a specific service
tail -f ~/.cortana/logs/cortana-harness-stdout.log

# All recent logs
ls -lt ~/.cortana/logs/ | head -20
```

---

## Caffeinate

Prevents the Mac from sleeping — critical for an always-on Studio.

```bash
# Check if running
pgrep caffeinate

# The launchd service runs:
caffeinate -dims
# -d: prevent display sleep
# -i: prevent idle sleep
# -m: prevent disk sleep
# -s: prevent system sleep
```
