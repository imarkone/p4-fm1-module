# Etapa 1: prototyp na kábloch (bez PCB, bez breadboardu)

[English](stage1-prototype.md) · **Slovenčina**

Cieľ: rozbehať jadro systému z hotových modulov a prepojovacích káblikov skôr, než navrhneme akékoľvek PCB.
Všetko, čo sa tu naučíme, ide do nosnej dosky. Pri rozdiele platí anglická verzia.

Bez breadboardu: kontakty v ňom sa ľahko uvoľnia a zostava sa nedá prenášať. Všetky moduly vrátane LED a enkódera idú na
40-pinovú lištu káblikmi samica–samica (pozri [Spôsob zapojenia](#spôsob-zapojenia)).

## Čo etapa 1 overí

1. Displej: obraz FM-1 240×240 v 2× zväčšení na vstavanom 4.3" paneli, funkčný dotyk.
2. Zvuk: jedna I2S zbernica, P4 je zdroj hodín; PCM5102A hrá, PCM1808 nahráva, obe synchrónne.
3. Ovládanie: enkóder EC11 s tlačidlom, RGB stavová LED.
4. USB host: SMK-25 II na OTG porte, MIDI správy viditeľné v sériovom logu.

MIDI cez TRS (IN/OUT) je etapa 2 (nosná doska); piny sú už rezervované.

## Diely

| Diel | Poznámka |
|---|---|
| Waveshare ESP32-P4-WIFI6-Touch-LCD-4.3 (WS-33874) | doska ESP32-P4 pripevnená na zadnej strane 4.3" dotykového panelu 480×800 (ST7701 + GT911). 40-pinová lišta (J3). Dva Type-C: USB-UART (napájanie, nahrávanie, sériový log) a USB 2.0 HS OTG |
| modul PCM5102A (typ GY-PCM5102, fialový, s 3.5 mm jackom) | linkový výstup |
| modul PCM1808 | linkový vstup; potrebuje 5 V (analóg) a 3.3 V (digitál) |
| modul enkódera (EC11, 20 impulzov na otáčku, tlačidlo; piny GND, S1, S2, Key, 5V) | pull-upy a RC filter na module; napájanie 3.3 V |
| RGB LED modul KY-016 (piny −, R, G, B) | spoločná katóda, odpory 150 Ω na module |
| káblíky samica–samica, 20 cm | moduly na 40-pinovú lištu |
| USB-A samica, káblový spájkovací konektor | vývody na J3 pre ovládač (pozri [Ovládač](#ovládač)) |
| M-VAVE SMK-25 II | USB-MIDI ovládač |

### Prečo obyčajná RGB LED a nie WS2812

Doska nemá používateľskú LED (len LED napájania). WS2812 potrebuje napájanie aspoň 3.5 V a jej dátový vstup
čaká asi 0.7 × VDD, čo je nad 3.3 V logikou P4. RGB LED so spoločnou katódou na troch PWM pinoch funguje na 3.3 V
bez prevodníka úrovní a bez časovania. Nosná doska použije ten istý diel (THT).

## 40-pinová lišta J3 (číslovanie podľa potlače Waveshare)

Pin 1 je tu 5 V, nie 3.3 V ako na Raspberry Pi. Pred zapojením skontroluj potlač.

| Pin | Signál | Pin | Signál |
|---|---|---|---|
| 1 | 5V (VCC_5V) | 2 | 3V3 |
| 3 | 5V (VCC_5V) | 4 | GPIO7 (I2C SDA, na doske) |
| 5 | GND | 6 | GPIO8 (I2C SCL, na doske) |
| 7 | GPIO37 (TX konzoly) | 8 | GPIO2 (INT dotyku, nepoužívať) |
| 9 | GPIO38 (RX konzoly) | 10 | GND |
| 11 | GPIO5 | 12 | GPIO3 |
| 13 | GND | 14 | GPIO4 |
| 15 | GPIO21 | 16 | GPIO28 |
| 17 | GPIO22 | 18 | 3V3 |
| 19 | GND | 20 | GPIO29 |
| 21 | GPIO24 (USB FS) | 22 | GPIO30 |
| 23 | GPIO25 (USB FS) | 24 | GPIO31 |
| 25 | USBD_N (USB HS OTG, nie GPIO) | 26 | GND |
| 27 | USBD_P (USB HS OTG, nie GPIO) | 28 | GPIO34 (strapping) |
| 29 | GND | 30 | GPIO35 (strapping) |
| 31 | GPIO32 | 32 | GPIO49 |
| 33 | GND | 34 | GPIO50 |
| 35 | GPIO46 | 36 | GPIO51 |
| 37 | GPIO47 | 38 | GPIO52 |
| 39 | GPIO48 | 40 | GND |

Na doske, mimo lišty: I2S kodeku ES8311/ES7210 na GPIO9–13, zapnutie zosilňovača GPIO53, reset LCD GPIO27,
podsvietenie GPIO26/GPIO33, reset dotyku GPIO23, microSD na GPIO39–44, ESP32-C6 cez SDIO.

Strojovo čitateľné pridelenie je [pinmap.csv](pinmap.csv); firmvér aj návrh v KiCade sa riadia ním.

## Zapojenie

### Zvuk (jedna I2S zbernica, P4 je master)

| Signál | GPIO na P4 (pin J3) | modul PCM5102A | modul PCM1808 |
|---|---|---|---|
| MCLK | GPIO21 (15) | SCK | SCK |
| BCLK | GPIO22 (17) | BCK | BCK |
| WS / LRCK | GPIO28 (16) | LCK | LRC |
| dáta do DAC | GPIO5 (11) | DIN | – |
| dáta z ADC | GPIO4 (14) | – | OUT |
| 3.3 V | 3V3 (2, 18) | – | 3.3V |
| 5 V | 5V (1, 3) | VIN | +5V |
| GND | GND (5, 13, 19) | GND | GND |

Modul PCM5102A má vlastné 3.3 V stabilizátory, takže VIN berie 5 V; jeho logika aj tak beží na 3.3 V a sedí s P4.

Nastavenie modulov:

- **PCM5102A (GY-PCM5102):** štyri mostíky na zadnej strane sú H1L = FLT, H2L = DEMP, H3L = XSMT, H4L = FMT.
  Nastav H1L = L, H2L = L, H3L = H (zrušenie stlmenia), H4L = L (I2S). Pri XSMT v stave L je modul ticho.
  Dve malé plôšky vedľa SCK na prednej strane uzemňujú SCK; musia ostať **rozpojené**, lebo SCK nesie MCLK z P4.
- **PCM1808:** MD0 = L, MD1 = L (slave režim), FMT = L (I2S); na module LaskaKit je FMT popísaný ako „PMT“.
  Čip berie systémové hodiny 256, 384 alebo 512 fs; firmvér pošle 256 fs.

### Enkóder a LED (etapa 1 používa hotové moduly)

| Pin modulu | GPIO na P4 (pin J3) | Poznámka |
|---|---|---|
| RGB modul (KY-016) R | GPIO32 (31) | 150 Ω na module |
| RGB modul G | GPIO49 (32) | 150 Ω na module |
| RGB modul − (spoločná katóda) | GND (33) | |
| RGB modul B | GPIO50 (34) | 150 Ω na module |
| modul enkódera S1 | GPIO47 (37) | 10 kΩ pull-up a RC filter na module |
| modul enkódera S2 | GPIO52 (38) | |
| modul enkódera Key | GPIO48 (39) | |
| modul enkódera GND | GND (40) | |
| modul enkódera „5V“ | **3V3 (18)** | nikdy nie 5 V: pull-upy by dali 5 V na piny P4, ktoré 5 V neznesú |

Odpory na moduloch sú rátané na 5 V; pri 3.3 V ide červená asi 8–9 mA, zelená a modrá 2–4 mA. Firmvér vyrovná
farby cez PWM.

### Spôsob zapojenia

- Samostatné káblíky samica–samica z pinov modulov na lištu; v etape 1 sa nič nespájkuje.
- Na nosnej doske sú LED, odpory aj enkóder prispájkované na PCB; nikto ich nezapája ručne.

### Displej

Je súčasťou dosky. Dotyk (GT911) je na I2C zbernici dosky (GPIO7/8); BSP ho číta periodicky, linka INT sa nepoužíva.

### Ovládač

OTG Type-C nemá výstup 5 V: jeho VBUS napája dosku cez obvod ideálnej diódy (AO3401 + MMDT3906) a spätný prúd je
zablokovaný. Ovládač zapojený cez obyčajnú OTG redukciu nedostane napájanie. Samotná doska sa napája z Type-C USB-UART.

Etapa 1 použije tú istú cestu ako nosná doska: USB-A samicu na vývodoch do J3.

| pin USB-A | pin J3 |
|---|---|
| 1 VBUS | 1 alebo 3 (5V) |
| 2 D− | 25 (USBD_N) |
| 3 D+ | 27 (USBD_P) |
| 4 GND | 26 |

Vodiče D−/D+ drž krátke (20 cm) a spolu skrútené. OTG Type-C na doske nechaj prázdny, zdieľa tie isté dátové linky.
Ovládač je full-speed zariadenie, prepojovacie káble stačia. Ako záloha funguje aj USB hub s napájaním na OTG Type-C.

Nosná doska to vyrieši natrvalo: dátové linky HS OTG sú aj na J3 (piny 25/27), takže nosná doska dostane vlastný
USB-A host port s 5 V zo svojho napájania.

## Rezervované pre etapu 2 (nosná doska)

| Funkcia | pin J3 |
|---|---|
| TRS MIDI OUT | GPIO46 (35) |
| TRS MIDI IN | GPIO51 (36) |
| USB-A host D− / D+ | USBD_N (25) / USBD_P (27) |
| napájanie 5 V do dosky | VCC_5V (1, 3) |
| voľné | GPIO3 (12), GPIO29 (20), GPIO30 (22), GPIO31 (24) |

## Piny, ktoré nepoužívať

- GPIO7/8: I2C na doske (kodek, ADC na potlačenie ozveny, dotyk).
- GPIO37/38: UART konzoly, strapping piny.
- GPIO34/35: strapping piny.
- GPIO2: INT dotyku na niektorých kusoch (R35 osadený alebo nie).
- GPIO24/25: USB full-speed PHY.

## Známe obmedzenia etapy 1

- Analógové napájanie PCM1808 ide priamo z USB 5 V; počítaj s trochou šumu. Nosná doska pridá filtráciu.
- Výstup modulu PCM5102A je linkový. Zapájaj do linkových vstupov (zvuková karta, aktívne monitory). Slúchadlá len
  potichu, kým nebude stupeň s NJM4556.
- Linkový vstup zatiaľ nemá ochranu; púšťaj doň len linkovú úroveň.
- Doska má medzi VCC_5V a napájaním systému tlačidlový vypínač (ECJ23001). Či sa po pripojení 5 V zapne sama, alebo
  treba stlačiť tlačidlo napájania, sa zistí pri prvom zapnutí.
