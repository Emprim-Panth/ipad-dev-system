# Memory Systems

Cortana has five interconnected memory systems. Together they ensure no context is lost between sessions, devices, or agents.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Memory Layer                          │
│                                                         │
│  Brain           Compass         LCM          Scout     │
│  (knowledge)     (project state) (conversation)(codebase)│
│  ┌───────┐       ┌───────┐      ┌───────┐    ┌───────┐ │
│  │ .md   │       │ .db   │      │ .db   │    │ FTS5  │ │
│  │ files │       │SQLite │      │SQLite │    │+Ollama│ │
│  └───┬───┘       └───┬───┘      └───┬───┘    └───┬───┘ │
│      │               │              │             │     │
│      ▼               ▼              ▼             ▼     │
│  corrections    goals/phases    conversation   code     │
│  patterns       blockers        history        search   │
│  decisions      tickets         recall         summaries│
│  anti-patterns  decisions       assembly       maps     │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────┐
│   Auto-Memory       │
│   (~/.claude/       │
│    projects/*/      │
│    memory/)         │
│   Session-level     │
│   learnings         │
└─────────────────────┘
```

---

## 1. Brain — Persistent Knowledge Store

**Location:** `~/.cortana/brain/`

The brain is the highest-value memory. It stores what Cortana has learned across all sessions — mistakes, patterns, decisions, anti-patterns. Every session reads from it on start, writes back on end.

### Structure

```
~/.cortana/brain/
├── DIRECTOR-BRIEF.md              ← Current state: what's happening NOW
├── BRAIN-PROTOCOL.md              ← Rules for reading/writing to brain
├── ENGINEERING-PROTOCOL.md        ← How work happens (5-phase protocol)
├── FORGE-LOCKDOWN.md              ← Path lockdown, structural integrity
├── brain-inject.sh                ← Generates context dump for external sessions
│
├── knowledge/
│   ├── corrections.md             ← HIGHEST VALUE — Evan corrected me here
│   ├── architecture-decisions.md  ← Settled architectural choices
│   ├── anti-patterns.md           ← What looks right but doesn't work
│   ├── patterns.md                ← Proven approaches to reuse
│   └── candidates.md              ← Observations pending promotion
│
├── identity/
│   ├── who-i-am.md               ← Evan's profile, the partnership
│   ├── operating-principles.md    ← How I make decisions
│   ├── behavioral-profile.md     ← Known biases and counteractions
│   ├── relationship-context.md   ← Working with Evan — observed patterns
│   └── slop-profile.md           ← Output defaults to counteract
│
└── projects/
    └── {project}.md               ← Per-project decisions & history
```

### Session Protocol

**On start:**
1. Read `DIRECTOR-BRIEF.md` — where we are NOW
2. Read `knowledge/corrections.md` — don't repeat mistakes
3. Read `identity/behavioral-profile.md` — how to make decisions

**On end:**
1. Update DIRECTOR-BRIEF "What Just Happened" section
2. Append corrections, decisions, patterns to appropriate files
3. Update behavioral-profile if significant decision was made

### Knowledge Types (ranked by value)

| Type | Value | Example |
|------|-------|---------|
| CORRECTION | Highest | "Use `ditto` not `cp -R` — cp strips extended attributes" |
| ANTI_PATTERN | High | "Don't use `try?` on database calls — silently discards errors" |
| DECISION | High | "XcodeGen for all iOS/macOS projects, single commit per task" |
| PATTERN | Medium | "GRDB query pattern: execute, fetch multiple/single rows" |
| CANDIDATE | Low | Observation pending promotion after repeated validation |

### CLI Commands

```bash
cortana-cli memory log "[CORRECTION] ..."   # Log a correction
cortana-cli memory log "[DECISION] ..."     # Log a decision
cortana-cli memory log "[PREFERENCE] ..."   # Log a preference
cortana-cli memory summary "What was done"  # End-of-session summary

cortana-cli knowledge search "query"        # FTS5 search
cortana-cli intelligence check "query"      # Full pre-implementation check
```

### For External Sessions (Codex, API)

```bash
# Generates a compact brain dump for injection
~/.cortana/brain/brain-inject.sh
```

---

## 2. Compass — Project State Tracking

**Database:** `~/.cortana/compass.db` (SQLite)

Tracks the live state of every project: current goal, phase, blockers, decisions, tickets.

### Commands

```bash
compass_status {project}     # Where this project is right now
compass_overview             # All projects, ranked by priority
compass_history {project}    # What happened: commits, sessions, decisions
compass_tickets {project}    # Open work, sorted by priority
compass_knowledge {project}  # Past mistakes and patterns for this project
compass_update {project}     # Set goal, phase, blockers, log decisions
```

### How It's Used

- **Session start:** `compass_status` to know where we are
- **During work:** `compass_update` when goals shift or blockers appear
- **Session end:** Auto-captured — files touched, phase, goal, summary
- **World Tree:** Reads Compass for project cards (goal, phase, git state, tickets)
- **Telegram bot:** `/compass` command exposes project state remotely

---

## 3. LCM — Lossless Context Manager

**Database:** `~/.cortana/lcm/conversations.db` (SQLite)
**MCP Server:** `cortana-lcm` (Node.js)

Maintains full conversation history across long sessions. When Claude's context window fills up and compacts, LCM still has the full history.

### Tools (via MCP)

```
lcm_ingest    — Call after every user/assistant turn (session_id, role, content)
lcm_assemble  — Get optimized context window before large tasks
lcm_grep      — Search compacted history for specific decisions/code/configs
lcm_describe  — Inspect a specific summary node
lcm_status    — Check token usage and compaction state
```

### Session ID Convention

Use project + date: `"WorldTree-2026-03-18"`. Consistent IDs = continuous history.

### Key Rule

Before answering questions about past decisions, commands, or configs — run `lcm_grep` first. Don't guess from memory.

---

## 4. Scout — Local Codebase Intelligence

**MCP Server:** `cortana-scout` (Bun + Ollama)
**Model:** `qwen3-coder` (18GB, ~88 tok/s on M4 Max)
**Index:** SQLite FTS5 per project

Scout reads files, understands code, and returns compressed summaries — all via the local Ollama model. Every file read through Scout saves ~80% of Claude's context window vs raw Read.

### Tools (via MCP)

```
scout_understand  project files[] question  — Read files + answer via local model
scout_map         project                   — Compressed project structure
scout_find        project intent            — Natural language code search
scout_summarize   project file              — Compressed file summary (cached)
scout_compress    content [context]         — Compress any tool output
scout_diff        project [args]            — Compressed git diff
scout_index       project                   — Index/refresh a project
scout_health                                — Check Scout + Ollama status
```

### When to Use Scout vs Read

| Need | Use |
|------|-----|
| Understand what a file does | `scout_summarize` |
| Understand how a system works | `scout_understand` with multiple files |
| Find where something is implemented | `scout_find` with natural language |
| Explore project structure | `scout_map` |
| Need exact line content for an Edit | `Read` (direct) |
| Reading a small config file (<50 lines) | `Read` (direct) |

---

## 5. Auto-Memory — Session-Level Learnings

**Location:** `~/.claude/projects/{project-path}/memory/`

Claude Code's built-in memory system. Stores learnings from individual sessions — user preferences, project facts, feedback, references.

### Types

| Type | Purpose |
|------|---------|
| `user` | Who Evan is, how to tailor responses |
| `feedback` | What to avoid, what to keep doing |
| `project` | Ongoing work, goals, decisions |
| `reference` | Pointers to external systems |

### How It Works

- Each memory is a markdown file with frontmatter (name, description, type)
- `MEMORY.md` is an index file loaded into every conversation
- Memories persist across conversations within the same project
- Auto-created when Cortana learns something worth remembering

---

## Semantic Search (QMD)

**Binary:** `~/.bun/bin/qmd`
**MCP Server:** `qmd` (configured in `~/.claude/mcp.json`)

Semantic search across all brain knowledge using Ollama embeddings.

```bash
qmd query "query" -c cortana-knowledge   # Search corrections, decisions, patterns
qmd query "query"                         # Search all collections
```

---

## How They Work Together

1. **New session starts** → Brain loads corrections + behavioral profile, Compass loads project state, LCM is ready to ingest
2. **Before writing code** → Search knowledge base for related mistakes/patterns, Scout explores relevant files
3. **During work** → LCM ingests every turn, Compass updates on goal shifts
4. **Session ends** → Brain gets new corrections/decisions, Compass captures summary, auto-memory stores session learnings
5. **Next session** → Everything is there. No context lost.
