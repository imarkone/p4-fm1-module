# Design decisions

**English** · [Slovenčina](decisions.sk.md)

Each entry: the decision, why, and what was rejected. Newest decisions are added at the end.

## D1. Port the FM-1 firmware, do not write a new synth engine

- **Decision:** add an ESP32-P4 board support layer under the existing FM-1 firmware (Felucca first) and run it unchanged.
- **Why:** Felucca 1.7.0 moved the FM-1 hardware layer into `felucca-bsp`; `fm1-base` already provides a host simulator
  and browser emulator. FM1VST shows the same idea works: firmwares run unmodified behind a panel abstraction.
- **Rejected:** emulating the FM-1 CPU (JieLi AC791N, dual pi32v2 @ 240 MHz) on the P4 with fm1-emu. Not real-time on a
  360 MHz RISC-V.

## D2. Licence: GPL-3.0 for software, CERN-OHL-S-2.0 for hardware

- **Why:** Felucca is GPL-3.0-only, so the combined firmware must be GPL-3.0. The hardware gets the equivalent
  strongly reciprocal hardware licence.

## D3. Main board: Waveshare ESP32-P4-NANO

- **Why:** ESP32-P4 + ESP32-C6, 32 MB PSRAM, microSD, MIPI-DSI, USB 2.0 HS OTG on a Type-A port (USB host for the
  controller), pin headers for a carrier board.
- **Rejected:** M5Stack Tab5 (tablet form factor), ESP32-P4 Knob (single encoder, older co-processor),
  Waveshare ESP32-P4-WIFI6-Touch-LCD-4B (720×720 = exact 3× scale, but only 5 free GPIOs and mono ES8311).

## D4. Display: 4" MIPI-DSI 480×800 touch

- **Why:** the FM-1 screen is 240×240. At an exact 2× scale it fills 480×480, leaving a 480×320 strip for extra views.
  Touch allows settings (e.g. MIDI channels) without extra controls.

## D5. Front panel = external USB-MIDI controller, mapped by CSV

- **Decision:** no FM-1 keys, encoders or buttons are rebuilt on the module. A USB-MIDI controller plugs into the P4
  USB-A host port. A CSV profile maps MIDI messages to FM-1 panel events (keys, encoders, buttons).
- **Reference controller:** M-VAVE SMK-25 II: 25 velocity keys (FM-1 has 27, octave shift covers the rest),
  8 endless 360° encoders (= FM-1 7 encoders + MASTER), 16 RGB pads (= FM-1 14 buttons + 2 spare).
- **Why:** mechanical keys and velocity pads are the hardest and most expensive part to build; a ready controller is
  durable and cheap. The module stays small.
- **Rules for the mapping layer:**
  - Notes used as FM-1 panel keys and notes played as music must be separated (dedicated MIDI channel or a mode pad),
    otherwise sounds trigger twice.
  - Encoders sending absolute 0–127 values are converted to relative steps; true relative mode is preferred.
- **Not documented, to be measured with a MIDI monitor:** whether the SMK-25 II encoders send relative values, and
  whether pad colours can be set by the host over MIDI.
- **Rejected:**
  - Building the P4 into a cheap MIDI keyboard (re-wiring its matrix is more work than a PCB; usually pots, no LED
    control).
  - NeoTrellis / NeoKey grids (module becomes too wide; no velocity; silicone pads do not survive drumming).
  - Novation Launchkey MK3 as reference: its knobs are pots with end stops (endless encoders came with MK4).
- **Buying note:** "SMK-25" is used for three different products. Only **SMK-25 II** (16 pads, 8 encoders) fits.
  SMK-25 / MVA-25 has 8 pads; the original SMK-25 mini has 1 knob.

## D6. Audio: PCM5102A + PCM1808 modules, not ES8388

| | DAC SNR / THD+N | ADC SNR / THD+N |
|---|---|---|
| ES8388 | 96 dB / −83 dB | 95 dB / −85 dB |
| PCM5102A + PCM1808 | 112 dB / −93 dB | 99 dB / −93 dB |

- **Why:** better figures, no I2C codec driver (hardware-configured), one I2S bus with the P4 as clock master, so
  input and output are sample-synchronous. Both exist as pin-header modules, which keeps the carrier board THT-only.
  ES8388 is QFN-28 and cannot be hand-soldered.
- **Cost:** PCM5102A has no headphone amplifier, hence D7. PCM1808 needs MCLK (256/384/512 fs) from the P4.
- **Open-source risk:** module clones differ in pinout. v1 targets one specific module, documented with photo and
  pinout. A later revision may place the chips directly (JLC SMT assembly) for reproducibility.

## D7. Headphone output: NJM4556AD

- **Why:** DIP-8, 70 mA per channel, a long-standing headphone driver choice. Socketed, so a substitute (e.g. 4580) fits.
- **Limits:** marked "Not For New Designs" by the maker but widely available. On a single 5 V supply the output is
  about 0.8 Vrms: loud enough on 32 Ω headphones, weak on 250 Ω and above. A ±5 V converter is possible later.
- Volume is digital (MASTER encoder via mapping), no potentiometer.

## D8. Connectors: all 3.5 mm, through-hole

Line out, line in, headphones, MIDI IN, MIDI OUT: 5× 3.5 mm TRS jacks, PCB-mounted, on one board edge at one height
so the 3D-printed front panel lines up.

## D9. MIDI: TRS IN and OUT on the carrier board

- **Why:** the FM-1 has TRS MIDI IN only (plus USB and BLE MIDI). OUT adds clock and note output to other gear.
- IN: H11L1 optocoupler (galvanically isolated, MIDI standard). OUT: series resistors per the 3.3 V MIDI spec.
- TRS Type A and Type B both exist in the market; a jumper per jack selects the wiring. Default: Type A (FM-1, MMA RP-054).
- Felucca does not send hardware MIDI OUT by itself; routing to the UART is added in the P4 board layer.

## D10. Input / output protection ("anyone can plug in anything")

| Port | Protection |
|---|---|
| Line in | DC-blocking capacitor, 0 / −12 dB jumper pad, series resistor, 1N4148 clamp diodes. PCM1808 full scale is 3 Vpp (≈1 Vrms), so +4 dBu pro line and Eurorack levels need the pad. |
| Line out | 100 Ω series resistors (short-circuit safe) |
| Headphones | ~10 Ω series resistors on the NJM4556 outputs |
| MIDI IN | Optocoupler isolation |
| MIDI OUT | Series resistors |

## D11. Debug and fallback controls

- EC11 encoder with push button and one RGB status LED, directly on GPIO (no module).
- Uses: hold at boot = safe mode / settings reset; menu navigation if touch fails.
- Real debugging is the serial log over the P4-NANO USB-C UART. The LED shows state when the display is dead
  (boot OK, no audio, no MIDI device, display error).
- Settings and mappings are CSV files on microSD, editable on a PC when everything else fails.

## D12. Carrier board: through-hole only, made by a PCB fab

- No perfboard. KiCad design, ordered from a Chinese PCB fab, hand-soldered. SMD only if a later revision uses
  fab assembly.
- The board sits behind a 3D-printed front panel; KiCad exports STEP so the panel is modelled to the real board.

## D13. Power: USB-C 5 V only

No battery in v1.

## D14. Stage 1 is a wired prototype with ready-made modules

- **Decision:** before any PCB, bring up P4-NANO + 4" DSI display + PCM5102A + PCM1808 + EC11 + RGB LED + SMK-25 II
  on jumper wires. See [stage1-prototype.md](stage1-prototype.md).
- **Why:** proves display, audio clocking, USB host and controls cheaply; the carrier board follows what works.
- **Pin allocation:** [pinmap.csv](pinmap.csv) is the single source of truth for firmware and KiCad.

## D15. Status LED: common-cathode RGB LED on three PWM pins, not WS2812

- **Why:** the P4-NANO has only a power LED. WS2812 needs ≥3.5 V supply and a data level above 3.3 V logic;
  a plain RGB LED runs directly from 3.3 V GPIOs, also as a through-hole part on the carrier board.

## D16. Display 4-DSI-TOUCH-A confirmed for ESP32-P4

- Waveshare's `esp32_p4_platform` BSP lists 4-DSI-TOUCH-A (ST7701, `esp_lcd_st7701`) as tested.
- The panel is natively portrait 480×800: the FM-1 240×240 frame at 2× fills the top 480×480, leaving 480×320 below
  without any rotation.
- Power: 5 V, ≥180 mA.

## D17. Stage 1 without a breadboard: soldered pigtails with Dupont connectors

- **Decision:** no breadboard. Modules connect to the P4-NANO header with female–female jumper wires. The RGB LED
  with its resistors and the encoder get soldered pigtails ending in Dupont female contacts; the LED plugs on as one
  2×2 housing on R9–R12.
- **Why:** a breadboard comes apart when carried or knocked and adds unreliable contacts while debugging.
  Soldered pigtails survive transport and testing.
- **Carrier board:** the LED, resistors and encoder are on the PCB; users wire nothing by hand.
