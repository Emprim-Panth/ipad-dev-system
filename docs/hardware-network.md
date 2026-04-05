# Hardware & Network

## The Machine

**Mac Studio M4 Max**
- RAM: 128 GB unified memory
- GPU: 40 cores (Ollama uses these for model inference)
- Storage: 2 TB (1.7 TB free as of setup)
- Always on — never sleeps, display off

This is the compute brain. Everything runs here. Every device just connects to it.

---

## Network: Tailscale

Tailscale creates a private WireGuard mesh between all your devices. Even on different networks (home wifi, cell data, airport wifi), devices connect to each other directly via Tailscale IPs.

### Install

```bash
# macOS (Studio + MacBook)
brew install tailscale

# iOS/iPadOS — App Store: "Tailscale"
```

### Setup

```bash
# On Mac Studio
sudo tailscale up

# On MacBook
sudo tailscale up

# On iPad — follow in-app flow
```

### Get your Studio's Tailscale IP

```bash
tailscale ip -4
# Example output: 100.119.132.84
```

Save this IP — it goes in your SSH config and Termius.

### Verify connectivity

```bash
# From MacBook or iPad (via SSH/Termius):
ping 100.x.x.x  # Your Studio's Tailscale IP
```

### Key facts

- Tailscale IP stays constant even if your home IP changes
- Works through NAT, CGNAT, firewalls — no port forwarding needed
- The Studio's Tailscale IP is what Termius and `~/.ssh/config` use
- Free tier: up to 3 users, 100 devices — more than enough

---

## Why This Architecture

Without Tailscale, you'd need:
- Port forwarding on your router (fragile, changes with ISP)
- Dynamic DNS (another moving part)
- VPN setup (more complexity)

Tailscale handles all of that. Install, log in, done.
