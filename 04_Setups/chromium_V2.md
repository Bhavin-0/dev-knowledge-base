---
type: setup
created: 2026-05-05
updated: 2026-05-16
status: active
tags: [chromium, hardening, ram-optimization]
topic: Chromium Hardened Setup
reviewed:
source:
environment:
---

# Chromium (Hardened + RAM Optimized Setup)

## Objective

Set up Chromium with:

- Reduced telemetry/background activity
- Lower RAM usage
- Stable ChatGPT + DevTools support
- Persistent hardened launch configuration
- Reliable terminal + launcher behavior

---

# Actual Environment Used

- OS: Linux
- Shell: Fish
- Chromium Install Type: Snap
- RAM Target: ~8GB systems running local LLMs
- Launch Architecture: Wrapper Script + PATH Override

---

# Important Reality

The original setup recommended native APT Chromium.

However, this system currently uses:

```bash
snap chromium
```

Because of this:

- `.desktop` launchers behaved inconsistently
- terminal launches ignored custom flags
- Snap wrapper injected its own behavior

Solution used:

```txt
Wrapper Script + PATH Override Architecture
```

---

# Final Architecture

Launch flow:

```txt
chromium / chromium-lite
        ↓
~/.local/bin/chromium-lite
        ↓
snap chromium launcher
        ↓
Chromium with hardened flags
```

This guarantees:
- consistent flags
- detached launches
- terminal-independent execution
- predictable behavior

---

# Main Wrapper Script

Create:

```bash
mkdir -p ~/.local/bin
vim ~/.local/bin/chromium-lite
```

Add:

```bash
#!/bin/bash

exec /usr/bin/snap run chromium \
--process-per-site \
--renderer-process-limit=4 \
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

# PATH Priority (Critical)

Verify:

```bash
which chromium-lite
```

Expected:

```bash
/home/<user>/.local/bin/chromium-lite
```

If needed:

```fish
set -gx PATH ~/.local/bin $PATH
```

Reload shell:

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

-----------For chromium---------
alias c='nohup chromium >/dev/null 2>&1 &'

-----------For chromium-lite-------------
alias cl='nohup chromium-lite >/dev/null 2>&1 &'

alias clog='nohup chromium-lite >~/.local/share/chromium-lite/chromium.log 2>&1 &'
```

Reload:

```fish
source ~/.config/fish/config.fish
```

---

# Alias Behavior

## cl

Silent detached launch.

Features:
- launches in background
- terminal closure does NOT affect Chromium
- no logs stored
- ideal for daily usage

---

## clg

Debug launch.

Features:
- launches in background
- terminal closure does NOT affect Chromium
- stores Chromium logs
- log file overwritten every launch

Useful for:
- startup issues
- crashes
- extension debugging
- GPU issues

---

# Create Log Directory

```bash
mkdir -p ~/.local/share/chromium-lite
```

---

# Flag Explanations

## --process-per-site

Purpose:
- reduces renderer process explosion

Benefit:
- lower RAM usage

Tradeoff:
- slightly weaker site isolation

Recommended:
- YES for low-memory systems

---

## --renderer-process-limit=4

Purpose:
- limits total renderer processes

Benefit:
- fewer Chromium processes
- lower RAM usage

Tradeoff:
- heavy websites share renderers more aggressively

Safe Value:
- 4

Unsafe aggressive values may destabilize Chromium.

---

## --disable-background-networking

Purpose:
- disables background Google networking activity

Reduces:
- telemetry
- background requests
- wakeups

---

## --disable-sync

Purpose:
- disables Chromium sync engine

Reduces:
- background workers
- memory usage
- Google account sync traffic

---

## --disable-translate

Purpose:
- disables integrated translation service

Reduces:
- background services

---

## --disable-features=MediaRouter,OptimizationHints

MediaRouter:
- disables Chromecast discovery
- reduces local network scanning

OptimizationHints:
- disables Google optimization prediction service

Benefit:
- lower background activity

---

## --no-default-browser-check

Purpose:
- disables startup browser checks

Benefit:
- cleaner startup behavior

---

## --no-first-run

Purpose:
- skips onboarding/setup logic

Benefit:
- cleaner startup

---

# Verifying Flags

Open:

```txt
chrome://version
```

Check:

```txt
Command Line:
```

Verify custom flags appear there.

---

# Browser Settings

Open:

```txt
chrome://settings
```

Recommended:

## Privacy

Disable:
- preload pages
- usage statistics

Cookies:
- block third-party cookies

Permissions:
- location
- camera
- microphone
- notifications

Disable unless required.

---

# Extensions

Keep maximum:

```txt
2–3 extensions
```

Recommended:
- uBlock Origin
- Privacy Badger

Optional:
- ClearURLs

Avoid:
- multiple ad blockers
- extension bloat

---

# Safe RAM Optimization Principles

Safe:
- renderer-process-limit=4
- process-per-site

Unsafe:
- --single-process
- disabling GPU aggressively
- random internet performance flags

---

# RAM Optimization Reality

Largest RAM savings come from:

1. fewer Chromium tabs
2. smaller LLM context windows
3. quantized models
4. fewer Electron apps
5. avoiding unnecessary VSCode sessions

Browser flags provide incremental optimization only.

---

# Useful Monitoring Commands

## Chromium Memory

```bash
ps aux | grep chromium
```

---

## System Memory

```bash
htop
```

---

## Top RAM Processes

```bash
ps aux --sort=-%mem | head
```

---

# Expected Outcome

- Lower Chromium RAM usage
- Reduced background Google activity
- Stable ChatGPT + DevTools usage
- Consistent launch behavior
- Better coexistence with local LLM workloads

---

# Final Note

This setup is optimized for:

```txt
Daily development + local LLM usage + controlled privacy + low RAM systems
```

Not for:

```txt
Extreme hardening or anonymity-focused browsing
```
