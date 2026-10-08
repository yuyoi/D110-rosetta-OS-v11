# Platform: D-110 hardware as the OS sees it

Source: D-110 service notes (parts list p.4, schematic p.7, IC data p.12, change info p.14), MAME
`src/mame/roland/roland_d10.cpp`, and the traced code. Addresses are D-110 OS v1.10.

## CPU
- **Intel N8097BH (IC18), MCS-96 family, 12 MHz, 8-bit external bus** (BUSWIDTH low). Little-endian.
- 256 bytes of on-chip registers/SFRs at `0x00-0xFF`. Registers are the working variables (`r70`, `r74`, ...).
  `0x18` = SP (starts at `0xF9D0`), `0x00` = `zero` register (always 0, usable as a source and as a base).
- **Word accesses must be at even addresses** (registers and memory). Long word ops (`mulu`/`divu` results) need
  4-aligned destinations.
- A word register is a byte pair: `r74` = `r74` (low) + `r75` (high). Byte and word use of one pair clobber each other.
- Arithmetic results go to a register. Memory is reached with `ld`/`st`, or as the source operand
  (`add r70, 0x8902[r76]`, long-indexed off `zero` for absolute addresses).
- Flags: `cmp`/`sub` set C = 1 when there is **no** borrow (`jc` = unsigned >=, `jnc` = unsigned <, `jh` = >).
  Signed: `jgt`/`jlt`/`jge`/`jle`.
- `ldbze` writes a whole word (zero-extended): it clobbers the next register.
- Interrupts: `di`/`ei`, `int_mask`. Vectors at `0x2000-0x2011` (timer overflow `0x22A3`, software timer `0x1A08`,
  serial `0x1DAC`, EXTINT `0x3138` = LA32 interrupt, TRAP `0x19D5`).

## Memory map
| CPU address | What |
|---|---|
| `0x0000-0x00FF` | registers / SFRs |
| `0x0100` | bank latch (write page number) |
| `0x0200` | system out (LED, reverb bits) |
| `0x021A` / `0x021C` | button matrix SC0 / SC1 (read); `0x021A` write = output latch |
| `0x0280` | LCD-side latch (use unknown) |
| `0x0300` / `0x0380` | LCD data / control (HD44780-type, 2 x 16) |
| `0x0400`, `0x0800` | reverb parameter latches (write only) |
| `0x0C00-0x0DFF` | **LA32** sound chip registers (per-partial banks, partial * 2) |
| `0x1000-0x7FFF` | **IC19** OS EPROM (file offset = CPU address) |
| `0x8000-0xBFFF` | 16 KB **bank window** |
| `0xC000-0xFFFF` | fixed RAM (battery backed SRAM) |

## Banking
- Write a page number to `0x0100`; bank space `page * 0x4000` appears at `0x8000-0xBFFF`.
- Pages: `0x00`-`0x01` IC19 (lower half: UI strings, test mode), `0x11` RAM, `0x20-0x27` **IC15** (control ROM,
  128 KB = 8 pages), `0x30`/`0x31` memory card.
- Convention: register `rb7` holds the current latch value. The LA32 interrupt (`int_extint`) switches pages and
  always writes `rb7` back on exit. So: code that changes the page sets `rb7` first, and restores the caller's `rb7`
  (+ latch) afterwards. `push rb6` saves `rb6`+`rb7` as one word.
- Code can run from the window: Rosetta runs from IC15 page `0x27`.

## Chips
| Board | Part | What |
|---|---|---|
| IC18 | N8097BH | CPU |
| IC19 | uPD27C256AD-20 (28-pin EPROM, 32 KB, socketed) | OS |
| IC15 | LH5310-DJ (28-pin mask ROM, 128 KB, odd pinout; "ic12" in dump sets) | control/tone ROM: timbres, PCM table, wave names, demo songs |
| IC7 / IC8 | HN62304B (32-pin mask ROM, 512 KB each, 27C040 pinout) | PCM samples |
| IC6 | HN623257 (32 KB) | BOSS reverb chip program, not CPU code |
| IC9 | MB87136APF | LA32 sound generator |
| IC17 | HM62256LP-15 | 32 KB SRAM (battery) |
| IC12 | TC74HC27P | logic, not a ROM (dump sets misname IC15 as ic12) |

## LA32 (partly hypothesis)
- 32 partials. Register banks of 32 words, indexed by partial * 2: `0x0C00/0x0C80` ramp pairs, `0x0C40/0x0C41`
  (cutoff for synth, sample position for PCM), `0x0CC0` pitch, `0x0D00/0x0D01` control byte + resonance (synth)
  or control + wave length (PCM). `0x0D00/0x0D01` behave as a 16-bit pair: write `0x0D00` then `0x0D01`.
- The LA32 interrupts (EXTINT) when an envelope ramp ends; `0x0C00` read = partial number.
- RAM shadows: `0xEF80/0xEF81` (= `0x0D00/0x0D01`), `0xF1C0` (= `0x0C41`), `0xEF40` pitch base.
- Partial parameter layout = munt's MT-32 layout (58-byte partial blocks). munt (github.com/munt/munt) is the best
  outside reference for the LA engine; read it, don't copy it.

## Ticks and MIDI
- Software timer 1 increments `rc4` about every 2.08 ms (estimate). Stock OS used it only for the demo sequencer.
- MIDI: serial interrupt `0x1DAC`; stock OS drops clock/start/continue. TX through `sub_1D8D` (byte in `rd0`).
