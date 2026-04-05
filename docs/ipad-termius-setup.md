# iPad Setup — Termius

Termius is the SSH client on the iPad. It's the window into the Studio.

---

## Install

**App Store:** Search "Termius — SSH & SFTP Client"

Free tier works. The paid tier ($9.99/mo or one-time) adds sync, snippets, and SFTP — worth it if you use this heavily.

---

## Add the Studio Host

1. Open Termius
2. Tap **Hosts** → **+** → **New Host**
3. Fill in:
   - **Label:** `Mac Studio`
   - **Hostname:** Your Studio's Tailscale IP (e.g. `100.119.132.84`)
   - **Port:** `22`
   - **Username:** Your Mac username (e.g. `evanprimeau`)
   - **Auth:** SSH Key (see below)

---

## SSH Key Setup

### Generate a key (if you don't have one)

```bash
# On your iPad in Termius: Settings → Keychain → + → Generate Key
# OR generate on Mac and import:
ssh-keygen -t ed25519 -C "ipad" -f ~/.ssh/id_ipad_ed25519
```

### Install the public key on the Studio

```bash
# Copy the public key to the Studio
ssh-copy-id -i ~/.ssh/id_ipad_ed25519.pub evanprimeau@100.119.132.84

# Or manually:
cat ~/.ssh/id_ipad_ed25519.pub >> ~/.ssh/authorized_keys
```

### In Termius
- Settings → Keychain → Import Key → paste or upload your private key
- On your Host entry, set Auth to use this key

---

## Keyboard Setup for tmux

tmux needs Ctrl+A as a prefix. On iPad, configure the extended keyboard bar:

1. Termius → Settings → Keyboard
2. Enable **Extended Key Bar**
3. Make sure these keys are in the bar:
   - `Ctrl`
   - `Esc`
   - `Tab`
   - `|` (pipe — used for vertical splits)
   - Arrow keys

**Tip:** Long-press `Ctrl` then tap `A` to send the tmux prefix on the touch keyboard.

---

## Useful Termius Features

### Snippets
Create saved snippets for your most common commands:

- Name: `Session List` | Command: `ts`
- Name: `Attach Main` | Command: `ts main`
- Name: `Health` | Command: `cortana-health`
- Name: `Ollama Status` | Command: `ollama list`

Access snippets from the keyboard toolbar — one tap to run.

### Split View
Termius supports split sessions on iPad (two terminal panes side by side). Useful for watching logs in one pane while working in another.

### Local Port Forwarding
If you want to access Studio web services (World Tree API, Ollama) from the iPad browser:

In Termius → Host settings → Port Forwarding → Add:
- Local: `11434` → Remote: `localhost:11434` (Ollama)
- Local: `8080` → Remote: `localhost:8080` (any local web service)

---

## The Daily Workflow

1. **Open Termius** — tap Mac Studio host
2. Connection opens in ~1 second (Tailscale handles routing)
3. You land in the `main` tmux session (if auto-attach is configured)
4. Type `ts` to see all sessions
5. `Ctrl+A S` to pick a project session
6. Work. Detach when done (`Ctrl+A D`).
7. Close Termius. Session stays running on Studio.

That's it. No reconnection ritual. No "where was I."
