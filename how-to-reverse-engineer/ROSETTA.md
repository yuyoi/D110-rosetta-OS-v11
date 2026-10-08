# How Rosetta is built

Source: `rosetta.py` (one file: IC19 hooks `build_ic19`, IC15 code `build_ic15`, settings, menu items, data).

## Two chips
- **IC19** gets ~300 B: a call system and hooks. Each hook does `push rb6; lcall enter; jne skip; lcall 0xB0xx;
  skip: pop rb6; stb rb7, 0x0100`. `enter` maps IC15 page `0x27` and checks the magic `0x5A1C` at `0xB000`.
  Stock IC15 = no magic = feature off, unit behaves stock.
- **IC15** page `0x27` (CPU `0x8000-0xBFFF` = IC15 `0x1C000-0x1FFFF`): code `0x8000-0xAFFF`, magic + **fixed jump
  table** at `0xB000` (`B002` banner, `B005` menu, `B008` note on, `B00B` note off, `B00E` CC70, `B011` partial
  hook, `B014` tick, `B017` aftertouch, `B01A` CC16/17), data `0xB020-0xBFFF`. Because the table is fixed, a new
  Rosetta version only needs IC15 reburned (IC19 MD5 `08ebae6e...` unchanged since v8).
- IC15 code calling IC19 routines that change the page: use `call19` (target in `TGT`, restores page 0x27).
  Reading IC15 page 0x20 (wave table) from page-0x27 code: `rd20`.

## Hooks (what runs when)
| Hook | IC19 site | Used for |
|---|---|---|
| note on / off | `jtab_241C[0/1]` | arp, chords, unison, mono/legato, recording, MIDI out |
| partial setup | `0x3BB4` end of `sub_3615` | drift, random cutoff/wave, detune, glide, Lab bits |
| tick | `0x22B9` idle path, `rc4` count | arp clock, glide, wave seq, LFO, mod matrix, Lab Motion, menu redraw |
| CC 70 / 16 / 17, aftertouch | CC table, `jtab_241C[2/5]` | wave scan, mod sources |
| MIDI clock | `0x1E83` | `F8` -> `r90`, `FA` -> `r91` (start) |
| menu | Quick screen Edit key -> `ros_ui` state handler | Rosetta menu |

## RAM and registers
- Registers `0x1A-0x3F`, `0x90-0x9F` (see `REG` in `rosetta.py`).
- Per-partial tables (index `r54` = partial * 2), shared synth/PCM bytes: `TAB` in `rosetta.py`.
- Volatile state `0xF630-0xF64F`, `0xF6A0-0xF6A3`, `0xF7C0-0xF7FF` (cleared at power-on by `init`).
- Settings in battery RAM: `0xF600-0xF62F`, `0xF670-0xF69F`, recorded sequence `0xF650-0xF66F`. Own magic
  `S_MAGIC_V`; a layout change = new magic (old settings -> defaults) or an `XMARK`-style upgrade that keeps them.

## Menu
- `ITEMS` list: name, setting, max, kind, min, display offset, value names. Kinds 0-11 (number, named, per part,
  Info, Off/number, Seq Record, Chord Learn, folder ...). Flags `K_SUB` (inside a submenu), `K_SUBP` (opens one),
  `K_SECTION`, `K_ALLOFF` (all notes off when changed).
- Keys: Group +/- item, Bank +/- value 1, Number +/- value 10, Part = part, Edit = open submenu, Exit = back.
- `draw` builds the LCD buffer and prints through `API_LCD`; the tick can redraw when `rb4 == ros_ui` (menu on screen).

## Add a feature (checklist)
1. Settings: add to `SETTINGS` (a free byte in a settings block); add menu item(s) to `ITEMS`.
2. Code: put the routine in `build_ic15`'s source; call it from the right hook (note on, partial, tick).
   Respect the hook's register contract (see `IC19_MAP.md`, "Rosetta").
3. Space: the build prints free bytes (code `0x8000-0xAFFF`, data `0xB020-0xBFFF`); far jumps use `je!`/`jbc!`.
4. Test: add checks to `test_rosetta.py`; run the whole file; "no word access at an odd address" must pass.
5. Bump `VERSION`, update `README.md` (feature list + status), commit, give the user an IC15 burn file and a test
   list that starts and ends with "Power off and unplug".
