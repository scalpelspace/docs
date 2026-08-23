---
title: SPI-CAN 40-pin Hat
layout: default
parent: Products
nav_order: 3
---

# SPI-CAN 40-pin Hat

Product codename: `can_hat_40`.

---

A 40-pin (2x20) GPIO hat that adds a CAN bus interface to single-board computers
such as the Raspberry Pi series. It seats directly on the standard 40-pin header
and bridges the host's SPI bus to a CAN transceiver, so any compatible SBC can
talk CAN without an integrated controller.

**Key features:**

- Standard **40-pin (2x20) GPIO header** for direct mounting on Raspberry Pi and
  compatible SBCs.
- **SPI** host interface for Classic CAN and CAN FD via the MCP2518FD
  controller.
- Switch selectable **bus termination** (split 2x 60 Ohm + centre-tap
  capacitor).
- Jumper selectable **silent mode** (listen-only) for bus monitoring.
- Operates at fixed **3.3 V logic**.

---

## Getting Started

1. Power down the host and seat the hat on the 40-pin GPIO header, aligning pin
   1 to pin 1.
2. Set bus termination on (`120` mark) or off (`NC` mark) depending on where the
   board sits on the bus (enable 120 Ohm only at the physical ends of the bus).
3. Connect the 3-pin CAN side to your bus (`GND`, CAN high, CAN low).
4. Power on the host and follow the Linux setup to enable SPI, load the
   MCP2518FD overlay, and bring up the CAN interface.

- By default, the hat uses the host's SPI0 bus (CE0) with the controller
  interrupt on GPIO22, which matches the device-tree overlay shown in the
  examples.
