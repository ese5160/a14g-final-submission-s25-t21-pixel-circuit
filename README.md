[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/AlBFWSQg)
# a14g-final-submission

    * Team Number: 21
    * Team Name: pixel & circuit
    * Team Members: Zhongyu Wang, Linhai Deng
    * Github Repository URL: https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit.git
    * Description of test hardware: (development boards, sensors, actuators, laptop + OS, etc) laptop

## 1. Video Presentation

https://youtube.com/shorts/kJqO3oqh_wA

---

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

![1.2](https://github.com/user-attachments/assets/6076e328-336d-4f9d-8df0-840ed20ad933)

---

### 3. Challenges

| # | Issue | Mitigation / Result |
|---|-------|---------------------|
| 1 | **Tight MCU RAM** – 14 kB total, 11 kB pre‑used by vendor Wi‑Fi stack | • Moved OLED font table to flash (`const PROGMEM`) <br>• Disabled WAV codec & speaker driver to free 1.7 kB <br>• Reduced FreeRTOS task stacks by 20 % |
| 2 | **No PWM on PA02** (speaker pin) | • Generated square wave via 10‑bit DAC as stop‑gap buzzer <br>• Added _“speaker → TCC1 WO[0]”_ note to PCB‑v2 issues list |
| 3 | **ISR ↔ MQTT race conditions** | • Created two FreeRTOS queues `xQueueDoorEvents` & `xQueueBuzzerCmd` so ISRs only enqueue, never touch sockets <br>• MQTT publish handled in Wi‑Fi task context |
| 4 | **Low‑budget PIR latency** | • \$2 Murata PIR selected for BoM limit → 3 m range & 900 ms rise‑time <br>• Debounce filter added, but long latency remains (see Future Work) |

---

### 4. Prototype Learnings

- **Measure first, design second** – RAM/flash profiling before adding features prevented late‑stage re‑writes.  
- **Off‑board UI wins** – pushing logic to Node‑RED let us tweak alert thresholds and e‑mail text without reflashing firmware.  
- **Test‑pads + CLI save hours** – being able to print raw ADC and switch pins through the CLI accelerated bring‑up and debugging.

---

### 5. Next Steps & Takeaways

#### Next Steps

1. **PCB v2**  
   * Move speaker to a true PWM pin (TCC1/WO[0])  
   * Add 2 MB SPI flash for snapshot buffering  
   * Integrate Li‑ion fuel gauge & charge‑status LED  

2. **Full‑duplex audio**  
   * Stream ESP‑EYE microphone to dashboard (WebRTC)  
   * Relay doorbell chime + pre‑recorded messages back to speaker  

3. **Ultra‑low‑power mode**  
   * Target \< 200 µA standby with PIR + WINC1500 M2M_wake interrupt  
   * Periodic RTC wake to push “heartbeat” MQTT packet  

4. **PIR upgrade**  
   * Replace analog Murata IRA‑S210ST01 with digital Panasonic EKMC series (6 m range, 170 ms latency)  
   * Remove 100 kΩ pull‑down and averaging filter once latency improves  

#### Course Takeaways

* **Altium first, code later** – learning Altium Designer taught us to allocate RAM/flash and pin‑mux **before** routing; early BoM + DRC saved one board spin.  
* **Complete E‑CAD workflow** – we now know multi‑sheet schematics, managed libraries, and 2‑layer placement / routing in Altium; gerbers and pick‑&‑place files went to JLC with zero clearance errors.  
* **Queues > critical sections** – FreeRTOS queues kept ISR‑to‑MQTT hand‑off race‑free.  
* **Cheap parts ≠ free** – a \$2 PIR saved \$4 but cost latency; breadboard sensors _before_ BoM lock.  
* **Node‑RED as cloud glue** – visual flows let us bolt on e‑mail, OTA and dashboard widgets without touching firmware.  
* **OTA is priceless** – Wi‑Fi OTAF‑U let us fix bugs and add features without ever opening the enclosure.

---

## 3. Hardware & Software Requirements

| Req ID | Requirement (from design spec) | Target Metric | Test / Validation Method | Result | Notes |
|-------:|--------------------------------|--------------:|--------------------------|:------:|-------|
| HW‑01 | **PIR motion detection latency / range** | < 1 s from movement to IRQ | Oscilloscope on PIR _AIN_ vs. UART “attention” print‑out | ⚠️ | Budget PIR; see cost‑driven challenge note |
| HW‑02 | **ADC noise floor** (12‑bit SAR) | ±3 LSB max @ 1 kHz | Logged 1 000 samples, calculated σ | ✅ | 3.3 V reference, RC filter |
| HW‑03 | **OLED legibility** indoors | 700 cd/m² min | Lux‑meter on white screen | ✅ | Contrast 100 % |
| HW‑04 | **Battery life (Li‑Ion 800 mAh)** | ≥ 24 h standby | Simulated with bench supply @ 30 µA sleep | ⚠️ | Needs deep‑sleep optimisation |
| SW‑01 | **FreeRTOS task scheduling jitter** (sensor task) | < 5 ms | vTaskGetRunTimeStats → std dev | ✅ | 1 kHz SysTick |
| SW‑02 | **MQTT round‑trip** dashboard→MCU | < 500 ms @ LAN | Timestamp at publish/ISR | ✅ | Wi‑Fi RSSI ‑57 dBm |
| SW‑03 | **OTA firmware update** | Triggered via Node‑RED, success rate > 95 % | 20 cycles, verify CRC | ✅ | Uses Atmel WINC1500 secure OTA |
| SW‑04 | **RAM usage** fits 14 kB | ≤ 13 kB after build | `arm-none-eabi-size` | ✅ | Freed audio stack |
| SW‑05 | **Flash usage** fits 256 kB | ≤ 220 kB image | Linker map | ✅ | Leaves 20 % margin |
| SW‑06 | **Email alert reliability** | 100 % on “attention” event | 50 door‑open cycles | ✅ | SMTP relay via Node‑RED |
| INT‑01 | **System boots on battery only** | 3.4 V → 4.2 V | Power‑cycle test | ✅ | Buck‑boost TPS61291 |

> **Legend**   
> ✅ Met ⚠️ Partially met / needs improvement ❌ Not met

### Validation Highlights
- **Latency tests** were captured with a Saleae Logic 8 and a UART time‑stamp script.  
- **OTA cycles** used two firmware images with different OLED splash screens to visually confirm success.  
- **Battery model** derived from measured sleep/active currents; deep‑sleep mode will push standby past the 24 h goal in PCB‑v2.

---

## 4. Project Photos & Screenshots

![1](https://github.com/user-attachments/assets/c5e2b057-9cc8-48d4-8ce6-dd802e0b4e99)

![2](https://github.com/user-attachments/assets/e7549c49-9389-4be4-8865-2ed3f04d349b)

![3](https://github.com/user-attachments/assets/408e9619-36ab-4047-997a-8ee7dc5547fd)


---

## Codebase

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device

