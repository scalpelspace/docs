---
title: Getting Started
parent: MC Stepper
grand_parent: Products
nav_order: 1
applies_to: [ rel_a ]
---

# MC Stepper: Getting Started

<p class="ds-applies">Applies to: Rel A</p>

## CAN interface with Arduino library

The recommended starting point for most users.

1. Connect a stepper motor to the `Motor` connector (`A+`, `A-`, `B+`, `B-`).
2. Connect the `Power and CAN` connector to your bus (CAN high, CAN low, ground)
   and supply (VIN, **_29 V maximum_**).
    - The RGB LED lights up on initialization.
3. Ensure the bus is terminated at both physical ends. On-board termination is
   not populated by default, see the [datasheet]({% link
   products/mc_stepper/rel-a/index.md%}#detailed-description).
4. Flash the `mc_stepper_basic` example from the
   [ScalpelSpace Bus](https://github.com/scalpelspace/scalpelspace_bus) Arduino
   library to a host with a CAN interface (e.g. the
   [SPI-CAN Breakout]({% link products/can_hat_mini/index.md %})) and confirm
   the motor moves.

The library's `McStepper` device handle covers ScalpelSpace's unified CAN ID
allocation scheme and device control.

## Advanced: Classic CAN interface

For host systems integrating the CAN bus protocol directly.

1. Wire power, CAN and the motor as above.
2. Assign a node ID via the ScalpelSpace node ID allocation protocol. See the
   [ScalpelSpace CAN bus protocol]({% link protocols/can_driver/index.md %}).
3. Implement the CAN bus message encoding/decoding. The
   [mc_stepper_driver](https://github.com/scalpelspace/mc_stepper_driver)
   repository holds the **source-of-truth** CAN bus DBC definition
   (`can_mc_stepper.dbc`), message table and C drivers.

## Advanced: UART serial interface

For development, debug and configuration from a desktop computer.

1. Connect a 3.3 V USB-to-UART adapter to the `UART breakout` connector.
2. Open a serial terminal at 115200 bps and run `help` for the command list.
   Commands are newline (`\n`) terminated.

The full command reference is maintained with the
[mc_stepper](https://github.com/scalpelspace/mc_stepper) firmware.
