# DVSwitch YSF Bridge — Install Gotchas

Adding DVSwitch (Analog_Bridge + MMDVM_Bridge + YSFGateway) to an ASL3 node on a Wyse 3040
to bridge AllStar ↔ Yaesu System Fusion. This is what cost days on the KO6NOI build.
**Read this and the wiki pages below before you install.** Two failures compounded there:
the wiki never got read up front, and a "maybe we need the older YSFGateway" hunch went
unspoken for a full day of testing.

---

## 0. Read the DVSwitch wiki first — it exists and it loads

The DVSwitch GitHub repos mark their docs "coming soon" and are useless. The real docs are
the groups.io wiki. If it's unreachable when you start, that's a **stop-and-wait**
condition — do not proceed on the installer's source code alone.

- Wiki index — https://dvswitch.groups.io/g/main/wiki
- Bridge DMR to YSF Narrow (canonical YSF example) — https://dvswitch.groups.io/g/main/wiki/Bridge-DMR-to-YSF-Narrow
- MMDVM_Bridge.ini reference — https://dvswitch.groups.io/g/main/wiki/5953
- DVSwitch.ini reference — https://dvswitch.groups.io/g/main/wiki/5954
- AllStar Private node template (`[1234]` USRP stanza) — https://dvswitch.groups.io/g/main/wiki/38379

---

## 1. DVSwitch's own YSF docs do NOT use YSFGateway

The canonical bridge points `[System Fusion Network]` in `MMDVM_Bridge.ini` straight at a
YSF reflector:

```
[System Fusion Network]
GatewayAddress=ysfreflector.dvswitch.org
GatewayPort=42166
```

No YSFGateway, no reflector list, no Wires-X menu, no local parrot — you connect to one
reflector defined in the `.ini`. Some installers (e.g. KD5FMU's `ASL3_Multi_App_Install`)
layer YSFGateway on top (`GatewayAddress=127.0.0.1:4200`) to add reflector switching via
`dvswitch.sh tune` and DTMF. That hybrid — an AllStar/USRP source feeding YSFGateway — is
**not documented anywhere by the DVSwitch project**. If you only need one reflector, skip
YSFGateway and follow the wiki example exactly.

---

## 2. The bundled YSFGateway binary is broken on Trixie and Bookworm

The current DVSwitch / KD5FMU install on Debian 13 Trixie pulls
**`YSFGateway-20211108` (Built 15:38:12 Sep 9 2025, GitID #8946594)**. It truncates
5-digit YSF reflector IDs to 4 digits internally, dropping the first digit:

```
dvswitch.sh tune 99999   →   gateway tries "9999"   →   "Invalid YSF reflector id/name"
```

- The bug is in the **binary**, not `dvswitch.sh` — the bridge sends `LinkYSF99999`
  correctly; the gateway mangles it on receipt.
- Name-based `Startup=` connections (looked up from `YSFHosts.txt`) still work — only the
  numeric `tune` path truncates. But this build also appears to drop the return-audio /
  parrot echo: on the KO6NOI build the forward path was packet-captured working (~400 YSF
  voice frames per key reaching the reflector) while nothing echoed back from any target,
  including YSFGateway's own local parrot.
- Multiple users confirm on both Trixie and Bookworm. The DVSwitch author is aware and
  unresponsive. Thread: https://dvswitch.groups.io/g/main/topic/120694643
- Community also reports the Bookworm/Trixie move broke NXDN and P25 node-list handling
  across DVSwitch generally. Buster + Debian 11 is reported to run YSF cleanly.

**Fix:** replace `/opt/YSFGateway/YSFGateway` with an older build — `YSFGateway-20200908`
or a Buster/Bullseye-era binary. Back up the existing binary first. Sources: the DVSwitch
Buster apt repo, G4KLX `YSFClients` releases, or copy from a working Buster/Bullseye box.

---

## 3. Placeholder IDs crash MMDVM_Bridge on the first YSF key

Stock configs ship with `Id=1234567` / `gatewayDmrId=1234567` / `repeaterID=123456789` /
`fallbackID=1234567`. With an AllStar/USRP source (no native digital subscriber ID),
keying up makes MMDVM_Bridge fail to resolve the placeholder ID, take a buggy fallback
path, over-read a buffer, and **segfault** — every key, before any audio leaves the box.

Put the real registered DMR/CCS7 ID (7 digits) in **all** of these:

| File | Field(s) |
|---|---|
| `Analog_Bridge.ini` | `gatewayDmrId` (7-digit ID), `repeaterID` (7-digit ID + 2-digit SSID, e.g. `01` → 9 digits — BrandMeister hotspot convention) |
| `DVSwitch.ini` | every `fallbackID` entry, and `UserID` |
| `YSFGateway.ini` | `[General] Id` |

After the fix, `Begin TX` in the MMDVM_Bridge log reads `src=<yourID>` and the crash
stops. Confirmed over ~18 key-ups.

---

## 4. Line up the DMR ID and BrandMeister account before the build

- RadioID.net DMR ID approval can take days.
- BrandMeister self-care registration needs the RadioID license status showing
  **"Verified"**.
- Neither is required for YSF-only operation, but if DMR is the goal, start both
  applications first — they gate everything downstream.
- BrandMeister issues a separate **Hotspot Security password** (set in SelfCare) that
  goes in `MMDVM_Bridge.ini [DMR Network] Password=` — not the same as the account login.

---

## 5. If you have a hunch about a version or a config, say it out loud

The "maybe we should be on the older YSFGateway" thought occurred on the KO6NOI build and
went unspoken for a full day of testing. A one-sentence "here's an alternative, here's the
trade" costs two minutes. Skipping it to keep momentum is exactly how the perfect storm
forms.

---

## KO6NOI build status (2026-08-29)

- **Crash fixed** — real DMR ID in all three config files, ~18 clean key-ups.
- **BrandMeister account created**; DMR left disabled (`Enable=0` both stanzas) — YSF only.
- **Forward audio path proven** by `tcpdump` — voice reaches the reflector end to end.
- **YSF echo / return audio unresolved** — blocked on the broken YSFGateway binary
  (section 2). Next step: downgrade the binary, then retest tune, connect prompt, and
  echo together.
