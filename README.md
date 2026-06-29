# UART Transmitter & Receiver — MATLAB Simulink Model

![MATLAB](https://img.shields.io/badge/MATLAB-Simulink-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-green?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-VLSI%20%7C%20SoC%20Design-blue?style=flat-square)
![Author](https://img.shields.io/badge/Author-Vignesh%20R-purple?style=flat-square)

---

## 📌 Project Overview

This project models a complete **UART (Universal Asynchronous Receiver Transmitter)** communication system using **MATLAB Simulink**. It includes both the **Transmitter** and **Receiver** subsystems built from scratch using logic-level blocks — directly mapping to real RTL design concepts used in SoC and VLSI chip design.

> Built as part of **The VLSI Journey** — a hands-on learning series on chip design and semiconductor systems.

---

## 🎯 What This Model Does

| Subsystem | Function |
|---|---|
| **Transmitter** | Converts an 8-bit parallel byte (0xB2 = 178) into a serial UART bitstream |
| **Receiver** | Detects start bit, samples 8 data bits, reconstructs the original byte |
| **Verification** | Display block confirms received byte matches transmitted byte |

### UART Frame Structure

```
IDLE | START | D0 | D1 | D2 | D3 | D4 | D5 | D6 | D7 | STOP | IDLE
  1      0    b0   b1   b2   b3   b4   b5   b6   b7    1
```

- **Baud Rate** : 9600 bps  
- **Data Bits** : 8  
- **Stop Bits** : 1  
- **Parity**    : None  
- **Frame Size**: 10 bits total  

---

## 🏗️ Model Architecture

```
┌─────────────────────────────────────────────────┐
│               UART TRANSMITTER                  │
│                                                 │
│  Constant(178) → Int to Bit → build_frame       │
│                                    ↓            │
│                 Counter Limited → serialize      │
│                                    ↓            │
│                                 TX LINE         │
└─────────────────────────────────────────────────┘
                        │
                        │ Serial Bitstream
                        ↓
┌─────────────────────────────────────────────────┐
│               UART RECEIVER                     │
│                                                 │
│  TX LINE → Unit Delay  → detect_start           │
│  TX LINE →             → detect_start           │
│                               ↓                 │
│  TX LINE →           → sample_bits              │
│                               ↓                 │
│                    Bit to Integer Converter      │
│                               ↓                 │
│                         Display (178) ✅         │
└─────────────────────────────────────────────────┘
```

---

## 🧱 Simulink Blocks Used

### Transmitter Blocks

| Block | Library | Purpose |
|---|---|---|
| Constant | Simulink → Sources | Input data byte (uint8 = 178) |
| Integer to Bit Converter | DSP System Toolbox | Parallel → serial bits |
| MATLAB Function (build_frame) | User-Defined Functions | Assembles START + DATA + STOP |
| Counter Limited | Simulink → Sources | Baud rate clock index (0 to 9) |
| MATLAB Function (serialize) | User-Defined Functions | Outputs one bit per clock step |
| Scope | Simulink → Sinks | View TX waveform |

### Receiver Blocks

| Block | Library | Purpose |
|---|---|---|
| Unit Delay (1/z) | Simulink → Discrete | Stores previous RX bit for edge detection |
| MATLAB Function (detect_start) | User-Defined Functions | Detects falling edge = START bit |
| MATLAB Function (sample_bits) | User-Defined Functions | Samples 8 data bits using persistent state |
| Bit to Integer Converter | DSP System Toolbox | Reconstructs byte from bits |
| Display | Simulink → Sinks | Shows received byte value |

---

## ⚙️ Simulation Settings

```
Solver type     : Fixed-step
Solver          : Discrete (no continuous states)
Fixed step size : 1/9600  (one baud period = 104.17 µs)
Stop time       : 12/9600 (full frame + margin)
Baud rate       : 9600 bps
```

---

## 📊 Simulation Results

| Parameter | Value |
|---|---|
| Transmitted byte | 178 (0xB2 = 10110010) |
| TX Waveform | 0 → 0,1,0,0,1,0,1,1 → 1 |
| Received byte | 178 ✅ |
| Done signal | 1 (HIGH when byte received) |

### TX Waveform (from Scope):
```
Time step :  0    1    2    3    4    5    6    7    8    9
Signal    :  0    0    1    0    0    1    0    1    1    1
             ↑                                           ↑
           START                                       STOP
```

---

## 🌍 Real World Applications

### 1. 🖥️ Microcontroller Communication
UART is the most widely used protocol for microcontroller-to-peripheral communication. Used in **Arduino, STM32, ESP32, PIC** microcontrollers for serial communication with sensors, displays, and GPS modules.

### 2. 📡 GPS Modules
GPS receivers like **u-blox NEO-6M** send location data (NMEA sentences) to microcontrollers over UART at 9600 baud — exactly the baud rate used in this model.

### 3. 🖨️ Industrial Equipment
Factory machines, PLCs (Programmable Logic Controllers), and industrial sensors use **RS-232 and RS-485** — both built on top of UART protocol — for reliable long-distance serial communication.

### 4. 📱 Bluetooth Modules
**HC-05, HC-06 Bluetooth modules** communicate with host microcontrollers via UART. When you pair a phone with a Bluetooth device, UART is handling the data transfer on the embedded side.

### 5. 🛰️ SoC IP Blocks
In modern **System-on-Chip (SoC)** designs (Qualcomm Snapdragon, Apple M-series, ARM Cortex), UART is a standard peripheral IP block used for debug interfaces (JTAG/SWD via UART), boot loaders, and firmware flashing. This Simulink model directly represents the functional behavior of that IP block before RTL coding.

### 6. 🔬 VLSI Chip Testing
During **chip bring-up and testing**, UART is the first communication interface used to verify that a newly fabricated chip is functional. Engineers send known bytes over UART and verify the response — exactly what this model simulates.

### 7. 🚗 Automotive Systems
**ECU (Engine Control Units)** in vehicles use UART-based **LIN (Local Interconnect Network)** protocol for low-speed communication between body control modules, sensors, and actuators.

### 8. 🏥 Medical Devices
Patient monitoring devices (pulse oximeters, ECG machines) use UART to transmit real-time patient data from sensor modules to display processors reliably.

---

## 🔗 VLSI / RTL Design Relevance

| UART Concept | RTL / VLSI Equivalent |
|---|---|
| Baud rate clock | Clock divider from PLL output |
| Start bit detection | Falling edge FSM trigger |
| Shift register (serialize) | SIPO register in RTL |
| Frame assembly (build_frame) | TX FIFO logic in IP block |
| Persistent variables | D flip-flop register elements |
| Done signal | Interrupt flag in register map |
| Receiver sampling | 16x oversampling logic in real UART |

---

## 🚀 How to Run

### Requirements
- MATLAB R2021a or later
- Simulink
- DSP System Toolbox

### Steps
```
1. Download this UART.slx file
2. Open MATLAB
3. Drag and  Drop U_ART_TRANSMITTER.slx in Files Window
5. Double-click that file to open in MatLab Simulink
6. Run the Project by clicking the RUN icon.
7. Check Display block for received byte (178)
```

---

## 📁 File Structure

```
UART-Simulink/
│
├── U_ART_TRANSMITTER.slx    ← Main Simulink model
├── README.md                ← This file
└── images/
    ├── model_overview.png   ← Screenshot of full model
    └── scope_output.png     ← TX waveform from Scope
```

---

## 🔮 Future Extensions

- [ ] Add **even/odd parity bit** for error detection
- [ ] Implement **noise injection** using Random Number block
- [ ] Replace MATLAB Functions with **Stateflow FSM** (RTL-like)
- [ ] Add **16x oversampling** in receiver (real UART behavior)
- [ ] Support **multiple baud rates** (9600 / 19200 / 115200)
- [ ] Implement **full duplex** (simultaneous TX and RX)

---

## 👨‍💻 Author

**Vignesh R**  
ECE Student | VLSI Design Engineer (Aspiring)  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-vignesh--r--vlsi-blue?style=flat-square&logo=linkedin)](https://linkedin.com/in/vignesh-r-vlsi)
[![GitHub](https://img.shields.io/badge/GitHub-vignesh--r--ece-black?style=flat-square&logo=github)](https://github.com/vignesh-r-vlsi)

---

## 📜 License

This project is Paid software by MathWorks in Simulink.

---
