# Plant Stress Analyzer

Desktop software for the **ESP32 + BME690 + AS7343** plant stress sensor. It runs the measurement, saves every reading to a local database, and helps compare treatments using an odor fingerprint (metal-oxide gas sensor) and a VNIR spectrum (reflectance and transmission).

> **Download:** get the latest installer from [Releases](../../releases/latest).
> Use `PlantStressAnalyzer_<version>_x64-setup.exe` (recommended) or the `.msi`. Windows 10/11, 64-bit.

---

## Features

- **Data acquisition**: start and stop measurements, with live charts of every gas reading, the odor fingerprint and the VNIR spectrum.
- **Experiments and factors**: organize samples by experiment, factor (e.g. *Treatment*) and group (e.g. *Control*, *Leaf*). Experiments can be password protected.
- **Data Viewer**:
  - add charts for single samples or a group average
  - **Mean ± SD** boxes
  - **Difference (A − B)** boxes
  - **Group** boxes that overlay several samples or boxes, with a colour you pick for each series
  - single-value bar charts
  - PNG export of any chart
- **Firmware check and update**: when the ESP32 is plugged in, the app reads its firmware version. If it differs from the firmware bundled with the app, one click flashes the matching firmware. No Arduino IDE is needed.
- **Automatic app updates**: the app checks this repository for new versions and can download, install and restart by itself. Updates are signed.

## Getting started

1. Install the app from [Releases](../../releases/latest). The first install is manual; later versions arrive as in-app updates.
2. Plug in the ESP32 with a USB cable. In the **Connect the ESP32** dialog, pick its serial port (baud rate 115200).
3. If the firmware status shows **Update** or **Install**, click it and wait until it reports *up to date*. Keep the cable plugged in while it runs.
4. Choose or create an experiment. Add factors and groups under **Factors/Treatments**.
5. On **Data Acquisition**, select the group of the sample, set the number of cycles and press **Start**.

The ESP32 warms up its gas sensor for about 90 s after every power-on or reset. The firmware check also resets the board, so a measurement started straight after plugging in begins once the warm-up is done.

## Hardware

| Part | Connection |
|---|---|
| ESP32 dev board (`esp32:esp32:esp32`) | USB to the PC |
| Bosch BME690 (gas, temperature, humidity, pressure) | I²C bus 0: SDA 21, SCL 22, address 0x76 |
| ams OSRAM AS7343 (spectral sensor; its on-board LED is used for reflectance) | I²C bus 1: SDA 33, SCL 32, address 0x39 |
| External LED for transmission | GPIO 4 |

## Measurement protocol (v14)

Each cycle runs these steps in order:

1. **Reflectance**: AS7343 with its LED on for 3 s.
2. **Transmission**: external LED on for 3 s.
3. **Gas sensor conditioning**: 115 °C (1 s) ↔ 400 °C (1 s), repeated until the 115 °C reading is stable (10–60 cycles).
4. **Gas channel scan**: 20 heater targets from 115 °C to 400 °C in 15 °C steps. Each channel runs:
   - **PRE**: 115 °C for 1 s
   - **TARGET**: a quick read at about 0 s (T0), then reads at 1, 2 and 3 s (T1–T3)
   - **POST**: 400 °C for 1 s

The **odor fingerprint** is log₁₀(PRE / T3) for each channel. The VNIR spectrum covers 12 bands from 405 to 855 nm.

## Data

- All readings are stored in a SQLite database at
  `%LOCALAPPDATA%\CMU BME690 Research\Plant Stress Analyzer\bme690_data.sqlite`.
  The database is kept when the app is updated or reinstalled.
- Experiments can be exported to CSV from the app.

## Version numbers

The app and the ESP32 firmware always share the same version number. Each app release bundles its matching firmware, so updating the app and then clicking **Update** on the firmware status keeps both in step.

## Credits

Copyright © 2026 Assistant Professor Dr. Chun-I Chiu,
Department of Entomology and Plant Pathology, Faculty of Agriculture, Chiang Mai University.

The installer includes [esptool](https://github.com/espressif/esptool) by Espressif Systems (GPL-2.0), used unmodified to flash the ESP32 firmware. Its licence is installed alongside it (`firmware/esptool-LICENSE.txt`).
