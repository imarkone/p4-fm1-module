# Firmware

ESP-IDF project for the ESP32-P4 board support layer that runs FM-1 community firmware (Felucca first, as a git
submodule) unchanged. Status: not started.

Planned parts:

- Panel abstraction: FM-1 keys, encoders, buttons, LEDs and screen mapped to P4 hardware and to the USB-MIDI
  controller through CSV profiles in [`../mapping/`](../mapping/).
- Audio: one I2S bus, P4 as clock master (MCLK/BCLK/LRCK), PCM5102A out, PCM1808 in.
- MIDI: USB host (controller), TRS MIDI IN/OUT on UART, MIDI OUT routing added in the board layer.
- Display: FM-1 240×240 frame shown at 2× on the 480×800 DSI panel.
- Debug: serial log on USB-C UART, RGB status LED, EC11 safe-mode at boot.

All user-facing text comes from [`../lang/`](../lang/); none is written in code.
