# MCP Servers

MCP (Model Context Protocol) servers extend Claude Code with custom tools. Each server provides specialized capabilities.

---

## Active MCP Servers

| Server | Type | Purpose |
|--------|------|---------|
| **cortana** | stdio (Bun) | Core Cortana MCP — brain access, compass, memory |
| **scout** | stdio (Bun) | Local codebase intelligence via Ollama |
| **cortana-lcm** | stdio (Node) | Lossless Context Manager for conversation history |
| **cortana-vision** | stdio (Bun) | Computer vision — screenshot analysis, UI inspection |
| **forge-workflow** | stdio (Bun) | Forge workflow automation |
| **peekaboo** | binary | System inspection and observation |
| **xcodebuildmcp** | npx | Xcode build integration |
| **context7** | npx | Upstash Context7 — library documentation |
| **probe** | npx | Probe Labs code analysis |
| **gitnexus** | binary | Git workflow tools |
| **qmd** | binary | Semantic search over brain knowledge |

---

## Configuration

### Primary config: `~/.claude/settings.json`

All MCP servers are defined in the `mcpServers` section:

```json
{
  "mcpServers": {
    "cortana": {
      "command": "bun",
      "args": ["/Users/you/.cortana/bin/cortana-mcp"]
    },
    "scout": {
      "command": "bun",
      "args": ["/Users/you/.cortana/bin/cortana-scout"],
      "env": { "SCOUT_MODEL": "qwen3-coder" }
    },
    "cortana-lcm": {
      "command": "node",
      "args": ["/Users/you/Development/cortana-lcm/dist/index.js"],
      "env": { "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}" }
    }
  }
}
```

### Secondary config: `~/.claude/mcp.json`

Additional MCP servers (like `qmd`) that use HTTP transport:

```json
{
  "mcpServers": {
    "qmd": {
      "command": "/Users/you/.bun/bin/qmd",
      "args": ["mcp"]
    }
  }
}
```

---

## Server Details

### cortana (Core MCP)
Entry point: `~/.cortana/bin/cortana-mcp` → `cortana-core/bin/cortana-mcp.ts`

Tools provided:
- `compass_status`, `compass_overview`, `compass_update` — project state
- `memory_log`, `memory_search` — knowledge capture and retrieval
- `brain_read`, `brain_write` — direct brain file access
- Agent dispatch tools

### scout (Codebase Intelligence)
Entry point: `~/.cortana/bin/cortana-scout` → `cortana-core/bin/cortana-scout.ts`
Model: `qwen3-coder` via Ollama

Tools: `scout_understand`, `scout_map`, `scout_find`, `scout_summarize`, `scout_compress`, `scout_diff`, `scout_index`, `scout_health`

See [Memory Systems](memory-systems.md) for full Scout documentation.

### cortana-lcm (Context Manager)
Entry point: `Development/cortana-lcm/dist/index.js`

Tools: `lcm_ingest`, `lcm_assemble`, `lcm_grep`, `lcm_describe`, `lcm_status`

See [Memory Systems](memory-systems.md) for full LCM documentation.

### cortana-vision (Computer Vision)
Entry point: `Development/cortana-vision/mcp/cortana-vision-mcp.ts`

Used for screenshot analysis, UI verification, visual QA of macOS/iOS apps.

---

## Plugins

Claude Code also uses these plugins:

| Plugin | Purpose |
|--------|---------|
| hermes@hermes | Communication layer |
| rust-analyzer-lsp | Rust language server |
| swift-lsp | Swift language server |

---

## Adding a New MCP Server

1. Add the server definition to `~/.claude/settings.json` under `mcpServers`
2. If it needs permissions, add to `settings.local.json` under `allow`
3. Restart Claude Code — MCP servers are loaded on session start

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/server.js"],
      "env": { "API_KEY": "..." }
    }
  }
}
```
