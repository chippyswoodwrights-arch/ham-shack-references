# Useful Sites & Resources

*Shared reference — links worth remembering, dug up during sessions. Not build notes;
those stay in their own project files. Add here when something's worth finding again.*

---

## Blogs / News

- **[hamradio.my](https://hamradio.my/)** — Malaysian ham radio blog (9M2PJU). Mix of SDR
  reviews, satellite tracking tools, DIY/Android projects, field-ops and emergency-comms
  content. Worth periodic browsing, not a one-time read. Found via the QRZ/logging research
  session, 2026-09-11.

---

## DIY / Hardware Projects (candidates, not yet built)

- **[kv4p HT](https://github.com/VanceVagell/kv4p-ht)** — open-source handheld transceiver:
  ESP32 board + Android app turns a spare phone into a real VHF/UHF HT. PCB files + 3D-print
  case included, actively developed (807 stars as of 2026-09-11). Flagged because we have
  spare Android phones on hand.
  - **[kv4p-usrp](https://github.com/dadecoza/kv4p-usrp)** — bridges kv4p HT into an
    AllStarLink node over USRP, the same protocol our existing node (52547, Wyse 3040,
    10.0.0.90) already speaks. Direct tie-in, not a separate ecosystem.
  - **[kv4p-ht-dmr](https://github.com/ceilingduster/kv4p-ht-dmr)** — DMR-capable fork.
  - Not yet priced out (ESP32 board + RF front-end cost unconfirmed) or built. Candidate
    project, evaluate BOM before committing.

---

## Logging / QSL Confirmation Ecosystem (reference, researched 2026-09-11)

- **[QRZ subscriptions](https://shop.qrz.com/collections/subscriptions)** — XML Logbook Data
  tier ($35.95/yr) is the one needed for 3rd-party logbook push (Log4OM → QRZ) and XML
  callbook lookups. Premium/Platinum add QRZ's own award certificates.
- **[QRZ Awards](https://www.qrz.com/awards)** — QRZ runs its own award program, independent
  of ARRL (US Awards, DX World, Grid Squared, etc.), based on QRZ Logbook contents.
- **[Wavelog](https://www.wavelog.org/)** ([GitHub](https://github.com/wavelog/wavelog)) —
  self-hosted, web-based logger, 2024 fork of Cloudlog, fast-growing (532 stars in ~20
  months vs. Cloudlog's 573 over a much longer life). OpenHamClock already has a built-in
  Wavelog/QRZ push integration (`logsync.js`) for OHC's own native logbook — separate from
  Log4OM, would need its own wiring if we ever log through OHC directly.
- **[Cloudlog](https://github.com/magicbug/Cloudlog)** — the older, still-active
  self-hosted logger Wavelog forked from.
- Standard practice confirmed: hams commonly push the same QSO to LoTW + eQSL + QRZ
  simultaneously — no single service has universal reach, so it's additive, not either/or.

---

## Open Items

- DMR/YSF operators reportedly log almost exclusively via QRZ (Matt's own observation,
  2026-09-11) — Log4OM's QRZ push is not currently enabled. Needs the $35.95/yr QRZ XML
  tier + turning on Log4OM's QRZ integration. Not started.
