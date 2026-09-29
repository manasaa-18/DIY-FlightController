# ESP32 Drone Flight Controller

![Platform](https://img.shields.io/badge/platform-ESP32-blue)
![Framework](https://img.shields.io/badge/framework-Arduino-00979D)
![Language](https://img.shields.io/badge/language-C%2B%2B-orange)
![Loop Rate](https://img.shields.io/badge/loop-~250Hz-green)

**A from-scratch quadcopter flight controller for the ESP32, written in readable Arduino C++.** It combines MPU6050 sensor fusion, cascaded angle/rate PID stabilization, RC receiver input, and ESC motor mixing in a single sketch. Use it to learn how flight control really works, or as a hackable base for your own DIY drone, with no Betaflight or ArduPilot required.

> ⚠️ **Safety first:** This project controls high-speed brushless motors powered by LiPo batteries. Always **remove propellers** during setup, wiring, and testing. See the [Safety](#safety) section before powering anything.

---

## Table of Contents

- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Quick Start](#quick-start)
- [Wiring](#wiring)
- [Configuration](#configuration)
- [How It Works](#how-it-works)
- [Safety](#safety)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Authors](#authors)

---

## Features

- **ESP32-based:** runs the entire control stack on one affordable microcontroller
- **MPU6050 IMU:** gyroscope and accelerometer data with calibration offsets
- **Complementary filter:** fuses sensor data into stable roll and pitch angles
- **Dual-loop (cascade) PID:**
  - **Angle PID:** turns pilot stick input into target rotation rates
  - **Rate PID:** holds those rates using gyro feedback
- **6-channel RC input:** standard PWM receiver support
- **Real-time motor mixing:** X-configuration quadcopter
- **PWM ESC control:** drives standard brushless ESCs
- **Built-in safety logic:** arming, throttle cutoff, output clamping, and angle limiting
- **Fast loop:** ~250 Hz control loop for responsive flight

---

## Hardware Requirements

| Component | Purpose |
|-----------|---------|
| **ESP32 Dev Module** | Main microcontroller |
| **MPU6050** | IMU (gyroscope + accelerometer) |
| **4× ESCs** | Electronic speed controllers |
| **4× Brushless motors** | Propulsion |
| **6-channel RC receiver (PWM)** | Pilot input |
| **LiPo battery** | Power source |
| **X-frame quadcopter** | Airframe |

### Software Requirements

- [Arduino IDE](https://www.arduino.cc/en/software) **or** [PlatformIO](https://platformio.org/)
- ESP32 board support package
- Libraries: `ESP32Servo`, `Wire` (built in)

---

## Quick Start

> 🔧 **Do not install propellers during any of these steps.**

### 1. Clone the repository

```bash
git clone https://github.com/builtbyMani/DIY-FlightController.git
cd DIY-FlightController
```

### 2. Install libraries

In the Arduino IDE, go to **Sketch → Include Library → Manage Libraries**, search for **ESP32Servo**, and click **Install**. `Wire` is included with the ESP32 core.

### 3. Select your board

In the Arduino IDE, go to **Tools → Board** and choose **ESP32 Dev Module**.

### 4. Wire the hardware

Follow the [Wiring](#wiring) tables below. Double-check all connections before continuing.

### 5. Upload the sketch

Open `flight_controller.ino`, connect your ESP32 via USB, select the correct port under **Tools → Port**, and click **Upload**.

### 6. Run the pre-flight checklist

Before the first flight, with **props off**:

1. Power the ESP32 and receiver.
2. Confirm all six RC channels respond correctly to your transmitter.
3. Keep the drone level and still during IMU calibration on startup.
4. Arm the controller and verify each motor spins in the direction shown in [Motor Mixing](#motor-mixing).
5. Tilt the frame by hand and confirm the motors respond to correct the tilt.
6. Confirm motors stop when the throttle is low or the controller is disarmed.

<!-- TODO: Add exact arming/disarming procedure (e.g., which switch or stick position) and ESC calibration steps. -->

---

## Wiring

### MPU6050 → ESP32

| MPU6050 | ESP32 |
|---------|-------|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

### ESC Signal Pins

| Motor | GPIO |
|-------|------|
| Motor 1 | GPIO 13 |
| Motor 2 | GPIO 12 |
| Motor 3 | GPIO 14 |
| Motor 4 | GPIO 27 |

<!-- TODO: Add RC receiver channel-to-GPIO mapping and a wiring diagram image. -->

---

## Configuration

### PID Tuning

Default gains live at the top of `flight_controller.ino`:

```cpp
// Angle loop (roll)
float PAngleRoll = 2;
float IAngleRoll = 0.5;
float DAngleRoll = 0.007;

// Rate loop (roll)
float PRateRoll = 0.625;
float IRateRoll = 2.1;
float DRateRoll = 0.0088;
```

These defaults may need adjusting for your build. Tune based on:

- **Frame size**
- **Motor power**
- **Propeller configuration**
- **Battery voltage**

### Receiver Channels

| Channel | Function |
|---------|----------|
| CH1 | Roll |
| CH2 | Pitch |
| CH3 | Throttle |
| CH4 | Yaw |
| CH5 | Auxiliary |
| CH6 | Auxiliary |

### Motor Mixing

Outputs are mixed for an **X-configuration** quadcopter:

| Motor Position | Spin Direction |
|----------------|----------------|
| Front Right | Counter-Clockwise (CCW) |
| Rear Right | Clockwise (CW) |
| Rear Left | Counter-Clockwise (CCW) |
| Front Left | Clockwise (CW) |

<!-- TODO: Clarify which Motor 1–4 (GPIO) maps to which position. -->

---

## How It Works

Every loop iteration (~250 Hz), the controller runs this pipeline:

1. **Read** RC receiver inputs
2. **Read** MPU6050 gyroscope and accelerometer data (with calibration offsets applied)
3. **Estimate** roll and pitch using a complementary filter
4. **Run the angle PID** to convert desired angles into desired rotation rates
5. **Run the rate PID** to convert rate errors into corrections using gyro feedback
6. **Mix** corrections with throttle into four motor outputs
7. **Send** PWM signals to the ESCs

### Why a cascade PID?

The two-loop design gives you:

- **Smoother control**
- **Faster response**
- **Improved stability**

---

## Safety

This project controls high-speed motors and LiPo-powered hardware. Improper wiring or tuning can damage equipment or cause injury.

**Built-in protections:**

- Motors cut off when throttle is low
- PID integrals reset on disarm
- Motor outputs are clamped
- Angle is limited to **±20°**

**Your responsibilities:**

- **Remove propellers** during setup, wiring, and initial tests
- Test in a safe, open area away from people
- Never handle the drone while the battery is connected and armed

---

## Project Structure

```
.
├── flight_controller.ino   # Main flight controller sketch
└── README.md
```

---

## Roadmap

- [ ] Kalman filter
- [ ] Altitude hold
- [ ] GPS stabilization
- [ ] Telemetry support
- [ ] OLED status display
- [ ] Battery voltage monitoring
- [ ] WiFi/Bluetooth tuning interface
- [ ] Autonomous flight modes

---

## Contributing

Contributions are welcome! To get started:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes with a clear message
4. Open a pull request describing what you changed and how you tested it

Please **bench-test with propellers removed** before submitting any change that affects motor output or control logic.

---

## License

<!-- TODO: Add a license (e.g., MIT) and a LICENSE file. -->

---

## Authors

Built by:

- **builtbyMani** .
- **manasaa-18** ·
