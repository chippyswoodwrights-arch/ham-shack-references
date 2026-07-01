# Ham Shack Hardware Inventory

*Cross-project reference. Update when gear arrives, is sold, or is repurposed.*
*Last updated: 2026-06-20 by KO6NOI*

---

## Radios

| Radio | Band | Mode | Status | Project | Notes |
|---|---|---|---|---|---|
| Kenwood TS-480SAT | HF | SSB/CW/Data | In service | Ham AI shack radio | COM7 → flrig → WSJT-X |
| Yaesu FT-70DR | VHF/UHF | FM / C4FM | In service | Personal HT | RT Systems programming pending |
| Motorola CDM1250 | VHF | FM | In service | AllStar node | 146.460 MHz, PL 127.3 Hz |
| Baofeng UV-5R | VHF/UHF | FM | On hand | Misc / lending | Programmed or pending |
| Baofeng UV-5RM Plus | VHF/UHF | FM | On hand | Misc / lending | Programmed or pending |

---

## Computers

| Device | OS | Status | Project | Notes |
|---|---|---|---|---|
| Main shack PC | Windows 11 | In service | Ham AI / flrig / WSJT-X | COM7 to TS-480SAT |
| Dell Wyse 3040 | Debian 13 Trixie | In service | AllStar node | IP 10.0.0.196, user=matt, SSH via wyse_key |
| Dell Wyse 3040 | Debian 13 Trixie | In service | OctoPrint (Ender 3) | IP 10.0.0.170, user=matt, SSH via wyse_octoprint |
| Dell Wyse 3040 | Debian 13 Trixie | In service | Pi-hole | IP 10.0.0.191, user=matt, admin at http://10.0.0.191/admin |
| Dell OptiPlex 3070 Micro | — | On hand, not set up | Home security — Frigate/Home Assistant | i3-9100T, 8GB RAM, no drive — needs power adapter + SSD |

---

## Interface & Audio

| Item | Status | Project | Notes |
|---|---|---|---|
| Rigblaster (model TBD) | In service | Ham AI | Audio bridge TS-480SAT ↔ PC |
| RIM-Maxtrac USB Lite (Maxtrac edition) | Shipped | AllStar node | CDM1250 accessory port → Wyse USB |
| DisplayPort to HDMI adapter (Plugable) | On hand | AllStar node | Wyse 3040 has DP only |

---

## Antennas

| Antenna | Band | Status | Location | Notes |
|---|---|---|---|---|
| Tunable copper J-pole | VHF 2m | On hand | To be mounted | AllStar node / simplex 146.460 |

---

## Power

| Item | Status | Project | Notes |
|---|---|---|---|
| Powerwerx SS-30 (30A 12V DC supply) | In service | Shack | Powers CDM1250 and accessories |

---

## Cables & Adapters

| Item | Status | Notes |
|---|---|---|
| PL-259 female to F male barrel adapter | On hand (repurposed) | CDM1250 antenna connector workaround |
| Snap-on ferrite clamps (2x 20-piece assorted) | Delivered | RF interference suppression |
| CDM1250 programming cable (BlueMax RKN4081, FTDI) | Shipped | COM10 when connected |

---

## Programming Hardware

| Item | Status | Notes |
|---|---|---|
| RT Systems USB-57B cable | Delivered | FT-70DR programming — 3.5mm TRRS to SP/MIC jack |

---

## Software Licenses / Purchased Software

| Software | License | Notes |
|---|---|---|
| RT Systems ADMS-70D | Delivered — ready to use | FT-70DR programming software — radio-specific, no universal package |

---

## On Order / ETA (Hardware Builds)

| Item | Source | ETA | Cost | Project |
|---|---|---|---|---|
| Dell OptiPlex 3070 Micro (i3-9100T, 8GB, no drive) | eBay — harddrivesonly | Delivered | $74.99 | Home security — Frigate/Home Assistant |
| Dell Wyse 3040 x2 | eBay — madman_parts | Delivered | $19.99 ea | OctoPrint (done) + Pi-hole (done) |
| RTL-SDR Blog V4 | — | Delivered | — | OpenWebRX waterfall |
| SKR Mini E3 V3.0 | BigTreeTech | Arriving Jun 25 | — | Ender 3 board upgrade — BLTouch native |
| BLTouch V3.1 (Antclabs) | Antclabs | Ordered | ~$40 | Ender 3 auto bed leveler — CR Touch cancelled (incompatible with SKR) |
| Silicone bed legs + springs | — | Arriving Jun 27 | — | Ender 3 bed stability fix |

**Still needed:**
- OptiPlex 3070 Micro power adapter: 65W, 19.5V, 4.5x3.0mm barrel — ~$10-15 Amazon
- Wyse 3040 power adapter x2: 5V 3A barrel — check junk drawer first, ~$10 Amazon if needed

---

## Spare Parts — Home Security / ThinkCentre / OptiPlex Builds

| Item | Spec | Qty | Destination |
|---|---|---|---|
| RAM DDR4 SO-DIMM | 16GB, 2666 MHz | 1 | Build 1 |
| RAM DDR4 SO-DIMM | 8GB, 2666 MHz | 1 | Build 1 (run with 16GB for 24GB total) |
| RAM DDR4 SO-DIMM | 4GB, 3200 MHz | 1 | Spare |
| Intel 660p SSD | 512GB M.2 2280 NVMe | 1 | Build 1 or 2 (favors OptiPlex) |
| Kioxia KBG40ZNS256G SSD | 256GB M.2 2230 NVMe | 1 | Build 1 or 2 (favors ThinkCentre M) |
| Intel AX201NGW | WiFi 6 + BT 5.2, M.2 2230 | 1 | Build 1 |
| Intel AX200NGW | WiFi 6 + BT 5.0, M.2 2230 | 1 | Build 2 |
| Wyze V3 cameras | 1080p, IR night vision, WiFi | 2 | Home security — shop |

**Status:** Need 1-2 ThinkCentre M or OptiPlex micro units from Tech For Less (~$40-80 each).
RAM covers one full build (24GB). Second build needs RAM — may ship with some installed.
Both builds destined for home security project (Frigate + Home Assistant).

---

## Notes

- **⛔ CDM1250 POWER FLOOR — CHECK THIS BEFORE BUYING ANYTHING:** KKD variant = 25W minimum floor. KHD variant = 1W minimum. Home simplex AllStar node needs KHD. Check the model number label on day one — before buying a RIM-Maxtrac, programming cable, or any other supporting hardware. KKD on a home node means random strangers within 10 miles can key up your node with zero authentication. Learned the hard way.
- **CDM1250 antenna port:** F connector (cable TV type) — NOT SO-239. Use PL-259 female to F male barrel adapter.
- **Wyse 3040 SSH:** `ssh -i ~/.ssh/wyse_key matt@10.0.0.196`
- **CDM programming:** COM10, BlueMax FTDI cable, CPS R06.12.05
- **TS-480SAT CAT:** COM7, flrig XML-RPC port 12345
