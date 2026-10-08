# Porting Rosetta to the D-10 (and other LA family units)

## Why it is realistic
- MAME's `roland_d10.cpp` drives the D-10 and the D-110 with one memory map: same CPU (8097), same LA32, same
  bank window. Check the file for which other models it lists.
- The D-110 v1.10 ROM contains "Roland D-10" ID strings (`0x7804`): the OSes come from one code base.
- Our tools (disassembler, assembler, simulator) are CPU-level and work unchanged.

## What is the same / what changes (estimate, not verified)
| Part | Expectation |
|---|---|
| Rosetta feature code (arp, seq, Lab, mod matrix, chords ...) | ~70% reusable as is |
| LA32 register use, partial data layout | likely the same; verify `sub_3615`-equivalent |
| Every stock address (~50: note on/off, all off, LCD, menu system, partial setup, pitch base, part records) | must be found again |
| RAM map | unknown: the D-10 has a rhythm pattern sequencer that stores patterns in RAM; `0xF500-0xF7FF` may be used |
| Free IC19 space | unknown: we used the D-110's dead demo code |
| Control ROM free space | unknown: check if the D-10's control ROM has the same demo-song area |
| Panel / keys | different buttons: menu key handling must be rewritten |
| Keyboard | notes come from the keyboard in Performance mode (Upper/Lower): decide which part Rosetta acts on |

## Steps
1. User: dump the D-10 OS ROM and control ROM; note chip markings and OS version. Hash against MAME's set.
2. Disassemble: `python mcs96_dis.py d10_os.bin --summary`; teach the tracer any new table formats.
3. Map in a new file `D10_MAP.md` with the same sections as `IC19_MAP.md`. Find in this order: MIDI dispatch + note
   on/off, CC table, per-partial note-on setup and its end, main loop idle path + tick counter, UI interpreter
   (likely the same format: search for the `"!+-=>ASidIJKDEF"` action string), LCD print API, part records and
   note slots, free ROM/RAM/registers.
4. Make `rosetta.py` machine-independent: move every stock address into a per-machine dict (`STOCK`, hook sites,
   RAM blocks) selected by a `--machine d110|d10` flag. The D-110 build must stay byte-identical (check the MD5).
5. New hooks for the D-10 in a `build_ic19_d10` (or a table-driven version of `build_ic19`).
6. Simulator tests parameterised by machine.
7. Hardware test on a D-10.

## Shortcut for the search
Many routines will be byte-similar to the D-110 ones with shifted addresses. Take a routine's bytes from the
D-110 dump (e.g. 24 bytes from the start of the note-on handler), mask absolute address operands, and search the
D-10 dump for the pattern. Confirm each hit by tracing it.
