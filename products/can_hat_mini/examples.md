---
title: SPI-CAN Breakout - Examples
layout: default
parent: SPI-CAN Breakout
grand_parent: Products
nav_order: 1
---

# Example Code

The following example code was made for an `Arduino Nano` and similar
derivatives showing use with classic CAN (500 kbps).

- The code uses the
  open-source [ACAN2517](https://github.com/pierremolinaro/acan2517) Arduino
  Library.
- The [ACAN2517FD](https://github.com/pierremolinaro/acan2517FD) library should
  also be functional for CAN FD applications.

> Note: The interrupt pin should be attached to a hardware interrupt pin. On the
> `Arduino Nano` this is restricted to `D2` or `D3`.

```c++
#include <SPI.h>
#include <ACAN2517.h>

// Example shown with Arduino Nano.
static const uint8_t CS_PIN = 10;  // D10.
static const uint8_t INT_PIN = 2;  // D2 or D3.
static const uint8_t LED_PIN = 13; // Onboard LED.

ACAN2517 can(CS_PIN, SPI, INT_PIN);

uint8_t test_val = 0;

void process_can_frame(const CANMessage& msg) {
  // Print recieved message.
  Serial.print("RX ID: 0x");
  Serial.print(msg.id, HEX);
  Serial.print("  DLC: ");
  Serial.print(msg.len);
  Serial.print("  Data: ");
  for (uint8_t i = 0; i < msg.len; i++) {
    if (msg.data[i] < 0x10) Serial.print("0");  // Leading zero.
    Serial.print(msg.data[i], HEX);
    Serial.print(" ");
  }
  Serial.println();

  // Very quick LED blink.
  digitalWrite(LED_PIN, !digitalRead(LED_PIN));

  // Send message 0x101 with 1 byte signal incrementing (repeat with overflow).
  CANMessage tx_msg;
  tx_msg.id = 0x101;
  tx_msg.len = 1;
  tx_msg.data[0] = test_val;
  bool status = can.tryToSend(tx_msg);
  test_val++;
}

void setup() {
  Serial.begin(115200);

  delay(50);

  Serial.println("BOOT");

  pinMode(LED_PIN, OUTPUT);
  pinMode(CS_PIN, OUTPUT);
  digitalWrite(CS_PIN, HIGH);
  pinMode(INT_PIN, INPUT_PULLUP);

  SPI.begin();

  // 40 MHz oscillator, 500 kbps classic CAN.
  ACAN2517Settings settings(ACAN2517Settings::OSC_40MHz, 500000);

  uint32_t error = can.begin(settings, [] {
    can.isr();
  });

  Serial.print("begin() error=0x");
  Serial.println(error, HEX);

  if (error != 0) {
    while (true) delay(100);
  }

  Serial.println("CAN init okay");
}

void loop() {
  // Drain receive FIFO.
  while (can.available()) {
    CANMessage msg;
    can.receive(msg);
    process_can_frame(msg);
  }
}
```

---

## Third-Party Licenses

> Arduino and `Arduino Nano` are trademarks of their respective owners. Use of
> these names does **not** imply any endorsement by the trademark holders.
