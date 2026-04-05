# Restart Guide

Quick reference for restarting any part of the system. Use this when something's stuck, hung, or misbehaving.

---

## Restart the Telegram Bot

The most common restart needed.

```bash
# Method 1: Kill by PID file
kill $(cat ~/.claude/telegram/bot.pid) 2>/dev/null
sleep 1
cd ~/.claude/telegram && source ~/telegram-venv/bin/activate && python bot_v2.py &

# Method 2: Kill by name (if PID file is stale)
pkill -f "python bot_v2.py"
sleep 1
cd ~/.claude/telegram && source ~/telegram-venv/bin/activate && python bot_v2.py &
```

**From Telegram** (if the bot is still alive enough to respond):
```
/do restart telegram bot
```

---

## Restart a tmux Session (Claude Code inside tmux)

If a Claude Code session is hung or needs a fresh start:

```bash
# Kill the specific session
ts kill <session-name>

# Recreate it
ts new <session-name>

# Attach to it
ts <session-name>

# Then restart your work inside:
cd ~/Development/<project>
claude   # Start Claude Code interactively
# OR
claude -p "continue with TASK-xxx" --dangerously-skip-permissions &
```

---

## Restart Ollama

If models are not responding or loading:

```bash
# Restart the Ollama service
brew services restart ollama

# Check status
ollama list
ollama ps  # Shows what's currently in GPU memory

# If models aren't loading into GPU:
OLLAMA_NUM_PARALLEL=4 OLLAMA_FLASH_ATTENTION=1 ollama serve &
```

---

## Restart the CORTEX (iMessage AI)

```bash
# Stop
pkill -f cortana-cortex

# Start
cortana-cortex &

# Check logs
tail -f ~/.claude/logs/cortex.log
```

---

## Restart All Cortana Services

```bash
# Restart everything at once
cortana-ctl restart all

# Or individually:
cortana-ctl restart telegram
cortana-ctl restart cortex
cortana-ctl restart patrol
```

---

## Restart the Studio SSH Server

Only needed if SSH stops accepting connections:

```bash
sudo launchctl stop com.openssh.sshd
sudo launchctl start com.openssh.sshd
```

Or from System Settings: Sharing → Remote Login → toggle off and on.

---

## Restart tmux Server Completely

**Nuclear option** — kills ALL sessions. Use only if tmux itself is broken.

```bash
tmux kill-server

# Then recreate your main session
tmux new-session -s main
```

---

## If the Studio Is Unreachable

1. **Check Tailscale:** Open the Tailscale app on your iPad/MacBook — is the Studio showing as online?
2. **Check the Studio is on:** It should never sleep. If it did (power outage, etc.), power it back on.
3. **Tailscale reconnect:** If Tailscale shows it offline, wait 30s — it reconnects automatically. Or on the Studio: `sudo tailscale up`

---

## Force-Kill a Stuck Claude Code Process

```bash
# Find it
pgrep -a claude

# Kill by PID
kill -9 <PID>

# Kill all Claude Code processes (careful)
pkill -9 -f "claude"
```

---

## Check What's Running

```bash
# All cortana processes
pgrep -la cortana

# Telegram bot
pgrep -la "bot_v2.py"

# Ollama
pgrep -la ollama

# All tmux sessions
ts

# System resource usage
htop   # or: top
```
