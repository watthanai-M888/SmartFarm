# Milesight Gateway - Network Fix (No Internet)

**Device:** Milesight Gateway  
**IP:** 192.168.1.150  
**Location:** Nakorn Sawan  
**Date:** 2026-06-02  

---

## Problem

Milesight gateway could not connect to internet even though WiFi (NSCAT_Zyxel-1) was connected.

**Root Cause:** Default gateway was set to `eth0` (Ethernet, 192.168.2.1) which has no internet access. WiFi (`wlan0`, 192.168.1.150) was connected to the internet but not used as the default route.

---

## Diagnosis Steps

### 1. SSH into device
```
Host : 192.168.1.150
User : root
Pass : LoRaWAN@2018
```

### 2. Check routing table
```sh
ip route show
```

**Output showed wrong default gateway:**
```
default via 192.168.2.1 dev eth0   ← no internet
192.168.1.0/24 dev wlan0           ← WiFi connected but not default
```

### 3. Test internet connectivity
```sh
ping -c 3 8.8.8.8
```
**Result:** 100% packet loss (no internet)

---

## Fix (Temporary - resets on reboot)

```sh
ip route del default
ip route add default via 192.168.1.1 dev wlan0
ping -c 3 8.8.8.8
```

**Result after fix:** 0% packet loss, internet working.

---

## Permanent Fix (via Web UI)

1. Open browser → `http://192.168.1.150`
2. Login to Milesight admin panel
3. Go to **Network** settings
4. Find **WAN** or **Routing / Priority** settings
5. Set **WLAN (WiFi)** as primary/default WAN interface
6. Save and Apply

---

## Network Summary

| Interface | IP            | Gateway       | Internet |
|-----------|---------------|---------------|----------|
| wlan0     | 192.168.1.150 | 192.168.1.1   | Yes      |
| eth0      | 192.168.2.150 | 192.168.2.1   | No       |
| wg0       | 10.10.1.16    | -             | VPN      |

**WiFi SSID:** NSCAT_Zyxel-1
