# DNS Connectivity Issue Fix (Linux - Zorin OS)

## Problem

Connected to WiFi hotspot but no internet access. Internet works again
after reconnecting.

------------------------------------------------------------------------

## Root Cause

The issue was identified as a **DNS resolution failure**, not a network
connectivity issue.

### Key Observation

-   `ping 8.8.8.8` → works ✅
-   `ping google.com` → fails ❌

This proves: - Network is fine - DNS is broken

------------------------------------------------------------------------

## System Behavior

-   System uses `systemd-resolved`

-   `/etc/resolv.conf` points to:

        nameserver 127.0.0.53

-   This is a local DNS stub resolver

-   It forwards requests to DNS provided by hotspot (which is unstable)

------------------------------------------------------------------------

## Incorrect Approaches Avoided

-   ❌ Editing `/etc/resolv.conf`
-   ❌ Installing new drivers
-   ❌ Changing DHCP config unnecessarily

------------------------------------------------------------------------

## Correct Fix

Override DNS at NetworkManager level.

### Commands Used

``` bash
nmcli connection show
nmcli connection modify "<wifi_name>" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection modify "<wifi_name>" ipv4.ignore-auto-dns yes
nmcli connection up "<wifi_name>"
```

------------------------------------------------------------------------

## Automation Script

Created a script: `fix_dns.sh`

### Script Content

``` bash
#!/bin/bash

echo "==== DNS Fix Script ===="

read -p "Enter WiFi connection name: " WIFI_NAME

if [ -z "$WIFI_NAME" ]; then
    echo "WiFi name cannot be empty"
    exit 1
fi

nmcli connection modify "$WIFI_NAME" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection modify "$WIFI_NAME" ipv4.ignore-auto-dns yes

nmcli connection down "$WIFI_NAME"
nmcli connection up "$WIFI_NAME"

echo "DNS updated"
resolvectl status | grep "DNS Servers" -A2
```

------------------------------------------------------------------------

## Script Setup

### Make Executable

``` bash
chmod +x fix_dns.sh
```

### Placement Options

#### Option 1: User bin (Recommended)

``` bash
mkdir -p ~/bin
mv fix_dns.sh ~/bin/
```

Add to PATH:

``` bash
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

#### Option 2: System-wide

``` bash
sudo mv fix_dns.sh /usr/local/bin/fix_dns
sudo chmod +x /usr/local/bin/fix_dns
```

------------------------------------------------------------------------

## Notes

-   `*` in file listing indicates executable permission
-   Do NOT modify `/etc/resolv.conf` manually
-   Issue caused by unstable DNS from mobile hotspot

------------------------------------------------------------------------

## Outcome

-   Stable DNS resolution
-   No need to reconnect WiFi repeatedly
-   Quick recovery using script

------------------------------------------------------------------------

## Future Improvements

-   Auto-run script on disconnect
-   Monitor DNS failure automatically
-   Switch to persistent DNS configuration

------------------------------------------------------------------------

Generated on: 2026-08-07 08:11:59
