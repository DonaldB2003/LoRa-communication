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
LoRa RA-02 Module Pinout <=> ESP32 Pin Mapping
|-------------------|------------|---------------|
| LoRa RA-02 Pin    | Function   | ESP32 Pin     |
|-------------------|------------|---------------|
| DIO0              | Interrupt  | GPIO 26       |
| RESET             | Reset      | GPIO 14       |
| NSS / CS          | Chip Select| GPIO 18       |
| MOSI              | MOSI       | GPIO 27       |
| MISO              | MISO       | GPIO 19       |
| SCK               | Clock      | GPIO 5        |
|-------------------|------------|---------------|

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

## 🌟 Why Use This Project?
 - Great for Learning: Explore LoRa communication, real-time data transfer, and display integration in a hands-on, practical way.
  - Flexible & Expandable: Easily adapt the code for other IoT applications—remote timers, wireless displays, or sensor data transmission.
  - Open Source & Beginner-Friendly: Clean, well-documented code with step-by-step setup instructions.
  - Whether you’re building a remote sports timer, experimenting with LoRa, or looking for a fun IoT project, LoRa Stopwatch Sync is the perfect starting point. Fork, clone, and start building your own wireless stopwatch system today!
