# Chat Starter Template

Use this as the first message of any new project chat. Takes about a minute.
Written 2026-09-23 in the Claude feedback (admin/training) chat.

## Why

A new chat only reads the global rulebook, the folder's CLAUDE.md + project notes, and the
memory INDEX. It does not open individual memory files, other projects' build notes, or
anything said in other chats. All project chats likely open in the Ham Radio folder, so a
GoldenEye or Octopussy chat starts with Ham AI notes loaded and its own notes missing.

## Template

```
[Project]. Goal: [one sentence for this session].
State: [what works, what's broken, last known good].
Changed: [the one thing that changed since it worked, if troubleshooting].
Read: [file names, e.g. project_octoprint.md, AllStar Node\build_notes.md].
Limits: [what's powered on, what not to touch].
Done: [how we both know the session succeeded].
Tell me where things stand and your plan before you do anything.
```

## Example (printer resume)

```
Octopussy. Goal: get a clean first layer on the PEI plate.
State: printer was fine on the stock plate (Benchy first try). Only change: PEI plate. Z offset unknown, notes conflict.
Read: project_octoprint.md, feedback_bltouch_calibration.md.
Limits: printer is on, nothing printing. No M401/M402, no EEPROM writes without asking.
Done: M503 read, one test square with a good first layer, new Z saved.
Tell me where things stand and your plan before you do anything.
```

## Where each project's notes live

| Project | Read first |
|---|---|
| Ham AI app | `Ham Radio\project_notes.md` (loads automatically) |
| AllStar / DVSwitch | `AllStar Node\build_notes.md`, `Ham Shack\DVSwitch_DMR_Bridge_Build.md`, `Ham Shack\DVSwitch_YSF_Notes.md`, memory `project_allstar_build.md`, `project_dvswitch.md` |

## Before working in any repo

Phone/remote sessions push to GitHub, not to this laptop. Start by checking `git status -sb`:
if it says `behind`, pull first. If it says `ahead`, local work hasn't been pushed yet.
| Octopussy / printer | memory `project_octoprint.md`, `feedback_bltouch_calibration.md` |
| Satellite rotator | `Satellite Rotator\README.md`, memory `project_satellite_tracker.md` |
| OptiPlex | memory `project_optiplex_ohc.md` |
| GoldenEye | memory `project_3d_printing.md` (GoldenEye laser status) |
