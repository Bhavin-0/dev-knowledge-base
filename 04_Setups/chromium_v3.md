---
type: setup
created: 2026-05-05
updated: 2026-05-18
version: v3
status: active
tags: [chromium, hardening, ram-optimization, dual-mode]
topic: Chromium Dual-Mode Hardened Setup
reviewed:
source:
environment:
---

# Chromium — Dual-Mode Hardened Setup

## Objective

Two separate Chromium modes from a single snap install:

| Mode | Alias | Wrapper | Purpose |
|---|---|---|---|
| Normal | `c` | `~/.local/bin/chromium` | Daily usage, full renderer budget |
| Lite | `cl` | `~/.local/bin/chromium-lite` | LLM-heavy sessions, capped RAM |

Both modes:
- Use hardened privacy flags
- Launch detached from terminal
- Bypass snap's default behavior via wrapper + PATH override

---

# Environment

- OS: Linux
- Shell: Fish
- Chromium Install: Snap
- RAM Target: ~8GB with local LLMs running alongside
- Architecture: Wrapper Script + PATH Override

---

# Why Wrappers Instead of .desktop Files

Snap Chromium injects its own behavior and `.desktop` launchers behave
inconsistently. Terminal launches ignore custom flags unless intercepted.

Solution:

```
alias / terminal command
        ↓
~/.local/bin/chromium  OR  ~/.local/bin/chromium-lite
        ↓
/usr/bin/snap run chromium
        ↓
Chromium with hardened flags applied
```

This guarantees consistent flags, detached launches, and
terminal-independent execution regardless of how Chromium is invoked.

---

# Architecture Overview

```
~/.local/bin/
├── chromium           ← Normal mode wrapper
└── chromium-lite      ← Lite/RAM mode wrapper

~/.config/fish/config.fish
├── alias c            ← invokes chromium (normal)
├── alias cl           ← invokes chromium-lite
└── alias clog         ← invokes chromium-lite with debug log

~/.local/share/chromium-lite/
└── chromium-YYYYMMDD_HHMMSS.log   ← debug logs (clog only)
```

---

# When to Use Which Mode

## Use `c` (Normal) when:
- RAM is available (LLM not running or idle)
- Opening multiple heavy tabs simultaneously
- Running multiple AI services at once (ChatGPT + Claude + Gemini + more)
- DevTools-heavy work

## Use `cl` (Lite) when:
- LLM is actively consuming RAM
- RAM pressure is visible (`htop` showing high usage)
- Browsing lighter pages alongside inference
- You want to limit Chromium's footprint deliberately

> **AI tabs in Lite mode are supported.** The renderer limit is set to 6,
> which comfortably handles ChatGPT, Claude, Gemini, and Perplexity
> simultaneously without slowdown.

---

# Wrapper 1: Normal Mode

**Path:** `~/.local/bin/chromium`

```bash
vim ~/.local/bin/chromium
```

Contents:

```bash
#!/bin/bash

exec /usr/bin/snap run chromium \
--process-per-site \
--disable-background-networking \
--disable-sync \
--disable-translate \
--disable-features=MediaRouter,OptimizationHints \
--no-default-browser-check \
--no-first-run \
"$@"
```

Make executable:

```bash
chmod +x ~/.local/bin/chromium
```

> **Note:** `exec` replaces the bash wrapper process with snap's chromium,
> giving a cleaner process tree. Without it, an extra bash process lingers.

---

# Wrapper 2: Lite Mode

**Path:** `~/.local/bin/chromium-lite`

```bash
vim ~/.local/bin/chromium-lite
```

Contents:

```bash
#!/bin/bash

exec /usr/bin/snap run chromium \
--process-per-site \
--renderer-process-limit=6 \
--disable-background-networking \
--disable-sync \
--disable-translate \
--disable-features=MediaRouter,OptimizationHints \
--no-default-browser-check \
--no-first-run \
"$@"
```

Make executable:

```bash
chmod +x ~/.local/bin/chromium-lite
```

---

# Flag Differences Between Modes

| Flag | Normal (`c`) | Lite (`cl`) | Reason |
|---|---|---|---|
| `--process-per-site` | ✓ | ✓ | Groups same-site tabs, saves RAM |
| `--renderer-process-limit=6` | ✗ | ✓ | Caps renderer count in lite mode |
| `--disable-background-networking` | ✓ | ✓ | Kills Google background calls |
| `--disable-sync` | ✓ | ✓ | No sync engine overhead |
| `--disable-translate` | ✓ | ✓ | No translation service |
| `--disable-features=MediaRouter,OptimizationHints` | ✓ | ✓ | No Chromecast, no prediction |
| `--no-default-browser-check` | ✓ | ✓ | Cleaner startup |
| `--no-first-run` | ✓ | ✓ | Skip onboarding |

> **Why renderer limit is 6 and not 4:**
> ChatGPT, Claude, Gemini, and Perplexity are each different origins.
> With `--process-per-site`, each gets its own renderer process.
> At limit=4, the 5th AI tab starts competing for shared renderers → slowdown.
> At limit=6, all common AI services run with dedicated renderers
> while still capping total RAM usage below uncapped default.

---

# PATH Priority (Critical)

Ensure this line exists **permanently** inside `~/.config/fish/config.fish`:

```fish
set -gx PATH ~/.local/bin $PATH
```

> Running `set -gx` in the terminal only lasts for that session.
> It **must** be in `config.fish` to survive shell restarts.

Verify both wrappers are found correctly:

```bash
which chromium
which chromium-lite
```

Expected output:

```
/home/<user>/.local/bin/chromium
/home/<user>/.local/bin/chromium-lite
```

Reload shell after any changes:

```fish
source ~/.config/fish/config.fish
```

---

# Aliases

Edit:

```bash
vim ~/.config/fish/config.fish
```

Add:

```fish
# --- Chromium Normal Mode ---
alias c='chromium > /dev/null 2>&1 & disown'

# --- Chromium Lite Mode ---
alias cl='chromium-lite > /dev/null 2>&1 & disown'

# --- Chromium Lite Debug (timestamped log) ---
alias clog='chromium-lite > ~/.local/share/chromium-lite/chromium-$(date +%Y%m%d_%H%M%S).log 2>&1 & disown'
```

Reload:

```fish
source ~/.config/fish/config.fish
```

---

# Alias Behavior

## `c` — Normal mode, silent

- Invokes `~/.local/bin/chromium` (normal wrapper)
- Launches detached from terminal
- Terminal closure has no effect on Chromium
- No logs stored
- Full renderer budget (no process cap)
- Use when RAM is available

---

## `cl` — Lite mode, silent

- Invokes `~/.local/bin/chromium-lite` (lite wrapper)
- Launches detached from terminal
- Terminal closure has no effect on Chromium
- No logs stored
- Renderer capped at 6 (AI-tab safe)
- Use during active LLM sessions

---

## `clog` — Lite mode, debug

- Invokes `~/.local/bin/chromium-lite` (lite wrapper)
- Launches detached from terminal
- Writes timestamped log to `~/.local/share/chromium-lite/`
- Each launch creates a **new** log file (no overwrite)
- Use when debugging crashes, GPU issues, or extension problems

Cleanup logs periodically:

```bash
ls ~/.local/share/chromium-lite/
rm ~/.local/share/chromium-lite/chromium-*.log
```

---

# Create Log Directory

Run once:

```bash
mkdir -p ~/.local/share/chromium-lite
```

---

# Flag Explanations

## --process-per-site

Groups all tabs from the same site into one renderer process.

Benefit:
- Prevents renderer process explosion
- Meaningful RAM savings with multiple tabs open

Tradeoff:
- Slightly weaker site isolation vs. process-per-tab default

Recommended: YES for both modes on 8GB systems.

---

## --renderer-process-limit=6 *(Lite mode only)*

Caps the total number of renderer processes Chromium can spawn.

Why 6 (not 4):
- ChatGPT, Claude, Gemini, Perplexity = 4 different origins = 4 renderers
- 2 remaining slots handle other open tabs without contention
- AI tabs remain responsive even under RAM pressure

Why not lower:
- Limit=4 causes AI tabs to share renderers → noticeable slowdown on heavy JS apps

Why not higher:
- Defeats the purpose of lite mode
- RAM savings decrease significantly above 6

---

## --disable-background-networking

Disables Chromium's background Google service calls.

What it stops:
- Telemetry pings
- Component update checks
- Safe Browsing database refreshes
- OCSP/CRL certificate revocation checks *(see Known Tradeoffs)*

What it does NOT stop:
- Normal web traffic (ChatGPT, Claude, all AI tools work fine)
- Any tab-initiated network request

---

## --disable-sync

Disables the Chromium sync engine entirely.

Stops:
- Background sync workers
- Google account sync traffic
- Bookmark/history/password sync

No effect on browsing performance.

---

## --disable-translate

Disables the built-in page translation service.

Stops:
- Language detection background service
- Translation UI

No effect unless you need page translation.

---

## --disable-features=MediaRouter,OptimizationHints

**MediaRouter:**
- Disables Chromecast/Cast discovery
- Stops local network scanning for Cast devices

**OptimizationHints:**
- Disables Google's page load prediction service
- Stops background hints fetching

No effect on regular browsing or AI tools.

---

## --no-default-browser-check

Skips the "make Chromium your default browser?" check at startup.

Benefit: Cleaner, faster startup behavior.

---

## --no-first-run

Skips the onboarding/setup wizard on first launch.

Benefit: Consistent startup regardless of profile state.

---

# Browser Settings (Both Modes)

Open in either Chromium instance:

```
chrome://settings
```

### Privacy & Security

Disable:
- Preload pages for faster browsing
- Send usage statistics

### Cookies

- Block third-party cookies

### Permissions (disable unless needed)

- Location
- Camera
- Microphone
- Notifications

---

# Extensions

Keep maximum **2–3 extensions** across both modes.

Recommended:
- uBlock Origin
- Privacy Badger

Optional:
- ClearURLs

Avoid:
- Multiple ad blockers (conflict + RAM increase)
- Extension bloat of any kind

> Extensions apply per profile, not per wrapper. Both `c` and `cl`
> share extensions if using the same Chromium profile.

---

# Verifying Flags Are Applied

Open in either mode:

```
chrome://version
```

Look for the `Command Line:` section and confirm your flags appear there.

---

# Useful Monitoring Commands

### Chromium processes and RAM

```bash
ps aux | grep chromium
```

### System memory overview

```bash
htop
```

### Top RAM consumers

```bash
ps aux --sort=-%mem | head
```

### Check which Chromium binary is being used

```bash
which chromium
which chromium-lite
```

### Snap version and revision

```bash
snap list chromium
```

---

# Known Tradeoffs

## Certificate Revocation (`--disable-background-networking`)

This flag also disables OCSP/CRL checks. Chromium will not verify
whether an HTTPS certificate has been revoked in real time.

- Risk level: Low for daily dev + LLM use
- Mitigation: DNS resolvers (1.1.1.1 / 9.9.9.9) provide partial protection
- If concerned: Remove this flag from both wrappers; background activity increases slightly

---

## Snap Auto-Updates

Snap updates Chromium automatically without prompting. The wrappers call
`/usr/bin/snap run chromium` which is stable across snap revisions.

If something breaks after an update:

```bash
snap list chromium        # check current revision
snap revert chromium      # roll back to previous revision if needed
```

---

## `--process-per-site` + `--renderer-process-limit=6` Interaction

These two work together:
- `--process-per-site` groups tabs by origin into shared renderers
- `--renderer-process-limit=6` caps how many such renderers exist

In practice: if you have 7+ distinct sites open simultaneously in lite mode,
the 7th site shares a renderer with another site. This is acceptable
behaviour — not a crash or error, just mild contention.

---

# RAM Optimization Reality

Browser flags provide incremental savings only.
The largest RAM reductions come from:

1. Fewer open tabs
2. Smaller LLM context windows
3. Quantized models (Q4 over Q8 where possible)
4. Fewer Electron apps running alongside (VS Code, etc.)
5. Avoiding multiple browser instances simultaneously

Use lite mode as a complement to these habits, not a replacement.

---

# Safe vs Unsafe Flags Reference

| Flag | Safe | Notes |
|---|---|---|
| `--process-per-site` | ✓ | Stable, well-tested |
| `--renderer-process-limit=6` | ✓ | Safe range for AI tabs |
| `--renderer-process-limit=4` | ⚠ | Too low for multiple AI tabs |
| `--renderer-process-limit=1-2` | ✗ | Causes crashes |
| `--single-process` | ✗ | Extremely unstable, do not use |
| `--disable-gpu` | ✗ | Breaks rendering on most pages |
| Random "performance" flags from internet | ✗ | Unpredictable stability |

---

# Expected Outcome

| Behaviour | Normal (`c`) | Lite (`cl`) |
|---|---|---|
| RAM usage | Moderate (hardened, no hard cap) | Lower (capped renderers) |
| AI tabs (ChatGPT, Claude, etc.) | Full performance | Full performance (6 renderer budget) |
| Background Google activity | Disabled | Disabled |
| DevTools | Full support | Full support |
| Terminal dependency | None | None |
| Snap auto-update safe | ✓ | ✓ |

---

# Final Note

This setup is optimized for:

```
Daily development + local LLM usage + multiple AI tabs + controlled privacy + 8GB RAM
```

Not for:

```
Maximum anonymity, Tor-level hardening, or extreme isolation
```
