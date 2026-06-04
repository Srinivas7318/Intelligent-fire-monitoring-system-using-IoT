# 🔥 Intelligent Fire Monitoring System with IoT

An ESP32-based real-time fire detection system using smoke, flame, and temperature sensors — with local buzzer alarm and live data upload to ThingSpeak via WiFi.

---

## 📌 Features

- Detects fire using three independent sensor inputs (smoke, flame, temperature)
- Sounds a local buzzer alarm instantly on fire detection
- Uploads sensor data to ThingSpeak IoT cloud every 20 seconds
- Monitors temperature, humidity, smoke level, and flame presence in real time
- Low-cost, breadboard-friendly hardware setup

---

## 🧰 Hardware Requirements

| Component | Quantity |
|---|---|
| ESP32 development board | 1 |
| MQ-2 smoke / gas sensor | 1 |
| Flame sensor module (IR) | 1 |
| DHT11 temperature & humidity sensor | 1 |
| Buzzer (with NPN transistor driver) | 1 |
| Breadboard + jumper wires | As needed |
| USB cable (for power / programming) | 1 |

---

## 🔌 Pin Connections

| Sensor / Component | ESP32 GPIO |
|---|---|
| MQ-2 analog output | GPIO 34 |
| Flame sensor digital output | GPIO 25 |
| DHT11 data pin | GPIO 26 |
| Buzzer (via transistor) | GPIO 27 |

> **Note:** GPIO 34 is input-only on ESP32. MQ-2 AO connects here for analog read (0–4095).

---

## 📐 Block Diagram

```
 ┌─────────────────┐        ┌──────────────────────────┐
 │  Flame Sensor   │───────▶│                          │
 │  (GPIO 25)      │        │         ESP32            │──────▶ Buzzer (GPIO 27)
 └─────────────────┘        │   Fire Detection Logic   │
                             │   + WiFi Stack           │
 ┌─────────────────┐        │                          │
 │  MQ-2 Sensor    │───────▶│                          │
 │  (GPIO 34)      │        └──────────┬───────────────┘
 └─────────────────┘                   │
                                        │ WiFi (HTTP GET)
 ┌─────────────────┐                   ▼
 │  DHT11 Sensor   │───────▶  ┌─────────────────┐
 │  (GPIO 26)      │          │   ThingSpeak     │
 └─────────────────┘          │  (IoT Cloud)     │
                               └─────────────────┘
```

---

## ☁️ ThingSpeak Configuration

| ThingSpeak Field | Data |
|---|---|
| Field 1 | Smoke value (0–1023) |
| Field 2 | Temperature (°C) |
| Field 3 | Flame state (0 = detected, 1 = clear) |
| Field 4 | Humidity (%) |

Update interval: **20 seconds** (ThingSpeak free tier minimum is 15 s)

---

## 🚨 Fire Detection Logic

Fire is triggered if **any one** of the following conditions is true:

```
smokeValue  > 300       (MQ-2 analog threshold)
temperature > 50.0 °C   (DHT11 reading)
flamePin    == LOW      (IR flame sensor active)
```

When triggered → `BUZZER_PIN` goes HIGH and `"🚨 FIRE DETECTED 🚨"` is printed to Serial.

---

## 🛠️ Software Setup

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) (1.8.x or 2.x)
- ESP32 board package installed via Board Manager
  - URL: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`

### Libraries Required

Install via **Arduino IDE → Sketch → Include Library → Manage Libraries**:

| Library | Install name |
|---|---|
| DHT sensor library | `DHT sensor library` by Adafruit |
| Adafruit Unified Sensor | `Adafruit Unified Sensor` |

WiFi and HTTPClient are bundled with the ESP32 board package — no separate install needed.

---

## ⚙️ Configuration

Open `fire_monitor.ino` and update these lines before uploading:

```cpp
// WiFi credentials
const char* ssid     = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// ThingSpeak Write API Key
String thingspeakApiKey = "YOUR_API_KEY_HERE";

// Thresholds (adjust after sensor calibration)
const int   SMOKE_THRESHOLD = 300;
const float TEMP_THRESHOLD  = 50.0;
```

---

## 🚀 Uploading

1. Connect ESP32 via USB
2. In Arduino IDE select:
   - **Board:** `ESP32 Dev Module`
   - **Port:** your COM / tty port
3. Click **Upload**
4. Open **Serial Monitor** at `115200 baud` to see live readings

---

## 📊 ThingSpeak Dashboard

1. Create a free account at [thingspeak.com](https://thingspeak.com)
2. Create a new **Channel** with 4 fields (Smoke, Temperature, Flame, Humidity)
3. Copy the **Write API Key** into the sketch
4. Use ThingSpeak's built-in charts or MATLAB visualisation to monitor live data

---

## 📁 Project Structure

```
esp32-fire-monitor/
├── fire_monitor.ino      # Main Arduino sketch
└── README.md             # This file
```

---

## ⚠️ Notes & Calibration

- **MQ-2 warm-up:** Allow 2–5 minutes after power-on for the MQ-2 sensor to stabilise before readings are reliable.
- **Smoke threshold:** Default is 300 (normalised 0–1023 scale). Run the system in a clean environment, note the baseline, and set the threshold ~100 counts above it.
- **Flame sensor polarity:** Most IR flame modules output `LOW` when a flame is detected. Verify with your specific module.
- **Buzzer driver:** Use an NPN transistor (e.g. 2N2222) between GPIO 27 and the buzzer to avoid exceeding the ESP32's 12 mA GPIO current limit.

---

## 📄 License

This project is open-source under the [MIT License](LICENSE).

---

## 🙏 Acknowledgements

- [Espressif ESP32 Arduino Core](https://github.com/espressif/arduino-esp32)
- [Adafruit DHT Library](https://github.com/adafruit/DHT-sensor-library)
- [ThingSpeak IoT Platform](https://thingspeak.com)
