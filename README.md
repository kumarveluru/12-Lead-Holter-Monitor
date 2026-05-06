
# 12-Lead Holter Monitor (10-Electrode Implementation) | KiCad PCB Design

A KiCad-designed **12-lead Holter monitor** implemented using a **10-electrode configuration**, achieving full clinical-grade diagnostic output. This portable, wearable cardiac monitoring system continuously captures 12-lead ECG data using the ADS1298 analog front-end and an ESP32-S3 microcontroller, with onboard SD card logging, RTC timestamping, and OLED display.

You will find the following:
- **Board Schematic:** Hierarchical multi-sheet schematic covering electrode input, ESD protection, ECG input filtering, AFE, MCU, power management, SD card, RTC, and OLED.
- **PCB Layout:** The physical design of the printed circuit board with antenna keep-out zone.
- **PCB Routing:** Signal and power routing across the two-layer board.

---

### Tools Used

The electronic design and development were done using **KiCad**, an open-source EDA software.
The version used for this project is **KiCad 9.0.7**.

#### Footprints used for LEDs:
`0603_1608Metric_Pad1.08x0.95mm_HandSolder`

#### Footprints used for Resistors, Capacitors, Inductors:
`0402_1005Metric_Pad1.22x1.35mm_HandSolder`


All the files are attached in the repository folder.

---

## System Architecture

The design is organised into **9 hierarchical KiCad sub-sheets**:

| Sub-sheet | File | Function |
|---|---|---|
| Electrode & ESD | `electrodes,esd.kicad_sch` | 10-pin LEMO connector + PRTR5V0U2X ESD diodes |
| ECG Input Filter | `ecg_input_filter_adc.kicad_sch` | 10× RC low-pass filter (~159 Hz cutoff) |
| Analog Front-End | `afe.kicad_sch` | ADS1298 — 8-channel 24-bit ECG ADC |
| MCU | `ESP32-S3-ckt.kicad_sch` | ESP32-S3-WROOM-1 — SPI, I2C, USB, GPIO |
| Power Management | `pm-ckt.kicad_sch` | MCP73831 charger + TPS63001 buck-boost |
| Buck-Boost | `buck-boost-ckt.kicad_sch` | TPS63001 stable 3.3V from 2.5–5.5V input |
| SD Card | `sdcard.kicad_sch` | MSD-1-A microSD card in SPI mode |
| RTC & OLED | `oled,rtc.kicad_sch` | DS3231SN RTC + CR2032 backup + OLED display |
| USB Connector | `usb-conn-ckt.kicad_sch` | USB-C receptacle (16-pin, USB 2.0) |

---

## 12 Leads from 10 Electrodes

The system uses **10 electrodes** placed at standard clinical positions (RA, LA, RL, LL, V1–V6) to derive all 12 ECG leads mathematically:

| Lead Group | Leads | Method |
|---|---|---|
| Limb Leads | I, II, III | Einthoven's Triangle: `I = LA−RA`, `II = LL−RA`, `III = LL−LA` |
| Augmented Leads | aVR, aVL, aVF | Goldberger's equations |
| Chest Leads | V1–V6 | Directly measured w.r.t. Wilson Central Terminal (WCT) via ADS1298 |

> **Key Insight:** 6 leads measured directly (V1–V6) + 6 mathematically derived (I, II, III, aVR, aVL, aVF) = Full 12-Lead ECG

---

## Key ICs & Design Decisions

### Analog Front-End — ADS1298
- 8-channel, 24-bit delta-sigma ADC purpose-built for ECG acquisition
- Generates Wilson Central Terminal (WCT) internally — no external resistor network needed
- AVDD and DVDD isolated via **600Ω @ 100 MHz ferrite bead** to prevent digital noise from contaminating µV-level ECG signals
- DRDY (active-LOW, open-drain) with 10kΩ pull-up for clean interrupt edge to ESP32

### ECG Input Filter — ~159 Hz Low-Pass RC
- `R = 10kΩ, C = 100nF` → `f_c = 1 / (2π × 10k × 100n) ≈ 159 Hz`
- One RC stage per channel — 10 channels total (RA, LA, LL, RL, V1–V6)
- Preserves full ECG bandwidth (0.05–150 Hz) while suppressing EMG artifact, RF pickup, and ADC aliasing

### ESD Protection — PRTR5V0U2X
- Individual 2-pin ESD diodes per electrode line, clamped to `an_vdd`
- Better channel isolation and lower parasitic capacitance vs. multi-channel arrays
- Protects ADS1298 ultra-sensitive analog inputs from electrostatic discharge events

### Electrode Connector — LEMO ECG.1B.310.CLN (10-pin)
- Medical-grade push-pull circular connector — maps exactly to 10 electrodes (RA, LA, LL, RL + V1–V6)
- Keyed single-key design prevents incorrect cable insertion
- Chrome-plated brass shell rated for 5000+ mating cycles

### MCU — ESP32-S3-WROOM-1
- Dual-core LX7, Wi-Fi/BT, native USB-FS
- SPI bus shared between ADS1298 (AFE) and SD card — separate CS lines
- I2C bus shared between DS3231 RTC and OLED display
- Pull-ups on CS_n, START, RESET keep AFE in safe idle state during boot

### Power Management

| IC | Role |
|---|---|
| MCP73831 | Single-cell Li-Ion/Li-Po charger, 500 mA (set via 2kΩ R20), USB-C VBUS input |
| TPS63001 | Buck-boost converter — stable 3.3V across full Li-Ion range (2.5V–5.5V) |
| D6 (Schottky) | Prevents reverse discharge from battery back to USB |

### Storage & Timekeeping
- **MSD-1-A microSD** (SPI mode) — continuous long-term ECG data logging
- **DS3231SN RTC** — temperature-compensated crystal (TCXO), I2C, interrupt-driven wake for ESP32 power saving
- **CR2032 coin cell** — RTC backup power across main supply power cycles

---

## Schematic

![Root Schematic](https://github.com/kumarveluru/12-Lead-Holter-Monitor/blob/main/Images/Root%20Schematic.png)

---

## PCB Layout

### Component Placement
![Component Placement](https://github.com/kumarveluru/12-Lead-Holter-Monitor/blob/main/Images/Placement.png)

### Overall PCB Layout
![PCB Layout](https://github.com/kumarveluru/12-Lead-Holter-Monitor/blob/main/Images/Layout.png)

---

## 3D View

### Top View
![3D Top View](https://github.com/kumarveluru/12-Lead-Holter-Monitor/blob/main/Images/3D%20Top%20View.png)

### Bottom View
![3D Bottom View](https://github.com/kumarveluru/12-Lead-Holter-Monitor/blob/main/Images/3D%20Bottom%20View.png)

---
## Board Dimensions

| Parameter | Value    |
|-----------|----------|
| Width     | 68.61 mm |
| Height    | 68.08 mm |

## Board Specifications

| Parameter | Value |
|---|---|
| Layers | 2 |
| Supply Voltage | 3.3V (regulated) / Li-Ion battery |
| ECG Channels | 8 differential (ADS1298) |
| Leads Generated | 12 (6 direct + 6 derived) |
| Electrodes | 10 (LEMO ECG.1B.310.CLN) |
| MCU | ESP32-S3-WROOM-1 |
| AFE | ADS1298 (24-bit, 8-ch) |
| Storage | MicroSD (SPI) |
| Display | OLED (I2C) |
| Connectivity | USB-C (USB 2.0), Wi-Fi/BT (ESP32-S3) |

---

## License

[![MIT License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Contact

V Kumar - V.Kumar@iiitb.ac.in  [GitHub Profile](https://github.com/kumarveluru)

Project Link: [https://github.com/kumarveluru/12-Lead-Holter-Monitor](https://github.com/kumarveluru/12-Lead-Holter-Monitor)

---

## Acknowledgments

I would like to express my sincere gratitude to my college, **International Institute of Information Technology (IIIT-B)**, for providing the resources and support to complete this project.  
I am especially grateful to my professor, **Dr. Kurian Polachan**, for their invaluable guidance, encouragement, and expertise throughout the development of this work.
