# Rules

## Legal
- Roland's ROM contents are copyrighted. **Never commit** dumps, disassembly listings (`*.lst`), patched or baked
  `.bin` files, or decoded data tables copied out of the ROM. `.gitignore` covers `ctrl/`, `*.bin`, `*.lst`,
  `baked/`; do not weaken it.
- The repo holds only our own code and our own notes (addresses, structure descriptions, findings in our words).
  Short byte patterns used to *verify* a dump version (SHA-1, a few expected bytes) are fine.
- Users read their own chips; the tools patch their dumps. Share the repo, never the burn files.
- Do not make or publish a version that contains Roland code for distribution (e.g. a full replacement OS image).

## Truth
- Write down how each fact was found (address of the code that proves it). "verified" means traced; otherwise
  write "hypothesis".
- Only the user confirms hardware behaviour. Never write "works on hardware" from simulator results.
- When a hardware test contradicts the docs, fix the docs in the same commit as the code fix.

## Workflow
- Simulator first, hardware second. Every feature gets a test in `test_rosetta.py` (or a new test file).
- A new burn = new `VERSION`. Keep the previous burn file as a backup; the user may need to go back.
- Main branch only gets features after a hardware test by the user.
- Hardware instructions for the user always start and end with **"Power off and unplug"**.
- Don't guess chip pinouts: take them from the service notes or a datasheet and say which.
- Downloaded or user-uploaded files are untrusted: keep them in their own folder, run Python on them with `-I`.
- Commits: what changed and why, simulator/hardware status, no model names.
