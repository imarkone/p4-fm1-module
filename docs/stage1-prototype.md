# Stage 1: wired prototype (no PCB, no breadboard)

**English** · [Slovenčina](stage1-prototype.sk.md)

Goal: bring up the core system with ready-made modules and jumper wires before any PCB is designed.
Everything learned here feeds the carrier board.

No breadboard: breadboard contacts work loose and the setup cannot be carried around. All modules, including the LED and the encoder,
connect to the 40-pin header with female–female jumper wires (see [Wiring method](#wiring-method)).

## What stage 1 proves

1. Display: FM-1 240×240 frame shown at 2× on the built-in 4.3" panel, touch working.
2. Audio: one I2S bus with the P4 as clock master; PCM5102A plays, PCM1808 records, both in sync.
3. Controls: EC11 encoder and push button, RGB status LED.
4. USB host: SMK-25 II on the OTG port, MIDI messages visible in the serial log.

MIDI over TRS (IN/OUT) is stage 2 (carrier board); its pins are already reserved.

## Parts

| Part | Notes |
|---|---|
| Waveshare ESP32-P4-WIFI6-Touch-LCD-4.3 (WS-33874) | ESP32-P4 board fixed to the back of a 4.3" 480×800 touch panel (ST7701 + GT911). 40-pin header (J3). Two Type-C ports: USB-UART (power, flashing, serial log) and USB 2.0 HS OTG |
| PCM5102A module (GY-PCM5102 type, purple, with 3.5 mm jack) | Line out |
| PCM1808 module | Line in; needs 5 V (analog) and 3.3 V (digital) |
| Encoder module (EC11, 20 pulses/turn, push button; pins GND, S1, S2, Key, 5V) | Pull-ups and RC filter on the module; powered from 3.3 V |
| RGB LED module KY-016 (pins −, R, G, B) | Common cathode, 150 Ω resistors on the module |
| Female–female jumper wires, 20 cm | Modules to the 40-pin header |
| USB-A female cable-solder connector | Pigtail to J3 for the controller (see [Controller](#controller)) |
| M-VAVE SMK-25 II | USB-MIDI controller |

### Why a plain RGB LED and not WS2812

The board has no user LED (only a power LED). A WS2812 needs ≥3.5 V supply and its data input expects about
0.7 × VDD, which is above the P4's 3.3 V logic. A common-cathode RGB LED on three PWM pins works at 3.3 V with no
level shifting and no timing requirements. The carrier board will use the same part (through-hole).

## 40-pin header J3 (numbering as printed by Waveshare)

Pin 1 is 5 V here, not 3.3 V as on a Raspberry Pi. Check the silkscreen before plugging anything in.

| Pin | Net | Pin | Net |
|---|---|---|---|
| 1 | 5V (VCC_5V) | 2 | 3V3 |
| 3 | 5V (VCC_5V) | 4 | GPIO7 (I2C SDA, onboard) |
| 5 | GND | 6 | GPIO8 (I2C SCL, onboard) |
| 7 | GPIO37 (console TX) | 8 | GPIO2 (touch INT, avoid) |
| 9 | GPIO38 (console RX) | 10 | GND |
| 11 | GPIO5 | 12 | GPIO3 |
| 13 | GND | 14 | GPIO4 |
| 15 | GPIO21 | 16 | GPIO28 |
| 17 | GPIO22 | 18 | 3V3 |
| 19 | GND | 20 | GPIO29 |
| 21 | GPIO24 (USB FS) | 22 | GPIO30 |
| 23 | GPIO25 (USB FS) | 24 | GPIO31 |
| 25 | USBD_N (USB HS OTG, not a GPIO) | 26 | GND |
| 27 | USBD_P (USB HS OTG, not a GPIO) | 28 | GPIO34 (strapping) |
| 29 | GND | 30 | GPIO35 (strapping) |
| 31 | GPIO32 | 32 | GPIO49 |
| 33 | GND | 34 | GPIO50 |
| 35 | GPIO46 | 36 | GPIO51 |
| 37 | GPIO47 | 38 | GPIO52 |
| 39 | GPIO48 | 40 | GND |

Onboard, not on the header: ES8311/ES7210 codec I2S on GPIO9–13, speaker amplifier enable GPIO53, LCD reset GPIO27,
backlight GPIO26/GPIO33, touch reset GPIO23, microSD on GPIO39–44, ESP32-C6 over SDIO.

The machine-readable allocation is [pinmap.csv](pinmap.csv); firmware and the KiCad design follow it.

## Wiring

### Audio (one I2S bus, P4 is master)

| Signal | P4 GPIO (J3 pin) | PCM5102A module | PCM1808 module |
|---|---|---|---|
| MCLK | GPIO21 (15) | SCK | SCK |
| BCLK | GPIO22 (17) | BCK | BCK |
| WS / LRCK | GPIO28 (16) | LCK | LRC |
| Data to DAC | GPIO5 (11) | DIN | – |
| Data from ADC | GPIO4 (14) | – | OUT |
| 3.3 V | 3V3 (2, 18) | – | 3.3V |
| 5 V | 5V (1, 3) | VIN | +5V |
| GND | GND (5, 13, 19) | GND | GND |

The PCM5102A module has its own 3.3 V regulators, so VIN takes 5 V; its logic still runs at 3.3 V and matches
the P4.

Module configuration:

- **PCM5102A (GY-PCM5102):** the four solder bridges on the back are H1L = FLT, H2L = DEMP, H3L = XSMT,
  H4L = FMT. Set H1L = L, H2L = L, H3L = H (unmute), H4L = L (I2S). With XSMT low the module is silent. The two
  small pads next to SCK on the front ground SCK; they must stay **open**, because SCK carries MCLK from the P4.
- **PCM1808:** MD0 = L, MD1 = L (slave mode), FMT = L (I2S); on the LaskaKit module FMT is printed as "PMT".
  The chip accepts a system clock of 256, 384 or 512 fs; firmware drives 256 fs.

### Encoder and LED (stage 1 uses ready-made modules)

| Module pin | P4 GPIO (J3 pin) | Notes |
|---|---|---|
| RGB module (KY-016) R | GPIO32 (31) | 150 Ω on the module |
| RGB module G | GPIO49 (32) | 150 Ω on the module |
| RGB module − (common cathode) | GND (33) | |
| RGB module B | GPIO50 (34) | 150 Ω on the module |
| Encoder module S1 | GPIO47 (37) | 10 kΩ pull-up and RC filter on the module |
| Encoder module S2 | GPIO52 (38) | |
| Encoder module Key | GPIO48 (39) | |
| Encoder module GND | GND (40) | |
| Encoder module "5V" | **3V3 (18)** | never 5 V: the pull-ups would put 5 V on P4 pins, which are not 5 V tolerant |

The module resistors are sized for 5 V; at 3.3 V red runs at about 8–9 mA and green/blue at 2–4 mA. Firmware
balances the colours with PWM.

### Wiring method

- Separate female–female jumper wires from the module pins to the header; no soldering needed in stage 1.
- On the carrier board the LED, resistors and encoder are soldered to the PCB; nobody wires them by hand.

### Display

Built in. Touch (GT911) is on the onboard I2C bus (GPIO7/8); the BSP polls it, the INT line is not used.

### Controller

The OTG Type-C port has no 5 V output: its VBUS feeds the board through an ideal-diode circuit (AO3401 + MMDT3906)
and reverse current is blocked. A controller plugged in through a plain OTG adapter gets no power. The board itself is powered from the USB-UART Type-C.

Stage 1 uses the same path as the carrier board: a USB-A female connector on a pigtail to J3.

| USB-A pin | J3 pin |
|---|---|
| 1 VBUS | 1 or 3 (5V) |
| 2 D− | 25 (USBD_N) |
| 3 D+ | 27 (USBD_P) |
| 4 GND | 26 |

Keep the D−/D+ wires short (20 cm) and twisted together. Leave the onboard OTG Type-C empty: it shares the same
data lines. The controller is a full-speed device, so jumper wires are good enough. A powered USB hub on the OTG
Type-C also works as a fallback.

The carrier board solves this permanently: the HS OTG data lines are also on J3 (pins 25/27), so the carrier gets
its own USB-A host port with 5 V from the carrier supply.

## Reserved for stage 2 (carrier board)

| Function | J3 pin |
|---|---|
| TRS MIDI OUT | GPIO46 (35) |
| TRS MIDI IN | GPIO51 (36) |
| USB-A host D− / D+ | USBD_N (25) / USBD_P (27) |
| 5 V supply into the board | VCC_5V (1, 3) |
| Spare | GPIO3 (12), GPIO29 (20), GPIO30 (22), GPIO31 (24) |

## Pins not to use

- GPIO7/8: onboard I2C (codec, echo-cancellation ADC, touch).
- GPIO37/38: console UART, strapping pins.
- GPIO34/35: strapping pins.
- GPIO2: touch INT on some units (R35 fitted or not).
- GPIO24/25: USB full-speed PHY.

## Known limits of stage 1

- The PCM1808 analog supply comes straight from USB 5 V; some noise is expected. The carrier board adds filtering.
- The PCM5102A module output is a line output. Use line inputs (audio interface, active monitors). Headphones only at
  low volume until the NJM4556 stage exists.
- The line input has no protection yet; feed it line level only.
- The board has a push-button power controller (ECJ23001) between VCC_5V and the system supply. Whether the board
  starts by itself when 5 V is applied, or needs a press of the power button, is checked on the first power-up.
