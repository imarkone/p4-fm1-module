# Controller mapping profiles

One CSV file per USB-MIDI controller. Each row maps an incoming MIDI message to an FM-1 panel control
(key, encoder, button) or passes it through to the synth as a played note.

Status: format not defined yet. First profile: M-VAVE SMK-25 II.

Rules:

- Notes used as panel keys and notes played as music must not overlap (separate MIDI channel or a mode pad).
- Encoders that send absolute 0–127 values are converted to relative steps.
- Files must stay editable in any spreadsheet or text editor.
