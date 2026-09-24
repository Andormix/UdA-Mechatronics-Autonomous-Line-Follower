# Autonomous PID Line-Follower Robot — UdA Mechatronics

[![Arduino](https://img.shields.io/badge/Arduino-C%2B%2B-00979D?style=flat&logo=arduino&logoColor=white)](#)
[![PID Control](https://img.shields.io/badge/Control_System-PID_Algorithm-FF6F00?style=flat)](#)
[![Hardware](https://img.shields.io/badge/Hardware-Pololu_%7C_DFRobot-000000?style=flat)](#)
[![University of Andorra](https://img.shields.io/badge/Academic-Universitat_d'Andorra-003366?style=flat)](#)

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/bffecef4-5e87-4c50-b168-953269a5a0ee"
    alt="Line Follower Robot Showcase"
    width="100%"
  />
</p>

Autonomous line-following robot developed in C++ for Arduino as part of the Computer Science curriculum at the **Universitat d'Andorra (UdA)**.

The robot uses a custom **Proportional-Integral-Derivative (PID)** feedback loop to process real-time signals from a six-sensor reflectance array and control the speed of two DC motors. The system also includes dynamic sensor calibration, differential motor compensation, and LED indicators that display the robot's current trajectory correction.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Project Description](#project-description)
- [Features](#features)
- [Hardware Specifications](#hardware-specifications)
- [Pin Configuration](#pin-configuration)
- [PID Control System](#pid-control-system)
- [Control Parameters](#control-parameters)
- [Motor Power Compensation](#motor-power-compensation)
- [Indicator Logic](#indicator-logic)
- [Execution Flow](#execution-flow)
- [Repository Structure](#repository-structure)
- [Requirements](#requirements)
- [Installation and Upload](#installation-and-upload)
- [Calibration Procedure](#calibration-procedure)
- [Future Improvements](#future-improvements)
- [Academic Context](#academic-context)
- [License](#license)

---

## Project Overview

The objective of this project is to build a fast and stable autonomous robot capable of following a black track using infrared reflectance sensors.

The embedded software performs the following operations:

1. Calibrates the reflectance sensors during startup.
2. Reads the position of the line in real time.
3. Calculates the deviation from the center of the track.
4. Applies PID control to calculate a correction value.
5. Adjusts the speed of the left and right motors independently.
6. Activates LED indicators according to the direction of the correction.

---

## Project Description

### English

This project implements an autonomous line-following robot using an Arduino-compatible microcontroller, a Pololu QTR-6A reflectance sensor array, and a DFRobot MD1.3 dual motor controller.

The robot estimates the position of the track using six analog infrared sensors. The measured position is compared with the center of the sensor array, producing an error value. This error is processed by a PID controller, which dynamically adjusts the motor speeds to keep the robot aligned with the line.

### Español

Este proyecto implementa un robot seguidor de línea autónomo utilizando un microcontrolador compatible con Arduino, una matriz de sensores de reflectancia Pololu QTR-6A y un controlador doble de motores DFRobot MD1.3.

El robot estima la posición de la línea mediante seis sensores infrarrojos analógicos. La posición medida se compara con el centro de la matriz de sensores para obtener un error. Este error se procesa mediante un controlador PID, que ajusta dinámicamente la velocidad de los motores para mantener el robot alineado con la trayectoria.

---

## Features

- Autonomous black-line tracking.
- PID-based feedback control.
- Six-channel analog reflectance sensor array.
- Dynamic sensor calibration at startup.
- Differential PWM motor control.
- Individual motor power compensation.
- Real-time correction for curves and deviations.
- Direction indicators using two LEDs.
- Arduino-compatible C++ firmware.
- Low-level motor control using PWM and digital direction signals.

---

## Hardware Specifications

| Component | Description |
| :--- | :--- |
| Microcontroller | Arduino Uno, Nano, or Mega |
| Sensor Array | Pololu QTR-6A Reflectance Sensor Array |
| Motor Driver | DFRobot MD1.3 2A Dual Motor Controller (`DRI0002`) |
| Motors | Two DC motors |
| Indicators | Two LEDs for trajectory indication |
| Power Supply | Battery compatible with the selected motors and motor driver |
| Programming Interface | USB connection through Arduino board |

---

## Pin Configuration

| Component | Function | Arduino Pin |
| :--- | :--- | :--- |
| Sensor 1 | Analog reflectance input | `A0` |
| Sensor 2 | Analog reflectance input | `A1` |
| Sensor 3 | Analog reflectance input | `A2` |
| Sensor 4 | Analog reflectance input | `A3` |
| Sensor 5 | Analog reflectance input | `A4` |
| Sensor 6 | Analog reflectance input | `A5` |
| Left Motor | PWM speed control (`E1`) | `D6` |
| Left Motor | Direction control (`M1`) | `D7` |
| Right Motor | PWM speed control (`E2`) | `D5` |
| Right Motor | Direction control (`M2`) | `D4` |
| Left Indicator | Left signal LED (`L_ESQUERRA`) | `D9` |
| Right Indicator | Right signal LED (`L_DRETA`) | `D8` |

> **Note:** Verify the motor driver wiring and polarity before powering the robot. Incorrect connections may cause unexpected motor direction or hardware damage.

---

## PID Control System

The robot uses the `QTRSensors` library to obtain the position of the line from the six reflectance sensors.

The sensor array returns a position value between `0` and `5000`. For six sensors, the center position is approximately `2500`.

### Error Calculation

```cpp
error = qtra.readLine(sensors) - 2500;
```

Where:

- `qtra.readLine(sensors)` is the measured line position.
- `2500` represents the center of the sensor array.
- `error` represents the deviation from the desired trajectory.

A positive error indicates that the line is displaced toward one side of the sensor array, while a negative error indicates displacement toward the opposite side.

### PID Formula

The correction value is calculated using the following equation:

```text
Correction = (KP × error)
           + (KI × accumulatedError)
           + (KD × changeInError)
```

In mathematical notation:

$$
\text{Correction} =
(K_P \times e) +
(K_I \times \sum e) +
(K_D \times \Delta e)
$$

Where:

- `e` is the current error.
- `Σe` is the accumulated error.
- `Δe` is the difference between the current and previous error.
- `KP` is the proportional gain.
- `KI` is the integral gain.
- `KD` is the derivative gain.

### Motor Speed Correction

The correction is applied differentially to the two motors:

```text
Left Motor Speed  = Base Left Speed  + Correction
Right Motor Speed = Base Right Speed - Correction
```

This causes the robot to increase the speed of one motor while reducing the speed of the other, allowing it to return toward the center of the line.

---

## Control Parameters

The initial PID tuning constants are:

```cpp
#define KP 0.04
#define KI 0.1
#define KD 0.2
```

| Parameter | Value | Description |
| :--- | :---: | :--- |
| `KP` | `0.04` | Proportional gain |
| `KI` | `0.1` | Integral gain |
| `KD` | `0.2` | Derivative gain |

### Parameter Tuning

The PID constants may require adjustment depending on:

- Track surface and line color.
- Sensor height above the track.
- Motor characteristics.
- Battery voltage.
- Robot weight.
- Wheel grip.
- Track curvature.
- Desired maximum speed.

General tuning recommendations:

- Increase `KP` to obtain a stronger response to current deviations.
- Increase `KD` to reduce oscillations and improve curve stability.
- Increase `KI` only when a persistent error remains over time.
- Avoid excessive `KI`, as it may cause integral windup and unstable behavior.

---

## Motor Power Compensation

The two motors may not produce exactly the same mechanical speed when the same PWM value is applied. To compensate for this difference, independent base speeds are used.

| Constant | Value | Description |
| :--- | :---: | :--- |
| `VEL_MIN` | `180` | Base speed for the left motor |
| `VEL_MIN_E` | `197` | Base speed for the right motor |
| `VEL_MAX` | `255` | Maximum PWM value |

The right motor uses a higher base speed to compensate for mechanical or electrical differences between both motors.

```text
Left Motor Base Speed  = 180
Right Motor Base Speed = 197
Maximum PWM Value      = 255
```

These values should be recalibrated if the motors, wheels, battery, or chassis are changed.

---

## Indicator Logic

The robot uses two LEDs to indicate the direction and magnitude of the current trajectory correction.

| Condition | Meaning | Left LED | Right LED |
| :--- | :--- | :---: | :---: |
| `-1000 <= error <= 1000` | Robot approximately centered | ON | ON |
| `error > 1000` | Robot displaced to the right | ON | OFF |
| `error < -1000` | Robot displaced to the left | OFF | ON |

The indicator logic is implemented through the `intermitents` control routine.

```text
Centered position:
    Left LED  = ON
    Right LED = ON

Right displacement:
    Left LED  = ON
    Right LED = OFF

Left displacement:
    Left LED  = OFF
    Right LED = ON
```

---

## Execution Flow

### 1. Initialization

The `setup()` function configures:

- Sensor input pins.
- Motor control pins.
- LED output pins.
- Serial communication, if enabled.
- Initial motor state.

### 2. Sensor Calibration

During startup, the robot performs a dynamic calibration routine using:

```cpp
qtra.calibrate(QTR_EMITTERS_ON);
```

The calibration process lasts approximately **7.6 seconds**. During this period, the sensor array must be moved manually from side to side over the black line and the surrounding surface.

### 3. Signal Acquisition

Inside the main `loop()`, the firmware reads the six analog sensors and calculates the position of the line:

```cpp
position = qtra.readLine(sensors);
```

### 4. Error Calculation

The measured position is compared with the center value:

```cpp
error = position - 2500;
```

### 5. PID Computation

The current error, accumulated error, and change in error are used to calculate the motor correction.

### 6. Motor Adjustment

The correction is applied to the left and right motor speeds. The resulting values are constrained between `0` and `VEL_MAX`.

### 7. Indicator Update

The LEDs are updated according to the current error and the direction of the correction.

---

## Repository Structure

```text
.
├── seguidor_linea.ino
└── README.md
```

### Main Files

| File | Description |
| :--- | :--- |
| `seguidor_linea.ino` | Main Arduino sketch containing initialization, sensor calibration, PID computation, motor control, and LED indicators |
| `README.md` | Project documentation and technical specifications |

---

## Requirements

### Software

- Arduino IDE `1.8.x` or newer.
- Arduino IDE `2.x` is also supported.
- C++ compiler included with the Arduino IDE.
- Pololu `QTRSensors` library.

### Hardware

- Arduino Uno, Nano, or Mega.
- Pololu QTR-6A reflectance sensor array.
- DFRobot MD1.3 dual motor controller (`DRI0002`).
- Two compatible DC motors.
- Two LEDs.
- Appropriate power supply or battery pack.
- Robot chassis and wheels.

---

## Installation and Upload

### 1. Clone the Repository

```bash
git clone https://github.com/Andormix/UdA-mecatronica-seguidor-linea.git
cd UdA-mecatronica-seguidor-linea
```

### 2. Install the Required Library

Install the **QTRSensors** library through the Arduino IDE:

1. Open the Arduino IDE.
2. Go to **Tools > Manage Libraries**.
3. Search for `QTRSensors`.
4. Install the library compatible with your Arduino environment.

### 3. Open the Arduino Sketch

Open the following file:

```text
seguidor_linea.ino
```

### 4. Configure the Arduino Board

In the Arduino IDE:

1. Connect the Arduino board through USB.
2. Select the appropriate board under **Tools > Board**.
3. Select the correct serial port under **Tools > Port**.
4. Verify that the configured pins match the physical wiring.

### 5. Upload the Firmware

Click **Upload** or use the following keyboard shortcut:

- Windows/Linux: `Ctrl + U`
- macOS: `Cmd + U`

### 6. Perform Calibration

Immediately after resetting the board, move the sensor array from side to side across the black line during the initial calibration period.

---

## Calibration Procedure

For correct operation, follow these steps:

1. Place the robot on the track.
2. Ensure that the sensor array is positioned at a consistent height above the surface.
3. Reset or power on the Arduino board.
4. During the calibration period, move the sensor array across:
   - The black line.
   - The white or lighter surface surrounding the line.
5. Allow the calibration routine to finish.
6. Place the robot on the track and observe its behavior.
7. Adjust the PID constants if the robot oscillates, reacts slowly, or loses the line.

> **Important:** Calibration should be repeated whenever the track surface, sensor height, lighting conditions, or sensor position changes significantly.

---

## Troubleshooting

### The robot moves in the wrong direction

Check:

- Motor polarity.
- Direction pins `D7` and `D4`.
- Motor driver wiring.
- The sign used in the PID correction formula.

### The robot oscillates heavily

Try:

- Reducing `KP`.
- Increasing `KD`.
- Reducing the base motor speeds.
- Checking that the sensor array is firmly mounted.

### The robot reacts too slowly

Try:

- Increasing `KP`.
- Slightly increasing the base motor speeds.
- Checking the battery voltage.
- Confirming that the sensors are correctly calibrated.

### The robot loses the line on curves

Try:

- Increasing `KD`.
- Reducing the maximum motor speed.
- Improving sensor calibration.
- Adjusting the height and angle of the sensor array.
- Checking the mechanical alignment of the wheels.

### The robot does not move

Check:

- Battery connection.
- Motor driver power supply.
- PWM pins `D5` and `D6`.
- Motor direction pins `D4` and `D7`.
- Common ground between the Arduino and motor driver.

---

## Future Improvements

Possible improvements for future versions include:

- Automatic PID parameter tuning.
- EEPROM storage for calibration values.
- Better handling of line loss.
- Adaptive speed control based on curve sharpness.
- Sensor filtering using moving averages or low-pass filters.
- Battery voltage monitoring.
- Wireless telemetry.
- OLED or LCD status display.
- Improved mechanical chassis design.
- Support for multiple track colors and surfaces.
- Configurable parameters through serial communication.

---

## Academic Context

This project was developed as part of the Computer Science and Mechatronics coursework at the **Universitat d'Andorra (UdA)**.

It combines concepts from:

- Embedded systems.
- C++ programming.
- Feedback control systems.
- PID algorithms.
- Electronics.
- Robotics.
- Real-time signal processing.
- Motor control.
- Hardware-software integration.

---

## License

This project is intended for academic and educational purposes.

Unless otherwise specified, the source code and documentation are provided for learning, experimentation, and non-commercial use. Please contact the repository owner before redistributing or using the project in commercial applications.

---

## Author

Developed by **Andormix** for the **Universitat d'Andorra**.

Repository:

```text
https://github.com/Andormix/UdA-mecatronica-seguidor-linea
```
