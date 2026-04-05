# Cortana AI Stack

The AI layer that runs continuously on the Studio. This is what turns "persistent terminal sessions" into an actual always-on development operation.

---

## Overview

```
~/.cortana/
├── brain/                  ← Persistent knowledge (corrections, patterns, decisions)
│   ├── DIRECTOR-BRIEF.md   ← Current state: active projects, what's next
│   ├── knowledge/
│   │   ├── corrections.md  ← High-value: past mistakes, don't repeat
│   │   ├── patterns.md
│   │   ├── anti-patterns.md
│   │   └── architecture-decisions.md
│   └── identity/           ← Behavioral profile, working style
├── bin/                    ← All cortana-* commands
├── state/                  ← Runtime state (session registry, patrol state)
├── comms/                  ← Inbox/outbox between bot and orchestrator
│   ├── inbox/
│   ├── outbox/
│   └── leads/
└── secrets/                ← API keys, tokens (never committed to git)

~/.claude/
├── telegram/               ← Telegram bot + modules
│   ├── bot_v2.py           ← Main bot
│   ├── session_registry.py
│   ├── terminal_patrol.py
│   ├── compass_bridge.py
│   ├── project_actions.py
│   └── world_tree_bridge.py
└── settings.json           ← Claude Code permissions config
```

---

## Local LLM Fleet (Ollama)

All models run locally on the Studio's GPU. No API cost for routine work.

```bash
# Check what's loaded
ollama list

# Check which models are currently in memory
ollama ps
```

| Model | Size | Purpose |
|-------|------|---------|
| `qwen2.5:72b` | 47 GB | Heavy reasoning, Cortana always-on conversation |
| `qwen3-coder` | 18 GB | All coding tasks, Scout codebase queries |
| `nomic-embed-text` | 274 MB | Embeddings for semantic brain search |

**Ollama config** (`~/.ollama/config`):
```
OLLAMA_NUM_PARALLEL=4
OLLAMA_MAX_LOADED_MODELS=4
OLLAMA_KEEP_ALIVE=24h
OLLAMA_FLASH_ATTENTION=1
```

---

## Key Processes

### Telegram Bot

The bot is the remote control interface. It runs as a background process.

```bash
# Check if running
cat ~/.claude/telegram/bot.pid && ps -p $(cat ~/.claude/telegram/bot.pid) 2>/dev/null

# Start
cd ~/.claude/telegram && source ~/telegram-venv/bin/activate && python bot_v2.py &

# Stop
kill $(cat ~/.claude/telegram/bot.pid)

# Restart (clean)
cortana-ctl restart telegram
```

### Terminal Patrol

Runs every 2 minutes inside the bot. Monitors all tmux sessions for errors, stalled output, or Claude Code waiting for input. Sends Telegram alerts automatically.

No separate start needed — it's part of the bot.

### Always-On iMessage Cortana (CORTEX)

Polls `chat.db` every 10 seconds for new iMessages from Evan. Responds using `qwen2.5:72b`. Can take actions: run builds, send messages, check calendars, spawn Claude sessions.

```bash
# Check if running
pgrep -f cortana-cortex

# Start
cortana-cortex &

# Logs
tail -f ~/.claude/logs/cortex.log
```

### Brain Indexer

Keeps `~/.cortana/brain-index.db` in sync with brain markdown files. Enables semantic search (`/brain search <query>` in Telegram, or via World Tree).

Auto-runs via file watchers. Manual reindex:
```bash
cortana-cli brain reindex
```

---

## Compass (Project State DB)

```bash
# All projects + current state
compass_overview

# Specific project
compass_status archon-cad

# Update project state
compass_update archon-cad --goal "ship v1.0" --phase "QA"
```

The DB lives at `~/.cortana/compass.db`. World Tree reads it for project cards. The Telegram bot exposes it via `/compass`.

---

## Claude Code Sessions

Claude Code runs as `claude -p` (headless, pipe mode) for agent work. No warm session pools — each task gets its own process, runs to completion, exits.

```bash
# Run a task headlessly
claude -p "fix the NaN panic in archon/src/commands.rs" \
    --dangerously-skip-permissions \
    --output-format stream-json

# Check permissions config
cat ~/.claude/settings.json
```

**Key setting** — both files must have this or Claude will hang waiting for permission prompts:
```json
{
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": ["Bash(*)"]
  }
}
```

Files: `~/.claude/settings.json` AND `~/.claude/settings.local.json`
