# Dev Infrastructure

> Control your entire development operation from anywhere — browser, iPad, laptop, phone. Persistent sessions, AI agents, local LLMs, all running on a Mac Studio 24/7. One bookmark to connect, zero context lost.

This repo documents the full system: how it works, how to rebuild it from scratch, and every command needed to operate it. Built by [Forge&Code](https://forgeandcode.com).

---

## The System at a Glance

```
Any Device (Browser)
    │
    ▼ Tailscale (encrypted mesh)
Mac Studio (runs 24/7)
    ├── code-server (full VS Code IDE in your browser)
    ├── tmux (persistent sessions — one per project)
    ├── Cortana AI stack (orchestrator, brain, agents, memory)
    ├── Ollama (local LLMs: 72B reasoning, 18B coding, embeddings)
    ├── Telegram bot (remote control from anywhere)
    ├── Scout (local codebase intelligence via Ollama)
    ├── LCM (lossless context manager across sessions)
    ├── Compass (project state tracking)
    └── World Tree (macOS dashboard app)
```

The Studio never sleeps. Sessions never die. You pick up exactly where you left off, from any device, on any network.

---

## Quick Reference

| Task | Command / URL |
|------|---------------|
| **Open VS Code in browser** | `http://your-studio.tailnet.ts.net:8080` |
| List all tmux sessions | `ts` |
| Attach to a session | `ts <name>` |
| Create a session | `ts new <name>` |
| Kill a session | `ts kill <name>` |
| Session picker (inside tmux) | `Ctrl+A` then `s` |
| Create new named session | `Ctrl+A` then `S` |
| Restart Telegram bot | `cortana-ctl restart telegram` |
| SSH from MacBook | `ssh studio` |
| SSH + attach main session | `ssh studio-tmux` |

---

## Table of Contents

### Getting Connected
1. [Code-Server Setup](docs/code-server.md) — VS Code in your browser from any device
2. [Tailscale Networking](docs/tailscale.md) — Secure access from anywhere
3. [tmux Setup](docs/tmux-setup.md) — Persistent terminal sessions
4. [iPad Setup (Termius)](docs/ipad-termius-setup.md) — Native SSH from iPad
5. [SSH Configuration](docs/ssh-config.md) — Key-based auth, aliases

### AI & Intelligence Stack
6. [Cortana AI Stack](docs/cortana-stack.md) — Orchestrator, brain, agents
7. [Memory Systems](docs/memory-systems.md) — Brain, Compass, LCM, Scout, auto-memory
8. [Agent Fleet](docs/agent-fleet.md) — Galactica & Pegasus team structure
9. [MCP Servers](docs/mcp-servers.md) — All MCP integrations
10. [Ollama Local LLMs](docs/ollama-setup.md) — Local model fleet

### Operations
11. [Hooks System](docs/hooks.md) — Claude Code lifecycle hooks
12. [Daemon Infrastructure](docs/daemons.md) — All background services
13. [Telegram Bot](docs/telegram-bot.md) — Remote control interface
14. [Shell Commands Reference](docs/shell-commands.md)
15. [Restart Guide](docs/restart-guide.md)
16. [Rebuild From Scratch](docs/rebuild-from-scratch.md)

### Architecture
17. [Hardware & Network](docs/hardware-network.md)
18. [Database Layout](docs/databases.md) — All databases and their purpose
19. [Directory Structure](docs/directory-structure.md) — Where everything lives

---

## Stack

| Component | Purpose | Cost |
|-----------|---------|------|
| Mac Studio M4 Max 128GB | Always-on compute, storage, AI models | Hardware |
| code-server | VS Code IDE in any browser | Free |
| Tailscale | Private encrypted mesh network | Free tier |
| tmux | Persistent terminal sessions | Free |
| Termius | iPad SSH client (optional — code-server replaces this) | ~$10/mo |
| Ollama | Local LLM inference engine | Free |
| Claude Code CLI | AI coding agent | API usage |
| Telegram + Python bot | Remote command interface | Free |
| World Tree | macOS dashboard for project state | Internal |
| Scout | Local codebase intelligence (Ollama-powered) | Free |
| LCM | Lossless Context Manager for session continuity | Free |
| Compass | Project state tracking DB | Free |
