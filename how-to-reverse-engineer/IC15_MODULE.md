# IC15 drop-in module (D-110): SST39SF040 on a DIP-28 carrier

A small PCB that plugs into the IC15 socket like the original LH5310 (28-pin DIP, 600 mil) and carries an
SST39SF040 (or SST39SF010A). All rewiring is on the PCB: no lifted pins, no wires. Program the SST on a T48 with
the owner's own baked file (`D110_IC15_Rosetta-vN_SST39SF040-x4.bin`), then plug it in.

Sources: LH5310 pinout from the D-110 service notes p.12 (see `IC12_MAP.md`), SST39SF040 datasheet pinout (DIP-32).
Status: wiring = the hand-wired adapter that works on hardware, moved onto a PCB. The PCB itself is not built yet.

## Parts
- PCB, footprint DIP-28 600 mil (pins underneath: round machined pin headers, 2 x 14, 15.24 mm row spacing)
- 32-pin DIP socket (ZIF or turned pin) for the SST39SF040, or the PLCC-32 version soldered (lower profile)
- 100 nF ceramic capacitor (decoupling, close to SST pin 32/16)
- Optional bank select: 2 x 10 kΩ resistors + 2-way DIP switch (or jumpers), see below

## Pin table (module header pin = board IC15 socket pin)

| Header pin | Signal (LH5310) | → SST39SF040 pin | SST signal |
|---:|---|---:|---|
| 1 | A15 | 3 | A15 |
| 2 | A12 | 4 | A12 |
| 3 | A7 | 5 | A7 |
| 4 | A6 | 6 | A6 |
| 5 | A5 | 7 | A5 |
| 6 | A4 | 8 | A4 |
| 7 | A3 | 9 | A3 |
| 8 | A2 | 10 | A2 |
| 9 | A1 | 11 | A1 |
| 10 | A0 | 12 | A0 |
| 11 | D0 | 13 | DQ0 |
| 12 | D1 | 14 | DQ1 |
| 13 | D2 | 15 | DQ2 |
| 14 | GND | 16 | VSS |
| 15 | D3 | 17 | DQ3 |
| 16 | D4 | 18 | DQ4 |
| 17 | D5 | 19 | DQ5 |
| 18 | D6 | 20 | DQ6 |
| 19 | D7 | 21 | DQ7 |
| 20 | /OE (IC15's only enable) | 22 | CE# |
| 21 | A10 | 23 | A10 |
| 22 | **A16** | **2** | A16 |
| 23 | A11 | 25 | A11 |
| 24 | A9 | 26 | A9 |
| 25 | A8 | 27 | A8 |
| 26 | A13 | 28 | A13 |
| 27 | A14 | 29 | A14 |
| 28 | Vcc (+5 V) | 32 | VDD |

## SST pins with no header pin
| SST pin | Signal | Connect to |
|---:|---|---|
| 24 | OE# | **GND** (always output-enabled; CE# from header pin 20 does the enabling) |
| 31 | WE# | **Vcc** (never write in the synth) |
| 30 | A17 | **GND**, or bank switch (below) |
| 1 | A18 | **GND**, or bank switch (below) |

## Optional: 4 banks on one chip
The SST39SF040 holds 512 KB = 4 x 128 KB. Instead of tying A17/A18 to GND, give each a 10 kΩ pull-down to GND and a
switch to Vcc. Then one module holds 4 different IC15 images (e.g. stock, Rosetta v9, Rosetta v12, a sample bank)
selected by the switch. **Only switch with the power off.** With the switch at 0/0 it behaves exactly like the
tied-to-GND version. Burn the 4 images as one 512 KB file (bank 0 at 0x00000, 1 at 0x20000, 2 at 0x40000,
3 at 0x60000); the baker's x4 file is the same image 4 times.

## SST39SF010A version (128 KB)
Same table. Pins 1 and 30 are NC on the SST39SF010A: leave them open (no bank switch). Use the 128K file.

## Checks before first power-on
- Continuity: every row of the table, plus OE#→GND, WE#→Vcc, A17/A18→GND (or the pull-downs).
- No short between header pin 28 (Vcc) and 14 (GND).
- Notch / pin 1 of the module matches pin 1 of the IC15 socket.
- In the unit: header pin 20 goes low during reads (meter or scope).

Power off and unplug before fitting or removing the module.
