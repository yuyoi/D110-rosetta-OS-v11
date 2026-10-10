# Rosetta OS v12 for the Roland D-110: user guide

> **Status:** v12 has passed all simulator checks but is **not yet confirmed on hardware**. The last version
> confirmed on a real unit is v9. Keep your v9/v11 chips or files as a fallback.

Before you touch any chip, **power off and unplug** the unit.

---

## 1. What you need

| Chip | What goes in | File |
|---|---|---|
| IC19 (OS EPROM, socketed) | Rosetta IC19 (27C256 / M27C256B; W27C512 with the doubled file) | `D110_IC19_Rosetta-v12_27C256.bin` (MD5 `08ebae6e…`, the same since v8) |
| IC15 (control ROM) | SST39SF040 on the IC15 adapter (see `how-to-reverse-engineer/HARDWARE.md`) | `D110_IC15_Rosetta-v12_SST39SF040-x4.bin` |
| IC7 / IC8 (samples) | stock chips, or your Rosetta Studio sample chips | – |

Make the files yourself with the baker: put your own IC19 dump in `roms/IC19 (main OS EPROM)/`, your IC15 file
(from Rosetta Studio, or your plain dump) in `roms/IC15 (bin from Rosetta Studio here)/`, and run
`python bake_rosetta.py --test`. Burn the files, read them back, compare the MD5 with `baked/BURN.txt`.

With a stock IC15 the unit works as a normal D-110 plus the IC19-only features (section 3).

## 2. First power-on
- The screen shows **` ROSETTA OS v12 ` / ` D-110  by JSW  `** for a moment. That banner comes from IC15, so seeing
  it means both chips are right. If you see `D-110 ROSETTA` instead, IC19 works but IC15 is not found.
- Settings are kept in battery RAM. A first start (or an upgrade from an old layout) loads the defaults.
- **Factory test mode** is still there: hold Number- + Enter at power-on.

## 3. Always there (IC19)
**Quick screen:** hold **Enter** and press **Edit**.

| Key | Does |
|---|---|
| Group +/- | filter cutoff (all 4 partials of the part) |
| Bank +/- | resonance |
| Number +/- | attack |
| Part +/- | release |
| Part | next part (P1-P8) |
| Edit | opens the Rosetta menu |
| Exit | back |

Cutoff and resonance change held notes live; attack and release apply from the next note. The filter only works
on synth partials (square/saw); PCM partials have no filter.

**MIDI CC knobs** (on each part's channel): CC74 cutoff, CC71 resonance, CC73 attack, CC72 release.

**Plain labels** in the stock editor: OS, Pitch, Vibr., Flt, Amp, Flt Env, Amp Env ...

## 4. The Rosetta menu
Open it with **Enter + Edit**, then **Edit**.

| Key | Does |
|---|---|
| Group +/- | previous / next item |
| Bank +/- | value -1 / +1 |
| Number +/- | value -10 / +10 |
| Part | which part a per-part item shows (P1-P8) |
| Edit | opens the submenu of an item marked `>` (Edit again or Exit = back) |
| Exit | back / leave the menu |

Items that say **Part** pick which part a feature works on (`All` or P1-P8). Changing settings that decide how
notes are played (chord, mono, unison, arp, glide/vintage part) sends all notes off first, so nothing hangs.

The top-level list:
**Wave Scan, Wave Seq >, Random >, Vintage >, Glide Time >, Mono Part >, Unison Voices >, Chord >,
Mod Matrix >, Arp Mode >, Lab >, Info**

---

## 5. Wave
**Wave Scan (CC70)** 0-127, per part. Moves PCM partials through the wave list while you play (also from CC70).

**Wave Seq >**: steps the PCM wave of each note over time (Wavestation style). The pitch follows each wave's tuning.
- Wave Seq: Off, Up 4, Up 8, Down 4, Ping 8, Octo, Strobe, Once 4, Random, User
- WSeq Speed 0-99, WSeq Part, WSeq User 1-8 (wave numbers for the User pattern)

## 6. Random >
Every new note gets its own random amount.
- **Drift Pitch** 0-31: small pitch offset per note.
- **Random Wave** 0-127: PCM wave offset range.
- **Random Cutoff** 0-100: cutoff offset range.

## 7. Pitch and voice
- **Vintage >** 0-99: every note wanders slowly in pitch (up to about ±28 cents at 99) and a little in cutoff,
  each note its own way. Inside: Vintage Part.
- **Glide Time >** 0-99. Inside: Glide Part.
- **Mono Part >**: makes one part mono. Inside: **Legato** (On = no retrigger when notes overlap, glide only).
- **Unison Voices >** 1-4. Inside: Unison Detune 0-99, Unison Part.
- **Chord >**: one key plays a chord. Off, Octave, Fifth, 5th+Oct, Major, Minor, Sus4, Major7, Minor7, Dom7,
  Minor9, Dim, **Learned**, **Scale**, **Scale 7**. Inside:
  - Chord Part
  - **Chord Learn**: Bank+ (`Play a chord`), play up to 5 notes on the Chord Part, let go. Chord switches to
    Learned. Bank- cancels.
  - **Scale Key** C-B and **Scale Type** (Major, Minor, Dorian, Phrygian, Lydian, Mixolydian, Locrian, harmonic
    and melodic minor) for Scale (triads) and Scale 7 (7th chords) on each degree.

## 8. Mod Matrix > (On / Off)
Four slots: **ModN Source**, **ModN Dest**, **ModN Amount** (-63..+63).
- Sources: Off, ModWheel, AftTouch, Velocity, Key, LFO, S&H, CC16, CC17
- Destinations: Cutoff, Pitch, Wave, Level, Reso
- **LFO Rate** 0-99, **LFO Sync** (Off, 4 Bars, 2 Bars, 1 Bar, 1/2, 1/4, 1/8, 1/8T, 1/16, 1/16T: follows MIDI
  clock, or the arp tempo without clock; S&H steps with it), **Mod Part**.
- Ideas: Velocity → Wave = velocity layers. AftTouch → Cutoff. S&H → Pitch with sync = stepped random.
- Level changes are heard from the next envelope stage. Cutoff/Reso only on synth partials.

## 9. Arp Mode >
- **Arp Mode**: Off, Up, Down, Up+Down, Random, Played, **Seq**
- Arp Octaves 1-4, Arp Rate (1/4, 1/8, 1/8T, 1/16, 1/16T, 1/32), Arp Tempo 40-240 BPM, Arp Gate % 5-99,
  Arp Latch, Arp Part
- **MIDI clock**: when the D-110 receives clock, the arp follows it; Start resets the pattern.
- **Arp MIDI Out**: Off, On (arp notes also go out of MIDI OUT on the arp part's channel), Only (sent out, not
  played inside).
- Arp notes go through mono, chord, unison and glide too.

**Groove**
- Chance % (each step plays with this chance), Ratchet (Off, 2x, 3x, 4x, Random), Octave Jump %, Accent (Off,
  Every 2/3/4, Random), Euclid Hits / Euclid Steps (Off or 1-16: spreads the hits evenly), Humanize Time,
  Humanize Vel.

**Seq (recorded pattern)**
- Up to 32 steps (notes, rests, ties), kept in battery RAM. The last key held transposes it (the first recorded
  note = that key). A demo pattern is there at the start.
- **Seq Record** (in the Arp submenu):
  - **Bank+** → `Step`: every key = one step. Number+ = rest, Number- = tie.
  - **Bank+ again before the first key** → `Live`: the first key starts the arp clock (`Rec`), keys land on the
    nearest step, a held key = tie.
  - **Bank-** = stop: the steps so far become the pattern.
  - **New in v12:** the screen counts live while you play (`Step 05/32`) and shows **`Done 05/32`** when the
    recording has stopped (Bank- or all 32 steps used).
- **Seq Length** 1-32. Set Arp Mode = Seq to play it.

## 10. Lab > (experiment, On / Off)
Raw bits of the LA32 sound chip. Strange, sometimes amazing, sometimes silent. Lab Part picks the part (All or P1-P8).
- **Lab Reso High** Off / 0-7: the top bits of the resonance byte. 7 = much stronger resonance than stock.
- **Lab Reso Low** Off / 0-31: the low bits, including values Roland never uses.
- **Lab Ctrl XOR** 0-255: flips the control byte of synth partials. With Reso High 7, values like 8 replace the note
  with transient "wind" sounds; every value is a different transient.
- **Lab PCM XOR** 0-255: the same for PCM partials.
- **Lab PCM Pos** 0-255: flips sample-position bits: other samples, other parts of the memory.
- **Lab Motion**: changes a value automatically.
  - Motion: Off, Up, Down, Ping, Random
  - Motion Dest: PCM Pos, Ctrl XOR, PCM XOR, All
  - Motion Rate: Note (every new note), 1/1 ... 1/64 (arp tempo or MIDI clock)
  - Motion Range 1-255, Motion Step 1-64
  - Retrigger On: each step plays the held notes again, so one held key runs through new sounds like a stutter.

## 11. Info
`Mem oooo P01 v12`: RAM test of the four areas Rosetta uses (`o` = ok, `X` = bad), the current part, the version.

---

## 12. Troubleshooting
| Problem | Try |
|---|---|
| Banner says `D-110 ROSETTA`, menu does nothing | IC15 not found: check the IC15 burn (MD5) and adapter |
| Nothing on the screen | power off and unplug; check chip orientation, bent pins, read each chip back and compare MD5; go back to stock chips one at a time |
| Hanging notes | Exit and change the part's patch, or send All Notes Off |
| Arp tempo slightly off | the internal tempo assumes a 2.08 ms tick; MIDI clock sync is not affected |
| Filter does nothing | the partial is PCM (no filter on PCM) |
| Settings look wrong after an update | settings from an older layout are reset or upgraded; set them again |
| Lab sound gone silent | set the Lab values back to 0 / Off, or Lab = Off |

Don't share the burn files (they contain Roland's code). Share the repo; people bake their own.
