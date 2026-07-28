# RepTracker

A wearable velocity-based training (VBT) device that tracks rep speed and count during weightlifting, and recommends optimal weight and rep targets for your next set — all delivered wirelessly to your phone via Bluetooth.

---

## How It Works

The device straps to your wrist and uses an IMU (MPU-6050) to detect the speed and direction of each rep. After a set, it analyzes velocity loss across your reps and compares it against research-based thresholds for hypertrophy to recommend whether to increase, maintain, or decrease weight for your next set.

```
Wrist movement
  → MPU-6050 IMU sensor detects acceleration
  → ESP32 counts reps and measures duration
  → Bluetooth Low Energy (BLE) transmits data
  → Web app receives and displays live rep count
  → Algorithm outputs weight recommendation
```

---

## Features

- **Live rep counting** — detects full up/down strokes, counts reps in real time
- **Velocity-based analysis** — tracks rep duration across the set and computes velocity loss
- **Smart recommendations** — research-informed algorithm adjusts weight based on rep count and fatigue
- **Wireless** — BLE connection to iPhone via Bluefy browser
- **On-device display** — 0.96" OLED shows rep count and recommendations without needing a phone
- **Web app** — hosted on GitHub Pages with workout tracking, history chart, and monthly calendar
- **Multi-exercise support** — select from 8 core exercises including Bicep Curl, Chest Press, Pull-Up, Shoulder Press, and more
- **Persistent history** — workout log with weight progress chart and workout day calendar

---

## Hardware

| Component | Purpose |
|---|---|
| ESP32 DevKit V1 | Microcontroller with built-in BLE |
| MPU-6050 GY-521 | IMU sensor — detects rep movement |
| SSD1306 0.96" OLED | On-device display |
| LiPo 3.7V 450mAh 502535 | Battery |
| LX-LBES TP4057 | USB-C LiPo charging module |

**Wiring (I2C bus — shared by MPU-6050 and OLED):**

| Sensor Pin | ESP32 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

---

## Firmware

Written in Arduino C++ for the ESP32. Key responsibilities:

- Reads accelerometer data from MPU-6050 via I2C
- Detects reps using threshold-based up/down stroke detection
- Measures rep duration for velocity loss calculation
- Analyzes set performance and generates weight recommendation
- Transmits data to web app via BLE notify/write characteristics
- Drives OLED display with live rep count and post-set summary
- Resets cleanly between sets without dropping BLE connection

**BLE UUIDs:**
```
Service:        4fafc201-1fb5-459e-8fcc-c5c9c331914b
Characteristic: beb5483e-36e1-4688-b7f5-ea07361b26a8
```

---

## Algorithm

The recommendation logic is based on published VBT research for hypertrophy. Velocity loss is computed by comparing the average duration of the first half of reps (baseline) to the last rep.

| Rep Count | Velocity Loss | Recommendation |
|---|---|---|
| Below 6 | Any | Decrease weight 5 lbs |
| 6–7 | > 100% | Decrease weight 5 lbs |
| 6–7 | ≤ 100% | Keep weight |
| 8 | Any | Keep weight |
| 9–10 | < 30% | Increase weight 5 lbs |
| 9–10 | ≥ 30% | Keep weight |
| Above 10 | Any | Increase weight 5 lbs |

Velocity is only used at the edges of the rep range (6–7 or 9–10). In the middle of the range (8 reps), rep count alone drives the decision.

The upper velocity loss threshold of 100% (last rep takes twice as long as baseline) reflects research showing that 40–50% velocity loss already represents near-failure — a productive zone for hypertrophy. Only genuinely damaging fatigue triggers a weight decrease.

---

## Web App

Live at: **[hotheodore.github.io/reptracker](https://hotheodore.github.io/reptracker)**

Built as a single HTML file using vanilla JavaScript and Web Bluetooth API. Open in **Bluefy** on iPhone (Safari does not support Web Bluetooth).

**Tabs:**

- **Workout** — connect to device, select exercise, enter weight, view live rep counter and post-set recommendation
- **History** — filterable log of all sets with weight progress chart per exercise
- **Calendar** — monthly activity view showing workout days, total sets, and best streak

---

## Supported Exercises

- Bicep Curl
- Chest Press
- Pull-Up
- Shoulder Press
- Tricep Extension
- Lat Pulldown
- Bent Over Row
- Squat

---

## Setup

### Firmware

1. Install [Arduino IDE 2](https://www.arduino.cc/en/software)
2. Add ESP32 board support via Boards Manager (Espressif Systems, v2.0.17)
3. Install libraries: `MPU6050` (Electronic Cats), `Adafruit SSD1306`, `Adafruit GFX`
4. Select board: **ESP32 Dev Module**, upload speed **115200**, flash mode **DIO**
5. Upload firmware to ESP32

### Web App

Open [hotheodore.github.io/reptracker](https://hotheodore.github.io/reptracker) in **Bluefy** on iPhone. No installation required.

To run locally: clone the repo and open `index.html` in a Bluetooth-capable browser.

---

## Repository Structure

```
reptracker/
├── index.html        # Web app (single file — UI, BLE logic, history, calendar)
└── README.md         # This file
```

Firmware lives in the Arduino IDE project folder (not yet committed to this repo).

---

## Roadmap

- [ ] Commit ESP32 firmware to repository
- [ ] 3D printed wearable enclosure (OpenSCAD model in progress)
- [ ] Battery integration — remove USB dependency
- [ ] Exercise auto-detection using accelerometer ML classifier
- [ ] Multi-user support
- [ ] Export workout history to CSV

---

## Built With

- ESP32 Arduino framework
- MPU-6050 accelerometer/gyroscope
- Adafruit SSD1306 + GFX libraries
- Web Bluetooth API
- GitHub Pages (hosting)

---

## References

- Weakley et al. (2021) — Velocity loss thresholds and hypertrophy outcomes
- Pareja-Blanco et al. (2020) — Effects of velocity loss on strength adaptations
- González-Badillo & Sánchez-Medina (2010) — Movement velocity as measure of loading intensity

---

*Built as an embedded systems and signal processing project — Engineering Science, Trinity University*
