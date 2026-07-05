# FOR FRED — SA818 Node on a Dell Wyse 3040
## Pi-to-3040 Translation Notes

**Written by KO6NOI for Fred's SA818 AllStar node build.**
**The build guides you received (hamvoip, ASL3 docs, AllScan, SHARI manual) are all written
for a Raspberry Pi. This document explains every place where the 3040 is different and what
to do instead.**

---

## The Core Difference

A Raspberry Pi has GPIO pins — a row of hardware pins on the board that can be wired
directly to external hardware. The SA818 module connects to those pins for power, serial
communication, audio, and PTT control.

The Dell Wyse 3040 has no GPIO. It's a standard x86 PC. Everything that the Pi does via
GPIO pins, the 3040 does via USB adapters. The SA818 doesn't care — it just needs the
right signals. You're delivering them differently.

**ASL3 software is identical. Debian install is identical. Node registration is identical.
Only the physical hardware interface changes.**

---

## What Changes and What To Buy

### 1. Serial Communication (UART)

**What the Pi guides say:** Connect SA818 TX/RX/GND to GPIO pins 8, 10, 6 (or similar).
The Pi has a built-in hardware UART on those pins.

**What you do on the 3040:** Buy a USB-to-serial adapter. Plug it into any USB port.
It shows up as `/dev/ttyUSB0` (or ttyUSB1, etc.) in Linux — same as a Pi's `/dev/ttyAMA0`
just with a different device name. Update the device path in your SA818 config accordingly.

**Adapter to buy:** CP2102 or CH340G USB-to-serial adapter. ~$5-8 on Amazon.
Search: "CP2102 USB to TTL serial adapter." Get one with 3.3V/5V jumper selection.

**Wiring:** Best educated guess based on a quick read of the source material — SA818 TXD →
adapter RXD, SA818 RXD → adapter TXD, GND → GND. Verify against the SA818 datasheet and
hamvoip wiring diagram before connecting.

**Voltage:** Best educated guess: 3.3V logic. Verify in the datasheet before wiring.

---

### 2. Audio In / Audio Out

**What the Pi guides say:** Audio connects to the Pi's 3.5mm jack or a dedicated hat (SHARI,
etc.) that has audio hardware built in.

**What you do on the 3040:** Use a USB audio adapter. The 3040 has a 3.5mm combo jack but
USB audio is cleaner and easier to configure in ASL3.

**Best option: CM108-based USB audio adapter.** The CM108 chip is specifically supported
by AllStar/app_rpt and handles PTT via the same USB device (see section 3 below).
Search: "CM108 USB audio adapter" — Cmedia CM108 is the chip name.
~$8-15 on Amazon. Confirm it's CM108 before buying — not all cheap USB audio adapters use it.

**Wiring:** Best educated guess based on a quick read of the source material — SA818 audio
out → adapter mic in, adapter audio out → SA818 audio in. Verify pin labels and connector
types in the SA818 datasheet and hamvoip wiring diagram before connecting.

---

### 3. PTT (Push To Talk)

**What the Pi guides say:** PTT is controlled via a GPIO pin. The software toggles that pin
high/low to key the SA818.

**What you do on the 3040:** This is where the CM108 adapter earns its place.
The CM108 chip has a GPIO pin (HID GPIO) that AllStar/app_rpt can toggle directly over USB
to key PTT — no separate hardware needed. This is the standard non-Pi AllStar PTT method.

In your ASL3 `rpt.conf` or `simpleusb.conf`, PTT is configured as:
```
ptttype=cm108
```
with the device path pointing to your CM108 adapter.

**Alternative if you can't find a CM108 adapter:** Use the RTS line of your USB-serial
adapter for PTT. ASL3 supports this but CM108 is cleaner and more commonly documented.

---

### 4. Power for the SA818

**What the Pi guides say:** The SA818 is powered from the Pi's 5V GPIO pins (or external
supply). Some builds use the Pi's regulated output directly.

**What you do on the 3040:** The 3040 provides 5V via USB ports. You can power the SA818
from a USB port using a USB breakout board, or use a separate 5V regulated wall supply.

**Check the SA818 datasheet for its actual supply voltage before connecting anything.**
The SA818V (VHF) and SA818U (UHF) specs may differ slightly. The hamvoip guide
(https://hamvoip.org/hamradio/818_transceiver_module/) has a wiring diagram — read it.

Do not power the SA818 directly from USB without confirming current draw is within USB limits.
A small powered USB hub or dedicated 5V/2A supply is cleaner.

---

## What Stays the Same

These parts of the Pi guides apply directly to your 3040 build — no changes needed:

| Topic | Same on 3040? |
|---|---|
| Debian Linux install | Yes — see Matt's Debian/Wyse build notes |
| ASL3 package install | Yes — identical commands |
| AllStar node registration | Yes — same process, same website |
| `rpt.conf` configuration | Yes — same file format |
| SA818 programming (frequency, CTCSS, squelch) | Yes — same Python script, different device path |
| Firewall / port forwarding (UDP 4569) | Yes — same requirement |
| Node linking and testing | Yes — identical |

---

## Build Order Recommendation

1. Get Debian running on the 3040 first — confirm SSH works, confirm network is solid.
   Matt's Debian install notes cover the Wyse 3040 specifically.

2. Get ASL3 installed and node registered before touching any SA818 hardware.
   Confirm the node shows up on AllStarLink.org. Easier to debug software problems
   without hardware in the loop.

3. Add the USB-serial adapter. Confirm it shows up as `/dev/ttyUSB0`. Run the SA818
   programming script to set your frequency, CTCSS tone, and squelch.

4. Add the CM108 USB audio adapter. Configure simpleusb.conf. Test audio loopback
   before connecting the SA818.

5. Wire the SA818. Test PTT. Make your first local transmission.

---

## Primary Sources — Read These

Before wiring anything, read the actual build guides. These are the sources Matt sent you:

- **SA818 wiring and enclosure:** https://hamvoip.org/hamradio/818_transceiver_module/
- **ASL3 SA818 integration:** https://allstarlink.github.io/adv-topics/sa818modules/
- **Full node build guide:** https://allscan.info/docs/diy-node.php
- **SHARI construction manual** (wiring detail transfers): https://kits4hams.com/wp-content/uploads/2022/10/SHARI-PiHat-Construction-Manual_v1.03.pdf

The SA818 datasheet is your ground truth for voltage, pinout, and current specs.
Do not wire anything based on a blog post alone — confirm against the datasheet.

---

## Known Pitfalls

### USB device names can swap on reboot
If you have both a USB-serial adapter and a CM108 audio adapter plugged in, Linux assigns
device names (`/dev/ttyUSB0`, `/dev/audio`, etc.) at boot based on which USB port initializes
first. Those names can swap between reboots. ASL3 config points to specific device names —
if they swap, nothing works and it's not obvious why.

Fix: set up udev rules that pin each device to a fixed name by USB serial number. Do this
before you finish configuring ASL3, not after you've been debugging for an hour.

### No internal WiFi on the Wyse 3040
The 3040 has no WiFi card slot. Use wired ethernet — for a node this is the right call
anyway. UDP audio over WiFi introduces latency and dropouts. If you absolutely need WiFi,
use a USB WiFi dongle, but wired is strongly preferred.

### Power adapter not included
The Wyse 3040 needs a 5V 3A supply with a 5.5x2.1mm barrel connector. It does not come
with one. Order it at the same time as the rest of your hardware or check your junk drawer.

### SA818 squelch needs tuning
The SA818 ships with default squelch settings that may be too loose for your RF environment.
If squelch is set too low, the node opens on noise and stays open — your linked nodes will
hear static. Tune squelch to your actual noise floor via the Python programming script
before going live.

### SA818 needs programming before it does anything
The SA818 does not work out of the box. Frequency, CTCSS tone, squelch level, and TX power
all need to be set via the SA818 Python programming script over the USB-serial adapter.
Do this step before wiring audio or testing the node — confirm the radio is programmed
correctly first.

### UDP 4569 must be open and port-forwarded
AllStar linking uses UDP port 4569. Your router needs to forward that port to the Wyse 3040's
local IP address, and your firewall needs to allow it. If this isn't done, your node will
register on AllStarLink.org but won't be able to link to other nodes. Easy to forget,
annoying to diagnose.

---

## Questions / Problems

File them as GitHub Issues in the shared repo, or reach out to Matt directly.
Matt ran into plenty of gotchas on his CDM1250 node build — different radio,
same ASL3 software stack. A lot of the troubleshooting is the same.

73 de KO6NOI
