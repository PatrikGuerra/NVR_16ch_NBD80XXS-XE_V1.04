

# NBD80XXS-XE V1.04 — NVR Board Research & Reverse Engineering Notes

> **Status:** Work in progress — UART access confirmed, root shell pending  
> **Last updated:** April 2026

## Mechanical reference

[Enclosure dimensions, mounting holes, heatsink, BT1 and approximate connector coordinates](MECHANICAL.md). Includes CSV tables and a top-view diagram; measured values and image estimates are labeled separately. See also [component height estimates and comparison drawings](HEIGHTS.md).

# Images

<table>
  <tr>
    <td><img src="images/frontBoard.jpg" alt="Screenshot 1" width="400"></td>
    <td><img src="images/backBoard.jpg" alt="Screenshot 2" width="400"></td>
  </tr>

</table>

---

## 1. Board Identification

| Field | Value |
|---|---|
| **Board Model** | NBD80XXS-XE V1.04 |
| **PCB Revision** | V1.04 |
| **PCB Serial** | 02225862 |
| **Date Code** | JK+20251028 / L20043A |
| **Regulatory** | UL 94V-0, RoHS (Pb-free) |
| **Also known as** | NBD88X16S-KL-V3 and NBD8016S-KL-V2 (all 16CH variant)|
| **OEM Manufacturer** | Hangzhou Xiongmai Technology Co., Ltd |

This board is a generic OEM NVR motherboard used by dozens of brands (Procon, Smar, Hiseeu, Besder, Sunba, Sannce, Zmodo, etc.) under different product names. The board itself never mentions "Xiongmai" anywhere on its silkscreen.

---

## 2. Hardware Specifications

### 2.1 Main SoC

| Field | Value |
|---|---|
| **Chip** | XM8536E (Xiongmai proprietary SoC) |
| **Architecture** | ARM |
| **Bootloader** | U-Boot 2014.04 (compiled Sep 28 2022) |
| **Note** | Not a standard HiSilicon Hi3536 — Xiongmai custom variant |

U-Boot banner confirmed via UART:
```
XM8536E Spl Boot
DDR Init OK
Uncompressing Uboot... done, booting the uboot.
U-Boot 2014.04 (Sep 28 2022 - 09:49:37)
CPU: XM8536E
```

### 2.2 Memory

| Field | Value |
|---|---|
| **RAM** | 256 MiB DDR |
| **Flash** | SPI NOR (size TBD — read via `sf probe 0`) |
| **RAM confirmed by** | U-Boot: `DRAM: 256 MiB` |

### 2.3 Other Identified Chips (from PCB markings)

| Chip Marking | Function | Location |
|---|---|---|
| **ТПК 2524T5 / BT16B03** | Unknown (near HDMI/USB area) | Center-front of board |
| **MOTORCOMM** SZ18201 | Ethernet PHY (AR8035 compatible) | Left side of board |
| **24Cxx** (SOIC-8, near UART) | EEPROM — stores MAC address + Cloud Serial ID | Near J2 connector |

### 2.4 Confirmed Interfaces (physical ports)

| Port | Connector | Notes |
|---|---|---|
| **HDMI** | Full-size HDMI | Video output |
| **VGA** | DB-15 (blue) | Video output |
| **USB** | 2× USB-A | Storage / mouse |
| **SATA** | Standard SATA data + power | HDD connection |
| **RJ45** | Ethernet | Network |
| **Audio IN** | RCA | Microphone input |
| **Audio OUT** | RCA + SPKN SPKP header | Speaker output |
| **433MHz RF** | J2 5-pin JST connector | Wireless remote (NOT UART) |

---

## 3. Power Supply

### 3.1 Input
- **Required:** 12V DC
- **Minimum recommended:** 12V 3A (for board + 1× HDD)
- **Tested with:** 12V 2A (marginal — insufficient under load with HDD)
- **Connector:** DC barrel jack

### 3.2 Internal Power Rails (test points on PCB back side)

| Test Point | Voltage |
|---|---|
| PT1 | +5V |
| PT2 | +0.8V |
| PT3 | +3.3V |
| PT4 | +1.8V |
| PT5 | +1.5V |

### 3.3 UART Voltage
- **Logic level:** 3.3V TTL (confirmed by multimeter)
- TX idle = **3.33V**
- RX idle = **~0.5V** (pulled low when nothing connected)

---

## 4. UART / Serial Console

### 4.1 Finding the Correct UART Port

> ⚠️ **Common mistake:** The J2 5-pin JST yellow connector (labeled "433" on silkscreen) is for a 433MHz RF wireless module — NOT the UART console. Do not use it for serial debugging.

The correct UART is on **connector J1** — 3 small gold pads in the upper-right area of the component-side of the board.

### 4.2 J1 Pinout (UART Console)

```
J1 pad layout (left to right):
┌─────┬─────┬─────┐
│ RX  │ TX  │ GND │
└─────┴─────┴─────┘
```

| Pad | Voltage (idle, NVR on) | Function |
|---|---|---|
| 1 | 0V | GND |
| 2 | 3.33V | TX (NVR transmit → your RX) |
| 3 | ~0.5V | RX (NVR receive ← your TX) |

> **Verify with multimeter before connecting.** The TX pad shows 3.33V steady at idle with brief drops during data transmission.

### 4.3 Serial Settings

| Parameter | Value |
|---|---|
| **Baud rate** | 115200 |
| **Data bits** | 8 |
| **Parity** | None |
| **Stop bits** | 1 |
| **Flow control** | None |
| **Config shorthand** | 8N1 |

### 4.4 Working Connection Setup

```
NVR J1 TX  ──────→  Arduino/Adapter RX
NVR J1 GND ──────→  Arduino/Adapter GND
Arduino RESET ───→  Arduino GND  (keeps ATmega in reset = pure USB-TTL passthrough)
```

> **Note:** Arduino Uno with RESET shorted to GND works as a USB-TTL bridge. A dedicated CP2102/CH340/FTDI adapter is preferred to avoid interference.

### 4.5 Confirmed Boot Log Output

```
XM8536E Spl Boot
DDR Init OK
Uncompressing Uboot... done, booting the uboot.

U-Boot 2014.04 (Sep 28 2022 - 09:49:37)

CPU: XM8536E
WDT: 300S
DRAM:  256 MiB
In:    serial
Out:   serial
Err:   serial
Net:   PHY id1 0 id2 128 dwmac.10010000
Press Ctrl+C to stop autoboot
...
Image Name:   Linux-3.10.103+
Image Type:   ARM Linux Kernel Image (uncompressed)
Data Size:    1613072 Bytes = 1.5 MiB
Load Address: 80008000
Entry Point:  80008000
```

### 4.6 Entering U-Boot Shell

The autoboot delay is very short (0–1 second). To interrupt:

1. Open terminal **before** powering on the NVR
2. Use `screen` or `minicom` (picocom intercepts Ctrl+C locally):
   ```bash
   screen /dev/ttyUSB0 115200
   # or
   sudo minicom -b 115200 -D /dev/ttyUSB0
   ```
3. Power on NVR and **hold Ctrl+C continuously** for 2–3 seconds

Expected prompt: `xm8536 #` or `hisilicon #`

### 4.7 Useful U-Boot Commands (once shell is obtained)

```bash
printenv                          # show all environment variables
printenv bootdelay                # check autoboot delay
setenv bootdelay 3                # increase boot delay for easier access
saveenv                           # persist settings to flash

# Boot directly to root shell (bypasses init):
setenv bootargs 'mem=192M console=ttyAMA0,115200 root=/dev/mtdblock3 rootfstype=squashfs init=/bin/sh'
boot

# Backup full firmware via TFTP:
sf probe 0
sf read 0x82000000 0x0 0x1000000
tftp 0x82000000 firmware_backup.bin 0x1000000
```

---

## 5. Firmware

### 5.1 Installed Firmware Version

```
V4.03.R11.C6380251.12201.040000.0000000
```

| Field | Meaning |
|---|---|
| `V4.03.R11` | XMeye firmware main version |
| `C6380251` | Hardware/board ID (16CH variant) |
| `12201` | Build variant (16 channels) |
| `040000` | Feature build flags |

### 5.2 Firmware Application

- **Main app:** Sofia (Xiongmai proprietary video surveillance binary)
- **Cloud process:** `dvrHelper` / `macGuarder` — maintains P2P cloud connection
- **Kernel:** Linux 3.10.103+
- **Filesystem:** SquashFS (read-only rootfs)
- **Config storage:** Likely JFFS2 or custom on separate MTD partition

### 5.3 Firmware Update Files (structure)

```
romfs-x.cramfs.img   ← root filesystem
web-x.cramfs.img     ← web interface
user-x.cramfs.img    ← user application (Sofia)
custom-x.cramfs.img  ← custom scripts (extapp.sh)
logo-x.cramfs.img    ← boot logo
u-boot.bin.img       ← bootloader
InstallDesc          ← install script (shell commands, executed during update)
```

---

## 6. Network & Services

### 6.1 Open Ports (confirmed via Nmap)

| Port | Service | Notes |
|---|---|---|
| **80/tcp** | HTTP | Web interface + weird .exe download |
| **554/tcp** | RTSP | Video stream |

### 6.2 RTSP Stream URL

```
rtsp://admin:PASSWORD@NVR_IP:554/user=admin_password=PASSWORD_channel=1_stream=0.sdp
```

### 6.3 Port 9530 Status

- **Closed** on this firmware version
- Port 9530 (dvrHelper backdoor) is NOT open — confirms this firmware variant patched or omitted the Xiongmai backdoor
- More secure than typical XMeye devices in this regard

---

## 7. Cloud / P2P (XMeye)

### 7.1 How It Works

The device maintains a **permanent outbound UDP connection** to Xiongmai cloud servers (hosted on Alibaba Cloud, Kingsoft, AWS Singapore/Germany). This connection:
- Is enabled by default
- Bypasses NAT and firewalls
- Allows remote access without any port forwarding
- Routes all video through Chinese-operated servers

### 7.2 Known CVEs

| CVE | CVSS | Description |
|---|---|---|
| CVE-2018-17915 | 8.1 | Unencrypted communication — video stream interception possible |
| CVE-2018-17917 | 5.3 | Hidden "default" account accessible via cloud |
| CVE-2018-17919 | 6.5 | Unsigned firmware updates — remote code injection possible |

### 7.3 Cloud ID Generation

The Cloud ID (UID) is **derived from the device MAC address** — not random. An attacker can enumerate valid Cloud IDs from known Xiongmai MAC OUI ranges.

### 7.4 Disabling P2P (planned — requires root shell)

```bash
# Method 1: Edit config XML
vi /mnt/mtd/Config/NetService.xml
# Change: <CloudEnable>1</CloudEnable>  →  <CloudEnable>0</CloudEnable>

# Method 2: Block outbound via iptables
iptables -A OUTPUT -d secu100.net -j DROP
iptables -A OUTPUT -d xmeye.net -j DROP
# Add to startup script for persistence
```

---

## 8. Thermal / Cooling

### 8.1 Observations

- The heatsink over the XM8536E SoC is **adhesive-mounted** and shows visible gaps — poor thermal contact
- Device reports overheating under normal load
- No active cooling

### 8.2 Recommended Fixes

1. **Thermal paste** — Remove heatsink, clean with IPA, apply MX-4 or equivalent, reseat with uniform pressure. Expected temperature reduction: 10–20°C.
2. **5V fan** — Solder to **PT1 (+5V)** and any **GND** pad on the power test point area (PCB back side). Use a 5V 40×40mm fan (common in 3D printers/routers).
3. **Power supply** — Replace 12V 2A with 12V 3A or 4A quality adapter.

---

## 9. Remote Access (Secure Alternative to XMeye P2P)

### 9.1 Goal

Access NVR remotely without relying on Xiongmai cloud infrastructure, without opening inbound ports, without VPN client on mobile devices.

---

## 10. Open Items / TODO

- [ ] Interrupt U-Boot autoboot and obtain `hisilicon #` / `xm8536 #` prompt
- [ ] Run `printenv` to document full U-Boot environment
- [ ] Obtain Linux root shell via `init=/bin/sh` boot argument
- [ ] Locate and disable P2P cloud process
- [ ] Dump full firmware via TFTP (`sf read` + `tftp`)
- [ ] Document full MTD partition layout
- [ ] Test RTSP stream compatibility with external NVR/VMS
- [ ] Set up Cloudflare Tunnel for secure remote access
- [ ] Identify all 5 pins of J2 433MHz connector (for future reference)
- [ ] Measure J1 pad 3 (RX) voltage to confirm TX/RX/GND order

---

## 11. Tools & References

### Software Used

| Tool | Purpose |
|---|---|
| `nmap` | Port scanning |
| `picocom` | Serial terminal (confirmed working) |
| `minicom` / `screen` | Recommended for U-Boot interaction (no Ctrl+C intercept) |
| Arduino Uno (RESET→GND) | USB-TTL bridge |

### Relevant GitHub Projects

| Project | URL | Notes |
|---|---|---|
| HI3536DV100 hacking | github.com/ubis/HI3536DV100 | OpenWrt for similar NVR SoC |
| OpenIPC firmware | github.com/OpenIPC/firmware | Alt firmware for HiSilicon/XM SoCs |
| hisilicon-dvr-telnet | github.com/Snawoot/hisilicon-dvr-telnet | Port 9530 backdoor PoC (not applicable here) |
| pwn-hisilicon-dvr | github.com/tothi/pwn-hisilicon-dvr | RCE via web interface |
| NBD8016R firmware gist | gist.github.com/orangecms/f61b00099490d50e5305acac0f99fdb9 | Firmware structure reference |

### Security Advisories

- CISA ICS Advisory: ICSA-18-282-06
- SEC Consult: XMeye P2P Cloud vulnerabilities (CVE-2018-17915/17917/17919)
- Habr full disclosure: habr.com/en/articles/486856/

---

*Document compiled from hardware analysis, serial console output, multimeter measurements, and research — April 2026*
