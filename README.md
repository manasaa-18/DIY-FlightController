ESP32-Based Drone Flight Controller

A custom ESP32-powered quadcopter flight controller built using Arduino framework, featuring real-time PID stabilization, IMU sensor fusion, RC receiver input handling, and ESC motor control. This project implements a complete low-level drone stabilization system from scratch without relying on external flight controller firmware like Betaflight or ArduPilot.

Features
ESP32-based flight controller
MPU6050 IMU integration (Gyroscope + Accelerometer)
Complementary filter for angle estimation
Dual-loop PID stabilization
Angle PID
Rate PID
PWM ESC motor control
6-channel RC receiver input support
Real-time motor mixing
Safety throttle cutoff & motor arming logic
High-speed loop timing (~250Hz)
Designed for DIY quadcopters
Hardware Used
Component	Description
ESP32	Main microcontroller
MPU6050	IMU sensor
ESCs	Electronic Speed Controllers
Brushless Motors	Quadcopter propulsion
RC Receiver	6-channel PWM receiver
LiPo Battery	Power source
Software Stack
Arduino Framework
C++
Wire Library (I2C Communication)
ESP32Servo Library
ESP32 PWM Timers
Flight Controller Architecture
Sensor Processing
Reads gyroscope and accelerometer data from MPU6050
Applies calibration offsets
Computes roll and pitch angles
Uses complementary filter for sensor fusion
Stabilization System

The controller uses a cascade PID structure:

1. Angle PID

Converts pilot angle commands into desired rotational rates.

2. Rate PID

Controls angular velocity using gyro feedback.

This provides:

smoother control
faster response
improved stability
Motor Mixing

Motor outputs are mixed for an X-configuration quadcopter:

Motor	Direction
Front Right	Counter-Clockwise
Rear Right	Clockwise
Rear Left	Counter-Clockwise
Front Left	Clockwise
Receiver Channels
Channel	Function
CH1	Roll
CH2	Pitch
CH3	Throttle
CH4	Yaw
CH5	Auxiliary
CH6	Auxiliary
PID Tuning Parameters
float PAngleRoll = 2;
float IAngleRoll = 0.5;
float DAngleRoll = 0.007;

float PRateRoll = 0.625;
float IRateRoll = 2.1;
float DRateRoll = 0.0088;

These values can be tuned depending on:

frame size
motor power
propeller configuration
battery voltage
Safety Features
Motor cutoff when throttle is low
PID integral reset on disarm
Motor output clamping
Angle limiting (±20°)
Project Structure
.
├── flight_controller.ino
├── README.md
Future Improvements
Kalman filter implementation
Altitude hold
GPS stabilization
Telemetry support
OLED status display
Battery voltage monitoring
WiFi/Bluetooth tuning interface
Autonomous flight modes
How It Works
Read RC receiver inputs
Read MPU6050 sensor data
Estimate drone orientation
Run angle PID loop
Run rate PID loop
Mix motor outputs
Send PWM signals to ESCs

The loop runs continuously with precise timing for stable flight control.

Wiring Overview
MPU6050 → ESP32
MPU6050	ESP32
VCC	3.3V
GND	GND
SDA	GPIO 21
SCL	GPIO 22
ESC Signal Pins
Motor	GPIO
Motor 1	GPIO 13
Motor 2	GPIO 12
Motor 3	GPIO 14
Motor 4	GPIO 27
Compilation

Install required libraries:

ESP32Servo
Wire

Then upload using:

Arduino IDE
PlatformIO

Board:

ESP32 Dev Module
Disclaimer

This project directly controls high-speed brushless motors and LiPo-powered systems. Improper tuning or wiring can damage hardware or cause injury. Test carefully with propellers removed during initial setup.

Author

Built by Mani
