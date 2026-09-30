---
title: ScalpelSpace CAN Protocol
layout: default
parent: Protocols
nav_order: 1
---

# ScalpelSpace CAN Protocol

Reference
implementation: [can_driver](https://github.com/scalpelspace/can_driver)

---

<details markdown="1">
  <summary>Table of Contents</summary>

<!-- TOC -->
* [ScalpelSpace CAN Protocol](#scalpelspace-can-protocol)
  * [1 Overview](#1-overview)
    * [1.1 Design Goals](#11-design-goals)
    * [1.2 Recommended Development Interfaces](#12-recommended-development-interfaces)
  * [2 CAN ID Scheme](#2-can-id-scheme)
    * [2.1 Why Classic CAN, 11-bit IDs](#21-why-classic-can-11-bit-ids)
    * [2.2 Why Message Type First](#22-why-message-type-first)
    * [2.3 Reserved Values](#23-reserved-values)
  * [3 Node ID Allocation](#3-node-id-allocation)
    * [3.1 Handshake](#31-handshake)
    * [3.2 Design Decisions](#32-design-decisions)
  * [4 DBC as the Source of Truth](#4-dbc-as-the-source-of-truth)
    * [4.1 Per-Device DBCs](#41-per-device-dbcs)
    * [4.2 Code Generation](#42-code-generation)
    * [4.3 System-Level Merged DBC](#43-system-level-merged-dbc)
  * [5 Known Limits and Trade-offs](#5-known-limits-and-trade-offs)
  * [6 Read More](#6-read-more)
<!-- TOC -->

</details>

---

## 1 Overview

`can_driver` is the central protocol and message scheme behind every
ScalpelSpace CAN bus product. It defines:

1. **A CAN ID scheme** that encodes both *what* a message is and *who* sent it.
2. **A node ID allocation protocol** so devices can join a network without
   manual addressing.
3. **A DBC-first workflow** where each device's messages are defined once in a
   DBC file, then generated into C code and merged into a single system-level
   DBC.

This page explains why the protocol is designed the way it is. For exact message
layouts, API signatures and script usage, see the
[can_driver README](https://github.com/scalpelspace/can_driver#readme).

### 1.1 Design Goals

- **Composable:** Multiple ScalpelSpace devices, including several copies of the
  same device, can share one bus with no ID conflicts.
- **Plug-and-play:** Devices get their addresses from the network at startup. No
  DIP switches, jumpers or per-unit firmware builds.
- **Portable:** The driver is plain C and targets low-cost microcontrollers with
  a classic CAN peripheral or an SPI-CAN controller.
- **Tool friendly:** Every message on the bus is described by a standard DBC
  file, so off-the-shelf CAN tools can decode traffic directly.
- **Predictable:** Bus priority follows from message type, not from which device
  happened to get which address.

### 1.2 Recommended Development Interfaces

Most developers do not need to work with `can_driver` directly. ScalpelSpace
provides higher-level libraries that wrap the protocol for common platforms:

| Interface                                                              | Platform                        |
|------------------------------------------------------------------------|---------------------------------|
| [scalpelspace_bus](https://github.com/scalpelspace/scalpelspace_bus)   | Arduino (microcontroller hosts) |
| [scalpelspace_ros2](https://github.com/scalpelspace/scalpelspace_ros2) | ROS 2 (Linux / robotics hosts)  |

These are the **recommended starting point for rapid prototyping**. They handle
node ID allocation, CAN ID packing and signal decoding, and expose each device
as a simple typed interface (e.g. "move this motor", "read this IMU").

They are intended as a **generalized interface across platforms**. A project
often has developers working at different levels, for example one person on an
Arduino-based controller and another on a ROS 2 robot stack. Both interfaces sit
on the same protocol and the same per-device DBCs, so each developer uses the
tooling native to their platform while talking to the same devices in the same
way.

Use `can_driver` directly when building new device firmware, porting to a
platform without an existing interface, or when full control over the bus is
required.

---

## 2 CAN ID Scheme

The 11-bit standard CAN ID is split into two fields:

| Field        | Bits  | Width | Range | Purpose                              |
|--------------|:-----:|:-----:|:-----:|--------------------------------------|
| `message_id` | 10..5 |   6   | 0..63 | Type of data (what the message is).  |
| `node_id`    | 4..0  |   5   | 0..31 | Device on the network (who sent it). |

```
CAN ID = (message_id << 5) | node_id
```

### 2.1 Why Classic CAN, 11-bit IDs

Classic CAN with standard 11-bit IDs is supported by practically every CAN
peripheral, transceiver and analysis tool. Choosing the lowest common
denominator keeps ScalpelSpace devices compatible with low-end MCUs and existing
buses, and keeps frames short. Extended (29-bit) IDs are not used.

The trade-off is a small ID space, so the scheme spends its bits carefully (see
[5 Known Limits and Trade-offs](#5-known-limits-and-trade-offs)).

### 2.2 Why Message Type First

CAN arbitration gives priority to the lowest numeric ID. With `message_id` in
the most significant bits:

- **Priority is set by the message type.** Every instance of a given message
  type has the same priority, whichever device sends it. Assigning node IDs
  cannot change which data wins the bus.
- **Each message type takes a contiguous block of 32 IDs.** Hardware acceptance
  filters can match a whole message type across all devices with a single mask,
  or one device's messages by masking on the low 5 bits.
- **A device's DBC is written once.** Each device defines its messages with
  `node_id = 0`. Its real CAN IDs are that base value with the assigned
  `node_id` added in the low bits, so identical devices share one firmware image
  and one DBC.

### 2.3 Reserved Values

| Value               | Meaning                                               |
|---------------------|-------------------------------------------------------|
| `node_id` 0         | Unassigned: a device that has not been allocated yet. |
| `node_id` 31        | Broadcast: addressed to every device.                 |
| `message_id` 1..55  | Available for application messages.                   |
| `message_id` 56..59 | Node ID allocation protocol.                          |
| `message_id` 60..63 | Reserved for future allocation use.                   |

This leaves **30 addressable devices** per network (node IDs 1..30).

The allocation messages sit at the lowest priority end of the ID space. They
only run during startup and must never delay application data.

---

## 3 Node ID Allocation

### 3.1 Handshake

One device on the network is the **allocator**. Every other device is an
**allocatee**. At startup the allocator runs a four-step handshake:

```
Allocator                          Allocatee(s)
    |                                   |
    |--- DISCOVER (broadcast) --------> |
    | <---------- ADVERTISE (each) -----|  UID hash, current node_id, alloc_mode
    |--- ASSIGN (broadcast, per UID) -> |
    | <-------------- ACK (per node) ---|
```

### 3.2 Design Decisions

- **Identity comes from hardware.** Each device identifies itself with a 48-bit
  hash derived from a hardware unique ID (for example, the MCU die ID). This ID
  is stable across resets and unique per unit without any factory provisioning
  step.
- **Sessions guard against stale traffic.** Each discovery run carries an
  incrementing `session_id`. Late replies from an earlier run are dropped
  instead of corrupting the current one.
- **Every device answers, including assigned ones.** Devices that already hold a
  node ID still advertise it. The allocator therefore sees which IDs are in use
  and never hands out a duplicate. This also makes it safe to restart discovery
  at any time.
- **Fixed-ID devices can opt out.** A device can advertise itself as *not
  reassignable*. The allocator reserves its ID and skips it. This supports
  devices with hardcoded addresses, or deployments that need a fixed mapping,
  while the rest of the bus stays dynamic.
- **Assignment policy is pluggable.** The default assigns the lowest free IDs in
  the order devices reply. Where a stable mapping matters (for example, "this
  IMU is always node 2"), deterministic strategies such as sorting by UID or a
  UID lookup table can be used, or a custom one can be supplied.
- **Timing is left to the application.** The protocol has no built-in timeouts.
  The host decides when discovery ends and when to retry. That keeps the core
  state machine small and free of timers, but the integrator has to provide a
  watchdog or deadline.
- **Backwards compatible encoding.** New fields such as `alloc_mode` are defined
  so that a zero value keeps the old behaviour. Older firmware that sent those
  bytes as zeroed reserved fields still decodes correctly.

---

## 4 DBC as the Source of Truth

### 4.1 Per-Device DBCs

Each ScalpelSpace device repository contains a DBC file at its root that defines
that device's messages and signals. This DBC is the **single source of truth**:
firmware, host drivers and analysis tools all derive from it.

### 4.2 Code Generation

A generator script converts a DBC into C message and signal definitions, which
the driver uses to pack and unpack frames. This replaces hand-written bit
shifting, which is easy to get wrong, and keeps firmware in step with the DBC.

### 4.3 System-Level Merged DBC

A second script builds a single DBC for a whole system. Given a list of device
repositories (or local DBC files) and the node ID each one should use, it:

- Patches each device's CAN IDs to its assigned node ID.
- Suffixes node and message names (e.g. `MOMENTUM_02`) so multiple copies of one
  device stay distinct.
- Leaves allocation protocol messages unchanged, since they are shared by all
  devices.
- Warns on duplicate node IDs or colliding CAN IDs.

The result can be loaded into standard CAN tooling to decode a live bus with
several ScalpelSpace products on it.

---

## 5 Known Limits and Trade-offs

| Limit                             | Reasoning                                                              |
|-----------------------------------|------------------------------------------------------------------------|
| 30 devices per network            | 5 node bits, 2 values reserved. Enough for target systems.             |
| 55 application message types      | 6 message bits, the upper values reserved for allocation.              |
| Signals limited to 32 bits        | Pack/unpack stays within `uint32_t` for low-end MCUs.                  |
| Classic CAN only, no extended IDs | Maximum hardware and tool compatibility.                               |
| Single allocator per network      | Keeps the protocol simple. Arbitrate the role in the app if needed.    |
| No built-in timeouts              | Keeps the state machine timer-free. The integrator supplies deadlines. |

---

## 6 Read More

The [can_driver README](https://github.com/scalpelspace/can_driver#readme) is
the technical reference for:

- The driver API and DBC-to-C code generation.
- Exact CAN IDs, payload layouts and `alloc_mode` values for the allocation
  protocol.
- Implementer notes for the allocator and allocatee state machines.
- Merged DBC script usage and the repo file format.

Related developer libraries are built on this protocol:

- [scalpelspace_bus](https://github.com/scalpelspace/scalpelspace_bus): Arduino
  development interface (recommended for rapid prototyping).
- [scalpelspace_ros2](https://github.com/scalpelspace/scalpelspace_ros2): ROS 2
  development interface (recommended for rapid prototyping).
