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
This project focuses on developing a **cost-effective, motor-driven limb-lengthening implant**, inspired by existing commercial systems like **Fitbone** and **Precice Nail**.  
It integrates a **DC motor**, a **three-stage planetary gear system**, and a **lead screw** to achieve **precise linear motion** for controlled bone distraction in **distraction osteogenesis**.

> 🧩 Goal: Create an affordable implant with smooth, accurate, and non-invasive motion control suitable for orthopaedic use in developing nations.

---

## ⚙️ System Architecture  

| Component | Function |
|------------|-----------|
| **DC Motor (6V, 260 RPM)** | Provides rotary motion |
| **Planetary Gear System (3-stage, 166.375:1)** | Increases torque, reduces speed |
| **Lead Screw (Pitch = 1.25 mm/rev)** | Converts rotary to linear motion |
| **Hall-Effect Encoder** | Provides feedback for position & direction |
| **Arduino Uno** | Implements PID control algorithm |
| **L298N Motor Driver** | Drives the DC motor (PWM + Direction Control) |

---

## 🔩 Working Mechanism  

1. The **DC motor** rotates at 260 RPM.  
2. The **planetary gear system** reduces speed and amplifies torque (166.375:1 ratio).  
3. The **lead screw** converts the output rotation to linear displacement of 1.25 mm/rev.  
4. The **encoder** sends position feedback to Arduino.  
5. The **PID controller** adjusts PWM output to achieve smooth, controlled motion.  

**Result:**  
> Controlled bone distraction at ~0.3 mm per 2 minutes — aligning with clinical standards for safe bone regeneration.

---

## 🧭 Control System — PID Implementation  

### ⚡ Control Law
\[
V_{PWM} = K_p e(t) + K_i \int e(t) dt + K_d \frac{de(t)}{dt}
\]

Where:  
- \( e(t) = x_{desired} - x_{measured} \)  
- \( K_p, K_i, K_d \) = proportional, integral, derivative constants  

### 🧩 System Components
- **Controller:** Arduino Uno  
- **Driver:** L298N H-Bridge Motor Driver  
- **Feedback:** Hall-effect Encoder (40,500 pulses/rev)  

**Why PID?**  
- **P** ensures fast correction  
- **I** removes steady-state error  
- **D** minimizes overshoot and noise  

For steady-state operation, **Kd = 0** (PI control) was found optimal.

---

## 🧮 Design Calculations  

| Parameter | Symbol | Formula / Description | Value |
|------------|----------|----------------------|--------|
| **Gear Ratio** | G | \( (5.5)^3 \) | 166.375:1 |
| **Lead Screw Pitch** | P | mm/rev | 1.25 |
| **Linear Velocity** | v | \( v = \text{RPM} \times P \) | — |
| **Target Speed** | — | 0.3 mm per 2 minutes | — |
| **Required Motor RPM** | ω_m | Calculated | 50–80 RPM |

Thus, PID tuning ensures motor RPM between **50–80** for desired linear displacement.

---

## 💻 Arduino Implementation  

```cpp
#include <PID_v1.h>

double setpoint, input, output;
double Kp = 1.5, Ki = 0.5, Kd = 0.0;  // D minimized to reduce steady-state noise
PID myPID(&input, &output, &setpoint, Kp, Ki, Kd, DIRECT);

const int motorPWM = 9;  // PWM pin to L298N ENA
const int in1 = 7;       // Direction control 1
const int in2 = 8;       // Direction control 2
volatile long encoderCount = 0;

void setup() {
  pinMode(motorPWM, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  attachInterrupt(digitalPinToInterrupt(2), readEncoder, RISING); // Encoder signal
  Serial.begin(9600);
  myPID.SetMode(AUTOMATIC);
  setpoint = 40500;  // target encoder counts for one revolution
}

void loop() {
  input = encoderCount;     // Feedback from encoder
  myPID.Compute();          // Compute PID output
  controlMotor(output);     // Drive motor
  Serial.println(encoderCount);
}

void readEncoder() {
  encoderCount++;           // Increment pulse count
}

void controlMotor(double pwmVal) {
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  analogWrite(motorPWM, constrain(pwmVal, 0, 255));
}
