# iPad Dev System

> Control your entire development operation from an iPad — persistent tmux sessions, AI agents, local LLMs, all running on a Mac Studio 24/7. One tap to connect, zero context lost.

This repo documents the full system: how it works, how to rebuild it from scratch, and every command needed to operate it.

---

## The System at a Glance

```
iPad (Termius)
    │
    ▼ SSH via Tailscale
Mac Studio (runs 24/7)
    ├── tmux (persistent sessions — one per project)
    ├── Cortana AI stack (orchestrator, brain, agents)
    ├── Ollama (local LLMs: 72B reasoning, 32B coding, embeddings)
    ├── Telegram bot (remote control from anywhere)
    └── World Tree (macOS dashboard app)
```

The Studio never sleeps. Sessions never die. You pick up exactly where you left off, from any device, on any network.

---

## Quick Reference

| Task | Command |
|------|---------|
| List all sessions | `ts` |
| Attach to a session | `ts <name>` |
| Create a session | `ts new <name>` |
| Kill a session | `ts kill <name>` |
| Session picker (inside tmux) | `Ctrl+A` then `S` |
| Restart Telegram bot | `cortana-ctl restart telegram` |
| Restart Claude/tmux session | See [Restart Guide](docs/restart-guide.md) |
| SSH from MacBook | `ssh studio` |
| SSH + attach main session | `ssh studio-tmux` |

---

## Table of Contents

1. [Hardware & Network](docs/hardware-network.md)
2. [tmux Setup](docs/tmux-setup.md)
3. [iPad Setup (Termius)](docs/ipad-termius-setup.md)
4. [SSH Configuration](docs/ssh-config.md)
5. [Cortana AI Stack](docs/cortana-stack.md)
6. [Telegram Bot](docs/telegram-bot.md)
7. [Restart Guide](docs/restart-guide.md)
8. [Shell Commands Reference](docs/shell-commands.md)
9. [Rebuild From Scratch](docs/rebuild-from-scratch.md)

---

## Stack

| Component | Purpose | Cost |
|-----------|---------|------|
| Mac Studio M4 Max 128GB | Always-on compute, storage, AI models | Hardware |
| Tailscale | Private mesh network (iPad ↔ Studio) | Free tier |
| tmux | Persistent terminal sessions | Free |
| Termius | iPad SSH client | ~$10/mo or one-time |
| Ollama | Local LLM inference engine | Free |
| Claude Code CLI | AI coding agent | API usage |
| Telegram + Python bot | Remote command interface | Free |
| World Tree | macOS dashboard for project state | Internal |
