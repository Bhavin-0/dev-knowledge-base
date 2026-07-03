# Linux Network Data Usage Monitoring Guide (Zorin OS)

> **Target OS:** Zorin OS (Ubuntu-based)
> **Goal:** Monitor internet usage (daily, weekly, monthly), identify bandwidth-hungry applications, and analyze network traffic efficiently.

---

# Table of Contents

1. Introduction
2. Which Tool Should You Choose?
3. vnStat (Recommended)
4. NetHogs
5. bmon
6. nload
7. iftop
8. iptraf-ng
9. Sniffnet
10. GNOME System Monitor
11. Built-in Linux Commands
12. Automatic Daily Reports
13. Data Reset & Backup
14. Comparison Table
15. My Recommendation

---

# 1. Introduction

Unlike Windows, Linux doesn't include a built-in data usage tracker that records daily or monthly internet consumption.

Fortunately, Linux has several excellent tools that specialize in:

- Historical bandwidth tracking
- Real-time monitoring
- Per-process monitoring
- Network interface statistics
- Traffic analysis

Depending on your needs, different tools are better suited.

---

# 2. Which Tool Should You Choose?

| Purpose | Best Tool |
|----------|-----------|
| Daily Data Usage | ⭐ vnStat |
| Monthly Data Usage | ⭐ vnStat |
| Real-time Speed | nload |
| Live Bandwidth Graph | bmon |
| Per Application Usage | NetHogs |
| Network Traffic Analyzer | iptraf-ng |
| Live Connection Monitor | iftop |
| Beautiful GUI | Sniffnet |

---

# 3. vnStat (Recommended)

## Why vnStat?

vnStat is the gold standard for bandwidth monitoring on Linux.

Advantages:

- Extremely lightweight
- Uses almost no CPU
- Uses almost no RAM
- Stores historical statistics
- Survives reboots
- Automatically tracks usage
- Perfect for laptops

Unlike other monitoring tools, vnStat doesn't need to remain open.

---

## Installation

```bash
sudo apt update
sudo apt install vnstat
```

---

## Enable Service

```bash
sudo systemctl enable --now vnstat
```

Check status

```bash
systemctl status vnstat
```

---

## Find Your Network Interface

```bash
ip link
```

Example

```
lo
enp2s0
wlp2s0
```

WiFi interfaces generally look like:

```
wlp2s0
wlan0
```

Ethernet:

```
enp3s0
eth0
```

---

## Initialize Database

Example:

```bash
sudo vnstat -i wlp2s0
```

Replace:

```
wlp2s0
```

with your interface.

---

## Useful Commands

### Overall Usage

```bash
vnstat
```

Example

```
Today

Download : 2.35 GiB
Upload   : 480 MiB

Total

Download : 58.4 GiB
Upload   : 9.2 GiB
```

---

### Daily Usage

```bash
vnstat -d
```

Example

```
Day        Download   Upload

01 Jul     2.3 GiB    250 MiB
02 Jul     1.9 GiB    320 MiB
03 Jul     5.1 GiB    610 MiB
```

---

### Monthly Usage

```bash
vnstat -m
```

Example

```
July

Download : 74.3 GiB
Upload   : 9.4 GiB
```

---

### Hourly Usage

```bash
vnstat -h
```

---

### Live Speed

```bash
	vnstat -l
```

Example

```
RX: 14.2 Mbit/s

TX: 1.3 Mbit/s
```

---

### Top Days

```bash
vnstat -t
```

---

### Export JSON

```bash
vnstat --json
```

Useful for dashboards.

---

# 4. NetHogs

## Purpose

Shows **which application** is using the internet.

Example:

```
Chrome

2.3 MB/s

VS Code

500 KB/s

Spotify

100 KB/s
```

Perfect when your internet suddenly becomes slow.

---

## Install

```bash
sudo apt install nethogs
```

Run

```bash
sudo nethogs
```

Quit

```
q
```

---

# 5. bmon

Bandwidth Monitor

Displays beautiful live graphs.

Install

```bash
sudo apt install bmon
```

Run

```bash
bmon
```

Useful for:

- Live download speed
- Upload speed
- Packet statistics

---

# 6. nload

Simple real-time bandwidth monitor.

Install

```bash
sudo apt install nload
```

Run

```bash
nload
```

Shows:

- Incoming traffic
- Outgoing traffic
- Current speed
- Average speed
- Total transferred

---

# 7. iftop

Like Linux's Task Manager for network connections.

Install

```bash
sudo apt install iftop
```

Run

```bash
sudo iftop
```

Shows

```
192.168.1.15

↓

google.com

3.2 MB/s
```

Great for identifying active connections.

---

# 8. iptraf-ng

A full-screen network analyzer.

Install

```bash
sudo apt install iptraf-ng
```

Run

```bash
sudo iptraf-ng
```

Provides

- TCP statistics
- UDP statistics
- Interface monitoring
- Packet counters

---

# 9. Sniffnet

Modern graphical interface.

Features

- Live graphs
- Country detection
- Process monitoring
- Connection statistics

Install via Flatpak

```bash
flatpak install flathub io.github.gabm.Sniffnet
```

Launch

```bash
flatpak run io.github.gabm.Sniffnet
```

Best for users who prefer GUI.

---

# 10. GNOME System Monitor

Already installed on most Ubuntu-based systems.

Open

```
System Monitor
```

Network tab shows

- Current upload speed
- Current download speed

Limitations

❌ Doesn't store historical data.

---

# 11. Useful Linux Commands

### Network Interfaces

```bash
ip link
```

---

### Current IP

```bash
ip addr
```

---

### Routing Table

```bash
ip route
```

---

### Active Connections

```bash
ss -tulpn
```

---

### Interface Statistics

```bash
cat /proc/net/dev
```

---

### Wireless Information

```bash
iwconfig
```

---

### Current Internet Speed (Approximate)

```bash
watch -n 1 cat /proc/net/dev
```

---

# 12. Automatic Daily Reports

Create a cron job.

Open

```bash
crontab -e
```

Example

```cron
0 23 * * * vnstat -d >> ~/network_usage.log
```

This saves daily statistics every night.

---

# 13. Reset Statistics

Delete vnStat database

```bash
sudo systemctl stop vnstat
sudo rm -rf /var/lib/vnstat/*
sudo systemctl start vnstat
```

---

# Backup Statistics

```bash
sudo cp -r /var/lib/vnstat ~/vnstat-backup
```

Restore

```bash
sudo cp -r ~/vnstat-backup/* /var/lib/vnstat
```

---

# 14. Comparison Table

| Tool | Historical | Live | GUI | Per-App | Lightweight |
|------|------------|------|-----|----------|--------------|
| vnStat | ✅ | ✅ | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| NetHogs | ❌ | ✅ | ❌ | ✅ | ⭐⭐⭐⭐ |
| nload | ❌ | ✅ | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| bmon | ❌ | ✅ | ❌ | ❌ | ⭐⭐⭐⭐ |
| iftop | ❌ | ✅ | ❌ | Partial | ⭐⭐⭐⭐ |
| iptraf-ng | Partial | ✅ | ❌ | ❌ | ⭐⭐⭐ |
| Sniffnet | Partial | ✅ | ✅ | Partial | ⭐⭐⭐ |

---

# 15. Final Recommendation

## If you only install one tool

Install:

```bash
sudo apt install vnstat
```

It provides:

- Daily usage
- Monthly usage
- Hourly usage
- Historical reports
- Automatic tracking
- Negligible CPU usage
- Negligible RAM usage

---

## My Recommended Toolkit

| Tool | Purpose |
|------|---------|
| vnStat | Historical bandwidth tracking |
| NetHogs | Find which application is using internet |
| nload | Live speed monitoring |
| Sniffnet | Beautiful graphical dashboard |

This combination covers nearly every network monitoring need on Zorin OS while remaining lightweight and easy to use.
