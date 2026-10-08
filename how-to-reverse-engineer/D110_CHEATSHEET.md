# D-110 OS v1.10 cheat sheet (IC19 SHA-1 `28635510...`)

All verified from the traced code unless marked (h) = hypothesis. Full detail: `../IC19_MAP.md`.

## Entry points / API
| Address | What |
|---|---|
| `0x2080` | reset |
| `0x208A` | API: print LCD string at `r78` (address byte + text, `00` end) |
| `0x2098` | API: MIDI event reader (`r70` status, `r75` data1, `r74` data2) |
| `0x22A8` | main-loop MIDI dispatch -> `jtab_241C` |
| `0x22B9` | main-loop idle path (Rosetta tick hook) |
| `0x29F5` | periodic pass, one partial per loop (pitch -> LA32) |
| `0x3138` | `int_extint`: LA32 envelope interrupt |
| `0x1DAC` | serial (MIDI) interrupt; `0x1E83` real-time byte filter |
| `0x1D8D` | MIDI out: send byte in `rd0` |

## MIDI handlers (`jtab_241C`, per matching part: `r45` note/CC, `r46` velocity/value, `r50` part*16)
`0x245D` note off, `0x24FC` note on, `0x3BBA` control change (table `0x3BCE`, 128 words, 0 = ignored),
`0x244F` program change, `0x242C` pitch bend, `0x244E` aftertouch (stock: ignored). Keep `r42`, `r44-r46`, `r50`.
`0x3DE2` all notes off. SysEx parser `0x42C5`, address map `0x470C`/`0x4718`.

## Sound
| Address | What |
|---|---|
| `0x3615` (`sub_3615`) | per-partial note-on setup; ends at `0x3BB4` (Rosetta partial hook) |
| `0xE1E4 + part*0xF6` | timbre temp area per part (14 common + 4 x 58 partial bytes) |
| `0xEE80[p*2]` | sounding partial -> its 58-byte block |
| `0xEF40[p*2]` | pitch base (0x155 per semitone) |
| `0xEF80/0xEF81[p*2]` | control byte / resonance shadow (LA32 `0x0D00/0x0D01`) |
| `0xF1C0[p*2]` | cutoff shadow (LA32 `0x0C41`) |
| `0xF180[p*2]` | velocity attenuation |
| `0xF283 + part*16` | part records (`+8` receive channel); `0xF285` note slot list head |
| `0xF3C0` / `0xF440` / `0xEE40` | next slot / first partial of slot / next partial |
| `0xF400` / `0xF460` | slot note / slot flags (bit 6 released) |
| `0x30F0` (`sub_30f0`) | release all notes of a part |

## UI
| Address | What |
|---|---|
| `0x53A3` | menu interpreter (`ld r78,#menu; ljmp 0x53A3` = a state handler) |
| `rb4` (reg `0xB4`) | current UI state handler; `rb6` = state stack depth, stack `0xF4E2` |
| `0x5391` | pop state; `0x53CB` redraw; `0x57D1` push state with handler `r78` |
| `0x4F8E` | top-level menu; key `0x19` (Enter+Edit) = old demo, now Quick screen |
| `0xF6AB` | LCD line buffer (address byte + 32 chars) |
| `0xF6CD` | current part (0xFF until picked) |
| Key codes | 01 Exit, 02 Patch, 03 Timbre, 04 Part+, 05 Group+, 06 Bank+, 07 Number+, 08 Write, 09 Edit, 0A Part, 0B System, 0C Part-, 0D Group-, 0E Bank-, 0F Number-, 10 Enter |

## Boot
`0x2264` reads SC1: `0xEA` (Enter + Part- + Bank-) = version screen (`0x2206`, 32 chars), `0xFC` (Number- + Enter)
= factory test mode (`0x8A00`, page 0). No ROM checksum anywhere.

## Free resources (used by Rosetta)
- IC19 code holes: dead demo code `0x2371-0x241B`, `0x3EFA-0x401B`, `0x503D-0x5111`, `0x7F57-0x7FFF`, FF at
  `0x2191`, `0x2019-0x2061`, `0x1FA9`. About 300 B total; little left.
- IC15: `0x1C000-0x1EFFF` (end of demo songs) and `0x1F000-0x1FFFF` (FF) = CPU page `0x27`. More demo song space
  `0xB000-0x1BFFF` if needed.
- RAM never touched by the OS: `0xF500-0xF5FF`, `0xF600-0xF6A3`, `0xF740-0xF7FF`.
- Registers never used by the OS: `0x1A-0x3F`, `0x90-0x9F`.
