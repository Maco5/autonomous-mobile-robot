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

The main loop pings the ultrasonic sensor every cycle: a 10 µs trigger pulse, then measures the echo and converts it to distance. From there, it's a simple state check:

- **Distance > 25 cm:** drive forward.
- **Distance < 25 cm:** cut PWM to 0 and stop.
- **Distance < 15 cm:** back up briefly, then do a differential turn (left motors reverse, right motors forward) until the path is clear again.

### Arm Control

The arm runs a hardcoded pick-and-place sequence where I manually tuned the PWM values for each servo's "home," "grip," and "drop" positions so the arm wouldn't fight itself or stall at the end of each travel.

## Challenges

The ultrasonic sensor was noisy at first because it kept returning garbage distances, especially when pointed at soft or angled surfaces where the sound scatters instead of reflecting cleanly. I fixed it by isolating the sensor physically from the chassis (it was picking up vibration) and tightening up the trigger timing so I was not catching leftover echoes from the previous ping.

## Future Improvements

The current firmware is built around `delay()`, which blocks execution for the duration of every call. This works for a purely reactive robot with one active behaviour at a time, but it prevents any form of concurrency, where the sensor can't be polled while the arm is moving, and behaviours can't overlap or interrupt each other.

The next iteration would replace the blocking structure with a `millis()`-based scheduler, allowing the main loop to interleave sensing, locomotion, and manipulation. This is a prerequisite for anything beyond reactive avoidance: sensor fusion, path planning, or coordinated navigation and pick-and-place routines.

Additional improvements worth exploring:
- **Sensor filtering:** a running median or Kalman filter on the ultrasonic readings to further reduce false triggers.
- **Closed-loop motor control:** adding wheel encoders for odometry and PID speed regulation, rather than open-loop PWM.
- **Modular firmware architecture:** separating sensing, control, and actuation into discrete modules with clean interfaces, making the codebase easier to extend.
