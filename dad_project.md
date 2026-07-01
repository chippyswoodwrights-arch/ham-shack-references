# Dad's Place — Home Monitoring & Smart Home

## Overview
Build a fully networked home monitoring system starting with a doorbell camera,
expanding to full camera coverage, and laying groundwork for smart home integration.
Entry point is the doorbell — each addition builds toward a complete system.

## Key Considerations
- 89 years old, computer capable for his age, tech support is Matt
- Hard of hearing — hearing aids connected to cell phone
- Visual alerts on phone are more reliable than audio chimes
- Long driveway to front door — camera at entry point is high value
- Consumer Reports subscriber — he will fact check any recommendation
- Runs Claude AI — has a quick start guide

## Phase 1 — Doorbell Camera (entry point)
- **Goal:** See who's at the door from phone before getting up
- **Requirements:** No subscription, phone app, night vision, simple setup
- **Leading candidate:** Reolink Video Doorbell (~$60-80) — no subscription,
  local SD storage, two-way audio, night vision, phone alerts
- **Alternative:** Eufy Video Doorbell (~$80-100) — same profile, slightly better video
- **Avoid:** Ring — subscription required to access recordings
- **Research:** Run through Consumer Reports before buying. Have him ask Claude first,
  then fact check with CR. Tell Claude "give me your honest assessment, no fluff."
- **Installation:** Check existing doorbell wiring vs battery option

## Phase 2 — Full Camera Coverage
- **Hub:** Wyse 3040 running Frigate (free, open source NVR)
- **Storage:** 1TB USB hard drive (~$40-50), motion-triggered recording only
- **Cameras:** Reolink or equivalent IP cameras with IR night vision (~$35-40 each)
- **Night vision:** Built-in IR LEDs on all outdoor cameras
- **Retention:** Set per-camera — shorter windows on high-traffic areas
- **Areas to cover:** TBD — need walkthrough of property

## Phase 3 — Smart Home Integration
- **Software:** Home Assistant on same Wyse 3040 as Frigate
- **Value for Dad:** Visual alerts for doorbell, motion, door sensors on phone/tablet
  Especially useful given hearing aids — see it before you hear it
- **Expandable:** Lights, thermostat, door sensors, all in one dashboard

## Hardware Notes
- Wyse 3040 (~$40, Tech For Less part 3YX35) — runs Frigate + Home Assistant
- One Wyse handles both if camera count stays reasonable (under ~6 cameras)
- Coral TPU accelerator (~$60) if AI object detection needed (person vs car vs animal)

## HF Antenna Planning
See dad_antenna.md for full site description and antenna plan.
Short version: Phase 1 Hustler vertical on roof, Phase 2 EFHW down the slope to adjacent lot,
Phase 3 beam if he gets serious. Exceptional site — elevation, open sky, adjacent lot, mature trees.

## Status
- [ ] Dad runs doorbell camera research through Consumer Reports
- [ ] Select and order doorbell camera
- [ ] Install doorbell camera
- [ ] Plan camera coverage areas (property walkthrough)
- [ ] Order Wyse 3040 + 1TB USB drive
- [ ] Set up Frigate
- [ ] Add cameras one at a time
