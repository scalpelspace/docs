---
title: Momentum
layout: default
parent: Products
nav_order: 1
---

# Momentum: GNSS, 9-DOF IMU, Barometer Sensor Shield

Product codename: `momentum`.

[Firmware](https://github.com/scalpelspace/momentum) | [Driver (SPI/CAN)](https://github.com/scalpelspace/momentum_driver) | [Arduino Library (SPI)](https://github.com/scalpelspace/scalpelspace_momentum) | [Arduino Library (CAN)](https://github.com/scalpelspace/scalpelspace_bus)

---

A Uno-style footprint sensor integration board that bridges an SPI or CAN
host to sensor telemetry. It combines a 9-DOF IMU, multi-constellation GNSS,
and a barometric pressure sensor on a single board, allowing rapid
integration of a full sensor suite into any project with an SPI or CAN bus
host controller.

**Key features:**

- **9-DOF Inertial Measurement Unit (IMU)** via the BNO086.
- **Multi-constellation GNSS** via the SAM-M10Q module.
- **24-bit absolute barometric pressure** via the BMP390.
- **SPI or Classic CAN** host interface for sensor telemetry.
- Firmware update with the onboard **USB-C serial interface** via the CP2102N.
- Addressable **RGB LED** for status display.
- Dual 5 V source via USB-C or supply pin, with automatic power switching.
- Jumper selectable **5 V or 3.3 V logic level** for the SPI host interface.

---

## Getting Started

### SPI Interface with Arduino Library

1. Check for pin conflicts on the SPI `CS` pin (pin 10). See
   [Momentum - Hardware, SPI Interface]({% link products/momentum/hardware.md%}#27-spi-interface)
   for details on swapping to the alternative pin.
2. Set the logic level jumper to match your SPI host. The
   **_default logic level is 5 V_**.
3. Connect the board as a shield to the host.
    - An amber LED lights up to show the 5 V supply pin is powered.
    - The RGB LED lights up on initialization.
4. Flash the example firmware using the Arduino library and confirm the sensor
   data link.

### Advanced: Classic CAN Interface

1. Connect the CAN side to your bus (`GND`, CAN high, CAN low).
2. Power the board with either the 5 V pin (priority) or USB-C.
    - An amber LED lights up when the 5 V supply pin is powered.
    - The RGB LED lights up on initialization.
3. Implement the CAN bus signal decoding. See
   [Momentum - Examples, CAN Integration]({% link products/momentum/examples.md %}).
