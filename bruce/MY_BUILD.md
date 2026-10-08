# My Bruce build

Fork of [pr3y/Bruce](https://github.com/pr3y/Bruce), copied at commit `a59213f` (2026-09-09).
Bruce is licensed **AGPL-3.0** (see `LICENSE`). Any build I distribute must stay AGPL and ship its source.

## Hardware

| Part | Notes |
|---|---|
| ESP32-S3-DevKitC-1-N16R8 | 16 MB flash, 8 MB octal PSRAM |
| nRF24L01 (2.4 GHz) | SPI |
| CC1101 (sub-GHz) | SPI, shares the bus with nRF24 |
| 1.3" I2C OLED | Not supported by upstream Bruce (see below) |

## What upstream Bruce supports

- Display: TFT_eSPI (SPI TFT only). No OLED / I2C display driver.
- Radios: `USE_CC1101_VIA_SPI` and `USE_NRF24_VIA_SPI` are pin-configured per board (see `boards/lilygo-t-display-s3/pins_arduino.h`).
- Closest existing env: `esp32-s3-devkitc-1-psram` (N16R8) in `platformio.ini`, which uses `boards/ESP-General`.

## Proposed pin plan (NOT applied yet)

Avoids strapping pins (0, 3, 45, 46), USB (19, 20), and the PSRAM/flash-internal pins (26–37 on N16R8).

| Function | GPIO |
|---|---|
| SPI SCK (shared) | 12 |
| SPI MOSI (shared) | 11 |
| SPI MISO (shared) | 13 |
| CC1101 CS | 10 |
| CC1101 GDO0 | 4 |
| CC1101 GDO2 | 5 |
| nRF24 CS | 9 |
| nRF24 CE | 8 |
| nRF24 IRQ | 6 |
| OLED I2C SDA | 41 |
| OLED I2C SCL | 42 |

GPIO 48 is the on-board RGB LED on DevKitC-1; GPIO 47 is spare.

Confirm these match your physical wiring before flashing.

## Open decision

How to drive the OLED:
1. Add a U8g2 (SH1106/SSD1306) I2C display backend alongside TFT_eSPI.
2. Run the device headless and use only the radios through the web/serial interface.
