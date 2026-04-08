---
type: debugging
created: 2026-04-08
updated: 2026-04-08
status: resolved
tags: []
reviewed:
environment:
---

# Bluetooth_No_Sound_After_Connection

## Symptoms 
- Device connects succesfully via bluetooth.
- No sound in earbuds/ headphones 
- output source : speaker only 
---

## Context

OS: Zorin OS (Linux)
Audio System: PipeWire
Scenario: Bluetooth earbuds connected but not available as audio output

## Cause

Package `libspa-0.2-bluetooth` was not installed

---

## Fix

1.  Install required backend:  
    sudo apt update  
    sudo apt install libspa-0.2-bluetooth
2. Restart audio services:  
    systemctl --user restart pipewire pipewire-pulse wireplumber
3. Reconnect Bluetooth device:
    - Turn Bluetooth OFF → ON
    - Remove device
    - Pair again
4. Verify:
    - Open `pavucontrol`
    - Select Bluetooth device as output
    - Set profile to **A2DP Sink (High Quality Audio)**

---

## Prevention

- If device conects but no audio ->  suspect backend, not drivers 

---

## Related Notes
- [[ ]]

