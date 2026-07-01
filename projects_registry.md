# Ham Shack Projects Registry

*Single view of all active, planned, and backlogged projects.*
*For full build notes see the individual project folders.*
*Last updated: 2026-06-25*

---

## Active Projects

### Ham AI Operating Assistant
- **Folder:** `C:\Users\mattc\OneDrive\Desktop\Ham Radio\`
- **Status:** Phase 2 in progress — core loop live on real hardware
- **What it is:** Python/Flask web app. Listens to WSJT-X FT8 decodes, flrig rig control,
  fldigi CW/PSK31, SSB voice via Whisper. Claude Haiku gives plain-English operating guidance.
- **Stack:** Python, Flask, Anthropic API (Haiku), WSJT-X UDP, flrig XML-RPC, fldigi XML-RPC,
  faster-whisper, sounddevice, pyttsx3
- **Phase 2 remaining:** Log4OM coexistence (WSJT-X secondary UDP port), voice band monitoring,
  fldigi PSK31/RTTY verification with live signals
- **Phase 3:** JS8Call, WSPR, RTTY, agent architecture (privilege checker, decode curator,
  band-conditions pre-processor)
- **Notes:** Sessions 1–10 complete. See project_notes.md for full session log.

### AllStar Node 52547
- **⛔ CDM1250 POWER WARNING:** Radio on hand is KKD variant — 25W minimum floor. KHD variant needed for home simplex node (1W minimum). Options: accept 25W, find KHD (~$150 eBay), or use different radio. Do NOT repeat this mistake on future builds — check model number label BEFORE buying RIM-Maxtrac or any supporting hardware.
- **Folder:** `C:\Users\mattc\OneDrive\Desktop\AllStar Node\`
- **Status:** Hardware assembled, software configured — pending RIM-Maxtrac arrival and audio cal
- **What it is:** Simplex AllStar home node. CDM1250 → RIM-Maxtrac → Wyse 3040 (Debian 13 + ASL3).
  146.460 MHz simplex, PL 127.3 Hz. Bridges Matt (C4FM/FT-70DR) and Fred (DMR/Anytone) via internet.
- **Wyse:** 10.0.0.196, SSH via wyse_key, ASL3 3.9.3 installed, node 52547 configured
- **CDM1250:** Programmed — CDM1250_AllStar.mdf. 3 channels: NODE/MON/W6EK. Portland PD residue removed.
- **Remaining:**
  1. RIM-Maxtrac arrives → connect to CDM1250 and Wyse
  2. Audio calibration (radio-tune-menu)
  3. Forward UDP 4569 on Comcast router
  4. Verify node green at allstarlink.org/nodelist
  5. Bench test, then connect to W6EK (51018)
- **Phase 2:** Echolink integration (after node stable)

### FT-70DR Programming
- **Status:** RT Systems ADMS-70D + USB-57B cable ordered, pending delivery
- **Goal:** Organized channel banks — analog FM repeaters / C4FM digital channels
- **MY-NODE channel (row 26):** 146.460 MHz programmed, tone needs updating to 127.3 Hz once node live
- **Files:** KO6NOI_FT70D_full.csv, 145.19000KO6NOI_FT70D.FT70D on desktop

---

### Deck Antenna — Ed Fong DBJ-1
- **Notes:** antenna_notes.md in Ham Radio folder
- **Status:** Parts sourcing — need Class 200 3/4" PVC pipe (Home Depot SKU #282-477)
- **Plan:** Rebuild UHF connector on existing element → insert into correct thin-wall pipe →
  paint → mount on 2x4 clamped to deck rail. Option B is copper pipe tunable build.
- **Key lesson:** Must use Class 200 (thin-wall irrigation pipe), NOT Schedule 40. Heavier pipe
  attenuates signal and detunes antenna — confirmed from failed first build.
- **Source:** Ed Fong DBJ-1, QST Feb 2003. https://edsantennas.weebly.com/about.html

### DIY CNC Router
- **Folder:** `C:\Users\mattc\OneDrive\Desktop\CNC Router\`
- **Status:** Planning — no parts purchased. Ender 3 build comes first.
- **What it is:** Small hobbyist CNC router for furniture repair shop use.
  Use cases: meat grooves on cutting boards, dovetail joints, small furniture parts, jigs, templates.
- **Work envelope:** 500x500mm (~20x20 inch), 3-4 inch Z height
- **Key hardware:** Makita trim router (in hand) as spindle. Wood torsion box base (full shop available).
  Linear rails, aluminum extrusion gantry, NEMA 17 steppers, Arduino + GRBL controller.
- **Estimated cost:** ~$215-235 (spindle and frame materials already owned)
- **Build sequence:** Ender 3 first → print Makita mount → build torsion box → assemble CNC
- **Notes:** Linear rails required (not V-wheels) — dovetail cutting forces demand rigidity.
  Ender 3 prints parts for the CNC. CNC cuts parts for furniture repair business.

### Ender 3 3D Printer
- **Folder:** `C:\Users\mattc\OneDrive\Desktop\Ham Radio\ender3\`
- **Status:** Printer confirmed working. Nozzle plugged (user error) — replacement nozzles on hand.
- **Blocker:** Swap nozzle, confirm test print
- **Slicer:** Cura (community standard for Ender 3)
- **Print server:** OctoPrint on Wyse 3040 (planned, after printer stable)
- **Ham use cases:** EFHW unun holder, antenna hardware, Arduino enclosures, shack mounts
- **Notes:** See ender3/build_notes.md for full use case list and Octopussy build plan
- **Octopussy build:** 2x Wyse 3040 + OctoPrint — one per printer (Matt's Ender 3 + buddy's da Vinci 1.0A after Repetier flash). Blocked on nozzle swap.

---

## Backlog — Vetted, Ready to Build

### Da Vinci 1.0A Repetier Firmware Flash
- **What:** Replace stock XYZ firmware with open source Repetier — removes RFID DRM, unlocks standard 1.75mm filament
- **Hardware:** XYZ da Vinci 1.0A (buddy's printer) + USB cable for flashing
- **Goal:** After flash, da Vinci accepts any 1.75mm filament (PLA/PETG/etc) instead of expensive XYZ cartridges
- **Benefit:** ~$12/kg vs $3/kg filament cost, standard tooling (Cura, OctoPrint)
- **Bonus:** Can then use OctoPrint on a Wyse 3040 to control the da Vinci remotely
- **⚠️ Critical:** Repetier flash is ONE-WAY — no revert to stock firmware. Must be certain before flashing.
- **Status:** Backlog. Requires primary source research (SoliForum thread, Repetier GitHub) before proceeding.
- **Next step:** Research phase in dedicated session. Fetch: https://www.soliforum.com/topic/9138/ and https://github.com/luc-github/Repetier-Firmware-4-Davinci

### OpenWebRX Browser Waterfall
- **Hardware:** RTL-SDR Blog V4 (~$35) + second Wyse 3040
- **What:** Full spectrum waterfall from any browser on the network
- **Ham AI integration:** Band-conditions agent gets real RF data — noise floor, signal presence,
  interference detection — instead of inferring from spot counts
- **Build notes:** openwebrx_build_notes.md (verified 2026-06-23 — all commands from primary sources)
- **Status:** IN PROGRESS — build notes complete, ready to install when Wyse 3040 units arrive

### NOAA/METEOR Weather Satellite Receive
- **Hardware:** RTL-SDR Blog V4 + $15 turnstile antenna + Wyse 3040
- **Software:** WXtoImg or SatDump (free)
- **Status:** Vetted. Can run on same Wyse as OpenWebRX.

### ADS-B Aircraft Tracking
- **Hardware:** FlightAware Pro Stick (~$20) + Wyse 3040
- **Software:** dump1090
- **Status:** Vetted. Trivial add alongside OpenWebRX.

### Home Weather Station + Grafana Dashboard
- **Hardware:** ESP32 (~$10) + DHT22 temp/humidity (~$5) + BMP280 barometric pressure (~$3)
- **Software:** InfluxDB (time-series database) + Grafana (dashboard) on dedicated Wyse 3040
- **What it does:** Wireless sensor data → InfluxDB → Grafana dashboard viewable from any browser
- **Ham crossover:** Feed solar indices, decode counts, and band conditions into same Grafana instance
  alongside local weather — one dashboard showing RF conditions + barometric pressure trend together
- **Why ESP32 over Arduino:** Built-in WiFi, no USB cable to Wyse needed, same price, bigger ecosystem
- **Sensors (~$20 total):** Temp, humidity, barometric pressure. Expandable: wind speed, rain gauge
- **Status:** Vetted. Start after Octopussy build is stable and a spare Wyse is available.

### Dad's Place — Home Monitoring & Smart Home
- **Notes:** dad_project.md
- **Status:** Planning — Phase 1 is doorbell camera
- **Stack:** Doorbell cam → Wyse 3040 + Frigate → Home Assistant → full smart home
- **Key need:** Visual phone alerts (hard of hearing, hearing aids on phone)
- **Next step:** Dad researches doorbell cameras via Claude + Consumer Reports

### Home Security System (Frigate + Home Assistant)
- **Notes:** dad_project.md (Dad's place), hardware_inventory.md (parts bin)
- **Status:** Parts on hand — need 1-2 ThinkCentre M or OptiPlex micro units
- **Hardware on hand:** 2x Wyze V3 cameras, 512GB Intel NVMe, 256GB Kioxia NVMe,
  24GB DDR4 RAM (16+8), 2x Intel WiFi 6 cards
- **Stack:** Frigate (NVR) + Home Assistant on ThinkCentre M or OptiPlex micro (Linux)
- **Builds:** One for Matt's shop/apartment, one for Dad's place
- **Next step:** Shop Tech For Less for ThinkCentre M or OptiPlex micro — whichever is cheaper
- **Docs:** frigate.video, home-assistant.io

### Shop Security Cameras (Wyze V3 + Frigate)
- **Hardware:** 2x existing Wyze V3 cameras + Wyse 3040
- **What:** Flash Wyze V3 with RTSP firmware → feed into Frigate (free, open source NVR)
  on Wyse. Local recording, no subscription, no Wyze cloud crashes.
- **Night vision:** Wyze V3 has built-in IR — already solved, no hardware change needed.
- **Status:** Concept. Firmware research needed before starting — instructions change frequently.
  Do not proceed from memory, fetch current source first.

### Wireless Shack Node — Untethered Operating
- **What:** Operate the TS-480SAT from anywhere in the house or on the property.
  Radio stays at the desk. All control and audio rides the mesh network.
- **Stack:**
  - Bluetooth headset → tablet → audio over WiFi → Wyse 3040 at desk
  - Wyse runs: PipeWire/JACK audio routing + LADSPA filters + RNNoise AI suppression + flrig
  - Wyse → Rigblaster → TS-480SAT (same cable already in place)
  - Tablet is thin client: mic input + flrig remote display
- **Filter chain on Wyse:** High-pass → noise gate → RNNoise (AI suppression) → compressor → Rigblaster
- **Mesh:** Extend existing network to property — tablet roams freely
- **BT latency note:** Fine for SSB voice on repeater. For FT8/digital, tablet needs to be
  wired — FT8 timing is millisecond-sensitive, BT adds 50-200ms.
- **Hardware needed:** Tablet (or laptop), BT headset, mesh AP if property coverage needed
- **Wyse role:** Same 3040 platform — audio processing brain + rig controller in one box
- **Status:** Concept. Design before build. Start after AllStar node and Octopussy are stable.

### Pi-hole Network Ad Blocker
- **Hardware:** Wyse 3040
- **What:** DNS-level ad blocker for the whole network. Blocks ads, trackers, and malware
  domains before they reach any device.
- **Status:** COMPLETE — live as of 2026-06-30
- **IP:** 10.0.0.191 (static), admin at http://10.0.0.191/admin
- **SSH:** ssh -i ~/.ssh/wyse_octoprint matt@10.0.0.191
- **Block list:** StevenBlack (82,928 domains)
- **Upstream DNS:** Cloudflare 1.1.1.1
- **Clients configured:** Windows PC (Wi-Fi adapter, manual DNS), Pixel phone (static IP, manual DNS)
- **Xfinity router:** Cannot set DNS at router level — Xfinity locks it down. Each device configured manually.
- **DNS over HTTPS:** Must be OFF in Chrome and on phone for Pi-hole to see traffic.

### Arduino SWR/Power Meter
- **Hardware:** Arduino + bridge components + OLED (~$15-20)
- **Status:** Vetted. Community designs documented on QRZ/GitHub.

### Arduino Automatic Antenna Tuner
- **Hardware:** Arduino + roller inductor + steppers (~$40-60)
- **Status:** Vetted. ATU-100 is the most common open source design.

### Arduino CW Keyer
- **Hardware:** Arduino + paddle + relay (~$15)
- **Status:** Vetted. K3NG CW keyer is the gold standard.

---

## Backlog — Concept / Evaluate Later

### Phase 3 Ham AI Agents
- Privilege-checker agent (isolated, auditable safety logic)
- Decode-curator agent (filters WSJT-X decodes before main Claude prompt)
- Band-conditions pre-processor agent (NOAA/RBN/PSKReporter → structured summary)
- Do NOT start until core loop stable through Phase 2

### Open HamClock Backend (OHB)
- Self-hostable aggregated data source: NOAA solar, Kp, PSK Reporter, WSPR, RBN, MUF, VOACAP
- Single query replaces hitting 4+ raw APIs
- Evaluate at Phase 3 agent design time
- OHB: ohb.works | GitHub: github.com/BrianWilkinsFL/open-hamclock-backend

### RTL-SDR Band-Conditions Integration
- RTL-SDR → SoapySDR/rtl_power → real noise floor and signal data for band-conditions agent
- Pairs with OHB: OHB for internet data, RTL-SDR for local RF observation
- Evaluate at Phase 3 agent design time

### DVSwitch — YSF/WIRES-X Digital Voice Bridge
- **Hardware:** Dedicated second Wyse 3040 (do NOT share with AllStar node)
- **What:** Bridges AllStar to YSF (Yaesu System Fusion / WIRES-X) so FT-70D can use C4FM
  digital voice through the home node. Stack: Analog_Bridge + MMDVM_Bridge + YSFGateway.
- **Why separate box:** Failure isolation. If DVSwitch crashes, AllStar node stays up.
  Clean rebuild path — wipe the DVSwitch Wyse without touching the node.
- **No extra hardware needed:** Software AMBE via MD380 emulator (`useEmulator = true`).
  No ThumbDV dongle required.
- **Target rooms:** W6EK WIRES-X 62545 (2m room), NorCal Repeater Group #43847 (70cm)
- **Why needed:** W6EK C4FM repeater (440.575 Auburn) is not reachable from EDH apartment.
  DVSwitch is the only path to C4FM/WIRES-X without hitting that repeater directly.
- **Status:** DEFERRED — do not start until AllStar node 52547 has been stable for weeks.
  Architecture and docs confirmed 2026-06-23. Ready to build when sequencing is right.
- **Docs verified:** Analog_Bridge.ini, MMDVM_Bridge, YSFGateway — all on GitHub (DVSwitch org)

### Echolink on Node 52547
- ASL3 has built-in Echolink support
- Register at echolink.org (requires license verification, 1-2 days)
- Do NOT start until AllStar node is live and stable

### JS8Call Integration (Phase 3)
- Conversational digital mode, high AI coaching value
- High priority for Phase 3

### WSPR Integration (Phase 3)
- Propagation study, pairs with PSK Reporter data

### Accessibility Platform (separate project, concept only)
- Full SSB QSO capability for deaf/hard of hearing operators
- Real-time voice transcription, TTS transmit, AI procedural guidance
- Revisit after Ham AI assistant reaches v1
- Potential: ARRL accessibility grants

---

## Platform Notes

### ESP32 — Core Wireless Microcontroller Platform
- **What:** Tiny microcontroller (~$10) with built-in WiFi and Bluetooth. Programs like an Arduino.
  Runs standalone on 3.3V after programming — phone charger, battery, whatever.
- **Dev board:** ESP32 DevKit — USB for programming, then fully wireless
- **IDE:** Arduino IDE (same as Arduino) or MicroPython
- **Why it matters:** Solves "I want to monitor or control X wirelessly" for almost any project at $10/unit

**Ham radio use cases:**
- Remote antenna switch (multi-band, WiFi controlled)
- Automatic band decoder (reads flrig frequency, switches antenna)
- SWR/power meter with Grafana logging
- Rotator controller for satellite tracking
- CAT interface over WiFi
- Shack temperature/RF environment monitoring

**General use cases:**
- Weather station sensors (feeds InfluxDB/Grafana)
- Soil moisture, door/window sensors
- Smart power strip for shack
- IR blaster (control TV/AC over WiFi)
- Home Assistant automation nodes

**Learning path:** Start with weather station project — low stakes, useful output, teaches the
full ESP32 → WiFi → InfluxDB → Grafana pipeline that applies to every subsequent project.

### Dell Wyse 3040 — Core Compute Appliance Platform
- **What:** Fanless x86_64 thin client, ~$40 refurbished (Tech For Less, part 3YX35)
- **Runs:** Debian 13 Trixie, Python, Flask, ASL3, OctoPrint, Pi-hole, Grafana, InfluxDB
- **Rule:** One Wyse, one job. Keeps failure domains isolated and debugging simple.
- **At $40 each:** The answer to most "can these share a box" questions is just "buy another one."

**Current/planned Wyse inventory:**
- Node Wyse (10.0.0.196) — AllStar node 52547, ASL3
- Octopussy Wyse 1 — OctoPrint for Matt's Ender 3
- Octopussy Wyse 2 — OctoPrint for buddy's da Vinci 1.0A
- Wireless Shack Wyse — flrig + PipeWire/JACK + RNNoise audio filtering
- DVSwitch Wyse — Analog_Bridge + MMDVM_Bridge + YSFGateway (deferred until AllStar stable)
- OpenWebRX Wyse — RTL-SDR Blog V4 + OpenWebRX+ waterfall (in progress)
- Future: Pi-hole, Grafana/InfluxDB

---

## Not Planned

| Item | Reason |
|---|---|
| Digital voice (D-STAR, DMR, Fusion) integration in Ham AI | Separate ecosystem, untestable without hardware |
| DVSwitch shared with AllStar node | Failure domain isolation — separate Wyse required |
| Satellite operating | Separate design needed |
| SSTV | Different AI use case |
| MSK144 meteor scatter | Phase 4, evaluate demand |
| Winlink/VARA | Phase 4, evaluate demand |
| Mobile AllStar node (QYT) | Narrow use case, deferred |
| Portable hotspot (Pi/MMDVM or SharkRF) | Home node + FT-70DR covers realistic scenarios |
