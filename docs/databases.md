# Database Layout

All persistent state lives in SQLite databases. Three active, several archived.

---

## Active Databases

| Database | Location | Purpose |
|----------|----------|---------|
| **world-tree.db** | `~/.cortana/world-tree.db` | Main app DB — sessions, messages, knowledge, agents, tickets, vision, security |
| **compass.db** | `~/.cortana/compass.db` | Project state — goals, phases, blockers, decisions |
| **brain-index.db** | `~/.cortana/brain-index.db` | Semantic search — brain chunk embeddings via Ollama |

### Supporting Databases

| Database | Location | Purpose |
|----------|----------|---------|
| **conversations.db** | `~/.cortana/lcm/conversations.db` | LCM conversation history |
| **Scout indexes** | `~/.cortana/scout/` | FTS5 per-project code indexes |

---

## Archived / Inactive

These were consolidated during the database cleanup:

| Database | Status | What Happened |
|----------|--------|---------------|
| cortana.db | Archived (18MB) | Pre-rewrite, replaced by world-tree.db |
| nerve.db | Archived (3MB) | Gateway killed, data migrated |
| vision.db | Migrated | Content moved to world-tree.db |
| security.db | Migrated | Content moved to world-tree.db |

---

## Access Patterns

```bash
# Quick query
sqlite3 ~/.cortana/compass.db "SELECT * FROM projects;"

# Read-only exploration
sqlite3 -readonly ~/.cortana/world-tree.db ".tables"
```

**Rule:** compass.db is read/write. brain-index.db is read-only (rebuilt by indexer). world-tree.db is managed by the World Tree app.
