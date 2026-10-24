# STM32 PIR Motion Detection Projects

Three progressive embedded projects built around the **HC-SR501 PIR motion sensor** on an **STM32 NUCLEO-F446RE** board, developed with STM32CubeIDE and the HAL library. Each project builds on the previous one, and each lives on its own branch.

| Branch | Project | Level |
|---|---|---|
| `main` | 1. Motion Detection and UART Monitoring | Beginner |
| `Buzzer-Alarm` | 2. Motion-Activated Alarm System | Intermediate |
| `Smart-Room-Prototype` | 3. Smart Room Monitoring System | Advanced |

## Hardware

- STM32 NUCLEO-F446RE development board
- HC-SR501 PIR motion sensor
- Passive buzzer
- LDR (photoresistor) and a 10 kΩ resistor
- Breadboard and jumper wires

The on-board LED (LD2) and the on-board user button (B1) are used, so no external LED or switch is required.

## Project 1: Motion Detection and UART Monitoring (`main`)

The simplest version of the system. When the PIR sensor detects motion, the on-board LED turns on and a message is sent to the computer over UART. When the sensor output goes low again, the LED turns off.

**How it works**
- The PIR output is read as a digital input.
- The program reacts only when the sensor state changes, so each event is logged once instead of being repeated continuously.
- UART output: `Motion Detected` / `Motion Ended`.

**What it demonstrates:** GPIO input, reading a digital sensor signal, UART logging.

## Project 2: Motion-Activated Alarm System (`Buzzer-Alarm`)

Extends Project 1 into a real alarm. Motion now triggers an external interrupt, and a passive buzzer driven by PWM sounds the alarm together with the LED.

**How it works**
- The PIR output triggers an external interrupt (EXTI) on a rising edge. The interrupt only raises a flag, and the main loop does the actual work.
- The system has two states: **Idle** and **Alarm**.
- In the Alarm state the buzzer beeps and the LED blinks in sync.
- If new motion arrives during the alarm, the alarm timer restarts.
- If no motion is detected for 10 seconds, the alarm turns off automatically.
- All timing is non-blocking (based on the system tick), so the program never freezes in a delay.
- UART output: `ALARM ON` / `Motion again, alarm extended` / `ALARM OFF`.

**What it demonstrates:** EXTI interrupts, PWM tone generation, timer-based timing, a simple state machine, and event handling.

## Project 3: Smart Room Monitoring System (`Smart-Room-Prototype`)

Combines everything into one system: motion detection, ambient light sensing, a user-controlled on/off button, LED, buzzer, and UART reporting.

**How it works**
- The on-board user button (B1) toggles the system between **armed** and **disarmed**. When disarmed, everything stays off and motion is ignored.
- When armed, the LDR measures ambient light through the ADC. If the room is dark, the LED turns on.
- If the PIR detects motion while armed, the alarm starts (buzzer beeps, LED blinks) and ends after 10 seconds without new motion.
- A small hysteresis on the light threshold prevents the LED from flickering when the light level sits near the limit.
- Button and light readings are sampled every 200 ms, which also works as simple debouncing.
- Events are reported over UART: `SYSTEM ARMED`, `SYSTEM DISARMED`, `Dark (LDR=...)`, `Light (LDR=...)`, `ALARM ON`, `ALARM OFF`.

**What it demonstrates:** ADC + GPIO + EXTI + UART together, multi-sensor management, a state-machine design, and a modular program structure.

## Wiring Summary

| Component | Connection |
|---|---|
| PIR sensor VCC / GND | 5V / GND |
| PIR sensor OUT | PB0 |
| Buzzer (+ / -) | PB6 (TIM4 CH1) / GND |
| LDR divider | 3.3V → LDR → PA0 → 10 kΩ → GND |
| LED | On-board LD2 (PA5) |
| Button | On-board B1 (PC13) |
| UART | USART2 over the ST-Link virtual COM port, 115200 baud |

Which components are used depends on the project: Project 1 uses only the PIR sensor, Project 2 adds the buzzer, and Project 3 adds the LDR and the user button.

## Notes

- **PIR warm-up:** the sensor needs about 30-60 seconds after power-up to stabilize, and it may trigger randomly during that time.
- **Testing the PIR:** mount the sensor in a fixed position. Moving the sensor in your hand makes the whole room look like motion.
- **PIR potentiometers:** keep the delay potentiometer at a short setting and the jumper in repeat-trigger (H) mode, because the alarm duration is handled in software.
- **Dark threshold:** the `DARK_TH` value in Project 3 depends on your LDR and lighting. Cover and uncover the LDR, read the values printed over UART, and set the threshold between them.

## How to Run

1. Clone the repository and check out the branch of the project you want.
2. Open the project in STM32CubeIDE.
3. Wire the hardware as described above.
4. Build and flash the NUCLEO-F446RE.
5. Open a serial terminal on the ST-Link COM port at 115200 baud to see the output.
