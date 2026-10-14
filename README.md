# UWB Anchor & Indoor Human Occupancy Tracking Station

An integrated, multi-sensor UWB anchor PCB designed for real-time indoor object tracking (via Double-Sided Two-Way Ranging) and non-invasive human presence/occupancy monitoring. Built on a compact **75 × 85 mm²** 4-layer stackup, this node timestamps sensor data with microsecond precision via Wi-Fi TSF and streams payload telemetry to a central data fusion server.

## Table of Contents

* [Project Overview](#project-overview)
* [Key Features](#key-features)
* [Hardware & Bill of Materials](#hardware--bill-of-materials)
* [Sensor Matrix](#sensor-matrix)
* [Time Synchronization Strategy](#time-synchronization-strategy)
* [PCB Design & Stackup](#pcb-design--stackup)
* [Repository Structure](#repository-structure)
* [Team & Course Credits](#team--course-credits)

## Project Overview

This project implements an open-hardware anchor node for spatial localization and environmental occupancy analytics. The node performs two core functions simultaneously:

1. **Ultra-Wideband (UWB) Ranging:** Acts as a fixed anchor measuring distances to mobile UWB tags using IEEE 802.15.4z DS-TWR.
2. **Multi-Variable Human Sensing:** Tracks room occupancy trends and movement through non-camera environmental sensors (CO₂, Temperature/Humidity, Acoustic level, PIR motion, and mmWave micro-movement radar).

All gathered metrics are bound to a unified global timebase and transmitted over a 2.4 GHz Wi-Fi network (UDP/TCP) to a host server executing sensor-fusion algorithms.

## Key Features

* **High-Performance MCU:** ESP32-S3-WROOM-1-N8 (Dual-core Xtensa LX7 @ 240 MHz, 512 kB SRAM, 8 MB Flash, native USB CDC/JTAG).
* **Precision UWB Transceiver:** Decawave DWM3000TR13 (DW3110 engine) over SPI, supporting channels 5 (6.5 GHz) and 9 (8.0 GHz) with integrated antenna and keep-out area.
* **Dual 3.3V Power Domain Architecture:** Dedicated high-efficiency DC-DC Buck converter power rail for MCU processing and wireless transmission current spikes, combined with an isolated low-noise LDO regulator power rail for the sensitive sensor matrix.
* **Privacy-Preserving Human Detection:** Multi-variable sensing without optical cameras.
* **Microsecond Timestamping:** Hardware Wi-Fi TSF timer (`esp_wifi_get_tsf_time()`) combined with periodic NTP/SNTP synchronization.

## Hardware & Bill of Materials

| Function | Component MPN | Description | Interface | JLC Code |
| :--- | :--- | :--- | :--- | :--- |
| **MCU** | `ESP32-S3-WROOM-1-N8` | Dual-core Wi-Fi/BLE SoC (8 MB Flash) | SPI, I²C, UART, I²S | C2913198 |
| **UWB Transceiver** | `DWM3000TR13` | Integrated UWB Transceiver Module | SPI (20 MHz) | C5299931 |
| **MCU Power Rail** | `DC-DC Buck Regulator` | High-efficiency 3.3V Step-Down Converter (MCU) | Power | TBD |
| **Sensor Power Rail** | `Low-Noise LDO` | Dedicated Ultra-Low Noise 3.3V Regulator (Sensors) | Power | TBD |
| **Power Connector** | `HYCW263-USBC04-850B` | 16-Pin USB Type-C Receptacle | 5V Input | C3034197 |
| **CO₂ Sensor** | `Sensirion SCD40` | Photoacoustic NDIR CO₂ (0 - 40,000 ppm) | I²C | C3037696 |
| **Temp/RH** | `Sensirion SHT40-AD1B-R3` | Relative Humidity & Temperature Sensor | I²C | C2848306 |
| **Acoustic Sensor** | `TDK InvenSense ICS-43434` | Digital Omnidirectional MEMS Microphone | I²S | C5656610 |
| **PIR Motion** | `AM312` | Passive Infrared Human Motion Detector | GPIO | C114881 |
| **mmWave Radar** | `24/60 GHz mmWave Radar` | Micro-movement & Breathing Radar | UART | C37897213 |

## Sensor Matrix

1. **Decawave DWM3000 UWB:** Configured for Double-Sided Two-Way Ranging (DS-TWR) operating at 5 - 20 Hz update rates per tag.
2. **Sensirion SCD40:** Measures ambient CO₂ concentration every 30 - 60 s as an indicator of human breathing density.
3. **Sensirion SHT40:** Provides high-accuracy ambient thermal/humidity telemetry (±0.2 °C, ±1.8% RH) to complement indoor air comfort models.
4. **TDK ICS-43434:** Samples acoustic sound pressure over sliding time windows to determine room activity levels.
5. **AM312 PIR:** Digital motion trigger generating interrupts upon macroscopic body movement.
6. **mmWave Radar Module:** Captures fine target presence, velocity, and chest micro-movements (respiration) via continuous UART streaming.

## Time Synchronization Strategy

To ensure precise multi-anchor data alignment on the host processing server:

1. **Coarse Clock Sync (NTP/SNTP):** Upon Wi-Fi connection, the anchor queries an NTP server to calibrate epoch time.
2. **Fine-Grain Microsecond Timing (Wi-Fi TSF):** Each sensor frame is tagged using `esp_wifi_get_tsf_time()`, providing absolute microsecond precision relative to the Wi-Fi Access Point Timing Synchronization Function.
3. **Fallback Mode (ESP-NOW):** In ad-hoc deployments without an infrastructure AP, an ESP-NOW broadcast timesync protocol is utilized.

## PCB Design & Stackup

* **Dimensions:** 75 × 85 mm²
* **Layer Count:** 4 Layers (Mixed RF + High-Speed Digital)
* **Layer Assignment:**
  * **L1 (Top):** Component placement, high-speed signals (SPI, I²S, USB 90 Ω diff pair), RF traces.
  * **L2 (GND):** Continuous Solid Ground Plane.
  * **L3 (Power):** Split power planes with dual 3.3V power domains (3.3V_MCU from Buck, 3.3V_SENSOR from LDO).
  * **L4 (Bottom):** Secondary routing and test points.
* **RF Guidelines:** 50 Ω coplanar waveguide routing for UWB antenna path with explicit ground keep-out regions under the chip antenna area.

## Repository Structure

```
├── docs/
│   ├── schematics/             # PDF Schematics
│   └── specifications/         # ECAM Project Brief & Specs
├── hardware/
│   ├── gerber/                 # Production fabrication files
│   ├── KiCad/                  # Project files (.kicad_pro, .kicad_sch, .kicad_pcb)
│   └── bom/                    # Bill of Materials (CSV/JLCPCB format)
├── firmware/
│   ├── main/                   # ESP-IDF / Arduino source code
│   └── components/             # Custom drivers for SCD40, DWM3000, ICS-43434
└── README.md
```

## Team & Course Credits

**Course:** Advanced PCB Design (Academic Year 2026–2027)  
**Institution:** ECAM Brussels Engineering School  

**Group Members:**
* Laurent Emond
* Vincenzo Acquaviva
* Lincoln Lassey
* Mathieu Mascot

*ECAM Engineering Department*   
