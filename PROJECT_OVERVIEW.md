# Autonomous Mobile Robot (AMR)

![Top View](IMG_8309.jpeg)
![Front View](IMG_8311.jpeg)
![Rear View](IMG_8312.jpeg)
![Undercarriage Motors](IMG_8313.jpeg)

## Overview

A differential-drive robot I built to get hands-on with embedded C++ and sensor integration. It runs on an Arduino Uno and does reactive obstacle avoidance with an ultrasonic sensor, and has a 3-servo arm for basic pick-and-place sequences.

The goal was to get comfortable writing firmware that talks to real hardware, such as reading a sensor, driving motors through an H-bridge, and coordinating it all in a control loop.

## Hardware

- **Microcontroller:** Arduino Uno R3 (ATmega328P)
- **Drive:** L298N H-bridge driving 4 DC geared motors on a 4WD chassis
- **Arm:** 3 hobby servos (base rotation, arm lift, claw)
- **Sensor:** HC-SR04 ultrasonic (40 kHz)
- **Power:** 7.4V LiPo, regulated to 5V for logic

## Pin Mapping

| Function | Component Pin | Arduino Pin |
|---|---|---|
| Ultrasonic Trigger | TRIG | D12 |
| Ultrasonic Echo | ECHO | D13 |
| Left Motor Direction | IN1 | D2 |
| Left Motor PWM | ENA | D5 |
| Right Motor Direction | IN3 | D4 |
| Right Motor PWM | ENB | D6 |
| Claw Servo | Signal | D9 |
| Arm Servo | Signal | D10 |
| Base Servo | Signal | D11 |

## How It Works

### Obstacle Avoidance

The main loop pings the ultrasonic sensor every cycle — a 10 µs trigger pulse, then measure the echo and convert to distance. From there it's a simple state check:

- **Distance > 25 cm:** drive forward.
- **Distance < 25 cm:** cut PWM to 0 and stop.
- **Distance < 15 cm:** back up briefly, then do a differential turn (left motors reverse, right motors forward) until the path is clear again.

Not fancy, but it handles an obstacle course reliably.

### Arm Control

The arm runs a hardcoded pick-and-place sequence. I manually tuned the PWM values for each servo's "home," "grip," and "drop" positions so the arm wouldn't fight itself or stall at the end of travel.

## What I Ran Into

The ultrasonic sensor was noisy at first — it kept returning garbage distances, especially when pointed at soft or angled surfaces where the sound scatters instead of reflecting cleanly. I fixed it by isolating the sensor physically from the chassis (it was picking up vibration) and tightening up the trigger timing so I wasn't catching leftover echoes from the previous ping.

## What I'd Change

The whole thing is built around `delay()`, which blocks the entire program while it runs. That's fine for a simple reactive robot, but it means the Uno can't do anything else — no watching the sensor while the arm moves, no overlapping behaviors. Rewriting the loop around `millis()` timers would be the first thing I'd do if I picked this back up, and it's the obvious step toward anything more interesting than pure obstacle avoidance.
