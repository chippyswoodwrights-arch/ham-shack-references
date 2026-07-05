# AllStar Node Build — Build Notes

## Project Summary

Simplex AllStar home node for KO6NOI (Technician, Auburn CA / CM98).
Hardware: Motorola CDM1250 → RIM-Maxtrac USB Lite (Maxtrac edition) → Dell Wyse 3040 (Debian 13 Trixie + ASL3).
Target: bridge to W6EK (51018), K6IS (61464), CARLA regional network.

---

## Parts List

| Component | Source | Status | Cost |
|---|---|---|---|
| Motorola CDM1250 | Already owned | On hand | $0 |
| Dell Wyse 3040 (part 3YX35) | Tech For Less | Shipped | ~$40 |
| RIM-Maxtrac USB Lite (Maxtrac edition) | repeater-builder.com | **Delivered** | $50 |
| *(USB cable to PC included with RIM-Maxtrac — no separate purchase needed)* | | | $0 |
| *(CDM1250 accessory connector — RIM plugs in directly, no interface cable needed)* | | | $0 |
| CDM1250 programming cable (RKN4081, FTDI USB) | bluemax49ers.com (KJ6ZWL) | Shipped | ~$33 |
| DisplayPort to HDMI adapter (passive, 1080p) | Amazon — Plugable or StarTech | Shipped, ETA Thu 2026-06-18 | ~$8 |
| Tuneable copper J-pole | Already owned | On hand | $0 |
| 12V DC power supply (Powerwerx SS-30, 30A) | Already owned | On hand | $0 |
| CDM1250 DC power cable | Already owned | On hand | $0 |
| Antenna connector adapter: PL-259 female to F male | On hand (repurposed) | On hand | $0 |
| Snap-on ferrite clamps (2x 20-piece assorted, 5 sizes) | Amazon | Delivered | $18 |
| Bird 50-A-MFN-10 RF attenuator (50W, 10dB, N-type) | eBay — SharonSells2000 | On order — ETA after July 4th | $34 |
| N to UHF adapter kit (4pc — all gender combos) | Amazon | On order | $9.59 |
| **Total confirmed new spend** | | | **~$192** |

---

## Universal Radio Programming Rule

**Always start from a new/blank codeplug. Never modify a donor codeplug.**

This applies to every radio, every brand, every build — CDM1250, Baofeng, Yaesu, Kenwood, Icom, Anytone, anything.

When you reprogram a used radio by modifying the existing codeplug, you inherit every setting the previous owner made — squelch modes, signaling systems, scan lists, TOT timers, option board flags, phone interconnects, emergency settings — across every channel and every tab. Most of it is invisible unless you audit every field on every tab of every channel. You will miss something.

**Correct process for any radio:**
1. Read the radio → save as a factory/donor backup (reference only, never modify this file)
2. Create a new blank codeplug in the programming software for that radio model
3. Configure radio-wide settings first (audio routing, accessory pins, power settings)
4. Add only the channels you actually want
5. Write it

You put in exactly what you need. You know what's in every field because you put it there. No auditing required.

The one thing worth preserving from the donor read is radio-wide hardware configuration (accessory pin assignments, audio routing) — but that's a small known set of fields you can copy deliberately, not a reason to keep the whole donor codeplug.

---

## Lessons Learned — What Went Wrong on First Build (2026-06-18)

These are real mistakes made during the first KO6NOI build. Fred should read this before starting.

### 1. Gather all hardware BEFORE powering on the Wyse
You need all of these at your desk before you touch the power button:
- USB keyboard (can't borrow the laptop keyboard mid-install)
- Monitor with HDMI input
- HDMI cable
- DisplayPort to HDMI adapter (Plugable DP-HDMI) — Wyse has DisplayPort out only
- Debian 13 Trixie boot USB (created with Rufus beforehand)
- Ethernet cable confirmed to be a data cable (not a charge-only cable)

Running to find a keyboard or cable mid-install is painful. Ask how we know.

### 2. Accept the network mirror during the Debian install
The installer offers to configure a network mirror. **Accept it.** Use `deb.debian.org`, protocol `https`.

If it fails once, try again — don't skip it. Skipping means apt has no package sources after install, which means you can't install openssh-server, which means you're stuck typing every command on the TV instead of SSH-ing in from your laptop.

### 3. Generate your SSH keypair on the laptop BEFORE the install
Without SSH key access, you can't drive the Wyse remotely after install (password SSH works from a terminal but not from automated tools).

Before you start the install, open PowerShell on your laptop and run:
```
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\wyse_key -N ""
```

Then during the Debian install, when it asks to install an SSH server — say yes. After first boot, copy the key over:
```
type $env:USERPROFILE\.ssh\wyse_key.pub | ssh matt@<wyse-ip> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

This gives Claude Code direct SSH access to drive the configuration from your laptop.

### 4. Check that your ethernet cable is a real data cable
During the KO6NOI build, the ethernet cable was a dud — no link at all after install. Swap it if you see both interfaces showing DOWN in `ip addr`. A known-good cable from a working device is the fastest test.

### 5. Debian 13 uses a different apt sources format than older guides show
Many online guides show the old `/etc/apt/sources.list` format. Debian 13 Trixie uses `/etc/apt/sources.list.d/debian.sources`. If you end up needing to fix apt manually, use:
```
sudo nano /etc/apt/sources.list
```
And add:
```
deb https://deb.debian.org/debian trixie main contrib non-free non-free-firmware
deb https://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware
deb https://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
```
Then `sudo apt-get update`.

### 6. Debian 13 SSH password authentication is disabled by default — fix it correctly (2026-06-30)

Debian 13 ships with `PasswordAuthentication no` explicitly set in `/etc/ssh/sshd_config`.
SSH uses the **first matching value** it finds — appending `PasswordAuthentication yes` at the end does NOT override it.

**Wrong (does not work):**
```
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
```

**Correct (replace the existing line):**
```
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart ssh
```

**Better — use the preseed to fix it automatically during install:**
The boot USB preseed.cfg now does this automatically via late_command. Any new Wyse install from this USB gets SSH password auth enabled, sudo installed, and passwordless sudo configured for matt — no manual steps after install.

**PAM faillock gotcha:** After many failed SSH attempts in a row, PAM locks the account.
Fix: `faillock --user matt --reset` (run as root on the target machine).

**SSH tool limitation:** Claude Code's Bash SSH tool cannot supply passwords interactively.
Always set up key-based auth first: `ssh-keygen` on PC → copy pub key to `~/.ssh/authorized_keys` on Wyse.
Without key auth, Claude cannot drive the Wyse remotely regardless of whether password SSH works.

---

## Lessons Learned — CPS Programming (2026-06-19)

Real mistakes made programming the CDM1250 for the AllStar node. Read this before touching CPS on a donor codeplug.

### 1. Audit ALL tabs on donor codeplug personalities — not just Basic
When you repurpose a commercial codeplug, you replace frequencies and tones on the Basic tab. Everything else carries over invisibly. On this build, all three channels had Portland PD residue in:
- **Options**: Time-Out Timer at 60 sec (kills transmissions over 1 min — fatal for a node)
- **Signaling**: DTMF-1 on Rx and Tx (Portland dispatch signaling)
- **Scan**: Scan-1 assigned (radio would attempt to scan Portland channels)
- **Phone**: Phone-1 assigned (Portland phone interconnect)
- **Advanced**: Option Board Feature checked (Portland had a signaling option board we don't have)

Always open every tab and verify. Don't assume the Basic tab is the only thing that matters.

### 2. TOT default is 60 seconds on commercial Motorola radios
"Time-Out Timer" is a commercial feature that cuts off transmissions longer than N seconds (prevents stuck mics from tying up a system). Default is 60 sec. For an AllStar node, set it to **Infinite** — AllStar audio can easily run longer than 60 seconds and the radio will just stop transmitting mid-conversation.

### 3. Option Board Feature — uncheck it if you don't have the board
Portland PD used an option board (likely MDC signaling). If "Option Board Feature" is checked and no board is installed, the radio may behave unexpectedly. It's on the Advanced tab, easy to overlook.

### 4. Alias field editing in CPS — triple-click doesn't work
Triple-click, double-click, Ctrl+A all fail to select the full alias text in CPS edit fields. The text includes a hidden prefix from the original personality name.
**Reliable method**: click field → End → BackSpace until clear → type new name → click the Rx Frequency field to commit (NOT Tab — Tab can trigger unexpected navigation).

### 5. Save As filename field — same problem
Triple-click in the Save As filename field appends instead of replacing.
**Reliable method**: click field → Home → Shift+End → Delete → type new name → Save.

### 6. Nav bar X button deletes the whole record, not individual rows
In zone/personality list dialogs, the X button in the nav bar (bottom of window) deletes the entire zone or personality record. To delete individual rows from a list (e.g., zone channel assignments), right-click the specific row → Delete from context menu.

### 7. Row selection is sticky — left-click the row number first
Right-clicking always operates on the LAST selected row, regardless of where you click. Before right-clicking a row, left-click its row number column to change the selection, then right-click.

### 8. Batch row deletions overshoot as the list shrinks
When deleting multiple rows from a zone channel list, rows shift upward after each deletion. A fixed y-coordinate that was correct for row 4 will now land on row 3 (or higher) once earlier rows are deleted. Delete one row at a time from the bottom up, or take a screenshot between each deletion to recheck coordinates.

### 9. Maximize the CPS window before editing personalities
When CPS is in a smaller window, the personality dialog's top section (Alias, Channel Bandwidth) is hidden behind the tree view. Maximize the window first — tree and form become separate draggable MDI windows and everything is visible.

### 11. Start from a NEW codeplug, not a donor codeplug
When repurposing a commercial radio, the instinct is to read the radio, modify the existing codeplug, and write it back. Don't.

**Correct sequence:**
1. Read the radio → save as factory backup (for reference only)
2. Create a **new** codeplug in CPS for the target radio model
3. Set Radio Configuration / accessory pins first (hardware-level settings)
4. Add only the personalities you actually want
5. Write it

A new codeplug has known CPS defaults everywhere. You put in exactly what you need and nothing else. Zero donor residue, zero tab auditing.

The donor-codeplug approach on this build cost ~45 minutes of tab-by-tab auditing across 3 channels to find Portland PD residue in Options, Signaling, Scan, Phone, and Advanced. The Radio Config accessory settings (4 fields, ~5 minutes) were the only thing worth preserving from the donor — not worth it.

### 10. Write Device success has no confirmation dialog
After File → Write Device → OK on COM port → "Verifying that the device can accept the current codeplug" in the status bar → then blank status = success. There's no popup saying "write complete." Blank status bar + no error dialog = done.

---

## Build Steps

1. **Register AllStarLink node number** at allstarlink.org (requires callsign verification — do early, can take hours to a day to process)
2. **Check CGNAT status** — log into router, check WAN IP. If 10.x / 172.16–31.x / 100.64–127.x = CGNAT. If CGNAT: plan WireGuard tunnel via 44Net Connect before ordering. Port 4569 forwarding will not work.
3. **Order** Wyse 3040, RIM-Maxtrac, programming cable
4. **Program CDM1250** using CPS R06.12.05 + programming cable:
   - Install FTDI driver first (CDM21236_Setup.zip from bluemax49ers.com/driver-downloads/) — do NOT use Prolific driver, that is a different cable type. Allow 4–8 min after first plug-in for Windows to finish.
   - CPS must be installed before connecting cable.
   - Set simplex frequency: 146.460 MHz, PL 127.3 Hz
     Source: NARCC database search 2026-06-14, El Dorado County 2m — simplex segment clear.
     100.0 Hz PL avoided (used by KD6CQ locally). 127.3 is clean in EDH area.
     Confirm with SFARC Monday net before finalizing. Program Baofengs and FT-70D to match.
   - Accessory Pin 3 = Data PTT (Input) ✅ Source: VARA-FM CDM-750 reference build + CDM1250 accessory connector doc
   - COS/CTCSS wiring — DELIBERATE DEVIATION from hookup sheet (documented below):
     Hookup sheet (primary source) specifies: Pin 12 → RIM Lite Pin 3 (COS), Pin 8 → RIM Lite Pin 4 (CTCSS)
     ⚠️ We are NOT following the hookup sheet two-pin approach. Instead:
       CDM Pin 8 only → RIM Lite Pin 4 (CTCSS input)
       CDM Pin 12 → not wired
     Reason: simplex node uses PL tone squelch. CDM Pin 8 programmed as PL+CSQ Detect (combined)
     feeds the CTCSS input. AllStar reads ctcssfrom=usb only. carrierfrom=no.
     A combined PL+CSQ signal on Pin 4 = tone squelch. No separate COS wire needed.
     Source for this approach: W2YMM (UNVERIFIED). RIM Lite v2 schematic (primary source)
     confirms Pin 4 = CTCSS only input — consistent with this wiring.
     If reverting to hookup sheet two-pin approach: set carrierfrom=usb, ctcssfrom=usb.
   - RIM Lite v2 solder jumpers: NONE to set. Lite has no polarity jumpers (unlike full RIM).
     COS_IN and CTCSS_IN connect directly through protection diodes to CM119A GPIO. (verified 2026-06-14 against schematic)
   - RIM Lite v2 chip confirmed: CM119A (not CM119B). rxboost=no confirmed correct.
   - AllStar software settings:
       carrierfrom=no  |  ctcssfrom=usb  |  txmixa=voice
     NOTE: hookup sheet says carrierfrom=usb — we deliberately use carrierfrom=no (single-pin PL squelch approach)
   - Accessory Configuration → Rx Audio Type = Flat Audio ✅ Source: VARA-FM CDM-750 reference build
     CPS path: Radio Config → Accessory Config → RX Audio Type (confirmed W9CR Waris wiki)
   - Channel Bandwidth = WIDE (25 kHz) ✅ Source: VARA-FM CDM-750 reference build
   - Squelch Type = CSQ ✅ Source: VARA-FM CDM-750 reference build
5. **Wipe Wyse 3040 eMMC and install Debian**. Boot from USB flash drive (create with Rufus — not Win32 Disk Imager).
   BIOS setup (press F2 at Dell logo; default password "Fireport" if set):
   - Maintenance → Wipe eMMC first — prevents partition leftover problems
   - Secure Boot → Disabled (check twice — known to need two passes to stick)
   - AC Recovery → Last Power State (node restarts after power outage)
   - Intel SpeedStep → Disabled
   - C-States Control → Disabled
   - UEFI Network Stack → Disabled
   - Integrated NIC → Enabled (NOT with PXE)
   - Virtualization Support → Disabled
   - USB Wake Support → Disabled; Wake on LAN → Disabled
   - Fastboot → Thorough
   UNVERIFIED source: allscan.info/docs/3040-setup.html (community guide, not ASL3 official docs)

   Debian install:
   - Press F12 at Dell logo → boot from USB
   - Select **Advanced (non-graphical) install** — required to get "Force GRUB install to EFI path" option
   - Choose "Force GRUB install to EFI path" = **Yes** — mandatory on Wyse 3040
   - Desktop environment: None (8GB eMMC — skip it)
   - Install SSH server; skip web server
   - Do not enable root login
   - ~20 min install time
   - GPIO/pinctrl errors at startup are benign — ignore

   If SecureBoot cannot be disabled: run `sudo asl-setup-dkms-mok` after ASL3 install,
   set a one-time password, reboot with monitor attached, enroll key at MOK prompt.
   Miss the prompt = must rerun. Source: allstarlink.github.io/adv-topics/uefi-secureboot/

   Post-install eMMC wear mitigation:
   - Point /var/log and /tmp at tmpfs early
   - Consider /var/log on USB stick for multi-year continuous operation
6. **Install ASL3** on Debian — follow allstarlink.github.io and ASL3 wiki
7. **Connect RIM-Maxtrac** to CDM1250 accessory port and Wyse 3040 USB
8. **Configure ASL3** using `sudo asl-menu` → Node Settings:
   - Enter node number and password
   - Select USB sound device (RIM-Maxtrac) — `asl-find-sound` identifies device string
   - devstr= can be left blank; ASL3 auto-assigns on first run
   - Set duplex type (half duplex for simplex node)
   - AllStar settings from hookup sheet: carrierfrom=usb, ctcssfrom=usb, txmixa=voice
   - Add `rxondelay=25` to simpleusb.conf manually — asl-menu does NOT set this
     Prevents COR line glitch keying on simplex nodes. 25ms = community convention, not official spec.
     Source: community recommendation (UNVERIFIED). ASL3 docs confirm parameter exists, max 60000ms.
   - Keep txmixa + txmixb sum below 1000 to avoid TX audio clipping
   - Registration file: rpt_http_registration.conf (NOT iax.conf)
   - Verify registration: node shows green at allstarlink.org/nodelist (allow up to 30 min)
   - Forward UDP port 4569 on Comcast router to Wyse 3040 LAN IP
   - Check logs if issues: `journalctl -xeu asterisk.service`
9. **Audio calibration** using `sudo /usr/sbin/radio-tune-menu`:
   - RX level: average not past 3 kHz mark; peaks not past 5 kHz mark
   - TX level: FM deviation under 3 kHz, clean audio
   - RX Boost: OFF (disable in nearly all cases)
   - Check ClipCnt — if > 0, input is overdriving, reduce RX level
   - Echo mode: parrot test for final verification
   - Save to EEPROM: `radio tune save`
10. **Tune J-pole** to match simplex frequency, mount on deck
11. **Bench test** locally before bridging to any network
12. **Connect to W6EK (51018)** once locally operational and tested
13. **SFARC Monday digital-nodes net** — in-person help for final wiring/config if needed

---

## To-Do List

### Order Tomorrow
- [ ] BlueMax49ers — "Motorola CDM1250 FTDI Programming Cable RKN4081" ~$33
       bluemax49ers.com/product-explorer/motorola-cdm1250-ftdi-programming-cable-rkn4081/
       Includes CPS R06.12.05 download. FT232RL chip, RJ45 connector, 6ft. Login was down 2026-06-13.
- [ ] Dell Wyse 3040 (part 3YX35) ~$40 — techforless.com
- [ ] RIM-Maxtrac USB Lite (Maxtrac edition) $50 — repeater-builder.com/products/usb-rim-lite.html
- [ ] Fair-Rite #43 snap-on ferrite clamps (assorted pack, qty 6-10) ~$15 — Amazon or DX Engineering
- [ ] DisplayPort to HDMI adapter (passive, 1080p) ~$8 — Amazon (Plugable DP-HDMI or StarTech DP2HDMI)
       One-time use: Wyse 3040 has DP only. Needed to see screen during Debian install. Can borrow if available.

### AllStarLink Portal (can do now)
- [x] Add server and request node number — COMPLETE 2026-06-14
       Node number: 52547 | Server: KO6NOI | Callsign: KO6NOI
       Password: [see local credentials file — not stored in git]

### While Waiting for Parts
- [ ] Download CPS R06.12.05 — FREE, no account needed
       W9CR Waris Wiki: https://wiki.w9cr.net/index.php/Waris
       File: HVN9025_v6.12.05.zip — scroll to CPS section
       BlueMax49ers does NOT include CPS with cable purchase — download separately
       ⚠️ Windows 10 fix: after installing CPS, also run commsbepx64_setup.exe from the
       CPS program files folder — fixes "Can not create unknown radio component" error
       Fix details: https://kd8tnv.com/cps-error.html
- [ ] Install FTDI driver on Windows programming PC before cable arrives
       Download CDM21236_Setup.zip from bluemax49ers.com/driver-downloads/
       Install first, then plug in cable — allow 4–8 min for Windows to finish
- [ ] Download Debian 13 Trixie amd64 ISO, create boot USB with Rufus
- [ ] Talk to SFARC contact — what does CPS call the COR output and CTCSS output on
       programmable Digital I/O pins 8 and 12? Look for "PL and CSQ Detect" or "Monitor Output"

### When Hardware Arrives — In Order
- [ ] Wyse 3040 BIOS setup (F2 at Dell logo, password "Fireport" if set):
       Wipe eMMC in Maintenance first → Secure Boot off (do it twice) →
       AC Recovery = Last Power State → SpeedStep off → C-States off →
       UEFI Network Stack off → NIC on (no PXE) → Virtualization off
- [ ] Install Debian 13 Trixie — Advanced installer, Force GRUB to EFI = Yes, no desktop, SSH only
- [ ] Install ASL3, run sudo asl-menu, enter node number and password
- [ ] Program CDM1250 with CPS — full procedure:

       CONNECT AND READ
       1. Install CPS R06.12.05 FIRST, then plug in programming cable (FTDI driver installs with CPS)
       2. Power on CDM1250
       3. Connect programming cable RJ45 to radio front mic jack, USB to PC
       4. Open CPS → File → Connect → select correct COM port → click Read
          Radio must be on and cable seated — you will see progress bar as codeplug reads
          If read fails: check COM port in Device Manager, verify cable seated, try lower baud rate

       CHANNEL PROGRAMMING
       5. Left panel → Channels → click channel 1 (or add new channel)
       6. Set Rx Frequency = 146.460000 MHz
       7. Set Tx Frequency = 146.460000 MHz (same — simplex)
       8. CTCSS/DPL → Tx Tone = 127.3 Hz, Rx Tone = 127.3 Hz
       9. Channel Bandwidth = Wide (25 kHz)
       10. Squelch Type = CSQ
       11. Emphasis = None (flat audio path for data)

       ACCESSORY PIN CONFIGURATION
       12. Left panel → Radio Configuration → Accessory Configuration
           Rx Audio Type = Flat Audio
           Ext PTT Audio = Flat Tx Audio
           Hook = unchecked
       13. Left panel → Radio Configuration → Accessory Pins
           Pin 3 = External PTT, Input, Active Level = Low
           Pin 8 = "PL and CSQ Detect/Talkgroup Detect", Output, Active Level = High
                   (flip to Low if node doesn't key — batboard builds use Low, W2YMM uses High)
                   Field name verified 2026-06-14: repeater-builder CDM acc conn page + batboard
           Pin 12 = not programmed (not used in single-pin PL squelch approach)

       WRITE TO RADIO
       14. File → Connect → Write (or click Write button)
           Progress bar will show — do not disconnect during write
       15. Power cycle radio after write completes
       16. Verify channel shows 146.460 on display
- [ ] Install ferrite clamps (#43 snap-on):
       DC power leads to CDM1250 — 2-3 turns through clamp, as close to radio as practical
       USB cable (Wyse to RIM-Maxtrac) — clamp at each end
       Coax feedline — 6-8 turns of coax through clamp at antenna base (common mode choke)
- [ ] Connect RIM-Maxtrac directly to CDM1250 20-pin accessory connector (RIM plugs in — no cable needed).
       Connect RIM-Maxtrac Mini-USB to Wyse 3040 USB-A using included USB cable.
- [ ] Manually edit /etc/asterisk/simpleusb.conf after asl-menu:
       asl-menu sets: node number, password, IAX port, channel driver selection ONLY
       These must be hand-edited — asl-menu does NOT set them:
         carrierfrom=no
         ctcssfrom=usb
         rxondelay=25
         rxboost=no
         deemphasis=no
         preemphasis=no
         legacyaudioscaling=0
       Source: allscan.info/docs/Node-Radio-Module-Settings.php (UNVERIFIED community guide)
- [ ] Audio calibration: sudo /usr/sbin/radio-tune-menu
       RX avg not past 3 kHz mark, peaks not past 5 kHz, TX deviation under 3 kHz
       RX Boost off. Save: radio tune save
- [ ] Forward UDP 4569 on Comcast router to Wyse 3040 LAN IP
- [ ] Verify node registration green at allstarlink.org/nodelist (allow up to 30 min)
- [ ] Bench test locally — key up, receive, confirm audio both ways
- [ ] Connect to W6EK (51018) once locally verified and clean

### rxboost — resolved 2026-06-14
RIM-Maxtrac uses CM-119 (not CM-119B). ASL3 docs: rxboost=no is correct default for all
CM-108/CM-119 chips. Only enable if rxmixerset is maxed out and audio is still too weak.
CDM1250 flat audio = 330mV RMS at 60% deviation — normal range, boost should not be needed.
W2YMM's "rxboost=Enabled" was likely specific to his radio's output level, not a general rule.

---

## Phase 2 — Echolink Integration

Add Echolink to node 52547 after the node is locally stable and tested. ASL3 has Echolink
support built in — same audio protocol (standard VoIP, no proprietary codec, no hardware dongle).
The node appears on both AllStar and Echolink simultaneously, same as W6EK does (AllStar 51018 +
Echolink 4128 on the same repeater).

**What this gets you:**
- Echolink users can connect to your node from any Echolink node or the Echolink phone app
- You can DTMF-connect from your radio to any Echolink node worldwide
- Huge net presence — many nets have run on Echolink for 20+ years
- Echolink app (iOS/Android) = anyone with a phone can get on the air through your CDM1250

**Steps (do NOT start until node is live and stable):**
1. Register callsign at echolink.org — free, requires license verification, allow 1-2 days
2. Fetch ASL3 Echolink docs before touching any config:
   https://allstarlink.github.io (search Echolink module)
3. Enable and configure Echolink module in ASL3 per official docs
4. Verify node appears in Echolink node list

**Note:** config steps above are placeholders — fetch ASL3 Echolink docs at setup time,
do not configure from memory.

---

## Design Decisions

### Project Philosophy
Ham radio was founded on repurposing surplus military equipment, hand-built gear, and clever
cheap workarounds — not high-tech exotic solutions. This build embodies that spirit:
surplus commercial radio ($0, already owned), $40 thin client, $50 interface board, open
source software. Total outlay under $200 for a fully functional linked node. Technology is
embraced where it serves the mission (ASL3, Claude Code for Linux config), not for its own sake.
Ferrite rule: buy a cheap assortment, stack more on any cable that needs it. Impedance stacks
linearly — three cheap clamps beat one premium core and you still have spares.

### Simplex node (not repeater)
Appropriate for single CDM1250 (single-band, no duplexer), single local user, apartment deck. No duplexer required for simplex operation.

### Wyse 3040 chosen as node computer
<4W fanless, Intel Atom x5-Z8350, 2GB RAM, 8GB eMMC. Adequate for Debian 12 (Bookworm) + ASL3. ~$40 refurb.
Risk: eMMC wear from 24/7 Asterisk logging. Mitigation: tmpfs for logs, or USB stick for /var/log. Long-term: internal M.2 B-key slot can accept M.2 SATA SSD via adapter (see Wyse 3040 service manual).

### ⛔ CDM1250 POWER FLOOR — READ THIS BEFORE YOU BUY ANYTHING

The CDM1250 comes in two VHF variants and NOBODY in the RIM-Maxtrac / AllStar ecosystem tells you this matters:

| Model | Power Range | Minimum Power |
|---|---|---|
| **KHD** (low power) | 1–25W | **1W** — correct for a home simplex node |
| **KKD** (high power) | 25–45W | **25W minimum** — too much for a home simplex node |

**The problem:** A home simplex node at 25W covers 10+ miles. Anyone in that radius can key up your node, get on the AllStar network under your node number, and you have zero authentication to stop them. The whole point of a home node is a short-range personal RF link — you and your HT, that's it.

**How to check your radio BEFORE buying supporting hardware:** Look at the model number label on the radio. It will contain either KHD (good) or KKD (problem). Check this on day one — before buying a RIM-Maxtrac, before buying a programming cable, before spending a single dollar on anything else.

**If you have a KKD:** Solution is a hardware RF attenuator inline on the coax — passive, always on, cannot be accidentally changed in software. Bird 50-A-MFN-10 (50W rated, 10dB) drops 25W programmed output to 2.5W at the antenna. Add two N-to-SO-239 adapters to mate with standard ham coax. Total cost ~$44. This is better than buying a new radio — hardware redundancy on top of the CPS power setting.

**Why the RIM-Maxtrac documentation doesn't mention this:** The board maker is solving the interface problem, not the power problem. They assume you'll figure out whether the radio is suited for your use case. They assume wrong. Do not make the same mistake that was made here.

---

### CDM1250 Antenna Connector — Not Standard
The CDM1250 VHF uses an **F connector** (the small screw-on barrel type used for cable TV/satellite) on the antenna port — NOT the SO-239/PL-259 that most ham gear uses. This catches builders off guard.

**Adapter needed:** PL-259 female to F male barrel adapter (~$2-5). Screws onto the radio's F connector and presents a standard SO-239 that your coax PL-259 plugs straight into. No special tools, no crimping. Buy one and leave it permanently on the radio.

If you're building a fixed node with a dedicated coax run: a barrel adapter is cleaner than reterminating the coax with an F connector.

### RIM-Maxtrac USB Lite (Maxtrac edition) for hardware COS
Wires hardware GPIO lines to CDM1250 accessory connector. True hardware COS — not VOX.
Pins 7+9 shorted for auto power-on after power outage (confirmed against VARA-FM reference build).
**CPS requirement:** COR must be assigned in CPS to whichever programmable Digital I/O pin the
RIM-Maxtrac hardware COS input physically connects to on the accessory connector. CDM1250 has no
dedicated COR pin — candidates are Pin 4, 8, 12, or 14. ⚠️ Confirm against RIM-Maxtrac schematic
PDF (rimdocs.pdf / v2 PDF from repeater-builder.com/products/usb-rim-lite.html) before programming.

### FT-70D chosen over FT-5DR
FT-5DR premium features evaluated against actual use case and rejected:
- Wires-X portable node: requires SCU-57 cable + Windows laptop running Wires-X software. FT-70D + Pi-based portable AllStar node is cheaper and more flexible (bridges via DVSwitch, not just Wires-X).
- Dual-band/dual-receive: professional background = separate radios/scanners preferred.
- GPS/APRS: worst performance in canyon/wilderness terrain where it would matter most.
- Bluetooth/lapel mic: extra failure points, no clear use case.
- Touchscreen: reported durability/scratching concerns.

### Deferred items (deliberate, not oversights)
- Mobile AllStar node for QYT: narrow use case, revisit if need recurs
- Portable hotspot (Pi/MMDVM or SharkRF): home node + FT-70D covers realistic scenarios
- DMR: revisit if Sacramento/PAPA-area digital access becomes relevant
- FT-5DR: revisit only if Wires-X-specific portable node capability becomes specifically wanted
- Baofeng-as-node: viable future cheap secondary project, not primary

---

## Session Log

### Session 1 — 2026-06-13
- Project plan finalized (simplex node, parts list, build steps)
- Peer review identified open items: COR pin (Pin 8), CGNAT check, antenna connector, 12V supply, CPS sourcing, eMMC wear, Wyse BIOS setting
- CLAUDE.md, REFERENCES.md, build_notes.md created
- No hardware ordered yet

### Session 2 — 2026-06-13 (continued)
- Verified all primary sources: CDM1250 accessory connector, RIM Lite v2 schematic, hookup sheet, VARA-FM build, W9CR Waris wiki, BlueMax49ers driver/cable, CPS R06.12.05 release notes
- AllStarLink account registered; callsign verified; node number pending (add server step remaining)
- Read ASL3 manual through: Important Considerations, Debian Install, Basic Operation/asl-menu,
  Troubleshooting, Audio Filters, Basic Operation, Advanced Topics overview
- Read USB Audio Interfaces page (allstarlink.github.io/adv-topics/usbinterfaces/) — documented
  in REFERENCES.md: devstr= auto-assign, asl-find-sound, CM-108 native support, radio-tune-menu,
  audio calibration targets, EEPROM save procedure
- All open pre-order items resolved except BlueMax49ers order (login down — order tomorrow)
- build_notes.md updated: Debian 13 Trixie (recommended), SecureBoot disable, audio calibration
  step added as Step 9, steps renumbered

**Next session agenda:**
1. Order BlueMax49ers CDM1250 FTDI programming cable (login was down — try again)
2. Order Dell Wyse 3040 from Tech For Less (~$40, part 3YX35)
3. Order RIM-Maxtrac USB Lite v2 from repeater-builder.com ($50)
4. Add server and request node number on allstarlink.org (account validated, this step pending)
5. Read remaining ASL3 manual sections: UEFI/SecureBoot, Audio Calibration (if separate page)
6. Confirm CDM1250 CPS pin assignments for COS (Pin 12) and CTCSS (Pin 8) with SFARC contact
   or Monday digital-nodes net before programming

### Session 3 — 2026-06-18
- Debian 13 Trixie installed on Wyse 3040 — Expert install, Force GRUB to EFI = Yes
- eMMC wiped in BIOS before install (Maintenance → Data Wipe)
- BIOS settings confirmed: Secure Boot off, AC Recovery = Last Power State, SpeedStep/C-States off
- tmpfs configured for /tmp and /var/log (eMMC wear protection)
- ASL3 3.9.3 installed via apt — Asterisk active and enabled at boot
- Node 52547 configured in asl-menu: Half-duplex hotspot, SimpleUSB
- simpleusb.conf settings confirmed: carrierfrom=no, ctcssfrom=usb, rxondelay=25, rxboost=no
- Duplicate template lines cleaned from simpleusb.conf
- apt sources fixed for Debian 13 Trixie (main + updates + security repos)
- SSH key access established — Claude Code can now SSH in directly via wyse_key
- IAX2 registration not active — deliberately deferred until radio connected
- Wyse LAN IP: 10.0.0.196, login: matt/[see local credentials file — not stored in git]

**Remaining before node goes live:**
1. RIM-Maxtrac arrives → connect to CDM1250 accessory port and Wyse USB
2. ~~Program CDM1250 with CPS~~ **DONE 2026-06-19** — CDM1250_AllStar.mdf in AllStar Node folder.
   3 channels: NODE (146.460/CSQ/TPL 127.3), MON (146.460/CSQ/CSQ), W6EK (145.430rx/144.830tx/CSQ/TPL 162.2).
   All Portland PD residue removed: TOT=Infinite, Signaling=None, Scan=Disabled, Phone=Disabled, Option Board=Off.
   **⚠️ REPROGRAMMING NEEDED — see checklist below before next CPS session.**
3. Audio calibration: sudo /usr/sbin/radio-tune-menu
4. Forward UDP 4569 on Comcast router to 10.0.0.196
5. Verify node green at allstarlink.org/nodelist
6. Bench test, then connect to W6EK (51018)

**CDM1250 Reprogramming Checklist (do before attenuator session):**

- [ ] Change NODE channel frequency: 146.460 → **147.625 MHz** (simplex, no offset)
- [ ] Set transmit power to **25W** (High zone) in CPS — attenuator takes care of the rest
- [ ] Verify P4 button assignment in CPS — currently shows H/L next to speaker icon on LCD.
      **Confirm whether P4 is Volume Level or Transmit Power.** If it's Power Level, document
      what High and Low actually output in watts (Low on KKD may still be 25W — verify).
- [ ] Confirm P3 (labeled CALL) is programmed as External Alarm — not a call alert tone.
      Decide if this is useful for node use or should be reassigned to something else.
- [ ] Confirm P1 (labeled MON) is Monitor/squelch defeat — expected, leave as-is.
- [ ] Confirm P2 (screen dimmer) — leave as-is.
- [ ] Check and increase **mic gain** in CPS — previous codeplug likely has it set low.
      Symptom: repeater opens fine but audio weak/difficult to copy on W6EK net (2026-06-29).
      Mic in use: RMN4038B (correct Motorola mic, not the issue). Deviation/audio routing suspect.
- [ ] Confirm **audio routing** settings — verify mic input is not set to accessory/external.
- [ ] Update CDM1250_AllStar.mdf with new frequency and save new backup before writing.

**Phase 2 — after node is stable:**
7. Add Echolink to node 52547 (see below)

---

## Variant Build — Fred's Low Power UHF Node (W6TTZ)

### Concept
Fred needs a UHF AllStar node for neighborhood-range use — 440MHz, ~0.5–1W, apartment friendly.
The CDM1250 path is wrong here: minimum 25W, requires attenuator, overkill for one operator linking to the network.
Better approach: Wyse 3040 + SA818 UHF RF module. Low power by design, cheap, same ASL3 stack.

### Why Not CDM1250 for Fred
- CDM1250 KKD minimum power: 25W — covers 10+ miles, anyone in range can key the node
- Attenuator workaround adds cost and complexity ($34+ Bird attenuator)
- SA818 starts at milliwatts — right-sized for the use case from day one

### Hardware Plan

| Component | Est. Cost | Notes |
|---|---|---|
| Dell Wyse 3040 | ~$20 | Same platform as KO6NOI node — known good |
| SA818 UHF module | ~$15–20 | 400–480MHz, 0.5W or 1W output, serial control |
| USB-to-serial adapter | ~$5 | FTDI FT232R — connects SA818 to Wyse USB |
| USB sound card | ~$10 | Audio interface — RX/TX audio between SA818 and ASL3 |
| Small UHF antenna | ~$10–15 | 1/4 wave whip or rubber duck for indoor use |
| 5V power supply | ~$10 | Powers Wyse and SA818 from same rail |
| **Total** | **~$60–70** | vs $192+ for CDM1250 path |

### SA818 Notes
- SA818 vs DRA818U: functionally similar, SA818 is more widely used in current community builds
- Serial control via AT commands — frequency, tone, squelch all programmable via software
- PTT via GPIO or serial — USB-to-serial adapter handles this
- Audio: line level in/out — USB sound card bridges to ASL3 SimpleUSB

### ASL3 Integration
- Same Wyse 3040 + Debian 13 + ASL3 install as KO6NOI node — Fred can follow the same build notes
- SimpleUSB driver handles the USB sound card audio path
- SA818 serial control handled via small Python or shell script at startup — sets frequency and tone
- carrierfrom / ctcssfrom config same as simplex node

### Key Differences from KO6NOI Build
- No RIM-Maxtrac needed — SA818 has direct PTT and audio pins
- No programming cable, no CPS, no commercial radio codeplug
- No attenuator needed — power is appropriate by design
- Smaller, simpler wiring — fewer failure points

### Open Questions (research before building)
- [ ] Verify SA818 PTT interface to ASL3 SimpleUSB — GPIO or serial line?
- [ ] Confirm SA818 audio levels compatible with SimpleUSB expected range
- [ ] Fetch SA818 datasheet before any config — do not rely on memory
- [ ] Node registration: Fred needs his own AllStarLink account and node number

### Status
Concept only. Fred takes General exam 2026-06-28. Build after node 52547 is live and stable —
Fred can reference KO6NOI lessons learned before starting his own build.
