# Frequencies in Use — KO6NOI Station

*Operational reference for all active frequencies across all projects.*
*Last updated: 2026-06-23*

---

## AllStar Node 52547

| Use | Freq | PL Tone | Mode | Notes |
|---|---|---|---|---|
| Simplex node (TX/RX) | 146.460 MHz | 127.3 Hz | FM | CDM1250 → AllStar node 52547 |

**PL tone selection rationale:** 127.3 Hz — verified clean in El Dorado Hills area via NARCC
database 2026-06-14. 100.0 Hz avoided (used by KD6CQ locally). Confirm with SFARC Monday net.

---

## Repeater Coverage from El Dorado Hills Apartment (CM98)

*Last verified: 2026-06-23. Source: EDH_RepeaterBook.csv + W6EK website. Ask-and-answered — do not re-litigate in future sessions.*

### Confirmed Reachable (verified 2026-06-23 by KO6NOI)

| Call | RX Freq | Offset | PL | Location | Digital | Notes |
|---|---|---|---|---|---|---|
| N6QDY | 147.255 MHz | +0.6 | 123.0 Hz | El Dorado Hills | None | Local |
| KD6CQ | 145.370 MHz | -0.6 | 100.0 Hz | El Dorado Hills | None | 100.0 Hz PL avoided for node simplex |
| K6IS | 145.190 MHz | -0.6 | 162.2 Hz | Folsom | None | AllStar 61464 |
| W6EK | 145.430 MHz | -0.6 | 162.2 Hz | Auburn | None | AllStar 51018, Echolink 4128, WIRES-X 62545 — primary club repeater |
| AG6AU | 147.975 MHz | -0.6 | 82.5 Hz | El Dorado Hills (low-level) | None | Covers EDH Blvd / Silva Valley Pkwy / Town Center with HT. Linked to AG6AU 147.825. Echolink only (via 147.825). Not in RepeaterBook. |

### Unconfirmed / Needs Verification

| Call | RX Freq | Offset | PL | Location | Digital | Notes |
|---|---|---|---|---|---|---|
| AG6AU | 147.825 MHz | -0.6 | 82.5 Hz | El Dorado Hills (high-level) | None | Echolink only. Linked to 147.975 low-level site. No AllStar. |
| KA6GWY | 146.805 MHz | -0.6 | 123.0 Hz | Placerville | None | Unconfirmed from apartment |

### Cannot Reach from Apartment

| Call | RX Freq | Offset | PL | Location | Digital | Notes |
|---|---|---|---|---|---|---|
| W6EK | 440.575 MHz | +5.0 | none | Auburn | C4FM/Fusion, WIRES-X #43847 | C4FM repeater — not reachable direct |
| W6SAR | 145.270 MHz | -0.6 | 156.7 Hz | Auburn | None | Search & rescue repeater |
| N6QDY | 443.925 MHz | +5.0 | 179.9 Hz | El Dorado Hills | None | 70cm — unconfirmed, likely close |
| AG6AU | 441.725 MHz | +5.0 | 82.5 Hz | Lotus | None | 70cm |

### Implication for Node and Digital Planning
- **AllStar node (52547)** on 146.460 simplex is the path to W6EK and the rest of the network — confirmed correct approach.
- **W6EK C4FM repeater (440.575)** cannot be hit from apartment — C4FM digital voice requires an alternative path.
- **DVSwitch** is technically viable (confirmed 2026-06-23): bridges AllStar to YSF via Analog_Bridge + MMDVM_Bridge + YSFGateway, no hardware dongle needed. ARCHITECTURE DECISION: runs on a dedicated second Wyse 3040, NOT on the AllStar node. Keeps failure domains separate — DVSwitch can be wiped/rebuilt without touching the node. DEFERRED — do not pursue until AllStar node has been stable for weeks.

---

## Repeaters in Regular Use

| Call | RX Freq | TX Freq | PL | Network | Notes |
|---|---|---|---|---|---|
| W6EK | 145.430 MHz | 144.830 MHz | 162.2 Hz | AllStar 51018, Echolink 4128, WIRES-X 62545 | Primary club repeater — via AllStar node |
| K6IS | 145.190 MHz | 144.590 MHz | 162.2 Hz | AllStar 61464 | Secondary target — confirm range |

---

## HF Digital (FT8)

| Band | Freq | Technician Legal? | Conditions |
|---|---|---|---|
| 80m | 3.573 MHz | Yes | Evening/night, regional |
| 40m | 7.074 MHz | Yes | Afternoon/evening, regional to DX |
| 15m | 21.074 MHz | Yes | Daytime, DX when solar conditions allow |
| 10m | 28.074 MHz | Yes | Daytime, excellent DX when band open |
| 20m | 14.074 MHz | No (General+) | Best overall DX band |

---

## HF CW / Voice (reference)

| Band | Technician CW/Data Segment | Technician Voice? |
|---|---|---|
| 80m | 3.525–3.600 MHz | No |
| 40m | 7.025–7.125 MHz | No |
| 15m | 21.025–21.200 MHz | No |
| 10m | 28.000–28.500 MHz | Yes (28.300–28.500 MHz phone) |

---

## FT-70DR Programmed Channels

*See FT-70DR programming files for full channel list.*

| Ch | Name | Freq | PL | Mode | Notes |
|---|---|---|---|---|---|
| 26 | MY-NODE | 146.460 MHz | 127.3 Hz | FM | AllStar node 52547 — tone to be confirmed once node live |

*Full channel plan: KO6NOI_FT70D_full.csv and .FT70D programmer file on desktop.*

---

## fldigi Monitoring Frequencies (reference)

| Mode | Freq | Notes |
|---|---|---|
| CW nets | 7.110 MHz | Confirmed active 2026-06-12 — fldigi pipeline verified here |
| PSK31 | 14.070 MHz | General/Extra only — reference for future upgrade |
| RTTY | 14.080–14.100 MHz | General/Extra only — contests |
