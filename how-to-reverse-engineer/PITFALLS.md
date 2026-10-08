# Pitfalls (bugs we hit on hardware or in the simulator, and the rules they taught; "(rule)" rows are rules from the code, not bugs we hit)

| Symptom | Cause | Rule |
|---|---|---|
| Quick screen v1: keys did nothing | pointer in word `r74`, max in `r75` = its high byte | never mix byte and word use of one register pair |
| v7/v7b: garbage values, edits did not stick | word read at an odd address (`ld r74,16[r78]`) | word data `even`-aligned; simulator flags odd accesses |
| v7b Info line half written | word store to an odd LCD buffer address | LCD buffer writes byte by byte (`put2`) |
| live resonance stopped the voice | LA32 `0x0D00/0x0D01` latch as a pair | write `0x0D00` (from shadow `0xEF80`) then `0x0D01` |
| Lab Motion value never moved | `ldbze r70` wrote `r71` too (input lost) | `ldbze` writes a word |
| first arp note blipped | step timer started near 0 | reset timers to "due now" deliberately |
| humanize only moved notes early | unsigned compare on a signed time | `jgt`/`jlt` for signed values |
| edits blocked on non-part items | a helper returned part = 0xFF for every kind | clear outputs on every path |
| data area overflowed into the next page | tables grew | build asserts sizes; move tables into code space |
| arithmetic on a memory variable | MCS-96 needs a register destination | load into a register, operate, store |
| (rule) `mulu`/`divu` long results | long destination not 4-aligned | 4-align long destinations |
| (rule) borrow flag after `sub`/`cmp` | MCS-96 C = 1 means *no* borrow | `jc` after compare = unsigned >= |
| branch out of range | short jumps only reach -128..127 | use `je!`/`jbc!` far forms |
| (rule) hooks must not break stock behaviour | registers the caller needs were changed | write down each hook's register contract and keep it |
| (rule) IC15 code calling IC19 routines | IC19 routine left another bank mapped | `call19` restores page 0x27 |
| (rule) changing chord / mono / unison / arp / vintage part settings | notes started under the old setting stay | `K_ALLOFF`: all notes off when such a setting changes |
| T48 "bad pin position" | chip placement/contact, not the profile | reseat; AM27C040 profile for HN62304B |
| (build lesson) SST in the IC15 slot | OE# pin touching the board pad below | lift SST pin 24 fully, ground it separately |

Before each burn: full test run, compare the patched image against the stock dump (only the intended areas may
differ), MD5 of the burn file, read back after burning and compare.
