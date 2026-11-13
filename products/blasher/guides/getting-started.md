---
title: Getting Started
parent: Blasher
grand_parent: Products
nav_order: 1
applies_to: [ rel_a ]
---

# Blasher: Getting Started

<p class="ds-applies">Applies to: Rel A</p>

## Flash firmware with the web app

1. Install Silicon Labs's
   [CP210x USB to UART Bridge VCP Drivers](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers)
   if the board does not show up as a serial port.
2. Plug the Blasher into USB and connect the 5-pin `UART breakout` to the
   target's `UART breakout`.
    - The target must be powered separately, Blasher does not power it.
3. Open [blasher.scalpelspace.com](https://blasher.scalpelspace.com) and click
   **Connect**.
4. Drop the firmware `.bin` image into the **Firmware image** box.
5. Click **Flash firmware**.

## Serial monitor

With the Blasher connected, switch the web app to the **Serial monitor** tab
for a simple serial terminal to the target.
