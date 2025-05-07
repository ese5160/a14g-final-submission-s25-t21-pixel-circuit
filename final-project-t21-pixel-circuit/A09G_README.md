# A09G Comm Protocols

```
Team Number: 21
Team Name: pixel & circuit
Team Members: Zhongyu Wang, Linhai Deng
GitHub Repository URL: https://github.com/ese5160/final-project-t21-pixel-circuit.git
Description of test hardware: (development boards, sensors, actuators, laptop + OS, etc) laptop
```
## 2. Your most important (high-risk) peripheral

**1. Current Situation**

**1.1 What the High-Risk Component Is**

In this project, the chosen high-risk peripheral is a PIR (Passive Infrared) motion sensor.

**1.2 What Makes It High Risk**

- Core complexity: Must balance low power, high sensitivity, and strong noise immunity to ensure accurate and reliable data.

- System-critical importance: The PIR sensor detects whether someone or something is in front of the door and triggers the rest of the system. Unstable or unreliable readings could lead to missed detections or false alarms.

- Hardware/software interface risks: It requires correct ADC or digital input configuration (depending on the sensor’s output type), possibly additional external circuits for signal conditioning, ensuring the signal quality and stability.

**1.3 How It Interfaces with the MCU and Other System Components**

- MCU pin: For example, on the SAM D21 microcontroller, use pin PA03 as the digital input from the PIR (or as an ADC input if needed).

- Power and level shifting: If the PIR operates at 5V while the MCU is 3.3V, a level shifter or hardware logic might be required. If the PIR output is 3.3V compatible, a direct connection may be enough.

- Integration with the system: Once motion is detected, the PIR drives its output pin high (or low). The MCU detects this transition via an external interrupt or polling, then possibly activates a camera or sends a mobile alert.

**2. Future Plans**

1. Minimum Viable Validation

Read and filter the PIR sensor’s trigger signal, apply any software debouncing or filtering, and verify that motion detection works effectively within a ~3–5 meter range.

When an interrupt is detected, log the timestamp to analyze PIR latency.

2. Integration into Main Application

After confirming basic functionality, link the PIR sensor with other components (camera, microphone, Wi-Fi module) to enable real-time streaming or user alerts.

3. Extended Optimization

Adjust sensor gain, lens coverage (or a Fresnel lens), and software thresholds if sensitivity is found to be too high or too low. Software filtering or detection algorithms might be refined.

**3. Critical/Essential Functionality**

- Detect motion and provide a trigger signal

The PIR module outputs a digital GPIO signal; a high (or low) level indicates motion detected.

The MCU either polls or sets an external interrupt on this pin; once a rising or falling edge is detected, the MCU handles the event accordingly.

- Core parameters

Detection range: Typically ~3–5 meters

Trigger duration: The high-level signal duration after motion detection

Environmental tolerance: Temperature changes, electrical noise, and lighting conditions can cause false triggers

**4. Testing Plan**

1. Basic Interrupt Test

Verify that when no one is moving in front of the sensor, the PIR output remains stable. When a person moves within range, the PIR triggers a high-level output, causing an MCU interrupt.

Test multiple distances (1m, 3m, 5m) and movement speeds, checking detection accuracy.

2. Signal Stability Test

Run the system for extended periods to see whether the PIR incorrectly triggers (false positives) or misses actual motion (false negatives).

3. Power Consumption Test (optional)

If battery-powered, measure the standby and active current. Evaluate whether continuous monitoring is feasible within the required power budget.

4. Interference/Noise Test (optional)

Check performance under varying temperature or lighting conditions to ensure consistent results. Introduce additional shielding or filtering if necessary.

**5. Conclusions & Next Steps**

- Current Findings

The PIR sensor reliably detects motion in test conditions and triggers the MCU. Further real-world doorfront testing is necessary to validate false alarm rates.

- Future Work

If test results are satisfactory, integrate into the main application (e.g., “visitor approaches → camera captures image → Wi-Fi upload”).

If false triggering is common, consider additional hardware shielding, lens optimization, or enhanced signal filtering.

**6. Hardware Not Yet Received?**

If the actual PIR hardware has not yet arrived, you can simulate it with a regular GPIO pin, using timed triggers in firmware to:

1. Confirm that the MCU captures external interrupts correctly.

2. Ensure the software logic to distinguish triggered vs. idle states is solid.

3. Verify event handling, time logging, and any subsequent network/data operations. When the real sensor arrives, simply swap in the actual GPIO input line for final validation.

**7. Source Code Location**

This repository contains custom driver or library code for the PIR sensor in the following directory:

- final-project-t21-pixel-circuit-main\Application\src

Within this directory, you will find:
- `PIR_Test.c` – source code for initializing and reading from the PIR sensor.
- `PIR_Test.h` – corresponding header file exposing the test task creation function.

I have upload the PIR_Test.c, PIR_Test.h and main21.c in github

and here is the video

https://youtube.com/shorts/MRvaU8BvbvM?feature=share
