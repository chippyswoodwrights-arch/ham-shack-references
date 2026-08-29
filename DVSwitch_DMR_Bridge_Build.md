# AllStar ↔ BrandMeister DMR Bridge — Build Procedure

Get an ASL3 node bridged to BrandMeister DMR, on a Wyse 3040 running Debian 13 Trixie,
**to the point of testing audio**. Follow the phases in order.

**Read [`DVSwitch_YSF_Notes.md`](DVSwitch_YSF_Notes.md) before you start.** This document is
the happy path; that one is the traps. Every `⚠` below points back to it.

## Not finished in this write-up (worked example, KO6NOI build, Aug 2026)

1. **Install method.** KO6NOI used a *patched* KD5FMU `ASL3_Multi_App_Install`. The cleaner
   Trixie path is `yo8aiv/install-dvswitch-trixie` (compiles from source). Phase 2 documents
   what was actually run; pick your install and adjust.
2. **Phase 7 audio levels are method, not values.** The node-side TX calibration
   (`simpleusb.conf` `txmixaset` / `preemphasis`, radio deviation) was still being worked
   when this was written, and an `AUDIO_USE_AGC` change on the RX path is pending. Phase 7
   tells you *how* to tune; your numbers are your own.
3. **Different hardware.** KO6NOI: CDM750 + SA818 node, FT-70D handheld. Your node radio and
   HT will differ — the node-audio section is yours to work, this is one worked example.

---

## Phase 0 — Prerequisites (start these first; they take days)

| # | Step | Notes |
|---|---|---|
| 0.1 | Apply for a **RadioID.net DMR ID** | Approval takes days. Your amateur license must show **"Verified"** on RadioID before BrandMeister will accept you. |
| 0.2 | Register a **BrandMeister SelfCare** account | `brandmeister.network` → Register. Needs 0.1 verified. 4-step wizard: Callsign + DMR ID → Email → Security → Review. |
| 0.3 | In SelfCare, set the **Hotspot Security password** | This is a **separate** value from your account login. It goes in `MMDVM_Bridge.ini [DMR Network] Password=`. ⚠ §4 |
| 0.4 | Pick your **BrandMeister master** | US: `3101`–`3104`. Address is `310x.master.brandmeister.network`, port `62031`. Pick from the dashboard → Masters. |
| 0.5 | Have a **working ASL3 node** | Separate build. This procedure assumes ASL3 is installed, your node is on the air, and audio to/from your node radio is already calibrated for analog. |

Neither 0.1 nor 0.2 is needed for YSF-only. For DMR they gate everything downstream — file
them the day you decide to build.

---

## Phase 1 — Read before you install

Read these DVSwitch groups.io wiki pages. **If the wiki will not load, stop and wait** —
do not proceed on installer source code alone (this cost the KO6NOI build days). ⚠ §0

- Wiki index — `https://dvswitch.groups.io/g/main/wiki`
- Bridge DMR to YSF Narrow — `.../wiki/6097`
- MMDVM_Bridge.ini reference — `.../wiki/5953`
- DVSwitch.ini reference — `.../wiki/5954`
- Analog_Bridge.ini reference — `.../wiki/6293`
- Analog_Bridge audio levels — `.../wiki/8463`
- AllStar Private node template — `.../wiki/38379`

Know going in: on Trixie/Bookworm the **bundled YSFGateway binary (20211108) is broken**
(⚠ §2). DMR does not use YSFGateway, so a DMR-only bridge can ignore it — but if you plan
to add YSF later, plan the binary downgrade now.

---

## Phase 2 — Install DVSwitch (co-located with ASL3)

DVSwitch runs on the **same box** as ASL3, everything on `127.0.0.1`. Do not put it on a
second machine — the `rpt.conf` footprint is required either way and a second box only adds
a network hop and an SSH credential.

KO6NOI ran a patched `KD5FMU/ASL3_Multi_App_Install` with the `-d` flag (DVSwitch only, no
AllScan/Supermon bundle). The installer's `install_dvswitch()` hard-codes the *Bookworm*
DVSwitch installer even on Trixie — the four `bookworm` → `trixie` references in that
function were patched before running. It pulled `analog_bridge`, `mmdvm_bridge`,
`md380-emu`, `ysfgateway`, `dvswitch-server`, and the PHP dashboard deps (62 packages,
~580 MB).

**Cleaner alternative for Trixie:** `yo8aiv/install-dvswitch-trixie` compiles
`MMDVM_Bridge`, `Analog_Bridge`, and `md380-emu` from source specifically for Debian 13.
Does not cover YSFGateway or the dashboard.

**Verify after install** — all services present, enabled, and idle (nothing configured yet):

```bash
systemctl is-enabled analog_bridge mmdvm_bridge md380-emu ysfgateway
systemctl is-active  analog_bridge mmdvm_bridge md380-emu
```

---

## Phase 3 — AllStar side (`/etc/asterisk/rpt.conf`)

**Back up `rpt.conf` first.**

### 3.1 The private USRP node stanza

Add a `[1999]` node (any unused private number). This is the canonical template from wiki
38379, **verbatim** — do not "improve" it:

```
[1999](node-main)
rxchannel = USRP/127.0.0.1:34001:32001  ; 34001 = port the bridge listens on (your TX)
                                        ; 32001 = port ASL listens on (your RX)
duplex = 0                              ; half duplex, no telemetry tones, no hang time
hangtime = 0
althangtime = 0
holdofftelem = 1
telemdefault = 0                        ; keep AllStar telemetry ("Allison") off the DMR side
telemdynamic = 0
linktolink = no
nounkeyct = 1
totime = 180000
```

### 3.2 The `[nodes]` entry

The stanza alone is not enough — add a local IAX registration:

```
1999 = radio@127.0.0.1:4569/1999,NONE
```

### 3.3 (Optional) DTMF talkgroup control

To switch mode/TG from the radio, add `[functions]` codes and a `custom/extensions.conf`
`[tgtune]` context. Source: `dvswitch.groups.io/g/allstarlink/wiki/22825`. Not required to
test audio — you can drive everything with `dvswitch.sh` from the shell first.

### 3.4 Restart and verify

```bash
sudo systemctl restart asterisk
sudo asterisk -rx 'rpt localnodes'      # should list your real node AND 1999
```

---

## Phase 4 — DVSwitch config

Three files under `/opt`. **Back up each before editing.** Values shown are the KO6NOI
worked example — substitute your own IDs, master, TG.

Notation: `<DMRID>` = your 7-digit RadioID number. `<ESSID>` = `<DMRID>` + a 2-digit suffix
(`01`, `02`, …) → 9 digits. BrandMeister hotspot convention; a single suffix digit makes an
invalid 8-digit ID.

### 4.1 `/opt/Analog_Bridge/Analog_Bridge.ini`

```
[GENERAL]
useEmulator     = true          ; software AMBE (md380-emu). File ships false for a
decoderFallBack = true          ; hardware DV3000 — true/true is the no-hardware combo.
emulatorAddress = 127.0.0.1:2470

[AMBE_AUDIO]
ambeMode     = DMR
gatewayDmrId = <DMRID>          ; 7-digit. ⚠ §3 — stock placeholder 1234567 segfaults on keyup
repeaterID   = <ESSID>          ; 9-digit (DMRID + 2-digit SSID)
txTg         = 9                ; default; you'll tune this at runtime
txTs         = 2
colorCode    = 1

[USRP]
txPort   = 32001               ; crossover with rpt.conf USRP/...:34001:32001 —
rxPort   = 34001               ; AB sends where ASL listens, listens where ASL sends
usrpAudio = AUDIO_UNITY        ; RX level (DMR → radio). Tune in Phase 7.
tlvAudio  = AUDIO_UNITY        ; TX level (radio → DMR). Tune in Phase 7.
```

### 4.2 `/opt/MMDVM_Bridge/MMDVM_Bridge.ini`

```
[General]
Id       = <ESSID>            ; 9-digit, must match Analog_Bridge repeaterID
Duplex   = 0

[Info]
RXFrequency = 433000000       ; RX and TX equal = simplex hotspot. Any legal freq;
TXFrequency = 433000000       ; unequal freqs make BM treat you as a duplex repeater.

[DMR]
Enable = 1                    ; ⚠ §3 — mode-on WITHOUT network-on segfaults this build

[DMR Network]
Enable   = 1                  ; must be 1 alongside [DMR] Enable
Address  = 3102.master.brandmeister.network
Port     = 62031
Password = <hotspot security password>   ; from SelfCare (0.3), NOT your account login
Slot1    = 0
Slot2    = 1
```

### 4.3 `/opt/MMDVM_Bridge/DVSwitch.ini`

```
[DMR]
txPort = 31100
rxPort = 31103
slot   = 2
hangTimerInFrames = 50        ; stock 0 = the node kerchunks once per word.
                              ; 50 = 3 s hang (file's own comment). Not on the dated wiki page.

; every fallbackID entry and UserID → <DMRID> (7-digit).
; stock placeholder 1234567 segfaults on keyup. ⚠ §3
```

### 4.4 Vocoder

```bash
systemctl is-active md380-emu
sudo ss -lunp | grep 2470      # md380-emu listening on 127.0.0.1:2470
```

---

## Phase 5 — Bring the bridge up

```bash
sudo systemctl restart analog_bridge
sudo systemctl restart mmdvm_bridge
sleep 5
/opt/MMDVM_Bridge/dvswitch.sh mode DMR
/opt/MMDVM_Bridge/dvswitch.sh tune 3100        # or your chosen TG
/opt/MMDVM_Bridge/dvswitch.sh show | grep -E 'ambe_mode|"tg"'
```

**Verify:**

- `grep 'Logged into the master successfully' /var/log/mmdvm/MMDVM_Bridge-$(date +%F).log`
- BrandMeister dashboard → your device page → status shows the repeater **verified**
- `dvswitch.sh show` → `ambe_mode: DMR`, correct `tg`

**Link the node to 1999** (temporary — does **not** survive an asterisk restart; make it
permanent later with a startup macro or DTMF):

```bash
sudo asterisk -rx 'rpt cmd <yournode> ilink 3 1999'
sudo asterisk -rx 'rpt lstats <yournode>'      # 1999 → OUT / ESTABLISHED
```

⚠ Any full `mmdvm_bridge` restart re-runs the autoresync script, which forces the bridge
back to **YSF** mode. After such a restart, re-run `dvswitch.sh mode DMR` + `tune`. An
`asterisk`-only restart does **not** disturb the bridge.

---

## Phase 6 — BrandMeister side

Dashboard → **DMR Devices** → your device → **Edit** → **Static Talkgroups**. Type your TG,
click the arrow to add it. A simplex hotspot shows **one** list (no Timeslot 1 / Timeslot 2
split — that's duplex-repeater only). BM confirms "Success"; traffic starts arriving within
seconds.

You do not strictly need a static to test — keying a TG dynamically subscribes you for
~15 min. A static just keeps it alive.

---

## Phase 7 — Test audio

### 7.1 RX (BrandMeister → your HT)

`dvswitch.sh tune 3100` (or any busy TG). Listen on the HT. You should hear network
traffic. If you hear kerchunks but no sustained voice → `hangTimerInFrames` is still 0
(Phase 4.3).

### 7.2 TX group call — round-trip proof

```bash
/opt/MMDVM_Bridge/dvswitch.sh tune 4000        # 4000 = disconnect
```

Key the HT ~2 s. BrandMeister sends a disconnect-confirmation transmission back down the
chain — you hear it on the HT = TX and RX both proven at the network layer. (Or key a busy
TG and check the BM dashboard "Last Heard" for your callsign.)

### 7.3 TX private call — the parrot

**The parrot needs a PRIVATE call. `dvswitch.sh tune 9990` sends a GROUP call and the
parrot ignores it.** Append `#`:

```bash
/opt/MMDVM_Bridge/dvswitch.sh tune 9990#        # tune must register mode=Private, not mode=Group
```

**Clear your static TG first** — a busy static floods the slot and a half-duplex bridge
can't key over it (dashboard → remove the static, or key TG 4000 to drop dynamics). Then
key a slow 15-count and listen for your own audio played back. Re-add the static after.

### 7.4 Level tuning (method — your values are your own)

On the `9990#` parrot, adjust live and judge by ear:

```bash
/opt/MMDVM_Bridge/dvswitch.sh tlvAudio AUDIO_USE_GAIN <n>   # TX  (radio → DMR)
/opt/MMDVM_Bridge/dvswitch.sh usrpAudio AUDIO_USE_GAIN <n>  # RX  (DMR → radio)
```

- **Robotic / metallic / choppy playback** = vocoder starvation, TX too quiet → raise `tlvGain`.
- **Crunchy / fuzzy on peaks** = clipping → lower it.
- Range 0.0–5.0, linear (3.0 = 3× unity).
- **If raising the gain makes it *worse*, stop.** The bottleneck is your node's own audio
  path (`simpleusb.conf` `txmixaset` / `preemphasis`, radio deviation), not DVSwitch. You
  can't buy node TX level with Analog_Bridge input gain — you just clip AB's output.
- A synthetic edge that survives at every clean gain setting is the software-vocoder floor.
  Hardware AMBE (DV3000) is cleaner.
- Objective target if you want one: BrandMeister's **Hoseline player VU meter** (pre-AGC:
  yellow < −20 dBm, green −20 to −3, red > −3). Normal ops meter around −24 (yellow), so
  dead-center green is not required. Needs Hoseline in a foreground browser with audio.
- **`AUDIO_USE_AGC`** (`usrpAGC` = threshold,slope,decay) levels a wide talker spread — one
  setting covers a hot net-control op and a quiet one. Tradeoffs: pumping on speech, raised
  noise floor / breathing in pauses. Test it deliberately, not layered onto other changes.
  Set live with `dvswitch.sh usrpAudio AUDIO_USE_AGC` then `dvswitch.sh usrpagc <thr> <slope>
  <decay>` — note the AGC subcommand is **lowercase `usrpagc`** (mixed-case `usrpAgc` errors).
  Tuning: lower threshold or raise slope for the quiet ops; raise decay (100 → 200–300 ms)
  to kill pumping/breathing. **AGC levels, it does not add loudness** — and `usrpGain` does
  **not** usefully trim post-AGC ([groups.io 74581152](https://dvswitch.groups.io/g/main/topic/74581152)).
  If the AGC'd output is too quiet overall, the fix is the node (`txmixaset`) and the HT
  knob, not DVSwitch. *KO6NOI worked example: `usrpAGC = -25,20,100`.*

**Bake the final values in:** edit `Analog_Bridge.ini [USRP]` — the `dvswitch.sh` commands
are runtime-only and revert on restart.

---

## Known-trap recap (all detailed in `DVSwitch_YSF_Notes.md`)

| Symptom | Cause | Section |
|---|---|---|
| Segfault on every keyup | Placeholder IDs (`1234567` / `123456789`) in the `.ini` files | §3 |
| Segfault at idle on start | `[DMR] Enable=1` without `[DMR Network] Enable=1` | §3 |
| Node kerchunks once per word | `DVSwitch.ini [DMR] hangTimerInFrames = 0` | §7 |
| DMR audio low / hiss pumps with voice | `usrpAudio` at `AUDIO_UNITY`; needs `AUDIO_USE_GAIN` | §7 |
| Own audio metallic/robotic on parrot | `tlvAudio` at `AUDIO_UNITY`; vocoder starved | §7 |
| Parrot never echoes | Group call to 9990 instead of `9990#` private call | §6 |
| TX keyups swallowed during a test | Busy static TG flooding the slot | §7 |
| Bridge reverts to YSF after a restart | `mmdvm-autoresync.sh` hard-coded to `mode YSF` | build status |
| `dvswitch.sh tune 99999` → "9999" | Broken bundled YSFGateway binary (YSF only) | §2 |
