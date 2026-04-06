# Hooks System

Claude Code hooks are shell commands that execute automatically in response to lifecycle events. Cortana uses them for context injection, monitoring, and behavioral enforcement.

---

## How It Works

All hooks call the same entry point:
```bash
bun /Users/you/.cortana/bin/cortana-hooks
```

The hook reads `HOOK_EVENT_NAME` from the environment and receives a JSON payload on stdin.

---

## Hook Events

| Event | When It Fires | What Cortana Does |
|-------|---------------|-------------------|
| `SessionStart` | Claude Code session begins | Inject brain context, load corrections, check briefings |
| `UserPromptSubmit` | User sends a message | Log to LCM, check for push-back triggers |
| `PreToolUse` | Before any tool executes | Safety checks, can BLOCK tool execution (exit code 2) |
| `Stop` | Claude finishes a response | Log to LCM, check for corrections to capture |
| `SessionEnd` | Session closes | Write back to brain, update Compass, log summary |
| `PreCompact` | Before context compaction | Preserve critical context, ensure LCM has everything |

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Allow / normal completion |
| 2 | **Block** (PreToolUse only — prevents the tool from executing) |

**Safety rule:** Non-PreToolUse events always exit 0. A crash in a hook must never crash Claude Code.

---

## Configuration

Hooks are defined in `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ],
    "UserPromptSubmit": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ],
    "PreToolUse": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ],
    "Stop": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ],
    "SessionEnd": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ],
    "PreCompact": [
      { "type": "command", "command": "bun /path/to/cortana-hooks" }
    ]
  }
}
```

---

## Implementation

**Entry point:** `~/.cortana/bin/cortana-hooks` (symlink to `cortana-core/bin/cortana-hooks.ts`)

The implementation reads the event name and JSON payload, then dispatches to the appropriate handler. Key exports:
- `handleHookEvent` — main dispatcher
- `evaluatePreToolUse` — decides whether to allow/block a tool call
