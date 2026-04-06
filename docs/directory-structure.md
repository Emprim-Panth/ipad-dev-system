# Directory Structure

Where everything lives on the Studio.

---

## Core Locations

```
~/
├── .cortana/                          ← All Cortana state and config
│   ├── brain/                         ← Persistent knowledge store
│   │   ├── DIRECTOR-BRIEF.md          ← Current operational state
│   │   ├── BRAIN-PROTOCOL.md          ← Read/write rules
│   │   ├── ENGINEERING-PROTOCOL.md    ← 5-phase work protocol
│   │   ├── FORGE-LOCKDOWN.md          ← Path lockdown spec
│   │   ├── brain-inject.sh            ← Context dump for external sessions
│   │   ├── knowledge/                 ← corrections, patterns, decisions, anti-patterns
│   │   ├── identity/                  ← behavioral profile, operating principles
│   │   ├── CIC/                       ← Orchestrator knowledge
│   │   └── projects/                  ← Per-project decisions
│   │
│   ├── bin/                           ← All cortana-* commands (~111 items)
│   │   ├── cortana-cli                ← Main CLI binary
│   │   ├── cortana-daemon             ← Daemon binary
│   │   ├── cortana-mcp → core link    ← MCP server entry
│   │   ├── cortana-scout → core link  ← Scout MCP entry
│   │   ├── cortana-hooks → core link  ← Hooks handler
│   │   ├── cortana-attach             ← SSH auto-attach wrapper
│   │   ├── cortana-session            ← Session launcher
│   │   ├── cortana-session-guardian   ← Session persistence monitor
│   │   └── ...
│   │
│   ├── agents/                        ← Agent fleet definitions
│   │   ├── orchestrator/              ← Cortana CIC config
│   │   ├── leads/                     ← Galactica + Pegasus leads
│   │   ├── workers/                   ← Galactica + Pegasus workers
│   │   └── learnings/                 ← Accumulated fleet knowledge
│   │
│   ├── config/                        ← Configuration files
│   │   ├── config.toml                ← Main Cortana config
│   │   ├── fleet.yaml                 ← Team/project assignments
│   │   ├── models.yaml                ← Model routing rules
│   │   ├── paths.yaml                 ← Canonical path registry
│   │   └── feature-flags.json         ← Feature toggles
│   │
│   ├── harness/                       ← Session dispatcher
│   │   ├── com.cortana.harness.plist  ← launchd config
│   │   ├── harness.sock               ← Unix socket
│   │   └── mcp-configs/               ← MCP configs for spawned agents
│   │
│   ├── daemon/                        ← Main daemon
│   │   ├── daemon.toml                ← Daemon config
│   │   └── com.cortana.daemon.plist   ← launchd config
│   │
│   ├── comms/                         ← Message passing
│   │   ├── inbox/                     ← Incoming messages
│   │   ├── outbox/                    ← Outgoing messages
│   │   └── leads/                     ← Lead-specific channels
│   │
│   ├── briefings/                     ← Daily morning briefs
│   ├── alerts/                        ← Active alert JSONs
│   ├── logs/                          ← All daemon/service logs
│   ├── templates/                     ← PRD-FRD template, etc.
│   ├── secrets/                       ← API keys (never committed)
│   │
│   ├── compass.db                     ← Project state DB
│   ├── world-tree.db                  ← Main app DB
│   ├── brain-index.db                 ← Semantic search DB
│   ├── lcm/conversations.db          ← LCM history
│   └── scout/                         ← Codebase indexes
│
├── .claude/                           ← Claude Code config
│   ├── CLAUDE.md                      ← Master instructions (identity, protocols, memory)
│   ├── CLAUDE-REFERENCE.md            ← CLI command reference
│   ├── settings.json                  ← Permissions, hooks, MCP servers
│   ├── settings.local.json            ← Local permission overrides
│   ├── mcp.json                       ← Additional MCP configs
│   ├── statusline.sh                  ← Context window monitor
│   ├── telegram/                      ← Telegram bot + modules
│   └── projects/                      ← Per-project auto-memory
│       └── {path}/memory/             ← Session-level learnings
│
├── .config/
│   ├── code-server/config.yaml        ← code-server config
│   └── ghostty/config                 ← Terminal config (auto-launches tmux)
│
├── .local/
│   ├── bin/                           ← User binaries (claude CLI, ts, etc.)
│   └── share/code-server/User/        ← VS Code settings for code-server
│
├── .tmux.conf                         ← tmux configuration
│
├── Library/LaunchAgents/              ← All launchd plists
│   ├── com.cortana.*.plist            ← Cortana services
│   └── homebrew.mxcl.code-server.plist
│
└── Development/                       ← All project repos
    ├── cortana-core/                  ← Core Cortana implementation
    ├── cortana-lcm/                   ← LCM implementation
    ├── cortana-vision/                ← Vision system
    └── {projects}/                    ← Revenue projects
```
