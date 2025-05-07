[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/AlBFWSQg)
# a14g-final-submission

    * Team Number: 21
    * Team Name: pixel & circuit
    * Team Members: Zhongyu Wang, Linhai Deng
    * Github Repository URL: https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit.git
    * Description of test hardware: (development boards, sensors, actuators, laptop + OS, etc) laptop

## 1. Video Presentation

https://youtube.com/shorts/kJqO3oqh_wA

## 2. Project Summary

### 1 · Device Description
* **What it is** – SmartDoor Vision retrofits a dorm‑room or apartment door with a PIR motion sensor and an ESP‑EYE camera.  
* **Why we built it** – Traditional peepholes provide zero remote awareness and no recording. SD‑Vision solves that limitation without modifying building wiring.  
* **Internet integration** – Events publish over MQTT to an Azure VM running Node‑RED. From the dashboard you can:
  * Watch live video & snapshots  
  * Read door status (“attention/safe”) in real time  
  * Press **Call the Police** to sound an on‑door buzzer and display message  
  * Trigger an over‑the‑air firmware update (OTAFU)

---

### 2 · Device Functionality

| Sub‑system | Part / Interface | Role |
|------------|------------------|------|
| **Sensor** | IRA‑S210 PIR → ADC (PA03) | Detect motion, push events |
| **Camera** | ESP‑EYE Wi‑Fi | Live MJPEG + snapshot |
| **Actuators** | SSD1306 128×32 OLED (I²C) & LED | Show *“⚠ Police on the way / Safe environment”* |
| | Piezo buzzer on **PA02 (PWM 2 kHz)** | Two‑second siren |
| **Comms** | WINC1500 Wi‑Fi · MQTT (QoS 1) | TX sensor, RX commands |
| **Firmware** | SAMD21 · FreeRTOS · queues | Non‑blocking drivers |
| **Cloud/UI** | Node‑RED dashboard | Status, live view, OTAFU |

<p align="center">
  <img src="door-system-block.svg" alt="System Block Diagram" width="80%">
</p>

---

### 3 · Challenges

| Challenge | Root Cause | Mitigation |
|-----------|------------|------------|
| **Analog noise on PIR** | Wi‑Fi bursts bled into PA03 trace | Split analog/digital ground, RC filter 100 kΩ/100 nF |
| **OLED locked I²C bus** | Blocking driver in tight loop | Re‑wrote as non‑blocking state machine |
| **OTA flash corruption** | Power drop during swap | Added CRC + rollback slot |

---

### 4 · Prototype Learnings

* **Plan pin‑mux early** – PA02 DAC vs. PWM cost ~1 h to debug.  
* **Noise windows** – Real hallways create borderline ADC levels; hardware WC + debounce mandatory.  
* **If rebuilding** – Use a digital PIR with built‑in comparator and an SPI display to free I²C.

---

### 5 · Next Steps & Takeaways

* **Improvements** – Add Li‑ion + boost for cable‑free power; migrate MQTT to TLS.  
* **Course takeaways** – Deepened skills in FreeRTOS task design, bare‑metal driver writing on SAMD21, and secure OTA deployment.

---

## 3. Hardware & Software Requirements

## 4. Project Photos & Screenshots

## Codebase

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device

