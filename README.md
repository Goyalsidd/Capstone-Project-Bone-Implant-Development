# 🦴 Development of a Bone Implant  
### _A Mechanically Optimized and Affordable Solution for Orthopaedic Care_  

[![Project Type](https://img.shields.io/badge/Type-Capstone%20Project-blue)]()  
[![Made with](https://img.shields.io/badge/Made%20with-Arduino-orange)]()  
[![Institute](https://img.shields.io/badge/IIT-Ropar-red)]()  

---

## 👥 Team Members  
- **Rajeev Kumar** — 2022MEB1335  
- **Siddharth Goyal** — 2022MEB1346  

**Supervisor:**  
Dr. **Jitendra Prasad**, Associate Professor  
Department of Mechanical Engineering, IIT Ropar  

---

## 🧠 Project Overview  
This project focuses on designing a **cost-effective limb-lengthening implant** inspired by advanced systems like **Fitbone** and **Precice Nail**.  
It integrates a **DC motor**, **three-stage planetary gear system**, and **lead screw** to achieve **precise linear motion** for bone distraction during **distraction osteogenesis**.

> 🧩 Goal: Develop an affordable, motor-driven internal bone implant with precise, smooth, and non-invasive control.

---

## ⚙️ System Architecture  

| Component | Function |
|------------|-----------|
| **DC Motor (6V, 260 RPM)** | Provides high-speed rotary input |
| **Planetary Gear System (3-stage, 166.375:1)** | Amplifies torque and reduces speed |
| **Lead Screw (Pitch = 1.25 mm/rev)** | Converts rotation into precise linear motion |
| **Hall-Effect Encoder** | Provides feedback for position and direction |
| **Arduino Uno** | Runs PID control algorithm |
| **L298N Motor Driver** | Drives DC motor using PWM control |

---

## 🔩 Working Mechanism  

1. **DC Motor** provides rotary motion.  
2. **Planetary gear system** reduces RPM and increases torque.  
3. **Lead screw** converts rotation into linear displacement.  
4. **Hall-effect encoder** provides feedback to Arduino for closed-loop control.  
5. **PID algorithm** continuously adjusts motor speed and direction to reach the target position.  

**Result:**  
Smooth, controlled bone distraction at a rate of **~0.3 mm every 2 minutes**, aligning with biological growth requirements.

---

## 🧭 PID Control System  

The control loop ensures precision and stability in motor actuation.

\[
V_{PWM} = K_p e(t) + K_i \int e(t) dt + K_d \frac{de(t)}{dt}
\]

Where:  
- \( e(t) = x_{desired} - x_{measured} \)  
- \( K_p, K_i, K_d \) are proportional, integral, and derivative gains  

### **Hardware Used**
- Arduino Uno (Controller)  
- L298N Motor Driver (H-Bridge)  
- DC Motor with Hall-effect Encoder  

### **Software Features**
- Encoder provides 40,500 pulses per revolution  
- PID tuning for stability (D term = 0 to reduce steady-state noise)  
- Real-time feedback ensures accurate positioning and minimal overshoot  

---

## 🧮 Calculations  

| Parameter | Symbol | Value |
|------------|----------|--------|
| **Gear Ratio** | \( G \) | \( (5.5)^3 = 166.375:1 \) |
| **Lead Screw Pitch** | \( P \) | 1.25 mm/rev |
| **Desired Linear Speed** | \( v \) | 0.3 mm / 2 min |
| **Required Motor RPM** | \( \omega_m \) | 50–80 RPM (achieved via PID control) |

**Linear Speed Equation:**  
\[
v = \text{RPM} \times P
\]

---

## 💻 Arduino Implementation  

```cpp
#include <PID_v1.h>

double setpoint, input, output;
double Kp = 1.5, Ki = 0.5, Kd = 0.0;  // D term minimized for steady state
PID myPID(&input, &output, &setpoint, Kp, Ki, Kd, DIRECT);

const int motorPWM = 9;
const int in1 = 7;
const int in2 = 8;
volatile long encoderCount = 0;

void setup() {
  pinMode(motorPWM, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  attachInterrupt(digitalPinToInterrupt(2), readEncoder, RISING);
  Serial.begin(9600);
  myPID.SetMode(AUTOMATIC);
}

void loop() {
  input = encoderCount;              // current position
  myPID.Compute();                   // compute PID output
  analogWrite(motorPWM, output);     // control motor speed
}

void readEncoder() {
  encoderCount++;  // Increment count for each pulse
}
