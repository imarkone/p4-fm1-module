# Mechanika hlavnej dosky

[English](mechanical.md) · **Slovenčina**

Odmerané z mechanického balíka Waveshare pre ESP32-P4-WIFI6-Touch-LCD-4.3 (výkres a STEP model z 11. 4. 2026).
Samotné súbory tu nešírime; sú na
[stránke zdrojov Waveshare](https://docs.waveshare.com/ESP32-P4-WIFI6-Touch-LCD-4.3/Resources-And-Documents).
Pri rozdiele platí anglická verzia.

Súradnice podľa STEP modelu: počiatok v strede skla, X pozdĺž kratšej strany (66.8 mm), Y pozdĺž dlhšej strany
(114.4 mm, +Y je hrana s USB-C), Z = 0 na prednej strane skla, záporné smerom dozadu. Všetko v mm.

## Displej

| Položka | Hodnota |
|---|---|
| krycie sklo (lens OD) | 114.4 × 66.8, polomer rohov 2.5 |
| viditeľná plocha (lens VA) | 94.4 × 56.96; okraj 10 na kratších stranách, 4.92 na dlhších |
| hĺbka skla + panelu | 5.35 |

## Doska na zadnej strane

| Položka | Poloha | Poznámka |
|---|---|---|
| hlavné PCB | X −30.0 … 30.0, Y −49.0 … 53.5 (60.0 × 102.5), zadná plocha Z −7.15 | hrúbka 1.6 |
| montážne stĺpiky | 4 × Ø5.5, závit M2.5, na X ±25, Y −43.75 a +48.25 (rozostup 50 × 92) | vrch Z −11.18, 4.03 nad PCB |
| 40-pinová lišta J3 | telo X 22.47 … 29.08, Y −30.82 … 20.48 | SMD **samica** 2 × 20, rozstup 2.54; vrch Z −12.35 (5.1 nad PCB). Pin 1 (5 V) na konci +Y, pri konektore RTC batérie |
| 2 × USB-C | X ±3.8 … ±13.4, Y 47.4 … 55.0 | káble vstupujú z hrany +Y; diely siahajú po Z −10.4 |
| tlačidlá (BOOT, RESET, napájanie), microSD | pozdĺž dlhej hrany oproti J3 (−X) | musia ostať prístupné |
| najhlbší diel | J3 na Z −12.35 | všetko ostatné ≤ Z −11.5 |

## Dôsledky pre nosnú dosku

- Nosná doska sa nasadí na J3 THT **samčou** lištou 2 × 20 na spodnej strane.
- J3 je o 1.17 vyššia ako stĺpiky, takže nosná doska potrebuje dištančné stĺpiky navrch existujúcich M2.5; nesmie
  visieť len na lište (zaťažujú ju jacky a USB káble).
- Obrys nosnej dosky nechá voľnú hranu s USB-C (+Y), tlačidlá aj slot microSD (hrana −X).
