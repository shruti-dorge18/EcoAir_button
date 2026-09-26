CurrentoAir – Smart Air Purifier

EcoAir is an ESP32-based smart air purifier system that monitors air quality using a dust sensor and automatically controls fan speed according to the detected dust concentration.

The system provides manual fan control, automatic air-quality-based control, timer operation, OLED monitoring, buzzer feedback, and Wi-Fi/web-server support.

---

🚀 Features

- 🌫️ Real-time dust-density monitoring
- 📊 Air-quality classification into:
  - LOW
  - MEDIUM
  - HIGH
- 🌀 Three fan-speed levels:
  - LOW
  - MEDIUM
  - HIGH
- ⏹️ Fan OFF control
- 🤖 Automatic fan-speed adjustment based on air quality
- ⏱️ 30-minute and 60-minute timer modes
- 📺 0.96-inch 128×64 OLED display
- 🔔 Buzzer feedback for touch-button interaction
- 👆 Touch-button controls
- ⚙️ Servo-based fan-speed control
- 📡 ESP32 Wi-Fi connectivity
- 🌐 HTTP web server support
- 💨 GP2Y1010AU0F optical dust sensor
- ⚡ Non-blocking automatic sampling using ESP32 timers
- 🔌 ESP-IDF-based implementation

---

🧠 System Overview

The ESP32 acts as the main controller of the EcoAir purifier.

                 ┌─────────────────────┐
                 │       ESP32          │
                 │    Main Controller   │
                 └──────────┬──────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
  Dust Sensor          Touch Buttons         OLED Display
 GP2Y1010AU0F          Speed / Auto /        128×64 I2C
   │                    Timer
   │
   ▼
Air Quality
Calculation
   │
   ▼
Fan Speed Decision
   │
   ▼
 Servo Motor
   │
   ▼
Fan Controller
   │
   ▼
  Fan

---

🔧 Hardware Components

Component| Purpose
ESP32 D0WD V3| Main controller
GP2Y1010AU0F| Dust/particle sensor
0.96" OLED 128×64| Display
MG90S Servo Motor| Fan-speed control
Touch Sensor ×3| User controls
Buzzer| User feedback
Fan| Air purification
150Ω Resistor| Dust sensor LED circuit
220µF Capacitor| Dust sensor LED supply stabilisation
5V Power Supply| Servo/fan-related power

---

🔌 Pin Configuration

OLED

OLED Pin| ESP32
SDA| GPIO 4
SCL| GPIO 5
VCC| 3.3V
GND| GND

I2C address:

0x3C

---

Touch Sensors

Function| ESP32 GPIO
Speed| GPIO 12
Auto| GPIO 15
Timer| GPIO 23

All touch sensors are powered from 3.3V.

---

Servo Motor

Servo Pin| ESP32
Signal| GPIO 18
VCC| 5V
GND| GND

The MG90S servo is controlled using LEDC PWM at 50 Hz.

---

Buzzer

Buzzer| ESP32
Positive| GPIO 19
Negative| GND

---

Dust Sensor – GP2Y1010AU0F

Sensor Connection| ESP32 / Circuit
LED control| GPIO 25
Analog output| ADC1 Channel 6
Supply| 5V
Ground| GND

The dust sensor LED is controlled by GPIO 25, and the analogue output is read using the ESP32 ADC.

---

⚙️ Fan Control

The fan speed is controlled using a servo motor.

The servo rotates to predefined angles corresponding to different fan speeds.

Fan State| Servo Angle
OFF| 0°
LOW| 95°
MEDIUM| 120°
HIGH| 140°

The servo uses:

Frequency = 50 Hz
Minimum pulse = 500 µs
Maximum pulse = 2500 µs

The function:

set_fan_speed()

updates the current fan state and moves the servo to the corresponding position.

---

🌫️ Air Quality Detection

The GP2Y1010AU0F sensor provides an analogue voltage corresponding to the concentration of airborne particles.

The ESP32:

1. Turns ON the sensor LED.
2. Waits for the required sampling interval.
3. Reads the ADC value.
4. Converts the ADC value to voltage.
5. Applies the sensor conversion formula.
6. Calculates an estimated dust concentration.
7. Classifies the air quality.

The current implementation uses the following thresholds:

Dust Concentration| Air Quality
≤ 35 µg/m³| LOW
> 35 and ≤ 75 µg/m³| MEDIUM
> 75 µg/m³| HIGH

These thresholds are application-level thresholds used by the EcoAir firmware.

---

🤖 Automatic Mode

EcoAir includes an automatic air-quality control mode.

When AUTO mode starts, the ESP32 performs a 60-second measurement cycle.

AUTO button
     │
     ▼
Start measurement
     │
     ▼
Take dust samples
     │
     ├── Sample 1
     ├── Sample 2
     ├── Sample 3
     ├── ...
     └── Sample 60
     │
     ▼
Calculate average dust
     │
     ▼
Determine air quality
     │
     ▼
Select fan speed

The firmware uses:

#define AUTO_SAMPLES 60
#define DUST_SAMPLE_INTERVAL_MS 1000

Therefore, approximately one sample is taken every second.

After 60 samples, the average dust concentration is calculated:

Average Dust =
Sum of all samples / Number of samples

The resulting air-quality level is then used for automatic fan control.

---

⏱️ Timer Mode

EcoAir supports timer-based operation.

Available timer durations:

30 minutes
60 minutes

The firmware stores the timer state using:

timer_active
timer_minutes
timer_end_time

When the timer expires, the fan can be turned OFF automatically.

---

📺 OLED Display

The system uses a 128×64 SSD1306-compatible OLED display through I2C.

The display shows information such as:

AQI: LOW

FAN: MEDIUM

DUST:25.4

MANUAL

During automatic measurement:

AQI: MEDIUM

FAN: HIGH

DUST:58.2

AUTO:35/60

During timer operation:

AQI: LOW

FAN: MEDIUM

DUST:20.5

TIMER:30M

The firmware uses a custom 5×7 bitmap font instead of an external OLED graphics library.

---

🔔 Buzzer

The buzzer provides audible feedback when the user interacts with the touch controls.

The function:

buzzer_beep()

turns the buzzer ON for approximately:

80 ms

and then turns it OFF.

---

👆 Touch Controls

EcoAir uses three touch-input buttons.

Speed Button

Used to control fan speed manually.

Typical sequence:

OFF → LOW → MEDIUM → HIGH → OFF

Auto Button

Starts automatic air-quality measurement and fan control.

Timer Button

Controls timer operation.

Available durations:

30 minutes
60 minutes

---

📡 Wi-Fi Connectivity

The ESP32 is configured to connect to a Wi-Fi network using:

#define WIFI_SSID     "Pawan"
#define WIFI_PASSWORD "12345678"

Security Note: Do not commit real Wi-Fi credentials to a public GitHub repository. Use a configuration file, environment-specific settings, or ESP-IDF NVS/configuration instead.»

The firmware maintains the connection state using:

static bool wifi_connected = false;

---

🌐 Web Server

The project includes an ESP-IDF HTTP server:

static httpd_handle_t web_server = NULL;

The web server is configured to use:

Port: 80

This allows the EcoAir system to be extended with a browser-based control/monitoring interface.

Possible web functionality includes:

- Current dust concentration
- Current air-quality level
- Fan speed
- Fan ON/OFF control
- Automatic mode
- Timer control
- Real-time purifier status

---

🛠️ Software Architecture

The firmware is developed using ESP-IDF and FreeRTOS.

Major software components include:

ESP-IDF
│
├── FreeRTOS
│   └── Task management
│
├── GPIO
│   ├── Touch sensors
│   ├── Buzzer
│   └── Dust sensor LED
│
├── I2C
│   └── OLED display
│
├── LEDC
│   └── Servo PWM
│
├── ADC OneShot
│   └── Dust sensor
│
├── Wi-Fi
│   └── Network connectivity
│
└── HTTP Server
    └── Web interface

---

📦 Important ESP-IDF Components

The project uses the following ESP-IDF modules:

freertos/FreeRTOS.h
freertos/taskh

driver/gpio.h
driver/i2c_masterh
driver/ledch

esp_adc/adc_oneshot.h

esp_wifi.h
esp_event.h
esp_netif.h
nvs_flash.h
esp_http_server.h

---

🧮 Dust Sensor Calculation

The ADC value is converted into voltage:

adc_voltage = ((float)raw / 4095.0f) * 3.3f;

The code then applies a voltage scaling factor:

sensor_voltage = adc_voltage * 1.5f;

The estimated dust concentration is calculated using:

dust =
    (sensor_voltage - 0.60f)
    / 0.50f
    * 1000.0f;

The calculated value is limited to:

0 – 1000 µg/m³

---

🖥️ OLED Rendering System

Instead of directly writing text to the OLED every time, the firmware maintains an internal framebuffer:

static uint8_t oled_frame[
    OLED_WIDTH * OLED_HEIGHT / 8
];

A second framebuffer is used to detect changes:

static uint8_t oled_last_frame[
    OLED_WIDTH * OLED_HEIGHT / 8
];

The display is updated only when the framebuffer changes.

This reduces unnecessary I2C communication.

---

🔄 Operating Modes

EcoAir supports multiple operating states.

Manual Mode

The user directly controls the fan speed.

User → Speed Button → Fan Speed

Automatic Mode

The system measures air quality and controls the fan accordingly.

Dust Sensor → Average Dust → Air Quality → Fan Speed

Timer Mode

The fan operates for a selected period and then stops.

Timer → Fan Running → Timer Expires → Fan OFF

---

📊 State Variables

Important system states include:

current_fan_speed
current_aqi
current_dust

power_state

timer_active
timer_minutes
timer_end_time

auto_mode
auto_measuring
auto_sample_count
auto_dust_sum
auto_end_time

wifi_connected
sleep_mode
display_mode

These variables allow the firmware to maintain the current operating state of the purifier.

---

🔒 Safety Considerations

The EcoAir system controls a physical fan and therefore requires proper electrical isolation.

If the fan is an AC mains fan:

- Do not connect the fan directly to an ESP32 GPIO.
- Use an appropriately rated isolated fan controller/PWM regulator.
- Ensure the controller supports the fan's voltage and current.
- Use proper insulation and enclosure.
- Keep the ESP32 low-voltage circuitry isolated from mains voltage.
- Use appropriate fuses and protection.
- Never work on exposed mains wiring while powered.

The servo should also be powered from an appropriate 5V supply rather than relying on the ESP32 3.3V output.

---

📁 Suggested Project Structure

EcoAir/
│
├── main/
│   ├── main.c
│   ├── CMakeLists.txt
│   └── ...
│
├── components/
│   └── ...
│
├── CMakeLists.txt
├── sdkconfig
└── README.md

---

🚀 Getting Started

1. Install ESP-IDF

Install ESP-IDF according to the official Espressif installation instructions.

2. Clone the Repository

git clone <YOUR_REPOSITORY_URL>
cd EcoAir

3. Configure Wi-Fi

Update the Wi-Fi configuration in the firmware:

#define WIFI_SSID     "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

Do not commit real credentials to GitHub.

4. Select ESP32 Target

idf.py set-target esp32

5. Build the Project

idf.py build

6. Flash the ESP32

idf.py -p PORT flash

Replace "PORT" with your ESP32 serial port.

7. Monitor Serial Output

idf.py monitor

Or combine flashing and monitoring:

idf.py -p PORT flash monitor

---

🧪 Serial Monitor Output

The firmware provides useful debugging information through ESP-IDF logging.

Example:

I (1234) ECOAIR: OLED initialised
I (1240) ECOAIR: Dust sensor initialised
I (1245) ECOAIR: Touch buttons initialised
I (1250) ECOAIR: MG90S servo initialised on GPIO 18

I (5000) ECOAIR: DUST = 28.45 ug/m3 | AQI = LOW

I (6000) ECOAIR: AUTO MODE STARTED
I (6000) ECOAIR: Measuring dust for 60 seconds...

I (7000) ECOAIR: AUTO SAMPLE 1/60 = 31.25
I (8000) ECOAIR: AUTO SAMPLE 2/60 = 30.84

---

🔧 Configuration

The main configurable parameters are defined at the top of the source code.

Servo

#define SERVO_OFF_DEG      0
#define SERVO_LOW_DEG      95
#define SERVO_MEDIUM_DEG   120
#define SERVO_HIGH_DEG     140

Dust Thresholds

#define DUST_LOW_LIMIT     35.0f
#define DUST_MEDIUM_LIMIT  75.0f

Automatic Sampling

#define AUTO_SAMPLES            60
#define DUST_SAMPLE_INTERVAL_MS 1000

Timer

#define TIMER_30_MINUTES 30
#define TIMER_60_MINUTES 60

Automatic Runtime

#define AUTO_RUNTIME_MINUTES 60

---

💡 Future Improvements

Possible improvements for future versions include:

-📱 Mobile application
- 🌐 Real-time web dashboard
- 📈 Historical air-quality graphs
- ☁️ Cloud data logging
- 📊 MQTT integration
- 🔔 Mobile notifications
- 🌡️ Temperature and humidity monitoring
- 🧠 Machine-learning-based air-quality prediction
- ⚡ More precise fan-speed control
- 🔋 Power-consumption monitoring
- 💾 SD-card data logging
- 🔐 Secure Wi-Fi configuration
- 🔄 OTA firmware updates
- 📡 Remote device control
- 💤 Low-power sleep mode

---

🏆 Project Highlights

EcoAir combines several embedded-system technologies into a single practical application:

ESP32
  +
Dust Sensing
  +
ADC
  +
I2C OLED
  +
Servo PWM
  +
Touch Input
  +
Automatic Control
  +
Wi-Fi
  +
Web Server
  =
Smart Air Purifier

The project demonstrates practical implementation of embedded systems, IoT, sensor interfacing, PWM control, real-time processing, and ESP-IDF firmware development.

---

👨‍💻 Technologies Used

- ESP32
- ESP-IDF
- C
- FreeRTOS
- GPIO
- I2C
- ADC
- LEDC PWM
- Wi-Fi
- HTTP Server
- SSD1306 OLED
- GP2Y1010AU0F Dust Sensor
- MG90S Servo Motor

---

📜 License

Add your preferred license here, for example:

MIT License

---

🙌 Contributors

- Pawan Nanaware
- Shruti Dorge
- Atharv Bhavsar
- Pranav Vidhate

---

📌 Note

This README describes the functionality visible in the provided EcoAir firmware. The source code supplied in the request ends partway through "process_auto_measurement()", so the exact behaviour of the remaining automatic-control, timer, touch-processing, Wi-Fi, and HTTP-server functions may differ from the descriptions above.
