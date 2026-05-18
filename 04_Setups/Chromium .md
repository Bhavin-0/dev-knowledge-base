---
type: setup
created: 2026-05-05
updated: 2026-05-05
status: active
tags: []
topic: 
reviewed:
source:
environment:
---
# Chromium (Hardened Setup)

## Objective

Set up Chromium with:

- Reduced data leakage (privacy-focused)
    
- Lower RAM overhead (without breaking ChatGPT & DevTools)
    
- Clean, reproducible configuration
    

---

## Environment

- OS: Linux (Debian/Ubuntu-based recommended)
    
- RAM: ~8GB
    
- Install type: **APT package (NOT Flatpak/Snap)**
    

---

## Key Principles (Read Once)

- Privacy ≠ break websites
    
- Optimization ≠ aggressive flags
    
- Stability > theoretical gains
    

---

## Tasks Covered

1. Privacy hardening (anti-tracking, no background calls)
    
2. RAM optimization (process + feature control)
    
3. Stability for heavy web apps (ChatGPT, DevTools)
    
4. Clean startup configuration (persistent flags)
    

---

## Steps

### 1. Install Correct Chromium Version

Avoid sandboxed versions (Flatpak/Snap → higher RAM)

```bash
sudo apt update
sudo apt install chromium-browser
```

---

### 2. Apply Persistent Startup Flags

Modify launcher so flags apply every time

```bash
cp /usr/share/applications/chromium.desktop ~/.local/share/applications/
nano ~/.local/share/applications/chromium.desktop
```

Replace:

```bash
Exec=chromium %U
```

With:

```bash
Exec=chromium --process-per-site --disable-background-networking --disable-sync --disable-translate --disable-features=MediaRouter,OptimizationHints --no-default-browser-check --no-first-run %U
```

---

### 3. Configure Browser Settings

Open:

```bash
chrome://settings
```

Apply:

#### Privacy & Security

- Safe Browsing → Standard
    
- Disable:
    
    - Preload pages
        
    - Usage statistics
        

#### Cookies

- Block third-party cookies
    

#### Permissions

Disable unless needed:

- Location
    
- Camera
    
- Microphone
    
- Notifications
    

---

### 4. Install Minimal Extensions (Strict Limit: 2–3)

Required:

- uBlock Origin
    
- Privacy Badger
    

Optional:

- ClearURLs
    

DO NOT install multiple blockers (causes conflicts + RAM increase)

---

### 5. Enable Security Isolation

Open:

```bash
chrome://flags
```

Enable:

- Strict-Origin-Isolation
    

---

### 6. System-Level Privacy (Important)

Set DNS:

- Cloudflare → 1.1.1.1
    
- Quad9 → 9.9.9.9
    

---

### 7. Optional RAM Optimization (Safe)

Avoid aggressive flags that break sites.

Safe additions if needed:

```bash
--renderer-process-limit=4
```

(Use ONLY if system struggles)

---

## Commands (Summary)

```bash
# Install Chromium
sudo apt update
sudo apt install chromium-browser

# Copy launcher
cp /usr/share/applications/chromium.desktop ~/.local/share/applications/

# Edit launcher
nano ~/.local/share/applications/chromium.desktop
```

---

## What NOT to Do

- Do NOT use Flatpak/Snap Chromium
    
- Do NOT install many extensions
    
- Do NOT use random “performance flags” from internet
    
- Do NOT disable core web features (breaks ChatGPT)
    

---

## Expected Outcome

- RAM usage reduced by ~20–30%
    
- No background Google calls
    
- Stable ChatGPT + DevTools performance
    
- No repeated setup needed
    

---

## Future Improvements (Optional)

- Use system-wide firewall (ufw) to block telemetry
    
- Use separate browser profile for testing
    
- Monitor memory:
    

```bash
ps aux --sort=-%mem | head
```

---

## Final Note

This setup is optimized for:

> Daily development + heavy web apps + controlled privacy

Not for:

> Maximum anonymity or extreme hardening