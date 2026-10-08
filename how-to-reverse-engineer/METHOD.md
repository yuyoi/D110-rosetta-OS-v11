# Method and tools

## Tools in this repo
| Tool | Use |
|---|---|
| `mcs96_dis.py` | disassembler + flow tracer. Follows code from the vectors, jump tables, UI menus and the RAM copy; prints labels, menus (`menu`/`key` rows) and data. |
| `test_mcs96_dis.py` + `mame_ref/` | checks our decoder against MAME's on every byte offset (`sh mame_ref/build.sh` first) |
| `mcs96_asm.py` | assembler; syntax = the disassembler's output, every line is re-decoded to check it |
| `mcs96_sim.py` | small simulator (no I/O, no timers, no interrupts): call a routine, stub ROM routines by address, inspect RAM |
| `patch_ic19.py`, `patch_ic15.py` | apply mods to the user's own dumps (SHA-1 check of v1.10) |
| `test_rosetta.py`, `test_ic19_quick.py` | full feature tests on the patched images in the simulator |

```
python mcs96_dis.py ctrl/ic19.bin -o ctrl/ic19.lst       # full listing (private: keep it out of git)
python mcs96_dis.py ctrl/ic19.bin --summary              # vectors, code/data map, I/O and RAM references
python mcs96_dis.py ctrl/ic19.bin --at 3615 40           # linear decode at an address
python mcs96_dis.py ctrl/ic19.bin --entry 65b0 -o x.lst  # add an entry point the tracer missed
```
Outside references: MAME `roland_d10.cpp` (memory map, I/O names), munt (LA32 + MT-32 data structures), the
service notes (pinouts, schematic), the owner's manual (MIDI implementation, SysEx addresses).

## How the D-110 OS was mapped (repeat this for a new unit)
1. **Dump and identify.** Read the chip, hash it (SHA-1), match it to MAME's ROM set. Find the version string.
2. **Memory map first.** From MAME's driver + the schematic: where ROM, RAM, I/O, bank latch are. Without this every
   address is ambiguous.
3. **Vectors -> code.** Start the tracer at reset and the interrupt vectors. Look at what it can't follow: indirect
   jumps (`br [rX]`), jump tables (`ljmp`-free `tijmp`-like patterns, word tables of code addresses), code run from
   the bank window, routines copied to RAM. Teach the tracer each new pattern; re-run; repeat until coverage stops
   growing (D-110: 15.3 KB -> 23.9 KB).
4. **Anchor on I/O.** Every access to an I/O address names a driver: `0x0300` = LCD, `0x021A` = buttons,
   `0x0C00` = LA32, serial SFRs = MIDI. Name those routines first; then their callers.
5. **Anchor on strings.** Find the LCD text, then the code that prints it: that gives the UI. The D-110 UI is a
   table-driven interpreter (`0x53A3`): decoding its format unlocked all 34 menus at once.
6. **Anchor on the MIDI path.** Serial interrupt -> ring buffer -> main-loop dispatcher -> per-status jump table ->
   note on/off, CC table (128 words), SysEx parser and its address map. These are the best hook points.
7. **Anchor on data.** Known structures (munt's timbre and PCM table layouts) tell you which code reads them
   (`0x8900[r74]` = PCM table after selecting page `0x20`).
8. **Find free space.** FF fill in ROMs; code only reachable from removed features (demo player, demo songs); RAM
   ranges no instruction references (check every `ld/st` with an absolute or base address, plus the stack range).
9. **Find free registers.** Registers no instruction touches (D-110: `0x1A-0x3F`, `0x90-0x9F`).
10. **Write it down** in the map file with the proving address, status word and date.

## How a patch is made
1. Pick the hook: a jump table entry, a CC table slot, a `ret` in a routine, an `ljmp` in the main loop.
2. Note the exact register contract at that point (which registers hold inputs, which must be kept, which are free,
   which bank is mapped, whether interrupts are masked).
3. Write the new code with `mcs96_asm` into free space; redirect the hook; keep the original instruction's effect.
4. Simulate: stub the ROM routines you call (note on/off, LCD), drive the hook, check RAM and calls.
5. Burn, test on hardware, record the result.

## Getting help from the simulator
- `Sim(ic19, ic15)`, `s.stubs[addr] = fn`, `s.call(addr)`, `s.ld(a, size)`, `s.st(a, v, size)`, `s.odd` (odd word
  accesses: must stay empty), `s.bank`.
- `test_rosetta.py` shows the patterns: `note()`, `tick()`, `keys()` (menu keys), `goto()` (menu item), LCD capture
  through a stub on `0x208A`, MIDI out capture through a stub on `0x1D8D`.
- The simulator has no interrupts: anything racing with `int_extint` or the serial interrupt must be reasoned about
  (mask `int_mask` around multi-step LA32 writes, like the stock code does).
