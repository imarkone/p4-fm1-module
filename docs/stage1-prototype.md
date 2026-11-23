# Stage 1: wired prototype (no PCB, no breadboard)

**English** · [Slovenčina](stage1-prototype.sk.md)

Goal: bring up the core system with ready-made modules and jumper wires before any PCB is designed.
Everything learned here feeds the carrier board.

No breadboard: breadboard contacts work loose and the setup cannot be carried around. Modules connect to the
P4-NANO header with female–female jumper wires; the LED and the encoder get soldered pigtails ending in Dupont
female connectors (see [Wiring method](#wiring-method)).

## What stage 1 proves

1. Display: FM-1 240×240 frame shown at 2× on the 4" DSI panel, touch working.
2. Audio: one I2S bus with the P4 as clock master; PCM5102A plays, PCM1808 records, both in sync.
3. Controls: EC11 encoder and push button, RGB status LED.
4. USB host: SMK-25 II connected to the P4-NANO USB-A port, MIDI messages visible in the serial log.

MIDI over TRS (IN/OUT) is stage 2 (carrier board); its pins are already reserved.

## Parts

| Part | Notes |
|---|---|
| Waveshare ESP32-P4-NANO | Male 2×13 headers on both sides; Type-C for power, flashing and serial log |
| Waveshare 4-DSI-TOUCH-A (4", 480×800) | ST7701 + GT911. Listed as tested in Waveshare's `esp32_p4_platform` BSP. Needs 5 V, ≥180 mA. Outline 108.30 × 65.10 mm, active area 86.40 × 51.84 mm |
| PCM5102A module (GY-PCM5102 type, purple, with 3.5 mm jack) | Line out |
| PCM1808 module | Line in; needs 5 V (analog) and 3.3 V (digital) |
| EC11 rotary encoder with push button | Soldered pigtail, 4 wires |
| 5 mm RGB LED, common cathode | Plus 3 series resistors, soldered pigtail, 4 wires |
| Female–female jumper wires, 20 cm | Modules to the P4-NANO header |
| M-VAVE SMK-25 II | USB-A on the P4-NANO |

### Why a plain RGB LED and not WS2812

The P4-NANO has no user LED (only a power LED). A WS2812 needs ≥3.5 V supply and its data input expects about
0.7 × VDD, which is above the P4's 3.3 V logic. A common-cathode RGB LED on three PWM pins works at 3.3 V with no
level shifting and no timing requirements. The carrier board will use the same part (through-hole).

## P4-NANO header pinout (top view, as printed by Waveshare)

Left header (pins 1–26, odd = outer column, even = inner column):

| Pin | Net | Pin | Net |
|---|---|---|---|
| 1 | 3V3 | 2 | 5V |
| 3 | GPIO7 (I2C SDA, onboard) | 4 | 5V |
| 5 | GPIO8 (I2C SCL, onboard) | 6 | GND |
| 7 | GPIO23 | 8 | GPIO37 (UART0 TX, console) |
| 9 | GND | 10 | GPIO38 (UART0 RX, console) |
| 11 | GPIO5 | 12 | GPIO4 |
| 13 | GPIO20 | 14 | GND |
| 15 | GPIO21 | 16 | GPIO22 |
| 17 | 3V3 | 18 | GPIO24 (USB FS) |
| 19 | GPIO25 (USB FS) | 20 | GND |
| 21 | GPIO26 (USB FS) | 22 | GPIO27 (USB FS) |
| 23 | GPIO32 | 24 | GPIO33 |
| 25 | GND | 26 | GPIO36 (strapping) |

Right header:

| Pin | Net | Pin | Net |
|---|---|---|---|
| 1 | 5V | 2 | ESP_LDO_VO4 |
| 3 | GND | 4 | GND |
| 5 | 3V3 | 6 | GPIO0 (XTAL 32k) |
| 7 | GND | 8 | GPIO1 (XTAL 32k) |
| 9 | GPIO3 | 10 | GND |
| 11 | GPIO2 | 12 | GPIO6 |
| 13 | GPIO54 | 14 | GPIO53 (onboard speaker amp enable) |
| 15 | GPIO47 | 16 | GPIO48 |
| 17 | GPIO46 | 18 | GND |
| 19 | GPIO45 | 20 | C6_U0RXD |
| 21 | C6_IO12 | 22 | C6_U0TXD |
| 23 | C6_IO13 | 24 | C6_IO9 |
| 25 | GND | 26 | GND |

Onboard, not on the headers: ES8311 codec I2S on GPIO9–13, microSD on GPIO39–44, Ethernet PHY, ESP32-C6 over SDIO.

The machine-readable allocation is [pinmap.csv](pinmap.csv); firmware and the KiCad design follow it.

## Wiring

### Audio (one I2S bus, P4 is master)

| Signal | P4 GPIO (header pin) | PCM5102A module | PCM1808 module |
|---|---|---|---|
| MCLK | GPIO20 (L13) | SCK | SCK |
| BCLK | GPIO21 (L15) | BCK | BCK |
| WS / LRCK | GPIO22 (L16) | LCK | LRC |
| Data to DAC | GPIO23 (L7) | DIN | – |
| Data from ADC | GPIO32 (L23) | – | OUT |
| 3.3 V | 3V3 (L17) | VIN | 3.3V |
| 5 V | 5V (L2/L4) | – | +5V |
| GND | GND | GND | GND |

Module configuration:

- **PCM5102A (GY-PCM5102):** set the four solder bridges on the back: FLT = L, DEMP = L, XSMT = H (unmute),
  FMT = L (I2S). With XSMT left open the module is silent.
- **PCM1808:** MD0 = L, MD1 = L (slave mode), FMT = L (I2S); on the LaskaKit module FMT is printed as "PMT". The chip accepts a system clock of 256, 384 or 512 fs;
  firmware drives 256 fs.

### Encoder and LED

| Part | P4 GPIO (header pin) | Notes |
|---|---|---|
| EC11 A | GPIO45 (R19) | Internal pull-ups |
| EC11 B | GPIO46 (R17) | |
| EC11 switch | GPIO54 (R13) | |
| EC11 common pin + other switch pin | GND (R18) | Join both on the encoder side, one wire |
| LED red | GPIO2 (R11) | via 220 Ω |
| LED green | GPIO3 (R9) | via 100 Ω |
| LED blue | GPIO6 (R12) | via 100 Ω |
| LED common cathode | GND (R10) | |

Resistor values keep each colour at roughly 3–6 mA from 3.3 V (red Vf ≈ 2 V, green/blue Vf ≈ 3 V).

### Wiring method

- **LED:** the three resistors sit inline in the R, G and B wires (soldered into the cable, covered with
  heat-shrink); the four wires end in Dupont female contacts. R9 (G), R10 (GND), R11 (R) and R12 (B) form a 2×2 block on the right header, so one
  2×2 Dupont housing plugs the whole LED on in one move and cannot be plugged in shifted by a row.
- **Encoder:** four wires (A, B, switch, GND) to Dupont female contacts on R19, R17, R13 and R18. On the encoder
  side the common pin and the second switch pin are joined, so a single GND wire is enough.
- **Modules:** female–female jumper wires from the module pins to the P4-NANO header.
- On the carrier board the LED, resistors and encoder are soldered to the PCB; nobody wires them by hand.

### Display

DSI cable from the panel to the P4-NANO DSI connector. Touch (GT911) shares the onboard I2C bus (GPIO7/8).
The panel needs 5 V at ≥180 mA; use the supplied power lead to a 5V and GND pin on the header.

### Controller

SMK-25 II into the P4-NANO USB-A port. The P4-NANO is powered from USB-C.

## Reserved for stage 2 (carrier board)

| Function | GPIO |
|---|---|
| TRS MIDI OUT | GPIO47 (R15) |
| TRS MIDI IN | GPIO48 (R16) |
| Spare | GPIO4, GPIO5, GPIO33 |

## Pins not to use

- GPIO7/8: onboard I2C (codec, touch).
- GPIO53: onboard speaker amplifier enable.
- GPIO37/38: console UART (serial log through the CH343 on USB-C), strapping pins.
- GPIO36: strapping pin.
- GPIO0/1: 32 kHz crystal.
- GPIO24–27: USB full-speed PHY.

## Known limits of stage 1

- The PCM1808 analog supply comes straight from USB 5 V; some noise is expected. The carrier board adds filtering.
- The PCM5102A module output is a line output. Use line inputs (audio interface, active monitors). Headphones only at
  low volume until the NJM4556 stage exists.
- The line input has no protection yet; feed it line level only.
