---
title: Hiding Network Attributes - Spoofing your MAC address
description: Various ways to spoof your MAC address to avoid detection
layout: post
date: 2025-06-13 11:17 -0700
categories: [Security, Anonymity]
tags: [Hiding Network Attributes]
---

## What is a MAC Address?
- Unique 48-bit identifier for each network interface (wired or wireless).
- First 3 octets: manufacturer (OUI), last 3: device-specific.
- Used for identification on local networks (LAN); not visible beyond the first router.
- Commonly logged by routers, ISPs, and public Wi-Fi.

---

## Why Spoof Your MAC?
- Anonymity on local networks.
- **MAC Address filtering** : some corporate networks control network access by witlisting device MACs, though 802.1x is more effective. In a corporate environment, the devices like switches, printers etc are whitelisted by their MAC address.
  - These devices also have their MAC addresses displayed on their labels or we can access them in settings menu without password protection.
  - Once you get this we can use this to evade device based bans by: 
    1. connecting ethernet cable to laptop used to attack
    2. try pinging, if not allowed then maybe MAC address not whitelisted
    3. find MAC of whitelisted devices by label, settings etc
    4. spoof your MAC address
    5. get a new ip address with spoofed MAC (renew DHCP lease) by `sudo dhclient -r eth0` to remove old assigned one and `sudo dhclient eth0` to get new one.
    6. Try pinging again
- Evade device-based bans.
- Obtain new DHCP IP from ISP.
- Blend in with other devices (e.g., mimic vendor MACs)[1].

---

## Key Commands

### Show MAC Address

```bash
ip link show <interface>
```
or, using macchanger : 
```bash
macchanger -s <interface>
```

### Set Random MAC (Temporary)

```bash
sudo ip link set <interface> down
sudo macchanger -r <interface>
sudo ip link set <interface> up
```

### Set Specific MAC

 ```bash
sudo ip link set <interface> down
sudo macchanger -m aa:bb:cc:dd:ee:ff <interface>
sudo ip link set <interface> up
```

### Restore Permanent MAC 
```bash
sudo ip link set <interface> down
sudo macchanger -p <interface>
sudo ip link set <interface> up
```

---

## Persistently Randomize MAC at Boot

Edit your crontab with `crontab -e` and add the line `@reboot macchanger -r <interface>`. This gives new random MAC on every reboot[1].

---

## Useful Tips

- **Check current and permanent MAC:**  
  `macchanger -s <interface>`
- **Vendor-specific MACs:**  
  `macchanger -l` lists vendor OUIs (first 3 octets).  
  To blend in, use:  
  `macchanger -A <interface>` (random vendor MAC)
- **Don't forget to bring interface down before changing MAC and up after.**
- **Automate for all interfaces you care about.**

---

## Block Quote

> MAC addresses are only visible within the local network. Spoofing helps with local anonymity but does not affect your identity beyond the LAN[1].

---

## Table: macchanger Options

| Option           | Description                        |
|------------------|------------------------------------|
| -s <iface>       | Show current & permanent MAC       |
| -r <iface>       | Set fully random MAC               |
| -m XX:XX:...     | Set specific MAC                   |
| -p <iface>       | Restore permanent MAC              |
| -A <iface>       | Set random MAC of same vendor      |
| -l               | List vendor prefixes (OUIs)        |

---

## Footnotes
