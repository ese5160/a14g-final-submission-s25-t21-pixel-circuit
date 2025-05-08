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

###  Device Description

* **What it is** – A battery‑powered smart‑door add‑on that live‑streams video, senses motion and pushes e‑mail alerts. A Node‑RED dashboard shows status, hosts an emergency button and triggers OTA firmware updates.  
* **Why we built it** – Dorm rooms lack intercoms; package theft and unwanted visitors are common. We wanted a retrofit solution—no drilling, no paid cloud.
* **Internet integration** – Events publish over MQTT to an Azure VM running Node‑RED. From the dashboard you can:
  * Watch live video & snapshots  
  * Read door status (“attention/safe”) in real time  
  * Press **Call the Police** to sound an on‑door buzzer and display message  
  * Trigger an over‑the‑air firmware update (OTAFU)

---

###  Device Functionality

| Element | Details |
| ------- | ------- |
| **Sensors** | Murata IRA‑S210ST01 PIR → ADC PA03. Threshold 1000 raw (≈1.2 V) → `attention` vs `safe`. |
| **Actuators** | *I²C OLED* (UG‑2864HSWEG01) + on‑board LED. Panic button toggles text **“the police is coming”** and LED.<br>*Speaker* footprint present but disabled (MCU RAM limits). |
| **Live video** | ESP‑EYE 2 MP camera streams MJPEG + built‑in mic (no extra MCU RAM). |
| **Snapshot & mail** | Each `attention` sends JPEG + timestamp to user‑supplied e‑mail via Node‑RED SMTP. |
| **OTA update** | Dashboard buttons **GOLDEN IMAGE** / **OTA UPDATE (FW)** push a firmware URL; board downloads and self‑flashes. |

![1.2](https://github.com/user-attachments/assets/041a1774-bdcf-436c-9d53-079da5d34b9d)

---

###  Challenges

| # | Issue | Mitigation / Result |
|---|-------|---------------------|
| 1 | **Tight MCU RAM** – 14 kB total, 11 kB pre‑used by vendor Wi‑Fi stack | • Moved OLED font table to flash (`const PROGMEM`) <br>• Disabled WAV codec & speaker driver to free 1.7 kB <br>• Reduced FreeRTOS task stacks by 20 % |
| 2 | **No PWM on PA02** (speaker pin) | • Generated square wave via 10‑bit DAC as stop‑gap buzzer <br>• Added _“speaker → TCC1 WO[0]”_ note to PCB‑v2 issues list |
| 3 | **ISR ↔ MQTT race conditions** | • Created two FreeRTOS queues `xQueueDoorEvents` & `xQueueBuzzerCmd` so ISRs only enqueue, never touch sockets <br>• MQTT publish handled in Wi‑Fi task context |
| 4 | **Low‑budget PIR latency** | • \$2 Murata PIR selected for BoM limit → 3 m range & 900 ms rise‑time <br>• Debounce filter added, but long latency remains (see Future Work) |
| 5 | **Node-RED not optimized for video** | • Switched from real video to periodic JPEG frames (~1–3 fps) <br>• Frame sent as Base64 via MQTT or HTTP POST, displayed with `<img>` in dashboard |
| 6 | **Web dashboard frame flicker** | • Introduced timestamp-based buffer to avoid image overwrite conflicts <br>• Only update display if new frame differs from previous |
| 7 | **Large JPEG frames cause MQTT choke** | • Capped resolution to QQVGA (160×120), frame <5 kB <br>• Allowed only 1 in-flight image at a time to prevent overload |
| 8 | **Browser-side image memory growth** | • Manually revoked `<img>` blob URLs and cleared DOM nodes every 10 frames <br>• Exploring `<canvas>`-based rendering in Future Work |

---

###  Prototype Learnings

- **Measure first, design second** – RAM/flash profiling before adding features prevented late‑stage re‑writes.  
- **Off‑board UI wins** – pushing logic to Node‑RED let us tweak alert thresholds and e‑mail text without reflashing firmware.  
- **Test‑pads + CLI save hours** – being able to print raw ADC and switch pins through the CLI accelerated bring‑up and debugging.

---

###  Next Steps & Takeaways

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

5. **Web-based video frontend**  
   * Replace Node-RED with Flask + WebSocket server for smoother JPEG streaming  
   * Use `<canvas>` or `<video>` HTML5 elements for improved rendering control  
   * Support frame annotations (bounding boxes, timestamps)  

6. **MJPEG / HLS streaming**  
   * Explore MJPEG over HTTP or HLS (HTTP Live Streaming) for better compatibility  
   * Reduce latency and improve real-time responsiveness of video feed  

7. **Video-aware flow control**  
   * Implement adaptive frame rate based on network condition and browser feedback  
   * Buffer overflow protection and dropped-frame detection via frame index tracking  

#### Course Takeaways

* **Altium first, code later** – learning Altium Designer taught us to allocate RAM/flash and pin‑mux **before** routing; early BoM + DRC saved one board spin.  
* **Complete E‑CAD workflow** – we now know multi‑sheet schematics, managed libraries, and 2‑layer placement / routing in Altium; gerbers and pick‑&‑place files went to JLC with zero clearance errors.  
* **Queues > critical sections** – FreeRTOS queues kept ISR‑to‑MQTT hand‑off race‑free.  
* **Cheap parts ≠ free** – a \$2 PIR saved \$4 but cost latency; breadboard sensors _before_ BoM lock.  
* **Node‑RED as cloud glue** – visual flows let us bolt on e‑mail, OTA and dashboard widgets without touching firmware.  
* **OTA is priceless** – Wi‑Fi OTAF‑U let us fix bugs and add features without ever opening the enclosure.

---

###  Project Links

Node-RED instance: [http://20.55.16.155:1880/ui](http://20.55.16.155:1880/#flow/e5ea7423b886c1a5)

Altium 365: [https://upenn-eselabs.365.altium.com/designs/AF1AAD73-CCAB-4D4B-BED5-52957B402218?activeDocumentId=Top20Level.SchDoc(1)&variant=%5BNo+Variations%5D&activeView=SCH&location=%5B1,97.61,18.77,22.79%5D#design](https://upenn-eselabs.365.altium.com/designs/AF1AAD73-CCAB-4D4B-BED5-52957B402218?activeView=SCH&activeDocumentId=Top20Level.SchDoc(1)&variant=[No+Variations]&location=[1,97.61,18.77,22.79]#design)

---

## 3. Hardware & Software Requirements

| Req ID | Requirement (from design spec) | Target Metric | Test / Validation Method | Result | Notes |
|-------:|--------------------------------|--------------:|--------------------------|:------:|-------|
| HW‑01 | **PIR motion detection latency / range** | < 1 s from movement to IRQ | Oscilloscope on PIR _AIN_ vs. UART “attention” print‑out | ⚠️ | Budget PIR; see cost‑driven challenge note |
| HW‑02 | **ADC noise floor** (12‑bit SAR) | ±3 LSB max @ 1 kHz | Logged 1 000 samples, calculated σ | ✅ | 3.3 V reference, RC filter |
| HW‑03 | **OLED legibility** indoors | 700 cd/m² min | Lux‑meter on white screen | ✅ | Contrast 100 % |
| HW‑04 | **Battery life (Li‑Ion 800 mAh)** | ≥ 24 h standby | Simulated with bench supply @ 30 µA sleep | ⚠️ | Needs deep‑sleep optimisation |
| HW‑05 | **ESP32-S3-EYE JPEG frame capture** | ≥ 1 fps @ QQVGA (160×120) | Measured frame interval with timestamped logs | ✅ | 2.3 fps avg; JPEG size ~4.6 kB |
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

### Final project, including any casework or interfacing elements that make up the full project

![1](https://github.com/user-attachments/assets/a1ce804e-ad0c-412c-8596-9393d25665dc)

### The standalone PCBA, top

![2](https://github.com/user-attachments/assets/9ea0ce55-8db3-4d01-a846-1ae562fa1a7e)

### The standalone PCBA, bottom

![3](https://github.com/user-attachments/assets/663e0a9a-58ee-44c1-b477-f69e6e45662e)

### Thermal camera images while the board is running under load

![4](https://github.com/user-attachments/assets/879712b7-0bb4-4ab0-8274-fd57b653cfb3)

### The Altium Board design in 2D view

![5](https://github.com/user-attachments/assets/5fcee5cd-0386-45ad-9c55-afe58563d6a5)

### The Altium Board design in 3D view

![6](https://github.com/user-attachments/assets/8b6a5885-1b30-4e48-8821-32642b067cef)

### Node-RED dashboard

![7](https://github.com/user-attachments/assets/3d77adbb-3565-4aa4-907d-e06d66b21bc1)

![8](https://github.com/user-attachments/assets/cbaaecd0-cf18-4f6e-9756-0d85dd1d8b37)

### Node-RED backend

![9](https://github.com/user-attachments/assets/4bb5520c-dec3-4cf9-9ca9-56336c8f7dc7)

![10](https://github.com/user-attachments/assets/1b322441-2be6-4c77-9e6f-c25eb60dea6a)

### Block diagram of your system

![11](https://github.com/user-attachments/assets/111a50f1-30c5-44fc-a5d2-f59f5df9ca6e)

---

## Codebase
 
- A link to your final embedded C firmware codebase:  
  [https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/tree/main/final-project-t21-pixel-circuit](https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/tree/main/final-project-t21-pixel-circuit)

- A link to your Node-RED dashboard code:  
  [https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/blob/main/final-project-t21-pixel-circuit/NODE-RED/flows%20(1).json](https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/blob/main/final-project-t21-pixel-circuit/NODE-RED/flows%20(1).json)

- Links to any other software required for the functionality of your device:  
  [https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/tree/main/video_stream_server](https://github.com/ese5160/a14g-final-submission-s25-t21-pixel-circuit/tree/main/video_stream_server)

##  Open-Source Components Used

This project leverages the following open-source components:

- [`video_stream_server`](https://github.com/espressif/esp-iot-solution/tree/master/examples/video_stream_server) from Espressif's [`esp-iot-solution`](https://github.com/espressif/esp-iot-solution) repository  
  Licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)  
  We extended this component by integrating MQTT-based image publishing and control features.

## Acknowledgements

- Espressif Systems for providing comprehensive ESP-IDF examples and solutions.
- Community documentation and tutorials referenced via [docs.espressif.com](https://docs.espressif.com/) and relevant GitHub issues/discussions.

