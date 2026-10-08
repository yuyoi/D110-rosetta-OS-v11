# How to reverse engineer the Roland LA OS (for AI models and humans)

This folder is the briefing for any AI model (or person) that continues the work on the Roland D-110 OS, the
Rosetta mod, a D-10 port or the GR-50. Read it before touching code. It says what is known, how it was found,
which tools exist, the rules, and the mistakes that already cost hardware test rounds.

## Read in this order
| File | What |
|---|---|
| `PROMPT.md` | paste this into a new AI session to start it right |
| `RULES.md` | legal, safety and workflow rules (non-negotiable) |
| `PLATFORM.md` | the hardware: CPU (MCS-96 / 8097), memory map, banking, I/O, LA32, chips |
| `METHOD.md` | the method and the tools: dump -> disassemble -> trace -> map -> patch -> simulate -> burn |
| `D110_CHEATSHEET.md` | the most used D-110 v1.10 addresses on one page (details: `../IC19_MAP.md`, `../IC12_MAP.md`) |
| `ROSETTA.md` | how the Rosetta mod is built: hooks, IC15 code, RAM, menu, how to add a feature |
| `PITFALLS.md` | every bug that already happened and how to avoid it |
| `PORT_D10.md` | plan for porting to the D-10 (and other LA family units) |
| `GR50.md` | what is known / unknown about the GR-50 and how to start on it |
| `HARDWARE.md` | chips, adapters, wiring, burning |

The big reference files are in the repo root: `IC19_MAP.md` (OS ROM map, everything verified so far),
`IC12_MAP.md` (control ROM map + IC15 wiring), `rosetta.py` (the mod source, heavily commented).

## Status words used everywhere
- **verified**: read from the traced code (or the parsed data is self-consistent).
- **hardware**: the user confirmed it on the real unit. Only the user can promote something to this.
- **simulator**: passes `mcs96_sim` tests, not yet on hardware.
- **hypothesis**: plausible, not confirmed. Say so in docs and commits.

## Where things stand (2026-10-08)
- D-110 OS v1.10 (IC19): ~24 KB of 32 KB traced; UI menu system, MIDI input, SysEx map, I/O, LA32 access decoded.
- IC15 (control ROM) fully mapped; the demo song area hosts new code (page 0x27).
- Rosetta v12: ~15 KB of new code/data in IC15 + ~300 B of hooks in IC19. v9 confirmed on hardware; v10-v12
  simulator only.
- D-10: not started (same CPU board family; the D-110 ROM even contains "Roland D-10" ID strings).
- GR-50: not started; nothing verified yet.
