# Tailscale Networking

Tailscale creates a private encrypted mesh network between all your devices. It's what makes "access from anywhere" possible without exposing anything to the public internet.

---

## What It Does

- Every device gets a stable private IP (100.x.x.x) and a DNS name
- Traffic is encrypted end-to-end via WireGuard
- No port forwarding, no firewall rules, no dynamic DNS
- Works through NAT, firewalls, cellular — just works
- Only YOUR devices can see each other

---

## Install

### Mac Studio
```bash
# Download from https://tailscale.com/download/mac
# Or via App Store
# Sign in with your account
```

The CLI binary lives at `/Applications/Tailscale.app/Contents/MacOS/Tailscale`. Add an alias:
```bash
alias tailscale="/Applications/Tailscale.app/Contents/MacOS/Tailscale"
```

### iPad
Install Tailscale from the App Store. Sign in with the same account.

### MacBook / Other Macs
Same as Studio — download the app, sign in with the same account.

---

## Verify Your Network

```bash
tailscale status
```

You should see all your devices:
```
100.x.x.x    your-mac-studio     you@   macOS   -
100.x.x.x    your-macbook-pro    you@   macOS   idle
100.x.x.x    your-ipad           you@   iOS     idle
```

---

## MagicDNS

Tailscale assigns each device a DNS name:
```
your-mac-studio.tailXXXXX.ts.net
```

Get your Studio's DNS name:
```bash
tailscale status --json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['Self']['DNSName'])"
```

This is your bookmarkable URL base. Combine with service ports:
- **code-server:** `http://your-studio.tailXXXXX.ts.net:8080`
- **SSH:** `ssh your-studio.tailXXXXX.ts.net`

---

## Tailscale SSH (Optional)

Tailscale has built-in SSH that skips traditional SSH key management:

1. Enable in the Tailscale admin console → Access Controls
2. Add SSH rules for your devices
3. Connect with just `ssh your-mac-studio` — Tailscale handles auth

This is optional — standard SSH over Tailscale works fine too.

---

## What's Exposed

With this setup, your Studio has these services reachable over Tailscale:

| Service | Port | URL |
|---------|------|-----|
| code-server | 8080 | `http://studio.tailnet.ts.net:8080` |
| SSH | 22 | `ssh studio.tailnet.ts.net` |
| Ollama | 11434 | `http://studio.tailnet.ts.net:11434` (if needed) |

None of these are exposed to the public internet. Only devices on your Tailscale account can reach them.

---

## Troubleshooting

**Can't reach Studio from iPad:**
1. Open Tailscale app on iPad — is it connected?
2. On Studio, run `tailscale status` — does the iPad show as online?
3. Try pinging: `ping 100.x.x.x` (Studio's Tailscale IP)

**DNS name not resolving:**
Use the IP directly (`100.x.x.x:8080`). MagicDNS occasionally needs a moment after reconnecting.

**Tailscale app not staying connected on iPad:**
iOS aggressively kills background VPN connections. Open the Tailscale app periodically, or enable "Always On VPN" in iOS Settings → VPN if available.
