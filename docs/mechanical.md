# Main board mechanics

**English** · [Slovenčina](mechanical.sk.md)

Measured from Waveshare's mechanical package for the ESP32-P4-WIFI6-Touch-LCD-4.3 (drawing and STEP model dated
2026-04-11). The files themselves are not redistributed here; download them from the
[Waveshare resources page](https://docs.waveshare.com/ESP32-P4-WIFI6-Touch-LCD-4.3/Resources-And-Documents).

Coordinates follow the STEP model: origin at the centre of the glass, X along the short side (66.8 mm),
Y along the long side (114.4 mm, +Y is the USB-C edge), Z = 0 on the front of the glass, negative towards the back.
All values in mm.

## Display

| Item | Value |
|---|---|
| Cover glass (lens OD) | 114.4 × 66.8, corner radius 2.5 |
| Visible area (lens VA) | 94.4 × 56.96; border 10 on the short sides, 4.92 on the long sides |
| Glass + panel depth | 5.35 |

## Board on the back

| Item | Position | Notes |
|---|---|---|
| Main PCB | X −30.0 … 30.0, Y −49.0 … 53.5 (60.0 × 102.5), back face Z −7.15 | 1.6 thick |
| Mounting standoffs | 4 × Ø5.5, M2.5 thread, at X ±25, Y −43.75 and +48.25 (pitch 50 × 92) | top face Z −11.18, 4.03 above the PCB |
| 40-pin header J3 | body X 22.47 … 29.08, Y −30.82 … 20.48 | SMD **female** 2 × 20, 2.54 pitch; top Z −12.35 (5.1 above the PCB). Pin 1 (5 V) at the +Y end, next to the RTC battery connector |
| 2 × USB-C | X ±3.8 … ±13.4, Y 47.4 … 55.0 | cables enter from the +Y edge; parts reach Z −10.4 |
| Buttons (BOOT, RESET, power), microSD | along the long edge opposite J3 (−X) | must stay reachable |
| Deepest part | J3 at Z −12.35 | everything else ≤ Z −11.5 |

## Consequences for the carrier board

- The carrier plugs onto J3 with a THT **male** 2 × 20 header on its underside.
- J3 stands 1.17 above the standoffs, so the carrier needs spacers on top of the existing M2.5 standoffs; it must not
  hang on the header alone (jacks and USB cables load it).
- The carrier outline keeps the USB-C edge (+Y), the buttons and the microSD slot (−X edge) free.
