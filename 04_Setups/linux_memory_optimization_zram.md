---
type: setup
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
status: active
tags: [linux, memory, zram, swap]
topic: Memory Optimization
reviewed:
source:
environment: Linux (systemd, zram-generator)
---

# zram_swap_optimization

## Problem
System slowdown under memory pressure due to inefficient swap usage. 
Default disk swap is slow, causing lag during heavy workloads (AI models, IntelliJ, etc.).

---

## Environment
Linux system using systemd with systemd-zram-generator.

---

## Steps

1. Remove conflicting zram tools (zram-tools).
2. Configure systemd-zram-generator.
3. Set zram compression and size.
4. Create/adjust disk swap as fallback.
5. Set swap priorities (zram > disk).
6. Tune kernel parameters for aggressive zram usage.
7. Verify configuration.

---

## Commands

```bash
# Remove old zram tool
sudo systemctl disable zramswap
sudo apt purge zram-tools

# Clean existing state
sudo swapoff -a
sudo zramctl --reset /dev/zram0 2>/dev/null

# Configure zram
sudo nano /etc/systemd/zram-generator.conf

# Add:
[zram0]
zram-fraction = 0.6
max-zram-size = 0
compression-algorithm = lz4
swap-priority = 100

# Apply config
sudo systemctl daemon-reexec
sudo systemctl restart systemd-zram-setup@zram0

# Create swapfile (4G recommended)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon --priority 10 /swapfile

# Make swap permanent
sudo nano /etc/fstab
# Add:
/swapfile none swap sw,pri=10 0 0

# Kernel tuning
sudo nano /etc/sysctl.d/99-memory.conf

# Add:
vm.swappiness=180
vm.vfs_cache_pressure=50
vm.watermark_scale_factor=125

# Apply kernel params
sudo sysctl --system

# Verify
swapon --show
zramctl
free -h