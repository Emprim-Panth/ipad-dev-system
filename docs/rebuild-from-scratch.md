# Rebuild From Scratch

Follow this in order. Estimated time: 2–3 hours for a full rebuild, 30 minutes for just the SSH/tmux layer.

---

## Phase 1 — Mac Studio Foundation (30 min)

### 1.1 Enable SSH

System Settings → General → Sharing → Remote Login → **On**

### 1.2 Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 1.3 Install core tools

```bash
brew install tmux tailscale git curl wget
```

### 1.4 Tailscale

```bash
brew install tailscale
sudo tailscale up
```

Log in at the Tailscale URL it gives you. Note your Studio's IP:
```bash
tailscale ip -4
# Save this: 100.x.x.x
```

### 1.5 SSH keys

```bash
ssh-keygen -t ed25519 -C "studio-$(date +%Y)" -f ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

---

## Phase 2 — tmux Setup (15 min)

### 2.1 Config file

```bash
# Copy from this repo
cp config-templates/.tmux.conf ~/.tmux.conf

# Apply without restarting
tmux source-file ~/.tmux.conf
```

### 2.2 Session manager (`ts`)

```bash
mkdir -p ~/.local/bin
cp scripts/ts ~/.local/bin/ts
chmod +x ~/.local/bin/ts
```

Add to `~/.zshrc`:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

### 2.3 Auto-attach on SSH

Add to `~/.zshrc`:
```bash
if [[ -n "$SSH_CONNECTION" && -z "$TMUX" ]]; then
    tmux new-session -A -s main
fi
```

### 2.4 Create initial sessions

```bash
ts new main
ts new agents
# Add more as you need them per project
```

---

## Phase 3 — MacBook / iPad SSH (15 min)

### MacBook

```bash
# ~/.ssh/config — replace IP with your Studio's Tailscale IP
cat >> ~/.ssh/config << 'EOF'

Host studio
    HostName 100.x.x.x
    User evanprimeau
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

Host studio-tmux
    HostName 100.x.x.x
    User evanprimeau
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    RequestTTY yes
    RemoteCommand tmux new-session -A -s main
EOF
```

Install your key on the Studio:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub evanprimeau@100.x.x.x
```

Test:
```bash
ssh studio-tmux
# Should drop you into a tmux session on the Studio
```

### iPad

1. Install Termius from App Store
2. Settings → Keychain → Import or generate a key
3. Copy public key to Studio's `~/.ssh/authorized_keys`
4. Add Host: IP = Studio's Tailscale IP, Username = your Mac username, Auth = key
5. Connect — you should land in tmux main session

---

## Phase 4 — Ollama + Local Models (30–60 min download time)

```bash
# Install Ollama
brew install ollama

# Configure for multi-model loading
mkdir -p ~/.ollama
cat > ~/.ollama/config << 'EOF'
OLLAMA_NUM_PARALLEL=4
OLLAMA_MAX_LOADED_MODELS=4
OLLAMA_KEEP_ALIVE=24h
OLLAMA_FLASH_ATTENTION=1
EOF

# Start Ollama as a service
brew services start ollama

# Download models (these are large — plan for download time)
ollama pull qwen2.5:72b          # ~47 GB — main reasoning model
ollama pull qwen3-coder          # ~18 GB — coding tasks
ollama pull nomic-embed-text     # ~274 MB — embeddings

# Verify
ollama list
```

---

## Phase 5 — Claude Code CLI (10 min)

```bash
# Install
npm install -g @anthropic-ai/claude-code

# Verify
claude --version

# Configure permissions (critical — without this, agents hang on prompts)
mkdir -p ~/.claude

cat > ~/.claude/settings.json << 'EOF'
{
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": ["Bash(*)"]
  }
}
EOF

cp ~/.claude/settings.json ~/.claude/settings.local.json
```

---

## Phase 6 — Telegram Bot (30 min)

### 6.1 Create bot

1. Telegram → @BotFather → `/newbot`
2. Copy the token

### 6.2 Get your user ID

1. Telegram → @userinfobot

### 6.3 Install

```bash
mkdir -p ~/.claude/telegram ~/.cortana/secrets ~/.cortana/state ~/.cortana/comms/{inbox,outbox,leads}

# Store token
echo "YOUR_BOT_TOKEN" > ~/.cortana/secrets/telegram-bot-token
chmod 600 ~/.cortana/secrets/telegram-bot-token

# Python venv
python3 -m venv ~/telegram-venv
source ~/telegram-venv/bin/activate
pip install python-telegram-bot requests

# Copy bot files (from wherever you have them)
# See the source at: ~/.claude/telegram/ on the Studio
```

### 6.4 Config

```bash
cat > ~/.claude/telegram/config.json << 'EOF'
{
  "allowed_users": [YOUR_TELEGRAM_USER_ID]
}
EOF
```

> **Important:** Do not store API keys in this file. The bot reads the token from `~/.cortana/secrets/telegram-bot-token`.

### 6.5 Start

```bash
cd ~/.claude/telegram
source ~/telegram-venv/bin/activate
python bot_v2.py &
echo $! > bot.pid
```

Test: send `/health` to your bot on Telegram.

### 6.6 Auto-start on boot

```bash
cp config-templates/com.cortana.telegram-bot.plist ~/Library/LaunchAgents/
# Edit the plist to set your actual paths
launchctl load ~/Library/LaunchAgents/com.cortana.telegram-bot.plist
```

---

## Verify Everything Works

```bash
# 1. tmux sessions persist
ts                          # Should show your sessions
Ctrl+A D                    # Detach
ssh studio                  # SSH back in
ts                          # Sessions still there ✓

# 2. Ollama responding
ollama run qwen3-coder "say hello"   # Should respond ✓

# 3. Claude Code running
claude -p "what's 2+2" --dangerously-skip-permissions   # Should respond ✓

# 4. Telegram bot
# Send /health to your bot → should get system status ✓

# 5. iPad
# Open Termius → tap Studio → should connect and show tmux ✓
```

---

## What's NOT in This Repo

These are personal/secret and must be set up manually:
- `~/.cortana/secrets/` — API keys, tokens
- `~/.cortana/brain/` — Personal knowledge base
- `~/.cortana/compass.db` — Project state
- SSH private keys
- Tailscale account (tied to your email)

The World Tree macOS app (dashboard) is a separate project — see the `WorldTree` repo.
