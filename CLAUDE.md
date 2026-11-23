# Project rules (for Claude, Codex and human contributors)

These rules apply to every change in this repository. Codex task prompts must repeat the relevant ones.

## Workflow

1. Discuss the approach first. No code, KiCad files or CAD files are generated until the maintainer explicitly
   approves with the phrase "môžeš generovať kód".
2. No Codex task prompt is sent unprompted; it is written only after explicit written go-ahead.
3. Record every design decision in `docs/decisions.md` (EN) and `docs/decisions.sk.md` (SK). The English text is
   authoritative.
4. Verify facts (pinouts, datasheet values, part availability) from datasheets and schematics before using them.
   Do not leave "to be verified" notes for the maintainer; only mark what genuinely needs a physical measurement.

## Language separation (critical)

- No human-language strings in code: no UI text, labels, messages or word lists in source files.
- All user-facing text lives in `lang/` (one file per language), referenced from code with a comment at the
  connection point.
- Code, comments, identifiers and commit messages are in English.

## Data files

- Hand-editable data (controller mappings, settings, BOM) uses **CSV**, not JSON.

## Hardware

- Carrier board: through-hole parts only, unless a revision explicitly switches to fab assembly.
- Every decision that affects the 3D-printed front panel (connector positions, heights) is exported as STEP.

## Licences

- Software, docs, mappings: GPL-3.0-only (`LICENSE`).
- Hardware under `hardware/`: CERN-OHL-S-2.0 (`hardware/LICENSE`).

## Documentation

- English is primary. Slovak versions use the `.sk.md` suffix next to the English file.
