# p4-fm1-module

[English](README.md) · **Slovenčina**

Open-source desktopový syntetizátorový modul postavený na **ESP32-P4**. Spúšťa komunitný firmvér písaný pre vreckový
syntetizátor **M-VAVE FM-1** (začíname s [Felucca](https://github.com/hugelton/Felucca)), s väčšou pamäťou, väčším
displejom, stereo audio vstupom aj výstupom, MIDI IN/OUT a externým USB-MIDI ovládačom ako ovládacím panelom.

> **Stav:** koncept / plánovanie. Hardvér ani firmvér zatiaľ neexistujú. Pozri [docs/decisions.sk.md](docs/decisions.sk.md).

## Myšlienka

Ekosystém FM-1 má už čisto oddelený firmvér od hardvéru (`felucca-bsp`, `fm1-base` s host simulátorom). Tento projekt
pridáva novú hardvérovú vrstvu (BSP) pre ESP32-P4 a existujúci firmvér na nej beží bez úprav.

Ovládací panel FM-1 (27 kláves, 7 enkóderov + MASTER, 14 tlačidiel s LED) sa hardvérovo nekopíruje. Namiesto neho sa
bežný USB-MIDI ovládač mapuje na panel FM-1 cez upraviteľný CSV profil.

## Cieľový hardvér (v1)

| Blok | Diel |
|---|---|
| Hlavná doska + displej | Waveshare ESP32-P4-WIFI6-Touch-LCD-4.3 (ESP32-P4 + ESP32-C6, 32 MB PSRAM, microSD, vstavaný 4.3" dotykový panel 480×800; obraz FM-1 240×240 v presnom 2× zväčšení) |
| Audio výstup | modul DAC PCM5102A (linkový výstup) |
| Audio vstup | modul ADC PCM1808 (linkový vstup s ochranou, útlm 0 / −12 dB) |
| Slúchadlá | zosilňovač NJM4556AD (DIP-8) |
| MIDI | 2× 3.5 mm TRS MIDI (IN cez optočlen H11L1, OUT), Type A/B voliteľné prepojkou |
| Debug | enkóder EC11 s tlačidlom + RGB stavová LED |
| Ovládač | M-VAVE SMK-25 II (25 kláves, 8 nekonečných enkóderov, 16 padov) cez USB host; iné ovládače cez CSV profily |
| Napájanie | USB-C, 5 V |

Nosná doska je **len THT (vývodové súčiastky)**, dá sa spájkovať ručne. Predný panel je 3D tlačený.

## Štruktúra repozitára

| Cesta | Obsah |
|---|---|
| `hardware/kicad/` | Nosná doska, nasadí sa na 40-pinovú lištu hlavnej dosky (KiCad) |
| `hardware/enclosure/` | Parametrická 3D krabica a predný panel |
| `firmware/` | ESP-IDF projekt, hardvérová vrstva pre ESP32-P4 |
| `mapping/` | CSV profily ovládačov (MIDI správa → ovládací prvok FM-1) |
| `lang/` | Všetky texty rozhrania, podľa jazyka |
| `docs/` | Rozhodnutia, BOM, zapojenie, postup zostavenia |

## Licencie

| Časť | Licencia |
|---|---|
| Firmvér, nástroje, mapovania, dokumentácia | [GPL-3.0-only](LICENSE) (povinné, firmvér stojí na Felucce, ktorá je GPL-3.0-only) |
| Hardvér (`hardware/`) | [CERN-OHL-S-2.0](hardware/LICENSE) |

## Poďakovanie

- Felucca od Leo Kuroshita (Hügelton Instruments), GPL-3.0-only
- Komunitné firmvérové projekty pre M-VAVE FM-1, na ktorých tento projekt stavia

Projekt nie je prepojený so spoločnosťou M-VAVE.
