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

## 6. DMR side — the BrandMeister parrot needs a PRIVATE call (`dvswitch.sh tune 9990#`)

`dvswitch.sh tune 9990` sends a **group call** to talkgroup 9990. The BrandMeister parrot
(9990, or the regional `xxx997` — `310997`/`311997` for the USA) only answers a **private
call**. A group call to 9990 goes nowhere and you get no echo, which looks exactly like a
broken return path and will send you chasing binaries, levels, and timeslots for days.

**Fix: append `#` to the ID → `dvswitch.sh tune 9990#`.** `dvswitch.sh` passes the string
straight through (`tune()` just does `remoteControlCommand "txTg=$1"`); Analog_Bridge itself
reads the trailing `#` as the private-call flag. Verify with `dvswitch.sh show` / the
MMDVM_Bridge log — the tune must register as `mode=Private`, not `mode=Group`. On the KO6NOI
build an 11-second key then echoed back an 11.3-second playback, 0% loss, first try.

Note this is only reachable *because* Analog_Bridge supports the `#` flag. An ASL/USRP
source has no native DMR call-type bit; without the `#` every synthesized frame is a group
call. Sources: node-ventures.com/digital, billmongan.com `digital_link`,
dvswitch.groups.io topic 69233738, BM wiki `index.php/Parrot`.

---

## 7. DMR bridge — the two non-default settings that made RX usable

With an AllStar/USRP source feeding the DMR side, stock defaults give kerchunk-per-syllable
audio that's also too quiet and hissy. Two changes fixed it on the KO6NOI build:

| File | Setting | Stock | Set to | Why |
|---|---|---|---|---|
| `DVSwitch.ini` `[DMR]` | `hangTimerInFrames` | `0` | `50` | At 0, MMDVM_Bridge tears down the TLV stream on every gap between DMR superframes, so the AllStar node never latches its receiver and kerchunks once per word. 50 = 3 s hang (the file's own comment says so). |
| `Analog_Bridge.ini` `[USRP]` | `usrpAudio` / `usrpAGC` (RX: DMR→radio) | `AUDIO_UNITY` | `AUDIO_USE_AGC` / `-25,20,100` | Fixed gain (`AUDIO_USE_GAIN` ~2.5–3.0) works but no one value serves both a hot net-control op and a quiet one — the hot one clips at the `usrpGain` stage. AGC (`usrpAGC` = threshold,slope,decay) levels the spread. **It levels, it does not add loudness** — `usrpGain` doesn't trim post-AGC ([groups.io 74581152](https://dvswitch.groups.io/g/main/topic/74581152)); overall level comes from the node (`txmixaset`) + HT knob. `dvswitch.sh` subcommand is lowercase `usrpagc`. |
| `Analog_Bridge.ini` `[USRP]` | `tlvAudio` / `tlvGain` (TX: radio→DMR) | `AUDIO_UNITY` / `0.35` (inert) | `AUDIO_USE_GAIN` / `4.0` | At unity the vocoder is starved — your own audio comes back metallic/robotic off the parrot. Walk it up on the `9990#` parrot by ear: clearer each step to 4.0; ~4.5 starts to fuzz on peaks (clip onset). A synthetic edge that survives at any gain is the OP25/md380-emu software-vocoder floor, not a level problem. |

Range on both gain knobs is 0.0–5.0 (linear multiplier; 3.0 = 3× unity). `AUDIO_USE_AGC` +
`usrpAGC` is the RX fallback for hot/quiet talker spread.

**Tuning method:** clear your static TGs first (a busy one floods the slot and swallows
your keyups on a half-duplex bridge), `dvswitch.sh tune 9990#` for the parrot,
`dvswitch.sh tlvAudio AUDIO_USE_GAIN <n>` to change the TX gain live, key a slow count,
listen, repeat. Bake the final value into `Analog_Bridge.ini` — the `dvswitch.sh` command
is runtime-only and reverts on restart. BrandMeister's Hoseline player also has a
per-transmission VU meter (pre-AGC: yellow <-20 dBm / green -20 to -3 / red >-3) if you
want an objective target — though normal ops meter around -24 (yellow), so dead-center
green isn't required.

Everything else (`[1999]` rpt.conf stanza, `Jitter=360`, all the `AUDIO_UNITY` shipped
defaults) matched the wiki references exactly — do not go changing those.

---

## KO6NOI build status (2026-08-29)

- **MMDVM_Bridge crash fixed** — real DMR ID in all three config files (section 3).
- **DMR bridge working both directions** — RX (good audio after the section 7 settings),
  TX group call (proven via TG 4000 round trip), TX private call (9990# parrot echo,
  section 6).
- **YSF echo / return audio still unresolved** — separate problem from the DMR parrot;
  blocked on the broken YSFGateway binary (section 2). Downgrade to `YSFGateway-20200908`,
  then retest tune, connect prompt, and echo.
- **`mmdvm-autoresync.sh` is hardcoded to `dvswitch.sh mode YSF`** — any mmdvm_bridge
  restart flips the bridge back to YSF; re-run `dvswitch.sh mode DMR` + `tune <tg>` after.
  Make it mode-aware before DMR is daily-use.
