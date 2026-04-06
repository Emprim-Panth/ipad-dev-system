# Agent Fleet

Cortana operates a multi-agent system organized into two teams, each with specialized roles. The structure follows a Battlestar Galactica naming convention.

---

## Hierarchy

```
Evan (CEO)
  └── Cortana (Admiral / CIC — orchestrator)
        ├── Galactica (software development)
        │   ├── Apollo (Mission Lead)
        │   ├── Chief Tyrol (Engineering Lead)
        │   ├── Starbuck (Architecture Lead)
        │   ├── Tigh (QA Lead)
        │   ├── Gaeta (Operations Lead)
        │   └── Workers: Helo, Hot Dog, Cally
        │
        └── Pegasus (game development)
            ├── Athena (Mission Lead)
            ├── Baltar (Technical Lead)
            ├── Doc Cottle (QA Lead)
            └── Workers: Skulls, Racetrack, Seelix, Kat
```

---

## Model Tiers

| Tier | Model | Used For |
|------|-------|----------|
| Orchestrator | claude-opus | Top-level routing, strategic decisions |
| Leads | claude-sonnet | Task planning, code review, architecture |
| Workers (default) | claude-sonnet | Standard implementation |
| Workers (simple) | claude-haiku | Trivial tasks, formatting, simple lookups |
| Local (conversation) | qwen2.5:72b | Always-on Cortana, Telegram responses |
| Local (code) | qwen3-coder | Scout queries, codebase intelligence |
| Local (embeddings) | nomic-embed-text | Semantic search, brain indexing |

**Rule:** Sonnet is default. Opus is for novel algorithms, intricate multi-system reasoning, or very long context work where Sonnet has already failed.

---

## How Agents Are Dispatched

1. **Task comes in** (from Evan, Telegram, or another agent)
2. **Cortana (CIC)** evaluates: which team? which lead? what model tier?
3. **Lead** gets the task brief, breaks it into subtasks
4. **Workers** execute subtasks with non-overlapping file assignments
5. **Lead** reviews, integrates, reports back to CIC
6. **Tigh/Doc Cottle** (QA) runs verification before shipping

### Task Brief Format

Every task gets a standardized brief:
```
## Task: {title}
**Scope:** {what files/modules}
**DoD:** {definition of done}
**Risks:** {what could go wrong}
**Verification:** {how to confirm it works}
```

---

## Agent Configuration

**Location:** `~/.cortana/agents/`

```
agents/
├── orchestrator/
│   ├── instructions.md    ← Cortana's role & behavior
│   └── config.yaml        ← Model, permissions
├── leads/
│   ├── galactica/         ← Apollo, Chief Tyrol, Starbuck, Tigh, Gaeta
│   └── pegasus/           ← Athena, Baltar, Doc Cottle, Dee, Six
├── workers/
│   ├── galactica/         ← Helo, Hot Dog, Cally
│   └── pegasus/           ← Skulls, Racetrack, Seelix, Kat
└── learnings/
    ├── galactica.md       ← Software team accumulated knowledge
    ├── pegasus.md         ← Game dev team knowledge
    ├── cic.md             ← Orchestrator knowledge
    ├── engineering.md     ← Cross-team engineering insights
    ├── architecture.md    ← Architecture patterns
    ├── qa.md              ← QA knowledge
    └── shared.md          ← Cross-fleet standards
```

---

## Fleet Configuration

**File:** `~/.cortana/config/fleet.yaml`

Defines which projects belong to which team:

| Team | Projects |
|------|----------|
| Galactica | BIM Manager, Archon-CAD, DocForge, ForgeSchedule, World Tree, BookBuddy |
| Pegasus | ForgeMaster (game) |

---

## Key Rules

1. **Non-overlapping files** — two agents never edit the same file simultaneously
2. **One commit per completed task** — not per file, not per session
3. **QA before ship** — Tigh or Doc Cottle must verify before any task closes
4. **Learnings write back** — new patterns, mistakes, or decisions get appended to the relevant learnings file
5. **Model tier enforcement** — don't burn Opus tokens on Haiku-level work
