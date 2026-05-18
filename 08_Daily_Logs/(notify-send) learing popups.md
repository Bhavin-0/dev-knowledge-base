---
type: daily
created: 2026-04-30
tags: []
reviewed:
---
# 2026-04-30

## Work Done

- Implemented a break reminder system using Bash script + cron
- Script triggers desktop notification using `notify-send`
- Configured cron job to run script periodically
- Verified cron execution via `journalctl -u cron`
- Debugged and fixed notification delivery from cron environment

---

## Problems Encountered

- Script executed via cron but no notification appeared
- Root cause: cron runs in a non-GUI, non-interactive environment
- Missing environment variables required for desktop notifications:
  - `DISPLAY`
  - `DBUS_SESSION_BUS_ADDRESS`

- Learned:
  - `DBUS_SESSION_BUS_ADDRESS` connects apps to the user session bus (used by desktop services like notifications)
  - Cron jobs do not inherit user session environment

---

## Fix Applied

- Exported required environment variables inside script:
```bash
export DISPLAY=:0
export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$(id -u)/bus