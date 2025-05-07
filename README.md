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



## 4. Project Photos & Screenshots

## Codebase

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device

