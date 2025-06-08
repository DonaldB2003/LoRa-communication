# LoRa Stopwatch Sync: Wireless Stopwatch Display with OLED
Welcome to LoRa Stopwatch Sync—an innovative IoT project that showcases seamless, long-range wireless communication using LoRa technology! This repository demonstrates how to transmit a stopwatch’s value from one LoRa module to another, displaying the real-time result on a vibrant OLED screen.

## 🚀 Project Highlights
 - Long-Range Communication: Uses LoRa modules for reliable, low-power data transfer over impressive distances—perfect for remote or outdoor scenarios.
 - Real-Time Stopwatch Updates: The transmitter node sends the current stopwatch time at regular intervals. The receiver node instantly displays this value on an OLED screen, ensuring up-to-the-second accuracy.
 - Clear OLED Display: Enjoy a crisp, high-contrast interface that makes the stopwatch easy to read, even in bright or low-light environments.
 - ESP32 Powered: Built on the versatile ESP32 platform, making development, customization, and expansion a breeze.

## 🛠️ How It Works
 - Transmitter Node: Tracks the stopwatch and sends updates via LoRa.
 - Receiver Node: Listens for incoming data and updates the OLED display in real time.

## Transmitter code

### pin configuration
LoRa RA-02 Module Pinout <=> ESP32 Pin Mapping(transmitter)

| LoRa RA-02 Pin    | Function   | ESP32 Pin     |
|-------------------|------------|---------------|
| DIO0              | Interrupt  | GPIO 26       |
| RESET             | Reset      | GPIO 14       |
| NSS / CS          | Chip Select| GPIO 18       |
| MOSI              | MOSI       | GPIO 27       |
| MISO              | MISO       | GPIO 19       |
| SCK               | Clock      | GPIO 5        |

!! Voltage Level: Ensure your LoRa RA-02 module is powered with 3.3V and not 5V, as it may damage the module.

```c
#include <SPI.h>
#include <LoRa.h>


// customised pin configuration
#define DIO0 26
#define RST 14
#define NSS 18
#define MOSI 27
#define MISO 19
#define SCLK 5

unsigned long counter = 0;  // Seconds counter

void setup() {
  Serial.begin(115200);
  while (!Serial);
  Serial.println("LoRa Time Transmitter");

  // Initialize LoRa with your pins
  SPI.begin(SCLK, MISO, MOSI, NSS);
  LoRa.setPins(NSS, RST, DIO0);

  while (!LoRa.begin(433E6)) {
    Serial.println(".");
    delay(500);
  }
  LoRa.setSyncWord(0xF3);
  Serial.println("LoRa Initialized!");
}

void loop() {
  // Calculate hours, minutes, seconds
  unsigned long totalSeconds = counter;
  int hours = totalSeconds / 3600;
  int remainder = totalSeconds % 3600;
  int minutes = remainder / 60;
  int seconds = remainder % 60;

  // Format time as HH:MM:SS
  char timeStr[9];
  snprintf(timeStr, sizeof(timeStr), "%02d:%02d:%02d", hours, minutes, seconds);

  // Send timestamp
  LoRa.beginPacket();
  LoRa.print(timeStr);
  LoRa.endPacket();

  Serial.print("Sent: ");
  Serial.println(timeStr);

  counter++;
  delay(1000);  // Wait exactly 1 second
}


```

## Receiver code
LoRa RA-02 Module Pinout <=> ESP32 Pin Mapping(receiver)

| LoRa RA-02 Pin    | Function   | ESP32 Pin     |
|-------------------|------------|---------------|
| DIO0              | Interrupt  | GPIO 26       |
| RESET             | Reset      | GPIO 14       |
| NSS / CS          | Chip Select| GPIO 18       |
| MOSI              | MOSI       | GPIO 27       |
| MISO              | MISO       | GPIO 19       |
| SCK               | Clock      | GPIO 5        |

!! Voltage Level: Ensure your LoRa RA-02 module is powered with 3.3V and not 5V, as it may damage the module.

```c
#include <SPI.h>
#include <LoRa.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// OLED Configuration
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// LoRa Pins for ESP32
#define DIO0 26
#define RST 14
#define NSS 18
#define MOSI 27
#define MISO 19
#define SCLK 5

void setup() {
  Serial.begin(9600);
  
  // Initialize OLED
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.println(F("LoRa Receiver"));
  display.display();

  // Configure LoRa module with ESP32-specific pins
  SPI.begin(SCLK, MISO, MOSI, NSS);
  LoRa.setPins(NSS, RST, DIO0);
  
  if (!LoRa.begin(433E6)) {
    Serial.println("LoRa init failed!");
    display.clearDisplay();
    display.setCursor(0,0);
    display.println(F("LoRa Init Failed"));
    display.display();
    while (1);
  }
  
  LoRa.setSyncWord(0xF3);
  Serial.println("LoRa Initialized!");
  display.clearDisplay();
  display.setCursor(0,0);
  display.println(F("LoRa Initialized!"));
  display.display();
}

void loop() {
  int packetSize = LoRa.parsePacket();
  if (packetSize) {
    String receivedData = "";
    while (LoRa.available()) {
      receivedData += (char)LoRa.read();
    }
    
    Serial.print("Received: ");
    Serial.println(receivedData);

    // Display on OLED
    display.clearDisplay();
    display.setTextSize(2);  // Larger text size for better visibility
    display.setCursor(0, 20);
    display.println(receivedData);
    display.display();
  }
  delay(10);
}


```



## 🌟 Why Use This Project?
 - Great for Learning: Explore LoRa communication, real-time data transfer, and display integration in a hands-on, practical way.
  - Flexible & Expandable: Easily adapt the code for other IoT applications—remote timers, wireless displays, or sensor data transmission.
  - Open Source & Beginner-Friendly: Clean, well-documented code with step-by-step setup instructions.
  - Whether you’re building a remote sports timer, experimenting with LoRa, or looking for a fun IoT project, LoRa Stopwatch Sync is the perfect starting point. Fork, clone, and start building your own wireless stopwatch system today!
