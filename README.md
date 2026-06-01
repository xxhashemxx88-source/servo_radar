# 🛰️ Ultrasonic Radar System using ESP32 and OLED

A portable, Wi-Fi enabled miniature radar system that scans surroundings, detects nearby objects, renders a real-time radar sweep on a 0.96" OLED display, **and** streams a live radar dashboard to any browser on the same network — all processed at the edge by an ESP32 microcontroller.

---

## 📖 Introduction

The objective of this project is to design and implement a miniature radar system capable of scanning its surroundings, detecting objects, and providing visual feedback in real-time — both locally on an OLED screen and remotely via a web dashboard.

This system utilizes an **ESP32 IdeaSpark Edition** microcontroller to coordinate a servo motor and an ultrasonic sensor, capturing the distance and angle of nearby objects. The ESP32 also acts as a **Wi-Fi Access Point**, hosting a web server that streams live radar data to any connected device through a browser.

---

## 🔧 Hardware Components

| Component | Model | Role |
|---|---|---|
| Microcontroller | ESP32 (IdeaSpark Edition) | Central processing unit — controls servo, sensor, display, and Wi-Fi AP |
| Ultrasonic Sensor | HC-SR04 | Measures distance to objects via acoustic pulses |
| Servo Motor | Micro Servo SG90 | Rotates the sensor across a 180° arc |
| OLED Display | 0.96" I2C SSD1306 | Renders the local radar visualization |

---

## ⚡ Wiring & Connections

### Ultrasonic Sensor (HC-SR04)

| Sensor Pin | ESP32 Pin |
|---|---|
| VCC | 5V / VIN |
| GND | GND |
| TRIG | GPIO 25 |
| ECHO | GPIO 26 |

---

### Servo Motor (SG90)

| Servo Pin | ESP32 Pin |
|---|---|
| VCC | 5V / VIN |
| GND | GND |
| Signal | GPIO 15 |

---

### OLED Display (I2C — SSD1306)

| OLED Pin | ESP32 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 (Default I2C) |
| SDA | GPIO 21 (Default I2C) |

> **I2C Address:** `0x3C`

---

## 📐 Pin Summary

```
ESP32
├── GPIO 25  → HC-SR04 TRIG
├── GPIO 26  → HC-SR04 ECHO
├── GPIO 15  → SG90 Servo Signal
├── GPIO 21  → OLED SDA (I2C)
├── GPIO 22  → OLED SCL (I2C)
├── 3.3V     → OLED VCC
└── 5V/VIN   → HC-SR04 VCC, Servo VCC
```

---

## ⚙️ Working Principle

The system operates in a continuous loop with five stages:

### 1. 🔄 Scanning (Actuation)
The ESP32 commands the servo motor to sweep from **0° to 180°** and back in **2° increments**, continuously rotating the ultrasonic sensor across the environment.

### 2. 📡 Detection (Sensing)
At each angle, the ESP32 triggers the HC-SR04. The sensor emits an acoustic pulse, waits for the echo, and calculates distance:

```
Distance = (Speed of Sound × Time of Flight) / 2
```

> The speed of sound in air is approximately **343 m/s (0.034 cm/µs)**. The round-trip time is halved to get one-way distance. Readings beyond **400 cm** or with no echo are clamped to `400`.

### 3. 🧮 Data Processing
- If the object is within the **maximum range (40 cm)**, angle and distance are recorded.
- Out-of-range readings are ignored for display purposes but still transmitted over Wi-Fi.

### 4. 🖥️ OLED Visual Output
The ESP32 draws a radar grid (4 concentric arcs) on the OLED, then plots the sweep beam and a filled circle at the detected object's position using trigonometry:

```
X = cx + (distance / MAX_DISTANCE) × R × cos(θ)
Y = cy − (distance / MAX_DISTANCE) × R × sin(θ)
```

The display also shows:
- `Dist: XX cm` or `Dist: --` (top left)
- `Ang: XX` (top right)
- `** DETECTED **` or `CLEAR` (bottom center)

### 5. 🌐 Wi-Fi Web Dashboard
The ESP32 broadcasts its own **Wi-Fi Access Point** and hosts a web server on port 80. Any device connected to the AP can open a browser and view a live radar dashboard rendered on an HTML5 Canvas.

---

## 🌐 Wi-Fi Access Point & Web Dashboard

The ESP32 creates its own wireless network at boot:

| Setting | Value |
|---|---|
| SSID | `ESP32_Radar` |
| Password | `123456789` |
| Dashboard URL | `http://192.168.4.1` |
| Data Endpoint | `http://192.168.4.1/data` |

### How to Connect
1. On boot, the OLED displays the network name, password, and IP address for **5 seconds**.
2. Connect your phone or laptop to the **`ESP32_Radar`** Wi-Fi network.
3. Open a browser and navigate to **`http://192.168.4.1`**.
4. The live radar dashboard will load and update every **50 ms**.

### Dashboard Features
- 🟢 Green sweep beam rotating in real time
- 🔴 Red dots marking detected objects at their polar position
- Live **angle** and **distance** readout below the canvas
- Retro green-on-black radar aesthetic

---

## 🗺️ System Architecture

```
┌─────────────┐     angle/distance      ┌──────────────────────────────────────┐
│  Servo SG90 │ ◀────── GPIO 15 ──────  │                                      │
└─────────────┘                         │          ESP32 Microcontroller        │
                                        │                                        │
┌─────────────┐     TRIG → GPIO 25      │  loop():                             │
│   HC-SR04   │ ──────────────────────▶ │  1. Move servo by 2°                 │
│  Ultrasonic │ ◀────── GPIO 26 ──────  │  2. Trigger HC-SR04                  │
└─────────────┘     ECHO ← GPIO 26      │  3. Calculate distance               │
                                        │  4. Draw radar on OLED               │
┌─────────────┐     I2C (21/22)         │  5. Serve data to web clients        │
│  SSD1306    │ ◀───────────────────    │                                      │
│    OLED     │                         └──────────────────────────────────────┘
└─────────────┘                                        │ Wi-Fi AP
                                                       ▼
                                           Browser @ 192.168.4.1
                                           (Live Canvas Radar)
```

<img width="1171" height="638" alt="image" src="https://github.com/user-attachments/assets/564ab170-e09a-4056-92bc-fa0b5dc41394" />

<img width="1171" height="640" alt="image" src="https://github.com/user-attachments/assets/0c6d3887-ea4a-4c37-9848-c7ce30238ea0" />






---

## 📦 Dependencies / Libraries

Install the following via the **Arduino Library Manager**:

| Library | Purpose |
|---|---|
| `ESP32Servo` | Servo motor control on ESP32 PWM pins |
| `Adafruit SSD1306` | OLED display driver |
| `Adafruit GFX Library` | Graphics primitives (lines, circles, text) |
| `WiFi` *(built-in)* | ESP32 Wi-Fi stack |
| `WebServer` *(built-in)* | HTTP server for the web dashboard |

---

## 🚀 Getting Started

1. Wire all components according to the tables above.
2. Install the required libraries via the Arduino Library Manager.
3. Open `radar.ino` in the Arduino IDE.
4. Select your **ESP32** board and the correct COM port.
5. Upload the sketch.
6. On boot, the OLED shows Wi-Fi credentials and the IP address.
7. Connect your device to **`ESP32_Radar`** (password: `123456789`).
8. Open **`http://192.168.4.1`** in your browser.
9. The radar starts scanning and streaming immediately.

---

## 📄 License

This project is open-source and free to use for educational and personal purposes.
