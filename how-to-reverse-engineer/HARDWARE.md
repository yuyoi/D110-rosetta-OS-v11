# Hardware: chips, adapters, burning (D-110)

Always: **power off and unplug** before touching chips. Keep every original chip. Read, write, read back, compare MD5.

## IC19: OS EPROM (socketed, straight swap)
- Stock: uPD27C256AD-20 (some units: Mitsubishi M5M27C256K), 28-pin, 32 KB.
- Burn a 27C256 (ST M27C256B works on hardware; T48 programmer). W27C512 / 27C512: use the image doubled (64 KB).
- Same notch orientation. No wiring.

## IC15: control ROM LH5310-DJ (28-pin, 128 KB) -> SST39SF040 (32-pin flash)
Board footprint is 28 pins. Seat the SST so its pins 3-30 sit in board pins 1-28 (pins 1, 2, 31, 32 overhang the
notch end). Then:
- SST pin 24 (OE#) lands on board pin 22 (A16): lift it fully, tie it to GND.
- Board pin 22 (A16) -> wire to SST pin 2 (A16).
- SST pin 30 (A17) lands on board pin 28 (Vcc): lift it, tie to GND.
- SST pin 1 (A18) -> GND.
- SST pins 32 (VDD) and 31 (WE#) -> Vcc (board pin 28 net).
- SST CE# (pin 22) lands on board pin 20 (IC15's private /CE). No trace cuts.
- Burn the x4 file (4 copies of 128 KB) so A17/A18 do not matter. SST39SF010A: same wiring, 128 KB file.
- Meter check before power: board pin 20 swings low during reads.
- LH5310 pinout (top view): A15 1, A12 2, A7 3, A6 4, A5 5, A4 6, A3 7, A2 8, A1 9, A0 10, D0 11, D1 12, D2 13,
  GND 14, D3 15, D4 16, D5 17, D6 18, D7 19, /OE 20, A10 21, A16 22, A11 23, A9 24, A8 25, A13 26, A14 27, Vcc 28.
  A plain 27C256 profile cannot read it without an adapter (A16 on pin 22, /OE on pin 20).

## IC7 / IC8: PCM ROMs HN62304B (32-pin, 27C040 pinout) - only for custom samples
- Read with the T48 profile AM27C040@DIP32.
- SST39SF040 in their place: board pin 31 (A18) -> SST pin 1; SST pin 31 (WE#) -> Vcc (2-wire adapter).

## Lessons
- Lift the SST OE# pin clear of any pad and ground it separately.
- Beep every net with a continuity tester before power.
- One set of pin labels per picture.
- After a new IC15 with new wave names: RAM-reset the D-110.

## Still undocumented (author to add)
- Socket/adapter and T48 profile used to read the stock IC15.
- Photos of the finished adapters, parts used.
