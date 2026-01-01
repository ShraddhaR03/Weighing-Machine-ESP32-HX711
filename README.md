# ESP32 HX711 Digital Weighing Machine

A digital weighing machine built with ESP32 and HX711 load cell amplifier. It supports:

- Live weight measurement
- Peak weight capture
- OLED display (SSD1306)
- Bluetooth communication
- Reset button & Bluetooth reset command
- Stable readings with dead zone filtering

## Hardware Used

| Component | Description |
|-----------|-------------|
| ESP32 | Main microcontroller |
| HX711 | 24-bit ADC for load cell |
| Load Cell | Weight sensor |
| SSD1306 OLED | Display |
| Push Button | Peak reset |
| Bluetooth | Mobile display |

## Connection

### HX711 to ESP32

| HX711 | ESP32 |
|-------|-------|
| VCC | 5V |
| GND | GND |
| DOUT | GPIO 4 |
| SCK | GPIO 2 |

### OLED to ESP32

| OLED | ESP32 |
|------|--------|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

### Peak Reset Button

| Button | ESP32 |
|--------|--------|
| One pin | GPIO 27 |
| Other pin | GND |

## Features

- Auto tare at startup
- Peak weight lock
- Bluetooth communication
- Debounced peak reset button
- Dead zone (to ignore micro-noise near zero)

##  Bluetooth Commands

| Command | Description |
|---------|-------------|
| `RESET` | Resets peak weight |

## Calibration

Adjust the `calibration_factor` in code for accurate readings:

```cpp
float calibration_factor = 87500;
