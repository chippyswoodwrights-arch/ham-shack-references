# People & Stations

*Key contacts, their gear, and what they're building.*
*Last updated: 2026-06-20*

---

## KO6NOI — Matt (Primary Operator)

- **License:** Technician (upgrade to General in progress)
- **Grid:** CM98 | **QTH:** El Dorado Hills / Auburn CA (zip 95762)
- **Email:** chippyswoodwrights@gmail.com
- **AllStar node:** 52547

### Station
| Gear | Details |
|---|---|
| HF rig | Kenwood TS-480SAT |
| HF interface | Rigblaster → COM7 → flrig → WSJT-X |
| Digital HT | Yaesu FT-70DR (C4FM / Yaesu Fusion ecosystem) |
| Node radio | Motorola CDM1250 → AllStar node 52547 (146.460 MHz, PL 127.3 Hz) |
| Node computer | Dell Wyse 3040 (10.0.0.196) |

### Active Projects
- Ham AI operating assistant (Python/Flask, Ham Radio folder)
- AllStar simplex node 52547 (AllStar Node folder)
- FT-70DR programming (RT Systems ADMS-70D + USB-57B, pending delivery)

### Technician HF Privileges (key segments)
See `licenses.md` for full privilege table.

---

## Fred

- **License:** TBD (studying / recently licensed)
- **QTH:** Westchester neighborhood, Los Angeles CA
- **Email:** reddogwoodworks@pacbell.net
- **Ecosystem:** DMR (Anytone 868UV or 878UV under consideration)

### Station (planned / in progress)
| Gear | Details |
|---|---|
| DMR HT | Anytone 868UV (~$140 at HRO) — DMR + analog FM, no GPS |
| Programming | HRO lifetime programming recommended (avoids Anytone CPS entirely) |

### Notes
- C4FM (Matt) and DMR (Fred) are incompatible protocols — cannot talk directly
- AllStar node 52547 bridges the gap via internet linking
- Fred should register at RadioID.net before any DMR programming
- Anytone CPS is Chinese software — if self-programming, run in VirtualBox VM with no network
- RT Systems does NOT make software for Anytone (CPS or CHIRP only)
- HRO nearest SoCal location: check hamradio.com/locations.cfm
- Talley nearest: 12976 Sandoval St, Santa Fe Springs CA 90670 — (562) 906-8000

---

## Key Repeaters & Nodes

| Call / Node | Freq | PL | Type | Network | Notes |
|---|---|---|---|---|---|
| W6EK | 145.430 rx / 144.830 tx | 162.2 Hz | Repeater | AllStar 51018 | Primary bridge target for node 52547 |
| K6IS | — | — | Node | AllStar 61464 | Secondary connection target |
| CARLA | — | — | Network | AllStar | Regional network target |
| KO6NOI | 146.460 (simplex) | 127.3 Hz | Node | AllStar 52547 | Matt's home node |

---

## Local Clubs & Nets

| Organization | Notes |
|---|---|
| SFARC | Sacramento Foothill Amateur Radio Club — Monday digital-nodes net, in-person help for node build |

---

## Digital Ecosystem Map

```
Matt (FT-70DR / C4FM) ──┐
                         ├── AllStar node 52547 (146.460 MHz) ── internet ── W6EK / CARLA / K6IS
Fred (Anytone / DMR) ───┘
```

C4FM and DMR cannot talk directly. Both connect to AllStar via RF → node → internet linking.
