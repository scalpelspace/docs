---
title: Getting Started
parent: SPI-CAN Breakout
grand_parent: Products
nav_order: 1
applies_to: [ rel_a ]
---

# SPI-CAN Breakout: Getting Started

<p class="ds-applies">Applies to: Rel A</p>

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
