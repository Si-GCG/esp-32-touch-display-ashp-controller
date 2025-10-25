# Grant Aerona3 Touchscreen Controller

A touchscreen controller for the Grant Aerona3 air-source heat pump (ASHP) using a Waveshare ESP32-S3 7" touchscreen display. The device provides local control (touch UI on the display) and remote control via Home Assistant (MQTT or ESPHome integration). This repository contains firmware, UI assets, wiring guidance and Home Assistant integration examples.

Status: WIP — Prototype firmware and documentation.

Table of contents
- Project overview
- Features
- Hardware
  - Recommended parts
  - Power considerations
  - Wiring & pin mapping (example / placeholders)
- Software
  - Architecture & components
  - Build options
  - Configuration (Wi‑Fi, MQTT, ESPHome)
- Home Assistant integration
  - MQTT topics & payloads (Coming soon)
  - Example Home Assistant YAML
- UI design & customization
- Flashing & OTA
- Troubleshooting
- Resources
- Contributing
- License

Project overview
This project replaces or augments a physical controller for the Grant Aerona3 ASHP with a custom touchscreen interface built on a Waveshare ESP32-S3 7" touchscreen display. The device enables:
- Local device control via an on-device touch UI
- Remote monitoring and control from Home Assistant
- Automated schedules and safety interlocks
- Extendable firmware and UI (LVGL or equivalent)

Features
- Full touchscreen UI for weather compensation mode, setpoint and status
- Wi‑Fi connectivity to integrate with Home Assistant
- Remote control via MQTT or ESPHome
- OTA updates (optional)
- Configurable safety limits and error display
- Logging of basic telemetry (temperatures, modes, run status)

Hardware

Recommended parts
- Waveshare ESP32-S3 7-inch capacitive touchscreen display (the ESP32-S3 model)
- Stable 5V power supply USB C
- JST / Dupont wires and appropriate connectors
- Optional enclosure and mounting hardware

Wiring & pin mapping (example / placeholders)
The display typically exposes:
- Display interface (SPI or parallel) — depends on the exact Waveshare model
- Touch controller (I2C or SPI)
- Backlight control (PWM / GPIO)
- Power and ground

Important: consult your exact Waveshare display manual for connector pinouts before wiring. Below is an example mapping pattern — replace with pins for your specific display and board.

Example (replace with actual pins for your hardware):
- 5V VCC (display) -> 5V power supply
- GND (display) -> GND
- SPI SCK (display) -> ESP32-S3 GPIO 18
- SPI MOSI (display) -> ESP32-S3 GPIO 23
- SPI MISO (if used) -> ESP32-S3 GPIO 19
- Reset (display) -> ESP32-S3 GPIO 4
- DC / RS (display) -> ESP32-S3 GPIO 2
- Touch I2C SDA -> ESP32-S3 GPIO 21
- Touch I2C SCL -> ESP32-S3 GPIO 22
- Backlight PWM -> ESP32-S3 GPIO 13

Note: The Waveshare ESP32-S3 model may already integrate the microcontroller and display, changing wiring needs. Use vendor docs.

Software

Architecture & components
- Firmware: ESP32-based firmware using:
  - UI: LVGL (recommended) or a lightweight custom renderer
  - SDL: allows you to quickly iterate using UI changes without having to flash the actual device. Navigate your terminal to the folder with your sdl_test.yaml file and make changes to that file then save then run the following command - esphome run sdl_yaml 
  - Connectivity: Wi‑Fi + MQTT client OR ESPHome
  - Control logic: local state machine to handle modes, setpoints, safety and remote commands
- Home Assistant: configure entities (climate, switches, sensors) using MQTT or ESPHome integration

Build instructions

1. ESPHome
   Edit the secrets.yaml to include your systems details
   Once you are happy with your build run the following commands from your project directory (where your ASHP_ESP32-S3_display.yaml and secrets.yaml files are)
   
   ```
   esphome compile ASHP_ESP32-S3_display.yaml
   esphome run ASHP_ESP32-S3_display.yaml
   ```

Configuration (Wi‑Fi, MQTT, ESPHome) (I am still working on MQTT)
- Wi‑Fi: SSID and password stored in secure config (ideally prompt on first boot or use provisioning).
- MQTT: Broker address, username, password, base topic
- ESPHome: If using ESPHome, implement display and custom components through its YAML.

Home Assistant integration

MQTT topics & payloads (example)
- Base topic: grant/aerona3
- Commands:
  - grant/aerona3/command/set_mode -> payload: "heat" | "cool" | "off" | "auto"
  - grant/aerona3/command/set_temperature -> payload: "21.5" (numeric)
  - grant/aerona3/command/set_fan -> payload: "low" | "medium" | "high" | "auto"
  - grant/aerona3/command/power -> payload: "on" | "off"
- State / telemetry:
  - grant/aerona3/state/mode -> "heat"
  - grant/aerona3/state/temperature -> "21.2"
  - grant/aerona3/state/target -> "22.0"
  - grant/aerona3/state/fan -> "auto"
  - grant/aerona3/state/error -> "E01" or empty string

Example Home Assistant (MQTT) configuration
Replace broker / credentials / topics as needed.

configuration.yaml example (MQTT Climate)
```yaml
climate:
  - platform: mqtt
    name: "Grant Aerona3"
    mode_command_topic: "grant/aerona3/command/set_mode"
    mode_state_topic: "grant/aerona3/state/mode"
    modes:
      - "off"
      - "heat"
      - "cool"
      - "auto"
    temperature_command_topic: "grant/aerona3/command/set_temperature"
    current_temperature_topic: "grant/aerona3/state/temperature"
    temperature_state_topic: "grant/aerona3/state/target"
    min_temp: 5
    max_temp: 30
    temp_step: 0.5
```

You can also expose binary switches (power) with MQTT switch entities.

ESPHome example
- ESPHome can simplify the HA side (native integration), but complex LVGL GUIs will likely need custom code outside ESPHome. Use ESPHome if you prefer configuration-by-YAML and built-in OTA + HA discovery.

UI design & customization
- The UI is implemented using LVGL (recommended) or a chosen GUI framework.
- Suggested screens:
  - Home: current mode, indoor temp, setpoint, quick controls (power, mode)
  - Settings: Wi‑Fi, MQTT (coming soon)
  - Diagnostics: logs, error codes, firmware version
- Assets: icons, fonts and translations are stored in /assets (TBD)
- The UI is modular — customize layouts, color themes, and widgets.

Flashing & OTA

Initial flashing (USB)
- Connect the ESP32-S3 via USB
- Use PlatformIO or esptool to upload the firmware binary

OTA updates
- Implement OTA (ArduinoOTA, ESP-IDF OTA or custom) to allow firmware updates over Wi‑Fi
- Secure OTA with authentication if exposed to untrusted networks

Troubleshooting
- Display not showing: check power, backlight, and wiring. Verify contrast/power pins and that the display uses the expected interface (SPI vs parallel).
- Touch not responding: check touch controller interface (I2C vs SPI) and calibrate.
- Wi‑Fi connection issues: verify credentials, AP signal, and power stability.
- MQTT messages not received: check broker settings, topic names and credentials. Use mosquitto_sub or MQTT Explorer to debug.
- Heat pump control: ensure any wiring to the heat pump is done only per the vendor's instructions and local safety codes. Use optoisolators or proper relays when switching mains.

Resuorces
https://github.com/espressif/esp-bsp/tree/master/components/lcd_touch/esp_lcd_touch_gt911

https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-7

https://docs.lvgl.io/8.3/examples.html#

Safety & regulatory notes
- This project interfaces with HVAC equipment. Incorrect wiring or software bugs can cause equipment damage or unsafe operation.
- If you're not confident with electrical work, consult a qualified HVAC/electrical technician.
- Never bypass built-in safety systems on the ASHP.

Contributing
- Open an issue for feature requests or bugs
- Fork, make changes in a feature branch, test, and open a pull request
- Follow code style in the repo and include tests where appropriate
- Please document any hardware-specific pinouts and wiring you test

Repository structure (suggested)
- /includes — add any modbus registers dependent on type (Sensors, Switches , Binary Sensors, Input Numbers etc)
- /fonts— hardware diagrams, vendor datasheets, pinouts
- /images — any images you want to include in your display


What I have included
- A complete README template covering project goals, components, guidance for building, wiring, and Home Assistant integration with sample MQTT topics and Home Assistant YAML.

License
This repository defaults to the GPL 2.0 license. 

Acknowledgements
- Waveshare for the hardware
- LVGL community for GUI components
- Home Assistant community for integration patterns
- Benny Diamond and his awesome project - https://github.com/bennydiamond/esphome_lvgl_hmi_garage

Contact / author
- Si-GCG (project owner)
