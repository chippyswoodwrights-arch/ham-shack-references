# Ham Shack — Reference Files

Shared reference documents for ham radio builds on surplus hardware.
Covers Debian Linux setup on thin clients (Wyse 3040 and similar),
AllStar node builds, OpenWebRX, and US amateur radio license privileges.

Built and maintained by KO6NOI — El Dorado Hills, CA.

---

## What's Here

| File | Contents |
|---|---|
| [Debian_Install_Notes.md](Debian_Install_Notes.md) | Debian 13 Trixie install on Wyse 3040 and similar thin clients — boot USB, preseed, SSH, gotchas |
| [preseed.cfg](preseed.cfg) | Unattended Debian install config — SSH enabled, sudo, passwordless for build user |
| [openwebrx_build_notes.md](openwebrx_build_notes.md) | OpenWebRX + RTL-SDR V4 on Wyse 3040 — browser-based waterfall server |
| [licenses.md](licenses.md) | US amateur radio license privilege tables — Technician, General, Amateur Extra |
| [frequencies.md](frequencies.md) | Active frequencies — AllStar nodes, repeaters, HF digital modes |
| [vendors.md](vendors.md) | Cable and connector sourcing, prices, local store lists |
| [vendor_intel.md](vendor_intel.md) | Vendor policies and gotchas — HRO lifetime programming, Bridgecom warning, RT Systems |
| [projects_registry.md](projects_registry.md) | Project log — active builds, backlog, concepts |

---

## Hardware This Covers

- **Dell Wyse 3040** — Intel Atom X5, 2GB RAM, 8GB eMMC, fanless, ~$20-40 used on eBay
- **RTL-SDR Blog V4** — wideband receive dongle, HF through UHF, ~$35
- **SA818S UHF module** — 1W 440 MHz radio module for AllStar nodes, ~$10-15

All builds run Debian 13 Trixie (x86_64). No Raspberry Pi required.

---

## Companion Projects

- [Ham AI Assistant](https://github.com/KO6NOI/Ham-Radio) — AI operating assistant for WSJT-X / FT8, built on Flask + Anthropic API

---

## License

MIT — use freely, no warranty, verify regulatory content against primary sources before transmitting.
