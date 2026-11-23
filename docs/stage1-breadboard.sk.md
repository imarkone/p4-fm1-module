# Etapa 1: prototyp na breadboarde (bez PCB)

[English](stage1-breadboard.md) · **Slovenčina**

Cieľ: rozbehať jadro systému z hotových modulov a prepojovacích káblikov skôr, než navrhneme akékoľvek PCB.
Všetko, čo sa tu naučíme, ide do nosnej dosky. Pri rozdiele platí anglická verzia.

## Čo etapa 1 overí

1. Displej: obraz FM-1 240×240 v 2× zväčšení na 4" DSI paneli, funkčný dotyk.
2. Zvuk: jedna I2S zbernica, P4 je zdroj hodín; PCM5102A hrá, PCM1808 nahráva, obe synchrónne.
3. Ovládanie: enkóder EC11 s tlačidlom, RGB stavová LED.
4. USB host: SMK-25 II zapojený do USB-A na P4-NANO, MIDI správy viditeľné v sériovom logu.

MIDI cez TRS (IN/OUT) je etapa 2 (nosná doska); piny sú už rezervované.

## Diely

| Diel | Poznámka |
|---|---|
| Waveshare ESP32-P4-NANO | samčie lišty 2×13 na oboch stranách; Type-C na napájanie, nahrávanie a sériový log |
| Waveshare 4-DSI-TOUCH-A (4", 480×800) | ST7701 + GT911. Waveshare BSP `esp32_p4_platform` ho uvádza ako otestovaný. Potrebuje 5 V, ≥180 mA. Rozmer 108.30 × 65.10 mm, aktívna plocha 86.40 × 51.84 mm |
| modul PCM5102A (typ GY-PCM5102, fialový, s 3.5 mm jackom) | linkový výstup |
| modul PCM1808 | linkový vstup; potrebuje 5 V (analóg) a 3.3 V (digitál) |
| enkóder EC11 s tlačidlom | do breadboardu |
| 5 mm RGB LED, spoločná katóda | + 3 odpory (pozri nižšie) |
| breadboard, káblíky samica–samica a samica–samec | |
| M-VAVE SMK-25 II | do USB-A na P4-NANO |

### Prečo obyčajná RGB LED a nie WS2812

P4-NANO nemá používateľskú LED (len LED napájania). WS2812 potrebuje napájanie aspoň 3.5 V a jej dátový vstup
čaká asi 0.7 × VDD, čo je nad 3.3 V logikou P4. RGB LED so spoločnou katódou na troch PWM pinoch funguje na 3.3 V
bez prevodníka úrovní a bez časovania. Nosná doska použije ten istý diel (THT).

## Rozloženie líšt P4-NANO (pohľad zhora, podľa Waveshare)

Ľavá lišta (piny 1–26, nepárne = vonkajší rad, párne = vnútorný rad):

| Pin | Signál | Pin | Signál |
|---|---|---|---|
| 1 | 3V3 | 2 | 5V |
| 3 | GPIO7 (I2C SDA, na doske) | 4 | 5V |
| 5 | GPIO8 (I2C SCL, na doske) | 6 | GND |
| 7 | GPIO23 | 8 | GPIO37 (UART0 TX, konzola) |
| 9 | GND | 10 | GPIO38 (UART0 RX, konzola) |
| 11 | GPIO5 | 12 | GPIO4 |
| 13 | GPIO20 | 14 | GND |
| 15 | GPIO21 | 16 | GPIO22 |
| 17 | 3V3 | 18 | GPIO24 (USB FS) |
| 19 | GPIO25 (USB FS) | 20 | GND |
| 21 | GPIO26 (USB FS) | 22 | GPIO27 (USB FS) |
| 23 | GPIO32 | 24 | GPIO33 |
| 25 | GND | 26 | GPIO36 (strapping) |

Pravá lišta:

| Pin | Signál | Pin | Signál |
|---|---|---|---|
| 1 | 5V | 2 | ESP_LDO_VO4 |
| 3 | GND | 4 | GND |
| 5 | 3V3 | 6 | GPIO0 (kryštál 32k) |
| 7 | GND | 8 | GPIO1 (kryštál 32k) |
| 9 | GPIO3 | 10 | GND |
| 11 | GPIO2 | 12 | GPIO6 |
| 13 | GPIO54 | 14 | GPIO53 (zapnutie zosilňovača reproduktora na doske) |
| 15 | GPIO47 | 16 | GPIO48 |
| 17 | GPIO46 | 18 | GND |
| 19 | GPIO45 | 20 | C6_U0RXD |
| 21 | C6_IO12 | 22 | C6_U0TXD |
| 23 | C6_IO13 | 24 | C6_IO9 |
| 25 | GND | 26 | GND |

Na doske, mimo líšt: I2S kodeku ES8311 na GPIO9–13, microSD na GPIO39–44, Ethernet, ESP32-C6 cez SDIO.

Strojovo čitateľné pridelenie pinov je v [pinmap.csv](pinmap.csv); firmvér aj KiCad návrh sa ním riadia.

## Zapojenie

### Zvuk (jedna I2S zbernica, P4 je master)

| Signál | GPIO na P4 (pin lišty) | modul PCM5102A | modul PCM1808 |
|---|---|---|---|
| MCLK | GPIO20 (Ľ13) | SCK | SCK |
| BCLK | GPIO21 (Ľ15) | BCK | BCK |
| WS / LRCK | GPIO22 (Ľ16) | LCK | LRC |
| Dáta do DAC | GPIO23 (Ľ7) | DIN | – |
| Dáta z ADC | GPIO32 (Ľ23) | – | OUT |
| 3.3 V | 3V3 (Ľ17) | VIN | 3.3V |
| 5 V | 5V (Ľ2/Ľ4) | – | +5V |
| GND | GND | GND | GND |

Nastavenie modulov:

- **PCM5102A (GY-PCM5102):** na zadnej strane zaletuj štyri mostíky: FLT = L, DEMP = L, XSMT = H (zrušenie stlmenia),
  FMT = L (I2S). Ak XSMT ostane rozpojený, modul je ticho.
- **PCM1808:** MD0 = L, MD1 = L (slave režim), FMT = L (I2S); na module LaskaKit je FMT popísaný ako „PMT“. Čip berie systémové hodiny 256, 384 alebo 512 fs;
  firmvér pošle 256 fs.

### Enkóder a LED

| Diel | GPIO na P4 (pin lišty) | Poznámka |
|---|---|---|
| EC11 A | GPIO45 (P19) | stredný pin enkódera na GND; interné pull-upy |
| EC11 B | GPIO46 (P17) | |
| EC11 tlačidlo | GPIO54 (P13) | druhý pin tlačidla na GND |
| LED červená | GPIO2 (P11) | cez 220 Ω |
| LED zelená | GPIO3 (P9) | cez 100 Ω |
| LED modrá | GPIO6 (P12) | cez 100 Ω |
| spoločná katóda LED | GND | |

Hodnoty odporov držia každú farbu na asi 3–6 mA z 3.3 V (červená Vf ≈ 2 V, zelená/modrá Vf ≈ 3 V).

### Displej

DSI kábel z panelu do DSI konektora na P4-NANO. Dotyk (GT911) je na I2C zbernici dosky (GPIO7/8).
Panel potrebuje 5 V a aspoň 180 mA; pribalený napájací káblik zapoj na pin 5V a GND na lište.

### Ovládač

SMK-25 II do USB-A na P4-NANO. P4-NANO sa napája z USB-C.

## Rezervované pre etapu 2 (nosná doska)

| Funkcia | GPIO |
|---|---|
| TRS MIDI OUT | GPIO47 (P15) |
| TRS MIDI IN | GPIO48 (P16) |
| Voľné | GPIO4, GPIO5, GPIO33 |

## Piny, ktoré nepoužívať

- GPIO7/8: I2C na doske (kodek, dotyk).
- GPIO53: zapnutie zosilňovača reproduktora na doske.
- GPIO37/38: UART konzoly (sériový log cez CH343 na USB-C), strapping piny.
- GPIO36: strapping pin.
- GPIO0/1: kryštál 32 kHz.
- GPIO24–27: USB full-speed PHY.

## Známe obmedzenia etapy 1

- Analógové napájanie PCM1808 ide priamo z USB 5 V; počítaj s trochou šumu. Nosná doska pridá filtráciu.
- Výstup modulu PCM5102A je linkový. Zapájaj do linkových vstupov (zvuková karta, aktívne monitory). Slúchadlá len
  potichu, kým nebude stupeň s NJM4556.
- Linkový vstup zatiaľ nemá ochranu; púšťaj doň len linkovú úroveň.
