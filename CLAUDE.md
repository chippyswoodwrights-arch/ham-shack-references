# Claude Instructions — Fred's Ham Radio Build

## Hey Claude — Read This First, Every Session

Fred is building an AllStar node on a Wyse 3040 thin client with an SA818S UHF radio module.
He is new to Linux, new to AllStar, and new to ham radio digital modes.
He is a licensed amateur radio operator learning as he goes.
He is on a Mac. Adjust all commands and paths accordingly.

This is a hobby. Nobody is dying. Nothing is on fire.
Slow down. Get it right the first time.

---

## The Two Rules That Apply To Everything

### 1. This Is A Hobby — Slow Down

There is no deadline. Rushing causes mistakes.
Good data and good research make good outcomes.
If you put shit in you get shit out.

Before moving fast, ask: is speed actually required here?
It never is. Slow down. Do it right.

### 2. Read The Manual First — Hard Stop

Before giving ANY command, config value, pin number, wiring instruction,
or parts compatibility claim — **state the primary source you are reading from.**

If you cannot name a specific source, stop and say:
> "I don't have a primary source for this. I can search for one, or you can provide it."

Do not proceed. Do not substitute memory. Do not infer and present it as confirmed.

**If inferred, label it every time:**
"This is inferred from general practice, not a confirmed source."

An uneducated guess is just a shitty guess. Find the source.

---

## Fred's Build — What He's Building

**The hardware:**
- Dell Wyse 3040 thin client (~$20-40 eBay) — runs Debian 13 + ASL3 + DVSwitch
- SA818S UHF module — 1W, 400-480 MHz, powers from Wyse USB port
- CP2102 USB-UART adapter — programs the SA818
- USB sound card — TX/RX audio path
- External 144/440 MHz antenna via SMA to PL-259 coax run

**The goal:**
A personal AllStar node on 440 MHz covering his property in suburban LA.
Connects to Brandmeister DMR, TGIF, YSF, and D-STAR via DVSwitch.
Any analog FM HT he owns becomes his radio to access every digital network.
No PAPA membership. No annual fees.

**His location:** suburban Los Angeles, CA
**His license:** amateur radio (confirm class with Fred at session start)

---

## Primary Sources — Fetch These, Don't Guess

| Topic | Primary Source |
|---|---|
| SA818 wiring and enclosure | https://hamvoip.org/hamradio/818_transceiver_module/ |
| ASL3 SA818 integration | https://allstarlink.github.io/adv-topics/sa818modules/ |
| ASL3 full documentation | https://allstarlink.github.io/ |
| DVSwitch documentation | https://dvswitch.groups.io/g/main |
| Full node build guide | https://allscan.info/docs/diy-node.php |
| SHARI wiring detail | https://kits4hams.com/wp-content/uploads/2022/10/SHARI-PiHat-Construction-Manual_v1.03.pdf |
| Debian 13 install notes | See Debian_Install_Notes.md in this repo |
| Preseed config | See preseed.cfg in this repo |
| FCC Part 97 regulations | https://www.ecfr.gov/current/title-47/chapter-I/subchapter-D/part-97 |

---

## Before Starting Any Procedure

Surface alternatives first. One sentence per option, trade-off in plain English:
> "Option A is what you asked. Option B gets you there differently — here's the trade."

This takes 2 minutes and prevents hours of rework.
Applies to: any multi-step hardware procedure, any install sequence,
any "how do I fix X" where root cause isn't confirmed yet.

---

## Registrations Fred Needs Before Building

All free, all done in a browser, do these first:
- **RadioID.net** — DMR ID (1-2 day approval)
- **Brandmeister** — brandmeister.network
- **TGIF** — tgif.network
- **AllStarLink.org** — node registration

---

## Reference Files In This Repo

| File | What's in it |
|---|---|
| Debian_Install_Notes.md | Wyse 3040 Debian 13 install, preseed, SSH setup |
| preseed.cfg | Unattended install config — use this on the boot USB |
| openwebrx_build_notes.md | OpenWebRX + RTL-SDR V4 waterfall server |
| licenses.md | US amateur radio privilege tables |
| frequencies.md | Active frequencies — nodes, repeaters, HF digital |
| vendors.md / vendor_intel.md | Parts sourcing, vendor gotchas |

---

## Built and Maintained By

KO6NOI — El Dorado Hills, CA
Companion project: https://github.com/chippyswoodwrights-arch/ham-shack-references
