---
title: SPI-CAN Breakout
layout: default
parent: Products
nav_order: 2
---

# SPI-CAN Breakout

Product codename: `can_hat_mini`.

[Arduino Library (CAN)](https://github.com/scalpelspace/scalpelspace_bus)

---

A small-footprint, breadboard-friendly breakout board that bridges an SPI host
to a CAN bus. It exposes a standard SPI interface on one side and a CAN
transceiver on the other, letting any microcontroller with SPI talk CAN without
an integrated controller.

**Key features:**

- Miniature **breadboard-compatible** footprint for quick prototyping.
- **SPI** host interface for Classic CAN and CAN FD via the MCP2518FD
  controller.
- Switch selectable **bus termination** (split 2x 60 Ohm + centre-tap
  capacitor).
- Jumper selectable 5 V or 3.3 V **logic level** for the host interface.
- Jumper selectable **silent mode** (listen-only) for bus monitoring.
- 1x **M2.5 mounting hole**.

---

## Getting Started

1. Wire the 8-pin header to your host:
    1. `5 V` power.
    2. `GND` ground (second pin optional).
    3. SPI lines (`COPI`, `CIPO`, `CS` and `SCK`).
    4. Optionally, but recommended: interrupt `INT` pin.
2. Set the logic level jumper to match your host, the **_default logic level is
   5 V_**.
3. Set bus termination on (`120` mark) or (`NC` mark) off depending on where the
   board sits on the bus (enable 120 Ohm only at the physical ends of the bus).
4. Connect the 3-pin CAN side to your bus (`GND`, CAN high, CAN low).
5. Flash the example firmware or and confirm the link.
6. Optional: For larger projects using ScalpelSpace products, use
   the [ScalpelSpace Bus](https://github.com/scalpelspace/scalpelspace_bus)
   Arduino Library for ScalpelSpace's unified CAN ID allocation scheme and
   device control.
