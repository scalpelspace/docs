---
title: Getting Started
parent: SPI-CAN 40-pin Hat
grand_parent: Products
nav_order: 1
applies_to: [ rel_a ]
---

# SPI-CAN 40-pin Hat: Getting Started

<p class="ds-applies">Applies to: Rel A</p>

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
