# AllStar Node Build — Reference Documentation

## Shared Ham Shack References (C:\Users\mattc\OneDrive\Desktop\Ham Shack\)
- vendors.md — cable/connector prices, sourcing decision rule, verified vendors with locations
- vendor_intel.md — HRO lifetime programming, Bridgecom warning, RT Systems, BlueMax, gotchas
- hardware_inventory.md — every radio, computer, cable and adapter across all projects
- people_and_stations.md — Matt, Fred, repeaters, nodes, digital ecosystem map
- licenses.md — Technician/General privilege tables, FT8 frequencies, upgrade path
- frequencies.md — all active frequencies: node, repeaters, HF digital, fldigi monitoring
- projects_registry.md — all projects: active, backlogged, concept, not planned

## Vendors & Sourcing
- Verified vendors, cable/connector prices, decision rules: ..\Ham Shack\vendors.md (shared — verified 2026-06-20)

## Radio Hardware
- Motorola CDM1250 CPS version R06.12.05 (last wideband-capable version)
  Release notes: ProSeries_CPS_R06.12.05_Notes.pdf (saved locally, verified 2026-06-13)
  Software: CPS_R06.12.05_HVN9025.zip (downloaded 2026-06-15 from W9CR Waris wiki — free, no account needed)
  Direct URL: https://wiki.w9cr.net/images/a/a1/HVN9025_v6.12.05.zip
  Official Motorola Solutions release dated 2012-02-10. Key enhancement: Windows 7 64-bit support.
  ⚠️ Windows 10/11 fix: after installing CPS, run commsbepx64_setup.exe from CPS program folder
  Fix details: https://kd8tnv.com/cps-error.html
  BlueMax49ers does NOT include CPS with cable — download separately from W9CR wiki.
  DO NOT use R06.12.09 — does not support wideband (25 kHz) channels.
- Motorola CDM1250 accessory connector pinout: see CPS manual / Repeater-Builder CDM series documentation
- RIM-Maxtrac USB Lite (Maxtrac edition) product page: https://repeater-builder.com/products/usb-rim-lite.html
- RIM Lite v2 schematic: RB_RIM_Lite_v2.pdf (saved locally 2026-06-13)
- RIM Lite v2 layout: RB_RIM_Lite_v2_Layout.pdf (saved locally 2026-06-13 — physical board layout only, no new wiring info)

  ⚠️ NOTE: The Lite uses a DE-9 (DB-9) connector, NOT the DB-25 of the full RIM.
  The full RIM doc (rimdocs.pdf) does NOT apply to the Lite's radio connector pinout.

  RIM Lite v2 DE-9 radio connector (X2) — verified against schematic 2026-06-13:
  | Pin | Signal | Notes |
  |-----|--------|-------|
  | 1   | TX audio — tone/CTCSS [txmixa] | Connect to radio mic input |
  | 2   | TX audio — voice [txmixb] | Low-pass filtered; connect to direct modulator input |
  | 3   | COS from RX | COS-only OR CTCSS/COS composite (Note B) |
  | 4   | CTCSS from decoder | CTCSS-only input |
  | 5   | PTT to TX | |
  | 6   | Discriminator (flat) audio from RX | |
  | 7   | PC_OK output | Low when PC comms OK |
  | 8   | Ground | |
  | 9   | Ground | |

  Note B (from schematic): Pin 3 accepts COS-only or CTCSS/COS composite.
  Pin 4 is CTCSS-only. Using both separately allows software squelch mode selection in ASL3.

  Hookup sheet — RIM Lite to Maxtrac/GM300: RB_RIM_Lite_to_Maxtrac.pdf
  URL: https://www.repeater-builder.com/products/RIM_pdfs/RB_RIM_Lite_to_Maxtrac.pdf
  Fetched and verified 2026-06-13. Complete wiring table:

  | RIM Lite (DE-9) | Signal | Maxtrac 16-pin | CPS Programming Required |
  |---|---|---|---|
  | 1 | Aux TX Audio (tone) | 5 | Flat TX Audio |
  | 2 | Main TX Audio (voice) | 2 | Voice TX Audio / Mic Input |
  | 3 | COS in | 12 | Program: Active High COS |
  | 4 | CTCSS in | 8 | Program: Active High COS & CTCSS |
  | 5 | PTT | 3 | (PTT input — confirmed) |
  | 6 | RX Audio (Discriminator) | 11 | Flat audio out |
  | 7 | N/A | — | |
  | 8+9 | Ground | 7 | |

  AllStar software settings confirmed by this document:
  - carrierfrom=usb
  - ctcssfrom=usb
  - txmixa=voice

  ⚠️ CDM PIN ALIGNMENT NOTE: This hookup sheet is for the Maxtrac/GM300 16-pin connector.
  The CDM1250 uses a 20-pin connector. Pin numbers that appear to align:
  - Pin 3 (PTT) ✅ confirmed same on CDM
  - Pin 5 (Flat TX Audio) ✅ confirmed same on CDM
  - Pin 7 (Ground) ✅ confirmed same on CDM
  - Pin 11 (RX Audio flat/filtered) ✅ confirmed same on CDM
  - Pin 8 (CTCSS output) — CDM Pin 8 = Digital I/O 4, programmable ✅ plausible
  - Pin 12 (COS output) — CDM Pin 12 = Digital I/O 7, programmable ✅ plausible
  No CDM-specific hookup sheet found. Verify Pin 8 and Pin 12 CPS assignments with
  SFARC Monday digital-nodes net or Repeater-Builder before programming.

- RIM-Maxtrac USB v1B documentation PDF (full RIM, NOT the Lite): https://www.repeater-builder.com/products/RIM_pdfs/rimdocs.pdf
  Fetched and verified 2026-06-13. Full RIM uses DB-25 — different connector, different pinout.
  Full RIM's own 25-pin interface connector pinout (for reference only — does not apply to Lite):
  | RIM Pin | Signal       | Notes |
  |---------|-------------|-------|
  | 1       | PTT          | Output from RIM to radio |
  | 2       | TX AUDIO AUX | |
  | 3       | TX AUDIO MAIN| |
  | 4       | GND          | |
  | 5       | COS IN       | Carrier detect input TO the RIM from radio |
  | 6       | CTCSS IN     | CTCSS decode input TO the RIM from radio |
  | 7       | RX AUDIO IN  | |
  | 8       | +5V          | |
  | 9–12    | I²C SERIAL EEPROM (CS/CLK/DO/DI) | |
  | 13–14   | GND          | |
  | 15      | LOCAL RPT.   | PC comms OK output |
  | 16–17   | GND          | |
  | 18–25   | I/O 1–8      | GPIO lines |

  Solder jumper settings for AllStar + CDM1250:
  - SJ3 position '1' (shipped default) — required for AllStar (ACID/XIPAR). DO NOT change.
  - SJ6: short if CDM COS output is open-collector (does not source voltage)
  - SJ7 position '1' (shipped default): COS input expects voltage-high when carrier present
  - SJ4: short if CDM CTCSS output is open-collector
  - SJ5 position '1' (shipped default): CTCSS input expects voltage-high when tone present

  ⚠️ STILL NEEDED: Model-specific hookup sheet for CDM1250 — maps RIM Pin 5 (COS IN)
  and RIM Pin 6 (CTCSS IN) to specific CDM1250 accessory connector pins. This determines
  which CDM pin to assign as Monitor Output in CPS. Check repeater-builder.com/products/
  usb-rim-lite.html for the CDM-series model-specific connection diagram.
- CDM1250 programming cable (RKN4081 FTDI USB): https://bluemax49ers.com/usb-cables-serial-cables/
  Product Explorer search interface — search "CDM1250" or "Motorola CDM" to find cable listing.
  Static fetch not possible (JavaScript search form). Verified 2026-06-13: site is a search tool,
  not a static catalog. Price ~$33 per original project notes; confirm at time of order.

- BlueMax49ers driver downloads: https://bluemax49ers.com/driver-downloads/
  Fetched and verified 2026-06-13.
  RKN4081 uses FTDI chip — install CDM21236_Setup.zip (Windows XP and later, all versions incl. Win11).
  Download: https://www.dropbox.com/s/gslh0hkqxr16b0y/CDM21236_Setup.zip?dl=1
  Do NOT install Prolific driver — that is for a different cable type.
  Allow 4–8 minutes for Windows to complete driver installation after first plug-in.

## CDM1250 / Waris Series Technical Reference
- W9CR Waris wiki: https://wiki.w9cr.net/index.php/Waris
  Evaluated and verified safe 2026-06-15. Maintained by W9CR — well known ham radio operator,
  long standing technical reference for Motorola Waris series radios. Referenced by repeater-builder.com.
  CPS R06.12.05 downloaded from here 2026-06-15: saved as CPS_R06.12.05_HVN9025.zip locally.
  Also hosts: CDM firmware upgrades, codeplug tools, service manuals, accessory connector docs.
  Fetched and verified 2026-06-13. Confirms CDM1250 20-pin TE-AMP connector pinout.
  Additional details beyond Repeater-Builder CDM page:
  - Pin 8 = Digital In/Out 4 / MPT Tx (5V logic) — MPT Tx role only in trunking; conventional mode = programmable I/O
  - Pin 15 RSSI = direct from SA616 IF IC, 0.793V (-135 dBm) to 2.420V (-30 dBm), linear -120 to -55 dBm
  - CPS navigation: Radio Config → Accessory Config → RX Audio Type (path to Flat Audio setting)
  - Audio alignment tones: 80 Hz and 2500 Hz (not 3000 Hz) for CDM audio level adjustment

## Configuration Reference Builds
- VARA-FM CDM-750 reference build (flat audio / accessory pin config — confirmed applicable to CDM1250):
  https://www.valleycenterarc.org/VARA-FM.html
  Fetched and verified 2026-06-13. Confirmed settings:
  - Pin 3 = Data PTT Input (all other pins = Null)
  - Rx Audio Type = Flat Audio
  - Pins 7+9 shorted = auto power-on after outage (RIM-Maxtrac hardware feature)
  - Channel Bandwidth = WIDE (25 kHz)
  - Squelch Type = CSQ
  NOTE: This document does NOT address COR/COS pin assignment. Pin 8 for COR was asserted
  in peer review but is NOT confirmed by this source or the CDM1250 accessory connector doc.

- CDM1250 accessory connector full pin table:
  https://www.repeater-builder.com/motorola/cdm/cdm-acc-conn.html
  Fetched and verified 2026-06-13. Complete 20-pin TE-AMP connector:

  | Pin | Signal | Direction | Notes |
  |-----|--------|-----------|-------|
  | 1   | Speaker Output (−) | Out | DO NOT ground — destroys amplifier |
  | 2   | External Mic Audio | In | High-pass filtered |
  | 3   | Digital In 1 [PTT] | In | 4.7k to 5V & 47k to Gnd |
  | 4   | Digital Out 2 | Out | 4.7k to 12V — programmable via CPS |
  | 5   | Flat TX Audio | In | No deviation limiting |
  | 6   | Digital In 3 | In | 10k to 5V |
  | 7   | Ground | — | |
  | 8   | Digital In/Out 4 | In/Out | 4.7k to 5V — programmable via CPS |
  | 9   | Digital In 5 / Wakeup [Emergency] | In | 47k to 5V — shorted to Pin 7 by RIM-Maxtrac |
  | 10  | Digital In 6 / Ignition | In | 4.7k to Gnd |
  | 11  | Flat/Filtered RX Audio | Out | Programmable flat or de-emphasized |
  | 12  | Digital In/Out 7 | In/Out | 4.7k to 5V — programmable via CPS |
  | 13  | Switched B+ | Out | |
  | 14  | Digital In/Out 8 | In/Out | 4.7k to 5V — programmable via CPS |
  | 15  | RSSI | Out | 0.8V (no signal) to 2.4V (full signal) — analog |
  | 16  | Speaker Output (+) | Out | Floating, same as Pin 1 |
  | 17  | BUS+ (Programming/SCI+) | Bi | Programming only |
  | 18  | Boot Control | — | |
  | 19–20 | N.C. | — | |

  CRITICAL NOTE ON COR: There is NO dedicated COR/COS/CTCSS-decode pin on the CDM1250.
  COR must be assigned via CPS to one of the programmable Digital I/O pins (4, 8, 12, or 14).
  Peer review asserts Pin 8 for COR — plausible (it is a programmable I/O), but which pin the
  RIM-Maxtrac hardware COS input physically connects to on the accessory connector is NOT
  confirmed. Download RIM-Maxtrac schematic PDF from repeater-builder.com/products/usb-rim-lite.html
  to verify the physical wiring before assigning any CPS COR pin.

  RSSI ALTERNATIVE: Pin 15 (RSSI, 0.8–2.4V analog) can serve as carrier detect via voltage
  threshold if the RIM-Maxtrac supports analog COS input. Check schematic.

## Software / Protocol
- AllStarLink node registration portal: https://allstarlink.org
- ASL3 official documentation and install guide: https://allstarlink.github.io
  Verified 2026-06-13. Key facts:
  - Supports Debian 13 Trixie (recommended) and Debian 12 Bookworm
  - Supports x86_64/amd64 (Wyse 3040) and Raspberry Pi 3/4/5
  - Built on Asterisk 20 LTS (major upgrade from legacy Asterisk 1.4)
  - Cockpit web console available for browser-based node management
  - Asterisk runs as non-root (security improvement)
  - HTTP-based registration
  - Install docs at allstarlink.github.io (JavaScript-rendered — browse directly, not fetchable)

- ASL3 USB Audio Interfaces page: https://allstarlink.github.io/adv-topics/usbinterfaces/
  Fetched 2026-06-13. Key facts for RIM-Maxtrac (CM-108 family):
  - devstr= can be left BLANK — ASL3 auto-assigns the device on first run
  - Use `asl-find-sound` utility to identify device string if needed
  - Natively supported chips: CM-108 (PIDs 0x000c/d/e/f), CM-108B (0x0012), CM-108AH (0x013c),
    CM-119 (0x0008), CM-119A (0x013a), CM-119B (0x0013), N1KDO (0x6a00) — all VID 0x0d8c
  - Non-native chips (AIOC, etc.): add VID:PID to /etc/asterisk/res_usbradio.conf
  - Audio tune menus: `sudo /usr/sbin/simpleusb-tune-menu` (SimpleUSB) or
    `sudo /usr/sbin/radio-tune-menu` (USBRadio / usbradio driver)
  - Save to EEPROM: `susb tune save` or `radio tune save`
  - RX audio target: average level not past 3 kHz mark; peaks not past 5 kHz mark
  - TX audio target: FM deviation under 3 kHz, clear audio
  - RX Boost: disable in nearly all cases
  - ClipCnt > 0 in stats = input overdriving — reduce RX level
  - Moving RIM-Maxtrac to a different USB port may change devstr; blank devstr= and re-discover
  - Tune settings are now in simpleusb.conf / usbradio.conf directly (no separate tune files in ASL3)
  - Channel syntax: rxchannel=SimpleUSB/1999 (not SimpleUSB/USB1999 — changed in ASL3)

- ASL3 Config Files — rpt.conf: https://allstarlink.github.io/config/rpt_conf/
  Fetched 2026-06-13. Key parameters for simplex node:
  - rxchannel= : radio interface type (required) — will be SimpleUSB/1999 or Radio/1999
  - duplex= : 0 = half duplex no telemetry, 1 = half duplex with tones (use 1 for simplex)
  - idrecording= : voice/Morse ID. Morse prefix: |i. Default ID interval: 300000 ms (5 min)
  - hangtime= : squelch tail duration ms (default 5000 = 5 sec)
  - totime= : transmit timeout ms (default 180000 = 3 min) — increase if connecting to busy hubs
  - archivedir= : audio logging — OFF by default, heavy CPU/storage if enabled
  - startup_macro= : commands to run at boot
  - Default function character: * / End character: #
  - File location: /etc/asterisk/rpt.conf
  - Default sounds: /usr/share/asterisk/sounds/en
  - The default rpt.conf is well-commented and works for most nodes out of the box

- ASL3 Config Files — simpleusb.conf: https://allstarlink.github.io/config/simpleusb_conf/
  Fetched 2026-06-13. Key parameters for RIM-Maxtrac on simplex node:
  - devstr= : USB device string — leave blank for auto-assign (ASL3 fills it in)
  - carrierfrom=usb : COR/COS from USB GPIO (required for RIM-Maxtrac hardware COS)
  - ctcssfrom=usb : CTCSS from USB GPIO (required for RIM-Maxtrac CTCSS input)
  - rxondelay= : ms to delay COR assertion after carrier — set 25 for simplex (prevents glitch keying)
  - txoffdelay= : ms transmitter must be off before COR valid (default 0)
  - rxboost=no : leave OFF for RIM-Maxtrac (CM119B is the exception that needs yes)
  - rxmixerset= : RX audio level (default 500, tune via radio-tune-menu)
  - txmixaset= : TX left channel audio level (default 500)
  - txmixbset= : TX right channel audio level (default 500) — keep txmixaset + txmixbset < 1000
  - deemphasis=no : use no for flat audio (discriminator audio from CDM1250)
  - preemphasis=no : use no for flat audio TX path
  - plfilter=no : highpass filter — off for flat audio
  - eeprom=0 : set to 0 for most nodes (1 = external EEPROM present)
  - legacyaudioscaling=0 : use 0 for new nodes
  - File location: /etc/asterisk/simpleusb.conf (tune settings stored here in ASL3, not separate file)
  - Both chan_simpleusb.so and res_usbradio.so must be loaded

- ASL3 Config Files — iax.conf: https://allstarlink.github.io/config/iax_conf/
  Fetched 2026-06-13. Key facts for simplex node:
  - Leave iax.conf defaults unchanged — asl-menu configures what needs changing
  - Registration is NOT in iax.conf in ASL3 — it's in rpt_http_registrations.conf
  - bindport=4569 UDP — this is the only port to forward on Comcast router
  - Outbound codecs: ulaw (primary), adpcm, gsm — defaults are correct
  - [radio] stanza handles incoming node connections — do not edit
  - requirecalltoken=no only needed if clients report rejection errors (Asterisk 20+ security)
  - [allstar-autopatch] is commented out by default — leave it, not needed for simplex node
  - File location: /etc/asterisk/iax.conf

- AllScan How-To guides (ASL3 node setup reference): https://allscan.info/docs/
- ASL3 / app_rpt documentation: https://wiki.allstarlink.org
- Debian 13 Trixie installation media: https://www.debian.org/distrib/
- Debian 12 Bookworm installation media: https://www.debian.org/distrib/

## Hardware
- Dell Wyse 3040 thin client (part 3YX35): https://www.techforless.com
  Specs: Intel Atom x5-Z8350, 2GB RAM, 8GB eMMC, <4W, fanless, WLAN+ethernet
- Wyse 3040 service manual (BIOS / internal M.2 B-key slot documentation): search Dell support site for "Wyse 3040 service manual"

## Frequency Coordination
- NARCC database: https://www.narcc.org — Database → Search
  El Dorado County 2m search run 2026-06-14 (18 records). Key findings:
  - No coordinated repeaters in simplex segments 146.400–146.580 or 147.420–147.570
  - 146.460 MHz selected for simplex node — clear of all coordinated activity in EDH area
  - PL 127.3 Hz selected — 100.0 Hz avoided (used by KD6CQ El Dorado Hills/Shingle Springs)
  - AG6AU 147.975 MHz (EDH) shows Uncoordinated in NARCC — operational but not coordinated
  - W6EK 145.430 MHz (Auburn/SFARC) is Placer County — not in El Dorado County results

## Local Network
- W6EK (SFARC repeater): 145.430 MHz, PL 162.2, AllStar node 51018, EchoLink 4128, Wires-X 62545
- K6IS: AllStar node 61464, 145.190 MHz (PL 162.2 east / 123.0 west)
- CARLA regional network: NorCal/Central CA/NV AllStar+DMR network

## Regulations
- FCC Part 97 (frequency privileges, power limits, emission types):
  https://www.ecfr.gov/current/title-47/chapter-I/subchapter-D/part-97
  §97.301: authorized frequency bands
  §97.305: authorized emission types
  §97.313: power limits
- ARRL Band Plan: https://www.arrl.org/band-plan
- Technician Privileges: https://www.arrl.org/what-privileges-does-a-technician-have

## Networking
- 44Net Connect (WireGuard tunnel for CGNAT workaround): https://44net.org
  Use if ISP WAN IP is in 10.x, 172.16–31.x, or 100.64–127.x (CGNAT) — port 4569 forwarding will not work

## Handheld Radio
- Yaesu FT-70D manual: https://www.yaesu.com/indexVS.cfm?cmd=DisplayProducts&ProdCatID=111&encProdID=8FA58F426C671510FA2E29EE76C05BD1
- FT-70D product page: https://www.yaesu.com (search FT-70D)
- Available from: DX Engineering (dxengineering.com), GigaParts (gigaparts.com) — ~$189–219

## Community Resources
- Repeater-Builder (CDM series docs, RIM-Maxtrac wiring): https://www.repeater-builder.com
- SFARC Monday digital-nodes net — in-person help resource for final wiring/config
- RadioReference forums (CDM1250 CPS hash verification): https://forums.radioreference.com
