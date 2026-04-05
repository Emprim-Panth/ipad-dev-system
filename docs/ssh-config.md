# SSH Configuration

## ~/.ssh/config

The SSH config file sets up named shortcuts so you type `ssh studio` instead of the full IP + username.

```
# ── Mac Studio ────────────────────────────────────────────────
# Replace 100.119.132.84 with your Studio's Tailscale IP

Host studio
    HostName 100.119.132.84
    User evanprimeau
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# ssh studio-tmux → SSH in + auto-attach to "main" tmux session
Host studio-tmux
    HostName 100.119.132.84
    User evanprimeau
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    RequestTTY yes
    RemoteCommand tmux new-session -A -s main
```

**Usage:**
```bash
ssh studio          # SSH to Studio, no auto-attach
ssh studio-tmux     # SSH to Studio + attach to main session immediately
```

---

## On the Studio: sshd_config

The default macOS SSH server configuration works. Just ensure SSH is enabled:

- **System Settings → General → Sharing → Remote Login** → On

Optional hardening (recommended):
```
# /etc/ssh/sshd_config — add these lines
PasswordAuthentication no      # Key-only auth (more secure)
ChallengeResponseAuthentication no
UsePAM yes
```

After editing:
```bash
sudo launchctl stop com.openssh.sshd
sudo launchctl start com.openssh.sshd
```

---

## SSH Key Management

### Generate a key per device
```bash
# iPad key
ssh-keygen -t ed25519 -C "ipad-$(date +%Y)" -f ~/.ssh/id_ipad_ed25519

# MacBook key (if separate)
ssh-keygen -t ed25519 -C "macbook-$(date +%Y)" -f ~/.ssh/id_macbook_ed25519
```

### Install keys on the Studio
```bash
# From the source device:
ssh-copy-id -i ~/.ssh/id_ed25519.pub evanprimeau@100.119.132.84

# Or manually on Studio:
cat >> ~/.ssh/authorized_keys << 'EOF'
ssh-ed25519 AAAA... ipad-2026
EOF
```

### Check what keys are authorized on Studio
```bash
cat ~/.ssh/authorized_keys
```

---

## Troubleshooting

**Permission denied (publickey):**
```bash
# Verbose mode shows which keys are tried
ssh -v studio

# Check authorized_keys permissions on Studio
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

**Connection refused:**
```bash
# Check SSH is running on Studio
sudo systemsetup -getremotelogin

# Check Tailscale is up on both devices
tailscale status
```

**Connection times out from iPad:**
```bash
# Verify Tailscale is active on iPad (check Tailscale app)
# Test the IP is reachable
ping 100.x.x.x
```
