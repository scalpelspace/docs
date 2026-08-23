---
title: SPI-CAN 40-pin Hat - Examples
layout: default
parent: SPI-CAN 40-pin Hat
grand_parent: Products
nav_order: 1
---

# Example Linux (Ubuntu) Setup

This setup is for Raspberry Pis running Ubuntu and similar Debian distros.

1. Install MCP251XFD required drivers (example showing [
   `can-utils`](https://github.com/linux-can/can-utils)).
   ```shell
   sudo apt update
   sudo apt install -y can-utils
   ```
2. Enable SPI and add the MCP2518FD overlay.
   ```shell
   sudo nano /boot/firmware/config.txt
   ```
   ```shell
   # Add the following lines to config.txt:
   dtparam=spi=on
   dtoverlay=mcp251xfd,spi0-0,oscillator=40000000,interrupt=22,spimaxfrequency=10000000
   ```
3. Reboot the system.
   ```shell
   sudo reboot
   ```
4. Run CAN.
    - For CAN classic setup (example for 500 kbps, 87.5% sample point, bus error
      reporting):
   ```shell
   sudo ip link set can0 down 2>/dev/null || true
   sudo ip link set can0 type can bitrate 500000 sample-point 0.875 berr-reporting on
   sudo ip link set can0 up
   ```
    - For CAN FD setup (example for 1 Mbps nominal rate, 80% sample point, 4
      Mbps data phase rate, bus error reporting):
   ```shell
   sudo ip link set can0 down
   sudo ip link set can0 type can bitrate 1000000 sample-point 0.80 dbitrate 4000000 dsample-point 0.80 fd on berr-reporting on
   sudo ip link set can0 up
   ```
