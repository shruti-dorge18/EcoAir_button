# ESP32 Air Purifier — EcoAir

An **ESP32-based smart air purifier** designed for indoor spaces up to **50 sq. ft.** using a **HEPA 13 filter**.

Integrated a H13 HEPA filter with a stated 99.97% filtration efficiency for particles down to 0.1 µm.

The system provides three operating modes:

* **Manual Speed Mode**
* **Timer Mode**
* **Automatic Mode**

Developed using **ESP-IDF** and **Visual Studio Code**.

---

## Components Used

| Component       | Specification                        |
| --------------- | ------------------------------------ |
| Microcontroller | ESP32 D0WD V3                        |
| Dust Sensor     | GP2Y1010AU0F                         |
| Display         | 0.96-inch OLED                       |
| Touch Sensors   | 3                                    |
| Servo Motor     | MG90S                                |
| Buzzer          | 1                                    |
| Fan             | 220 V AC, 0.5 A, 60 W, 2800 ± 50 RPM |
| Resistors       | 150 Ω, 10 kΩ, 20 kΩ                  |
| Capacitor       | 220 µF                               |

---

## Project Structure

```text
Air_Purifier/
├── CMakeLists.txt
├── sdkconfig
├── sdkconfig.old
├── README.md
│
└── main/
    ├── CMakeLists.txt
    ├── final_EcoAir.c
    └── idf_component.yml
```

### System Flow

```text
                    ESP32
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
     Dust Sensor     OLED       Touch
          │                       │
          ▼                       ▼
   Dust Level Analysis       Control Modes
    LOW / MEDIUM / HIGH       │
          │                   │
          └─────────┬─────────┘
                    ▼
             Fan Control Logic
                    │
                    ▼
               MG90S Servo
                    │
                    ▼
               Fan Speed
```

---

## Development Environment

* **ESP32**
* **ESP-IDF**
* **Visual Studio Code**
* ESP-IDF VS Code Extension

> Arduino IDE is not used in this project.

---

## Hardware Connections

### OLED Display

| OLED Pin | ESP32  |
| -------- | ------ |
| SDA      | GPIO 4 |
| SCL      | GPIO 5 |
| VCC      | 3.3 V  |
| GND      | GND    |

The OLED continuously displays the current dust level.

### Touch Sensors

| Sensor   | Function | GPIO    |
| -------- | -------- | ------- |
| Sensor 1 | Speed    | GPIO 12 |
| Sensor 2 | Auto     | GPIO 15 |
| Sensor 3 | Timer    | GPIO 23 |

All touch sensors are powered using **3.3 V** and **GND**.

### MG90S Servo

| Servo Wire | Connection |
| ---------- | ---------- |
| Yellow     | GPIO 18    |
| Red        | 5 V        |
| Brown      | GND        |

### Buzzer

* Positive → **GPIO 19**
* Negative → **GND**

The buzzer provides feedback when a touch sensor is detected.

### GP2Y1010AU0F Dust Sensor

| Sensor Wire | Connection                |
| ----------- | ------------------------- |
| Blue        | 150 Ω resistor → 5 V      |
| Green       | GND                       |
| White       | GPIO 25                   |
| Yellow      | GND                       |
| Black       | Voltage divider → GPIO 34 |
| Red         | 5 V                       |

Dust Sensor — Blue Wire

The blue wire is connected to the 5 V supply through a **150 Ω resistor**. A **220 µF capacitor** is used for filtering.

```text
5V
 │
 │
150 Ω
 │
 ├──────── Blue wire
 │
 └──── + 220 µF capacitor
             │
             └──── GND
```

Dust Sensor — Black Wire

The black wire is connected to **GPIO 34** through a voltage-divider circuit using **10 kΩ and 20 kΩ resistors**.

```text
Dust Sensor Black
       │
      10 kΩ
       │
       ├──────── GPIO 34
       │
      20 kΩ
       │
      GND
```

> **Note:** The dust sensor monitors the **dust parameter only**. For **AQI monitoring**, use a dedicated **PM2.5 sensor**.

### PM2.5 Sensor Connections

| PM2.5 Sensor Pin | ESP32 Connection |
| ---------------- | ---------------- |
| TX               | GPIO 0           |
| RX               | GPIO 2           |
| VCC              | 5 V              |
| GND              | GND              |

---

## Fan Speed Control

The MG90S servo controls the fan operating level:

| Servo Angle | Fan State |
| ----------- | --------- |
| 0°          | OFF       |
| 95°         | LOW       |
| 120°        | MEDIUM    |
| 140°        | HIGH      |

The ESP32 does **not directly drive the 220 V AC fan**. The servo is used as part of the fan-speed control mechanism.

---

## Dust-Level Classification

| Dust Concentration | Condition           | Fan Speed |
| ------------------ | ------------------- | --------- |
| 0–35 µg/m³         | Clean               | LOW       |
| 36–75 µg/m³        | Moderate            | MEDIUM    |
| 76–500 µg/m³       | High Dust           | HIGH      |
| >500 µg/m³         | Sensor Upper Region | HIGH      |

The OLED displays:

```text
DUST LEVEL: LOW
DUST LEVEL: MEDIUM
DUST LEVEL: HIGH
```

---

# Operating Modes

## 1. Manual Speed Mode

The **Speed touch sensor** cycles through:

```text
OFF → LOW → MEDIUM → HIGH → OFF
```

Servo positions:

```text
OFF    → 0°
LOW    → 95°
MEDIUM → 120°
HIGH   → 140°
```

The OLED displays the dust level and current fan speed.

---

## 2. Timer Mode

The **Timer touch sensor** provides:

* **30 minutes**
* **60 minutes**

When selected:

1. The timer starts.
2. The fan continues at its current/selected speed.
3. The OLED displays the dust level and timer.
4. When the timer expires, the servo moves to **0°**.
5. The fan switches OFF.

Example:

```text
DUST LEVEL: LOW
TIMER: 30 mins
```

---

## 3. Automatic Mode

The **Auto touch sensor** automatically adjusts the fan speed according to dust concentration.

### Process

1. Current fan speed is retained.
2. One dust reading is taken every second.
3. Measurements are collected for **1 minute**.
4. The average dust concentration is calculated.
5. Fan speed is selected according to the dust range.
6. The selected speed remains active for **60 minutes**.
7. The fan then returns to **MEDIUM** speed.

| 1-Minute Average | Fan Speed | Servo |
| ---------------- | --------- | ----- |
| 0–35 µg/m³       | LOW       | 95°   |
| 36–75 µg/m³      | MEDIUM    | 120°  |
| 76–500 µg/m³     | HIGH      | 140°  |
| >500 µg/m³       | HIGH      | 140°  |

During the 1-minute measurement period, the fan continues at its previous speed.

The OLED displays:

```text
DUST LEVEL: LOW
Fan Speed: LOW
AUTO MODE ON
```

After 60 minutes:

```text
Fan → MEDIUM
Servo → 120°
```

Pressing Auto again after completion starts a new measurement cycle.

---

## Control Flow

```text
START
  ↓
Initialize ESP32
  ↓
Initialize OLED / Dust Sensor / Servo / Touch Sensors
  ↓
Main Loop
  │
  ├── Speed Touch → Change Fan Speed
  │
  ├── Timer Touch → Start 30/60 min Timer
  │
  └── Auto Touch → Measure Dust → Calculate Average
                                  ↓
                             Select Speed
                                  ↓
                             Run 60 min
                                  ↓
                            Return MEDIUM
  ↓
Update OLED
  ↓
Continue Loop
```

---

## Electrical Safety

The fan operates at:

* **220 V AC**
* **0.5 A**
* **60 W**
* **2800 ± 50 RPM**

The ESP32 must **not be connected directly to the 220 V AC fan**.

An appropriate **isolated AC switching/control circuit** must be used between the ESP32 and the mains-powered fan.

A **5 V phone charger adapter** was used to power the ESP32 and servo motor.

---

## Future Improvements

* Using PM2.5 AQI sensor for better air quality monitoring
* Wi-Fi/mobile monitoring
* Historical dust-level logging
* Filter replacement indication
* Improved fan-speed control
* Improved enclosure and airflow design

---

## Team Members

1. **Shruti Dorge** — [GitHub](https://github.com/shruti-dorge18)
2. **Atharva Bhavsar** — [GitHub](https://github.com/AtharvaBhavsar14)
3. **Pawan Nanaware** — [GitHub](https://github.com/PWN-N)
4. **Prathamesh Vidhate** — [GitHub](https://github.com/pranav007-cell)
