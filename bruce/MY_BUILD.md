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

## Pin plan (from my wiring list)

Schematic: [docs/wiring.svg](docs/wiring.svg)

Power: 3V3 to nRF24, CC1101, and OLED VCC; GND to all of them.

| Function | GPIO |
|---|---|
| SPI SCK (shared nRF24 + CC1101) | 12 |
| SPI MOSI (shared) | 11 |
| SPI MISO (shared) | 13 |
| CC1101 CSN | 15 |
| CC1101 GDO0 | 4 |
| CC1101 GDO2 (optional, not wired) | - |
| nRF24 CSN | 10 |
| nRF24 CE | 9 |
| nRF24 IRQ (optional, not wired) | - |
| OLED SDA | 8 |
| OLED SCL | 7 |

Avoids strapping pins (0, 3, 45, 46), USB (19, 20), and PSRAM/flash-internal pins (26-37 on N16R8). GPIO 48 is the on-board RGB LED.

Note: the 1.3" OLED is usually SH1106, which the Adafruit SSD1306 library does not drive correctly. Use U8g2 with an SH1106 driver, or check the controller on your module.

## Open decision

How to drive the OLED:
1. Add a U8g2 (SH1106/SSD1306) I2C display backend alongside TFT_eSPI.
2. Run the device headless and use only the radios through the web/serial interface.

## Firmware status

- Env: `my-s3-n16r8` in `boards/my-s3-n16r8/my-s3-n16r8.ini`. It extends `esp32-s3-devkitc-1-psram` and overrides the ESP-General pin defines with `-U` then `-D`, so the upstream board is untouched.
- Radios: CC1101 enabled (CSN 15, GDO0 4). nRF24 enabled (CSN 10, CE 9). Shared SPI on 12/11/13.
- I2C bus: `GROVE_SDA=8`, `GROVE_SCL=7`. The firmware's I2C bus config uses these macros.
- **Not compiled yet.** PlatformIO's tool download fails TLS verification in this sandbox (the proxy CA is not trusted by its HTTP client). Build it on your own machine with `pio run -e my-s3-n16r8`.
- **Not implemented:** the OLED display driver. The 1.3" I2C OLED will not display anything until a U8g2 SH1106 backend is added.

## Known risk

The ESP-General board header defines `SDA = 8` and `SCL = 9` as Arduino globals. The `SCL = 9` default collides with nRF24 CE (GPIO9) if anything calls `Wire.begin()` with no arguments. A grep of `src/` and `include/` found no such call, but the M5 and Adafruit libraries are not checked. Confirm with a boot test or a serial log before wiring the OLED.
