# 🎙️ Voice-Controlled Smart Switch — ESP32 + Servo + Google Assistant

Control a physical wall switch with your voice using an ESP32, a servo motor, and Google Assistant — no extra hardware needed beyond what you likely already have.

> **"Ok Google, turn on fan switch"** → Servo rotates and flips the switch ✅

---

## 📸 How It Works

```
Google Assistant
      │
      ▼
 Sinric Pro Cloud
      │
      ▼
  ESP32 (Wi-Fi)
      │
      ▼
 Servo Motor → Physical Switch
```

---

## ✨ Features

- 🗣️ Voice control via Google Assistant
- 🌐 Control from **anywhere** — works over the internet
- 💡 No extra hardware — just ESP32 + servo + Wi-Fi
- 📱 Also works from Google Home app
- 🔄 Servo returns to neutral position after each action
- 🔌 Expandable — add Alexa, multiple switches, scheduling

---

## 🛒 Hardware Required

| Component | Details |
|-----------|---------|
| ESP32 Dev Module | Any standard ESP32 board |
| Servo Motor | SG90 or similar |
| Wi-Fi Network | 2.4 GHz only (ESP32 doesn't support 5 GHz) |
| Jumper Wires (male to female) | For connections |
| External 5V Supply  | for supplying power to the esp32 |

---

## 🔌 Wiring

| Servo Wire | ESP32 Pin |
|------------|-----------|
| Red (VCC) | 5V |
| Brown/Black (GND) | GND |
| Orange/Yellow (Signal) | GPIO 13 |

> ⚠️ **If the servo behaves erratically**, power it from an external 5V source and connect the servo's GND to the ESP32's GND.

---

## 📦 Software & Libraries

Install these in **Arduino IDE** via `Sketch → Include Library → Manage Libraries`:

- [`SinricPro`](https://github.com/sinricpro/esp8266-esp32-sdk) — by Sinric
- [`SinricProSwitch`](https://github.com/sinricpro/esp8266-esp32-sdk) — included with SinricPro
- [`ESP32Servo`](https://github.com/madhephaestus/ESP32Servo)

---

## ⚙️ Setup Guide

### Step 1 — Create a Sinric Pro Account

1. Go to [sinric.pro](https://sinric.pro) and create a free account
2. Navigate to **Devices → Add Device**
3. Set **Device Type = Switch**
4. Name it (e.g., `Room Switch`) and save

### Step 2 — Copy Your Credentials

From the Sinric Pro dashboard, collect:

| Value | Where to find |
|-------|--------------|
| `APP_KEY` | Credentials page |
| `APP_SECRET` | Credentials page |
| `DEVICE_ID` | Inside your created device |

### Step 3 — Upload the Code

Clone this repo or copy the code below. Fill in your credentials and Wi-Fi details, then upload to your ESP32.

```cpp
#include <WiFi.h>
#include <SinricPro.h>
#include <SinricProSwitch.h>
#include <ESP32Servo.h>

const char* ssid     = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

#define APP_KEY    "YOUR_APP_KEY"
#define APP_SECRET "YOUR_APP_SECRET"
#define DEVICE_ID  "YOUR_DEVICE_ID"

Servo myservo;

bool onPowerState(const String &deviceId, bool &state) {
  if (state) {
    // ON — flip switch forward
    myservo.write(67); //adjust the degree of rotation as required//
    delay(500);
    myservo.write(90);//adjust as required//
    Serial.println("Switch ON");
  } else {
    // OFF — flip switch backward
    myservo.write(115); //adjust the degree of rotation as required//
    delay(500);
    myservo.write(90);//adjust as required//
    Serial.println("Switch OFF");
  }
  return true;
}

void setup() {
  Serial.begin(115200);

  myservo.attach(13);
  myservo.write(90); // neutral position

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Connected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  SinricProSwitch &mySwitch = SinricPro[DEVICE_ID];
  mySwitch.onPowerState(onPowerState);

  SinricPro.onConnected([]()    { Serial.println("Connected to SinricPro");    });
  SinricPro.onDisconnected([]() { Serial.println("Disconnected from SinricPro"); });

  SinricPro.begin(APP_KEY, APP_SECRET);
}

void loop() {
  SinricPro.handle();
}
```

**Board settings in Arduino IDE:**
- Board: `ESP32 Dev Module`
- Baud rate: `115200`

### Step 4 — Connect Google Assistant

1. Open **Google Home** app
2. Tap **Add device → Works with Google**
3. Search for **Sinric Pro** and log in
4. Your switch will appear as a device

### Step 5 — Test It!

Say:
- **"Ok Google, turn on room switch"** → Servo moves to 67°, returns to 90°
- **"Ok Google, turn off room switch"** → Servo moves to 115°, returns to 90°
