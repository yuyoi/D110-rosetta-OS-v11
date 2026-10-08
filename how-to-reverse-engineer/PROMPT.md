# Starter prompt for a new AI session

Copy everything between the lines into the first message of a new session (Claude Code or similar), with this repo
attached. Edit the TASK line.

---
You are continuing a reverse-engineering and firmware-mod project for the Roland D-110 (LA synth module, MCS-96 /
8097 CPU) called Rosetta. Before doing anything:

1. Read `how-to-reverse-engineer/README.md`, `RULES.md`, `PLATFORM.md`, `PITFALLS.md`, then the file for the task
   (`ROSETTA.md` for new features, `METHOD.md` + `D110_CHEATSHEET.md` for reverse engineering, `PORT_D10.md` or
   `GR50.md` for other units). Use `IC19_MAP.md` / `IC12_MAP.md` in the repo root as the reference.
2. Rules: never commit Roland ROM bytes, dumps, listings or patched/baked .bin files (they are gitignored: keep it
   so). The user supplies their own dumps in `ctrl/` or `roms/`. Mark facts as verified / simulator / hypothesis;
   only the user marks something "hardware confirmed".
3. Every code change: build, run `python test_rosetta.py <ic19> <ic15>` (or `python bake_rosetta.py --test`), add a
   test for the new behaviour, check "no word access at an odd address". Bump `VERSION` in `rosetta.py` for a new
   burn. Keep the previous version's burn file as a backup.
4. Hardware test steps you write for the user always start and end with "Power off and unplug".
5. Keep answers short. Ask before big design changes.

TASK: <describe the task here>
---

Tip: one task per session. Long sessions get slow and expensive; the docs carry the context between sessions.
