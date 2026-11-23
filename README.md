# p4-fm1-module

**English** · [Slovenčina](README.sk.md)

An open-source desktop synth module built on the **ESP32-P4**. It runs the community firmware written for the
**M-VAVE FM-1** pocket synth (starting with [Felucca](https://github.com/hugelton/Felucca)), with more memory, a larger
display, stereo audio in/out, MIDI IN/OUT and an external USB-MIDI controller as the front panel.

> **Status:** concept / planning. No hardware or firmware yet. See [docs/decisions.md](docs/decisions.md).

## Idea

The FM-1 ecosystem already has a clean split between firmware and hardware (`felucca-bsp`, `fm1-base` with a host
simulator). This project adds a new board support layer for the ESP32-P4 and runs the existing firmware unchanged on it.

The FM-1 front panel (27 keys, 7 encoders + MASTER, 14 buttons with LEDs) is not rebuilt in hardware. Instead, an
off-the-shelf USB-MIDI controller is mapped onto the FM-1 panel through an editable CSV profile.

## Target hardware (v1)

| Block | Part |
|---|---|
| Main board | Waveshare ESP32-P4-NANO (ESP32-P4 + ESP32-C6, 32 MB PSRAM, microSD, USB-A host) |
| Display | Waveshare 4" MIPI-DSI 480×800 touch (FM-1 240×240 screen shown at exact 2× scale) |
| Audio out | PCM5102A DAC module (line out) |
| Audio in | PCM1808 ADC module (line in, protected, 0 / −12 dB pad) |
| Headphones | NJM4556AD (DIP-8) headphone driver |
| MIDI | 2× 3.5 mm TRS MIDI (IN via H11L1 optocoupler, OUT), Type A/B selectable by jumper |
| Debug | EC11 encoder with push button + RGB status LED |
| Controller | M-VAVE SMK-25 II (25 keys, 8 endless encoders, 16 pads) over USB host; other controllers via CSV profiles |
| Power | USB-C, 5 V |

The carrier board is **through-hole only**, so it can be hand-soldered. The front panel is a 3D-printed part.

## Repository layout

| Path | Content |
|---|---|
| `hardware/kicad/` | Carrier board for the P4-NANO (KiCad) |
| `hardware/enclosure/` | Parametric 3D enclosure and front panel |
| `firmware/` | ESP-IDF project, ESP32-P4 board support layer |
| `mapping/` | CSV controller profiles (MIDI message → FM-1 panel control) |
| `lang/` | All user-facing text, per language |
| `docs/` | Design decisions, BOM, wiring, assembly notes |

## Licences

| Part | Licence |
|---|---|
| Firmware, tools, mappings, docs | [GPL-3.0-only](LICENSE) (required, the firmware builds on Felucca, which is GPL-3.0-only) |
| Hardware (`hardware/`) | [CERN-OHL-S-2.0](hardware/LICENSE) |

## Credits

- Felucca by Leo Kuroshita (Hügelton Instruments), GPL-3.0-only
- The M-VAVE FM-1 community firmware projects this work builds on

Not affiliated with M-VAVE.
