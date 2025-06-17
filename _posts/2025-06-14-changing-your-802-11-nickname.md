---
title: Hiding Network Attributes - Changing your 802.11 nickname
description: Various ways to change your 802.11 nickname to avoid detection
layout: post
date: 2025-06-14 06:17 -0700
categories: [Security, Anonymity]
tags: [Hiding Network Attributes]
---

## What are the 802 standards?
- The IEEE 802 standards are some network specifications developed to govern LANs, MANs and PANs, to allow interoperability across many networking devices from different manufacturers.
### 802 Architecture in OSI model
- These standards define protocols for the physical and data link layers of the OSI model, to provide unified framework for wired and wireless technologies.
- Here the data link layer is subdivided into:
  - **Logical Link Control (LLC)**: Manages frame synchronization and error checking.
  - **Media Access Control (MAC)**: Handles addressing, collision detection, and channel access.

  | OSI Layer        | IEEE 802 Subdivisions         | Example Standards          |
  |-------------------|-------------------------------|----------------------------|
  | Data Link Layer   | LLC (802.2)                   | Ethernet (802.3), Wi-Fi (802.11) |
  |                   | MAC                           | Bluetooth (802.15.1)       |
  | Physical Layer    | PHY Specifications            | Fiber optics (802.8)       |

  ---
### Major IEEE 802 standards

| Standard      | Scope/Description                             | Key Features & Notes                                                                                                 | Status        |
|---------------|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------|---------------|
| **802**       | Overview & Architecture                       | Defines reference models, MAC address structure, protocol identifiers                                                | Active        |
| **802.1**     | Bridging & Network Management                 | VLANs (802.1Q), Network Access Control (802.1X), Spanning Tree Protocols, Link Aggregation, Audio/Video Bridging     | Active        |
| **802.2**     | Logical Link Control (LLC)                    | LLC sublayer for data link layer                                                                                     | Disbanded     |
| **802.3**     | Ethernet                                      | CSMA/CD, twisted pair/fiber, 10 Mbps–400 Gbps, PoE, 10G PON                                                          | Active        |
| **802.4**     | Token Bus                                     | Token-passing bus access method                                                                                      | Disbanded     |
| **802.5**     | Token Ring                                    | Token ring MAC layer                                                                                                 | Disbanded     |
| **802.6**     | MANs (DQDB)                                   | Distributed Queue Dual Bus for MANs                                                                                  | Disbanded     |
| **802.7**     | Broadband LAN                                 | Broadband LAN practices using coaxial cable                                                                          | Disbanded     |
| **802.8**     | Fiber Optic TAG                               | Fiber optic LAN practices                                                                                            | Disbanded     |
| **802.9**     | Integrated Services LAN                       | IsoEthernet, voice/data integration                                                                                  | Disbanded     |
| **802.10**    | Interoperable LAN Security                    | Security for LAN/MAN                                                                                                 | Disbanded     |
| **802.11**    | Wireless LAN (Wi-Fi) & Mesh                   | Wi-Fi (2.4/5/6 GHz), WPA3 security, OFDMA (ax), mesh networking                                                     | Active        |
| **802.12**    | Demand Priority (100BaseVG)                   | Demand-priority access method                                                                                        | Disbanded     |
| **802.13**    | Reserved                                      | Reserved for Fast Ethernet                                                                                           | Not used      |
| **802.14**    | Cable Modems                                  | Cable modem standards                                                                                                | Disbanded     |
| **802.15**    | Wireless Personal Area Network (WPAN)         | Bluetooth (802.15.1), Zigbee (802.15.4), UWB, Body Area Networks, Visible Light Communication                        | Active        |
| **802.16**    | Broadband Wireless Access (WiMAX)             | Fixed/mobile wireless MAN, OFDM, WiMAX certification                                                                 | Hibernating   |
| **802.17**    | Resilient Packet Ring (RPR)                   | Dual counter-rotating ring topology for high reliability                                                             | Disbanded     |
| **802.18**    | Radio Regulatory TAG                          | Radio regulatory matters for IEEE 802 standards                                                                      | Active        |
| **802.19**    | Wireless Coexistence                          | Standards for coexistence among wireless systems                                                                     | Active        |
| **802.20**    | Mobile Broadband Wireless Access              | Mobile broadband wireless access                                                                                     | Disbanded     |
| **802.21**    | Media Independent Handoff                     | Handover between different network types (Wi-Fi, cellular, etc.)                                                     | Hibernating   |
| **802.22**    | Wireless Regional Area Network (WRAN)         | Broadband wireless access using TV white spaces (cognitive radio)                                                    | Hibernating   |
| **802.23**    | Emergency Services                            | Emergency services networking                                                                                        | Disbanded     |
| **802.24**    | Vertical Applications TAG                     | Smart grid, industrial, medical, and other vertical applications                                                     | Active        |

#### IEEE 802.11 (Wi-Fi) Standard Generations

| Standard   | Year | Frequency Bands        | Max Speed      | Key Features & Notes                    | Wi-Fi Generation |
|------------|------|-----------------------|----------------|-----------------------------------------|------------------|
| 802.11     | 1997 | 2.4 GHz               | 1-2 Mbps       | Original standard, now obsolete         | -                |
| 802.11b    | 1999 | 2.4 GHz               | 11 Mbps        | First widely adopted standard           | Wi-Fi 1          |
| 802.11a    | 1999 | 5 GHz                 | 54 Mbps        | Higher speed, less interference         | Wi-Fi 2          |
| 802.11g    | 2003 | 2.4 GHz               | 54 Mbps        | Backward compatible with 11b            | Wi-Fi 3          |
| 802.11n    | 2009 | 2.4 & 5 GHz           | 600 Mbps       | MIMO antennas, higher speeds            | Wi-Fi 4          |
| 802.11ac   | 2012 | 5 GHz                 | >1 Gbps        | Multi-user MIMO, high throughput        | Wi-Fi 5          |
| 802.11ax   | 2021 | 2.4, 5, 6 GHz (6E)    | Multi-Gbps     | OFDMA, better efficiency, more devices  | Wi-Fi 6/6E       |
| 802.11be   | 2023 | 2.4, 5, 6 GHz         | Up to 40 Gbps  | Extremely high throughput (EHT)         | Wi-Fi 7          |
| 802.11bn   | 2024 | TBD                   | TBD            | Ultra High Reliability (UHR), Wi-Fi 8   | Wi-Fi 8          |

## What is a 802.11 nickname?

- The access point broadcasts its network name as a SSID (Service Set Identifier)
- The 802.11 nickname is a client side identifier that is transmitted suring WiFi connection establishment in management frames for:
  - **Probe requests** : Sent by clients to discover nearby networks.
  - **Authentication Frames** : Used during authentication handshakes.
  - **Association Request Frames** : These are sent to join a network.
- These management frames are sent without encryption so this nickname is exposed to anyone monitoring.
- The 802.11 frame header consists of four address fields with MAC addresses for receiver and transmitted but the 802.11 nickname is not part of the header field but is in the frame body of these management frames, often in:
  - Vendor Specific Information Elements (IEs) : Custom fields added by driver/OS implementations.
    - The general format of an IE consists of a 1-octet Element ID field, a 1-octet Length field (specifies number of octets after this length field), an optional 1-octet Element ID Extension field, and a variable length information field.
  - Non standard extensions to the 802.11 specification.

## How to prevent revealing hostname to the Access Point?
- First, while setting up your computer, make sure to use a hostname which is not obviously tracable to you.
- In linux, first get the wireless_tools package with `sudo pacman -S wireless_tools` if you dont have it since iwconfig is deprecated.
- Then use the command: 
```bash
sudo iwconfig wlan0 nickname "Fucko The Clown"
```
> The iwconfig nickname command relies on Wireless Extensions (WEXT), a deprecated API for Linux wireless drivers. Most modern drivers no longer support this feature since it is not really required for network operation so on a new system you will get an error on running the command above.
{: .prompt-info }

- Workaround for new systems:
```bash
sudo hostnamectl set-hostname "generic-name"
```
- Since the nickname defaults to the hostname we need to rename the machine, then reboot for the changes to take effect.

