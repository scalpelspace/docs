---
title: Momentum - Examples
layout: default
parent: Momentum
grand_parent: Products
nav_order: 1
---

# Example Code

## SPI Interface with Arduino Library

The recommended starting point for most users. See the
[ScalpelSpace Momentum](https://github.com/scalpelspace/scalpelspace_momentum)
Arduino library for a ready-to-run driver and self-contained examples.

## CAN Interface with Arduino Library

For users looking to implement CAN, a good starting point is
the [ScalpelSpace Bus](https://github.com/scalpelspace/scalpelspace_bus) Arduino
Library for ScalpelSpace's unified CAN ID allocation scheme and device control.

## Advanced: Custom SPI Integration and Classic CAN Interface

For host systems integrating the SPI protocol directly or utilizing CAN bus. See
the [momentum_driver](https://github.com/scalpelspace/momentum_driver)
repository for the **source-of-truth** CAN bus DBC definition and SPI protocol
outline.
