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

### IC15 alternative: a "27C301-type" 32-pin EPROM, no lifted pins (UV / OTP)
No 28-pin programmable 1 Mbit chip exists (the mask-ROM pinout has no pin free for VPP or WE). But one family of
32-pin EPROMs was made as a replacement for this 28-pin socket: the non-JEDEC "27C301 / 27C1000" layout with A16
on pin 24 and /OE on pin 2. Members: **Hitachi HN27C301** (G = UV window, P/AP = OTP), **Toshiba TC571000**,
**NEC uPD27C1000**, **Mitsubishi M5M27C100** (K = UV, P = OTP), **Fujitsu MBM27C1000** (-Z = UV, P = OTP),
**Macronix MX27C1001** (Macronix naming is reversed: MX27C1000 is the JEDEC one), **Intel 27C100 / 27011**. NOS and
pulls only (eBay, UTSource); all are in the XGecu T48 list under their own names. Not electrically erasable.
- Seat the chip hanging **2 pins over the notch end**: chip pins 3-30 into socket pins 1-28. Every address, data,
  GND and /CE pin then lands right (chip 22 /CE -> socket 20, chip 24 A16 -> socket 22), and chip pin 30 (NC) sits
  on socket Vcc harmlessly.
- The 4 overhanging pins get wires, nothing is lifted or cut: **pin 32 (Vcc), pin 1 (VPP), pin 31 (/PGM) -> +5 V**
  (shortest: to chip pin 30, which is in the Vcc socket pin); **pin 2 (/OE) -> GND** (to chip pin 16 / socket 14).
  Without the pin 32 wire the chip is unpowered; without the pin 2 wire its outputs never turn on.
- Cleaner: put the wires on a **32-pin turned-pin socket** instead (its pins 3-30 go into the IC15 socket, straps
  1/31/32 -> 30 and 2 -> 16 soldered on the socket). Then EPROMs plug in and out normally.
- Burn the 128 KB file (`D110_IC15_Rosetta-vN_128K.bin`). 120-200 ns parts are fine. Not yet tried on a D-110.

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
