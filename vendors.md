# Ham Radio Vendors & Sourcing Guide

*Applies to all projects: Ham AI assistant, AllStar node, shack builds, antenna work.*
*Verified 2026-06-20 by KO6NOI.*

---

## RF Coaxial Cable

**Buy from a professional RF distributor. Times Microwave LMR-400/LMR-240 is the standard.**

| Source | LMR-240 | LMR-400 | Type | Notes |
|---|---|---|---|---|
| Talley (walk-in) | $1.28/ft | $1.73/ft | List price | By the foot, no minimums, will-call |
| Ham Radio Outlet | $1.22/ft | $1.75/ft | Retail | Walk-in or online |
| DX Engineering | $1.23/ft | $1.75/ft | Online | Free ship over $99 |
| Gigaparts | ~$1.08/ft (ABR brand) | OOS | Online | Pre-cut lengths only, not Times Microwave |

*Talley list price beats online retailers on LMR-400 by $0.02/ft. Account holders get better pricing.*
*Prices verified 2026-06-20.*

---

## RF Connectors & Adapters

### Who Actually Makes Connectors

There are only a handful of real manufacturers — everything else is rebranded:

| Tier | Brand | Notes |
|---|---|---|
| Gold standard | **Amphenol** | Made in USA since WWII. Mil-spec capable. What repeater sites use. |
| Professional | **RF Industries** | San Diego. What commercial radio shops stock. Talley house brand. |
| Ham market | **Lands Precision (LP)** | Taiwanese. Sold at HRO. Acceptable for ham use. |
| Industrial | **TE Connectivity** | DigiKey/Mouser. Legitimate but overkill for most ham use. |
| Unbranded | **Amazon / no-name** | Chinese production. Inconsistent batch to batch. See decision rule below. |

### The Decision Rule

**Ask: can I easily get back to this connector if it fails?**

- **Yes → Amazon is fine.** Indoor adapters, barrel connectors, HT accessories, node builds,
  anything under 50W you can reach from a chair. $1–2 each in 5-packs.
- **No → buy name brand.** Anything up a mast, outdoors, exposed to weather, high power,
  or somewhere you'd need a ladder to reach. RF Industries or Amphenol. Buy from HRO, Talley, or DXE.

Failure modes on cheap connectors: center pin retention, plating corrosion, solder cup cracking
under vibration. These don't show up immediately — they show up as an intermittent 6 months later.

### Connector Prices

#### PL-259 Plugs (male, for LMR-400 / RG-8 / RG-213)

| Type | Talley List | Talley Acct | HRO | DigiKey | DXE 6-pack ea |
|---|---|---|---|---|---|
| Solder (RG-8/LMR-400) | $7.86 | $4.18 | $4.35 (LP) | ~$5.50 (Amphenol) | $5.67–$6.67 |
| Crimp (LMR-240/RG-8X) | $8.21 | $4.45 | — | — | — |
| Crimp (RG-58) | $5.97 | $3.74 | — | — | — |

#### SO-239 Jacks & Barrel Couplers (female)

| Item | HRO | GigaParts | DXE |
|---|---|---|---|
| Panel jack, 4-hole flange | — | $3.99 (Max-Gain) | $7.49 |
| Barrel coupler F-F (inline) | $2.99 | — | — |
| Barrel coupler F-F (bulkhead) | $7.99 | — | — |

#### Cross-Series Adapters

| Adapter | Best Source | Price | Notes |
|---|---|---|---|
| SO-239 ♀ to F ♂ (CDM1250) | Amazon 5-pack | ~$8–12 | Not stocked at ham retailers |
| BNC ♂ to SO-239 ♀ | DX Engineering | $6.00 ea (2-pack) | |
| N ♂ to SO-239 ♀ | DX Engineering | $32.59 (Amphenol) | Call DXE |

---

## Verified Vendors

### Talley LLC (professional RF distributor)
- **What they are:** Western US wireless infrastructure distributor. Supplies cell tower crews,
  commercial radio shops, wireless contractors. Andrew/CommScope/Times Microwave/RF Industries.
- **Public access:** Yes — list price, no account needed, no minimums, will-call at warehouse.
  Account holders get better pricing (roughly 20–25% off list) but they won't give ham operators
  contractor pricing without a legitimate business account.
- **Best for:** Cable (LMR-400/LMR-240), N-type connectors, commercial-grade hardware.
  PL-259/SO-239 stocked but secondary — call ahead to confirm.
- **Not for:** Cross-series adapters, specialty ham gear. Their web catalog is incomplete —
  call the counter, don't rely on the website.
- **KO6NOI area:** 11288 Pyrites Way, Gold River CA 95670 — (916) 273-1300 — Mon–Fri 6am–5pm
- **Fred area:** 12976 Sandoval St, Santa Fe Springs CA 90670 — (562) 906-8000 — Mon–Fri 6am–5pm
- **All locations:** talleycom.com

### Ham Radio Outlet (HRO)
- **What they are:** Dedicated ham radio retail chain. Walk-in public retail, no account.
- **Best for:** PL-259/SO-239/BNC/N-type/F-type connectors and adapters (full family in stock),
  cable, radios, accessories. Knowledgeable staff. Best walk-in certainty for connector variety.
- **Free lifetime programming** on any radio purchased from them. No locks, bring it back anytime.
- **KO6NOI area:** 4813 Auburn Blvd, Sacramento CA 95841 — (916) 659-7373 — Mon/Wed–Sat 10am–5:30pm
- **All locations:** hamradio.com/locations.cfm

### DX Engineering (online)
- **What they are:** Ham-focused online retailer. Strong connector and cable selection.
- **Best for:** Connectors (DXE house-brand crimp+solder PL-259 at ~$5.67 ea in 6-packs is
  the sweet spot), adapters, LMR cable. Free shipping over $99.
- **Website:** dxengineering.com

### GigaParts (online)
- **What they are:** Ham-focused online retailer.
- **Best for:** SO-239 panel jacks (Max-Gain Systems at $3.99 ea is best value), pre-cut cable.
- **LMR-400:** Backordered. Cable is ABR Industries brand (LMR equivalent, not Times Microwave).
- **Website:** gigaparts.com

### Bridgecom
- **WARNING: avoid.** Locks radios after programming — must return to them for any changes.
  Anti-operator practice. Buy elsewhere.

---

## Ruled Out (with reasons)

| Vendor | Why |
|---|---|
| Graybar | B2B contractor focus, most inventory shows OOS for public, not worth the drive |
| RFC Wireless (Gold River) | B2B systems integrator only — no parts counter, no walk-in |
| Sacramento Electronic Supply | Bulk commercial only — cable in 500–1000ft spools, $25 minimum |
| Fry's Electronics | All locations closed permanently 2021 |
| Best Buy / Home Depot / Lowe's | F-type TV coax only, no ham/RF connectors |
| Universal Radio | Mostly out of stock |

---

*Full local sourcing guide with Fred's area: RF_Parts_Local_Sourcing_Guide.md*
*Prices verified 2026-06-20. Call ahead to confirm stock before driving.*
