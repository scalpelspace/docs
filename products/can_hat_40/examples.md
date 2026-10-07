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
   gpio=22=ip,pu
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

## Automatically Starting the CAN Interface

The `ip link` commands above do not persist across reboots. Use **one** of the
methods below to bring `can0` up automatically at boot. Both assume steps 1-3 of
the setup above are complete and `can0` shows up in `ip link show`.

### Option A: systemd Service (Recommended)

A small systemd unit that runs the same `ip link` commands whenever `can0`
appears. This works on any systemd-based distro regardless of which network
manager is in use.

1. Create the service file.
   ```shell
   sudo nano /etc/systemd/system/can0.service
   ```
2. Add the following contents.
    - For CAN classic (500 kbps, 87.5% sample point, bus error reporting):
   ```ini
   [Unit]
   Description=SocketCAN interface can0
   BindsTo=sys-subsystem-net-devices-can0.device
   After=sys-subsystem-net-devices-can0.device

   [Service]
   Type=oneshot
   RemainAfterExit=yes
   ExecStartPre=-/usr/sbin/ip link set can0 down
   ExecStart=/usr/sbin/ip link set can0 type can bitrate 500000 sample-point 0.875 berr-reporting on restart-ms 100
   ExecStart=/usr/sbin/ip link set can0 up
   ExecStop=/usr/sbin/ip link set can0 down

   [Install]
   WantedBy=sys-subsystem-net-devices-can0.device
   ```
    - For CAN FD (1 Mbps nominal, 80% sample point, 4 Mbps data phase, bus error
      reporting), replace the first `ExecStart=` line with:
   ```ini
   ExecStart=/usr/sbin/ip link set can0 type can bitrate 1000000 sample-point 0.80 dbitrate 4000000 dsample-point 0.80 fd on berr-reporting on restart-ms 100
   ```
3. Enable and start the service.
   ```shell
   sudo systemctl daemon-reload
   sudo systemctl enable --now can0.service
   ```

Notes:

- `WantedBy=sys-subsystem-net-devices-can0.device` starts the service as soon as
  the kernel creates `can0`, so it does not race the MCP251XFD driver during
  boot. `BindsTo=` stops it again if the device disappears.
- `restart-ms 100` automatically recovers the controller 100 ms after a bus-off
  event. Without it, the interface stays down after bus-off until it is manually
  restarted, which is usually undesirable for an unattended system.
- To change the bitrate later, edit the file, then run
  `sudo systemctl daemon-reload && sudo systemctl restart can0.service`.

### Option B: systemd-networkd

If the system already uses `systemd-networkd` (the default on Ubuntu Server via
netplan), CAN settings can be declared in a `.network` file instead.

1. Create the network file.
   ```shell
   sudo nano /etc/systemd/network/80-can0.network
   ```
2. Add the following contents.
    - For CAN classic (500 kbps, 87.5% sample point, bus error reporting):
   ```ini
   [Match]
   Name=can0

   [Link]
   RequiredForOnline=no

   [CAN]
   BitRate=500000
   SamplePoint=87.5%
   BusErrorReporting=yes
   RestartSec=100ms
   ```
    - For CAN FD (1 Mbps nominal, 80% sample point, 4 Mbps data phase, bus error
      reporting), replace the `[CAN]` section with:
   ```ini
   [CAN]
   BitRate=1000000
   SamplePoint=80%
   DataBitRate=4000000
   DataSamplePoint=80%
   FDMode=yes
   BusErrorReporting=yes
   RestartSec=100ms
   ```
3. Enable `systemd-networkd` (if not already enabled) and apply the config.
   ```shell
   sudo systemctl enable --now systemd-networkd
   sudo networkctl reload
   sudo networkctl reconfigure can0
   ```

Notes:

- `RequiredForOnline=no` prevents `systemd-networkd-wait-online` from delaying
  boot while waiting for the CAN interface.
- On Ubuntu Desktop, NetworkManager does not manage CAN interfaces, so enabling
  `systemd-networkd` alongside it is safe as long as only CAN interfaces are
  matched by files in `/etc/systemd/network/`.

## Verifying CAN Interface

After a reboot, confirm the interface came up with the expected settings:

```shell
ip -details link show can0
```

Look for `state UP`, `can state ERROR-ACTIVE`, and the configured `bitrate`/
`dbitrate`. For Option A systemd services, the service status and logs are
available with:

```shell
systemctl status can0.service
journalctl -u can0.service
```

Then check for traffic with `can-utils`:

```shell
candump can0
```
