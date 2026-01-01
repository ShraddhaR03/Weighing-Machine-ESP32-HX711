#include "HX711.h"
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include "BluetoothSerial.h"

// ---------------- HX711 Pins ----------------
#define LOADCELL_DOUT_PIN  4
#define LOADCELL_SCK_PIN   2

float calibration_factor = 87500;

HX711 scale;

// ---------------- OLED ----------------
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// ---------------- Bluetooth ----------------
BluetoothSerial SerialBT;

// ---------------- Peak Reset Button ----------------
#define PEAK_RESET_PIN 27   // D27
float peak_weight = 0.0;

void setup() {
  Serial.begin(115200);
  Serial.println("Initializing 10kg Scale...");

  // Bluetooth
  SerialBT.begin("ESP32_Weight_Scale");
  SerialBT.println("Bluetooth Ready");

  // HX711 (UNCHANGED)
  scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);
  scale.set_scale(calibration_factor);

  Serial.println("Taring... Please wait.");
  delay(1000);
  scale.tare();
  Serial.println("Ready. Place weight on scale.");

  // OLED
  Wire.begin(21, 22);
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED failed");
    while (1);
  }
  display.clearDisplay();
  display.setTextColor(WHITE);

  // Button
  pinMode(PEAK_RESET_PIN, INPUT_PULLUP);
}

void loop() {
  // -------- HX711 READ --------
  float weight = scale.get_units(1);

  if (weight < 0.05 && weight > -0.05)
    weight = 0.00;

  // -------- Peak Logic --------
  if (weight > peak_weight) {
    peak_weight = weight;
  }

  // -------- Serial --------
  Serial.print("Weight: ");
  Serial.print(weight, 2);
  Serial.print(" kg | Peak: ");
  Serial.print(peak_weight, 2);
  Serial.println(" kg");

  // -------- OLED --------
  display.clearDisplay();

  display.setTextSize(2);
  display.setCursor(0, 0);
  display.print("Peak");

  display.setTextSize(3);
  display.setCursor(0, 25);
  display.print(peak_weight, 2);
  display.print("kg");

  display.display();

  // -------- Bluetooth Output --------
  SerialBT.print("Peak: ");
  SerialBT.print(peak_weight, 2);
  SerialBT.println(" kg");

  // -------- Button Reset --------
  if (digitalRead(PEAK_RESET_PIN) == LOW) {
    peak_weight = 0.0;
    Serial.println("Peak Reset (Button)");
    SerialBT.println("Peak Reset (Button)");
    delay(300); // debounce
  }

  // -------- Bluetooth RESET Command --------
  if (SerialBT.available()) {
    String cmd = SerialBT.readStringUntil('\n');
    cmd.trim();
    cmd.toUpperCase();

    if (cmd == "RESET") {
      peak_weight = 0.0;
      Serial.println("Peak Reset (Bluetooth)");
      SerialBT.println("Peak Reset OK");
    }
  }

  delayMicroseconds(250);
}
