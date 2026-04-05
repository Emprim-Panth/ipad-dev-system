# Shell Commands Reference

All the commands you'll actually use day-to-day, organized by what you're trying to do.

---

## Session Management

```bash
ts                          # List all active tmux sessions
ts <name>                   # Attach to named session (or create if new)
ts new <name>               # Create new session
ts kill <name>              # Kill a session
```

**Inside tmux:**
```
Ctrl+A S                    # Interactive session picker
Ctrl+A N / P                # Next / Previous session
Ctrl+A D                    # Detach (leave session running)
Ctrl+A Q                    # Detach ALL clients (free the Studio from other devices)
```

---

## AI & Models

```bash
ollama list                 # All installed models
ollama ps                   # Models currently loaded in GPU memory
ollama pull <model>         # Download a model
ollama run qwen2.5:72b      # Interactive chat with a model

claude                      # Start Claude Code interactively
claude -p "task"            # Run Claude Code headlessly (pipe mode)
claude --version            # Check version
```

---

## Cortana System

```bash
cortana-health              # Quick system health check
cortana-status              # Full status (all services, sessions, projects)
cortana-ctl restart telegram  # Restart Telegram bot
cortana-ctl restart all     # Restart all managed services

compass_overview            # All projects: goal, phase, blockers
compass_status <project>    # One project's current state
compass_update <project>    # Update project state interactively
```

---

## Git & Projects

```bash
# Standard git (nothing custom)
git status
git log --oneline -10
git diff
git add -p                  # Stage interactively (good habit)
git commit -m "message"
git push
```

---

## Tailscale & Network

```bash
tailscale status            # All connected devices
tailscale ip -4             # This machine's Tailscale IP
tailscale ping <ip>         # Ping another Tailscale device
sudo tailscale up           # Reconnect if disconnected
```

---

## Process & System

```bash
pgrep -la <name>            # Find processes by name
kill $(cat <file>.pid)      # Kill process by PID file
htop                        # Interactive process viewer (brew install htop)
df -h                       # Disk usage
free -m                     # Not available on macOS, use: vm_stat
```

---

## Logs

```bash
tail -f ~/.claude/logs/telegram-bot.log   # Telegram bot log
tail -f ~/.claude/logs/cortex.log         # iMessage/CORTEX log
tail -f ~/.cortana/logs/orchestrator.log  # Orchestrator log (if running)

# All logs directory
ls ~/.claude/logs/
```

---

## Tmux Scripting (Automation)

```bash
# Send a command to a running session without attaching
tmux send-keys -t <session> "ls -la" Enter

# Capture last N lines of a session's output
tmux capture-pane -t <session> -p | tail -20

# Check if a session exists
tmux has-session -t <name> 2>/dev/null && echo "exists" || echo "no"
```
