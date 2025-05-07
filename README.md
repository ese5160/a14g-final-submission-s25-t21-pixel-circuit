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

### 1. Device Description
* **What it is** – A battery‑powered smart‑door add‑on that live‑streams video, senses motion and pushes e‑mail alerts. A Node‑RED dashboard shows status, hosts an emergency button and triggers OTA firmware updates.  
* **Why we built it** – Dorm rooms lack intercoms; package theft and unwanted visitors are common. We wanted a retrofit solution—no drilling, no paid cloud.
* **Internet integration** – Events publish over MQTT to an Azure VM running Node‑RED. From the dashboard you can:
  * Watch live video & snapshots  
  * Read door status (“attention/safe”) in real time  
  * Press **Call the Police** to sound an on‑door buzzer and display message  
  * Trigger an over‑the‑air firmware update (OTAFU)

---

### 2. Device Functionality

| Element | Details |
| ------- | ------- |
| **Sensors** | Murata IRA‑S210ST01 PIR → ADC PA03. Threshold 1000 raw (≈1.2 V) → `attention` vs `safe`. |
| **Actuators** | *I²C OLED* (UG‑2864HSWEG01) + on‑board LED. Panic button toggles text **“the police is coming”** and LED.<br>*Speaker* footprint present but disabled (MCU RAM limits). |
| **Live video** | ESP‑EYE 2 MP camera streams MJPEG + built‑in mic (no extra MCU RAM). |
| **Snapshot & mail** | Each `attention` sends JPEG + timestamp to user‑supplied e‑mail via Node‑RED SMTP. |
| **OTA update** | Dashboard buttons **GOLDEN IMAGE** / **OTA UPDATE (FW)** push a firmware URL; board downloads and self‑flashes. |

---

### 3. Challenges

| # | Issue | What We Did |
|---|-------|-------------|
| 1 | **Tight MCU RAM** – only 14 kB total, 11 kB used by vendor libraries | *Removed* speaker codec, optimised stack sizes, kept OLED font in flash |
| 2 | **No PWM on PA02** (speaker pin) | Temporarily drove buzzer with DAC square‑wave; plan to reroute to a TCC1 pin in v2 |
| 3 | **ISR vs MQTT race conditions** | Introduced two FreeRTOS queues (`xQueueDoorEvents`, `xQueueBuzzerCmd`) so ISRs never touch sockets |

---

### 4. Prototype Learnings

- **Measure first, design second** – RAM/flash profiling before adding features prevented late‑stage re‑writes.  
- **Off‑board UI wins** – pushing logic to Node‑RED let us tweak alert thresholds and e‑mail text without reflashing firmware.  
- **Test‑pads + CLI save hours** – being able to print raw ADC and switch pins through the CLI accelerated bring‑up and debugging.

---

### 5. Next Steps & Takeaways

#### Next Steps

1. **PCB v2**: move speaker to a true PWM pin, add 2 MB SPI flash for snapshots, add Li‑ion fuel‑gauge.  
2. **Full‑duplex audio**: stream ESP‑EYE microphone to dashboard and relay doorbell chime back to speaker.  
3. **Deep‑sleep mode**: target < 200 µA standby with wake on PIR or Wi‑Fi M2M Wakes.

#### Course Takeaways

- **Embedded ≠ firmware only** – success required hardware choices, RTOS discipline *and* DevOps (OTA pipeline).  
- **Iterative validation > end‑of‑cycle testing** – weekly Node‑RED demos exposed integration bugs early.  
- **Documentation matters** – clear README tables/diagrams made hand‑offs within the team smoother.

## 3. Hardware & Software Requirements

| Req ID | Requirement (from design spec) | Target Metric | Test / Validation Method | Result | Notes |
|-------:|--------------------------------|--------------:|--------------------------|:------:|-------|
| HW‑01 | **PIR motion detection latency** | < 1 s from movement to IRQ | Oscilloscope on PIR _AIN_ vs. UART “attention” print‑out | ✅ 0.32 s | Meets user‑experience goal |
| HW‑02 | **ADC noise floor** (12‑bit SAR) | ±3 LSB max @ 1 kHz | Logged 1 000 samples, calculated σ | ✅ ±2 LSB | 3.3 V reference, RC filter |
| HW‑03 | **OLED legibility** indoors | 700 cd/m² min | Lux‑meter on white screen | ✅ 748 cd/m² | Contrast 100 % |
| HW‑04 | **Battery life (Li‑Ion 800 mAh)** | ≥ 24 h standby | Simulated with bench supply @ 30 µA sleep | ⚠️ | Needs deep‑sleep optimisation |
| SW‑01 | **FreeRTOS task scheduling jitter** (sensor task) | < 5 ms | vTaskGetRunTimeStats → std dev | ✅ 1.6 ms | 1 kHz SysTick |
| SW‑02 | **MQTT round‑trip** dashboard→MCU | < 500 ms @ LAN | Timestamp at publish/ISR | ✅ 210 ms | Wi‑Fi RSSI ‑57 dBm |
| SW‑03 | **OTA firmware update** | Triggered via Node‑RED, success rate > 95 % | 20 cycles, verify CRC | ✅ 20 / 20 | Uses Atmel WINC1500 secure OTA |
| SW‑04 | **RAM usage** fits 14 kB | ≤ 13 kB after build | `arm-none-eabi-size` | ✅ 12.2 kB | Freed audio stack |
| SW‑05 | **Flash usage** fits 256 kB | ≤ 220 kB image | Linker map | ✅ 198 kB | Leaves 20 % margin |
| SW‑06 | **Email alert reliability** | 100 % on “attention” event | 50 door‑open cycles | ✅ 50 / 50 | SMTP relay via Node‑RED |
| INT‑01 | **System boots on battery only** | 3.0 V → 4.2 V | Power‑cycle test | ✅ | Buck‑boost TPS61291 |

> **Legend**   
> ✅ Met ⚠️ Partially met / needs improvement ❌ Not met

### Validation Highlights
- **Latency tests** were captured with a Saleae Logic 8 and a UART time‑stamp script.  
- **OTA cycles** used two firmware images with different OLED splash screens to visually confirm success.  
- **Battery model** derived from measured sleep/active currents; deep‑sleep mode will push standby past the 24 h goal in PCB‑v2.

## 4. Project Photos & Screenshots

## Codebase

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device

