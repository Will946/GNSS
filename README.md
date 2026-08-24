# ZED-F9P RTK GPS Receiver

Board for an autonomous cart that follows the edge of a lawn and sprays weed killer along the border. Goal is centimeter level positioning using RTK corrections so the cart can hold a precise line along a mapped boundary. Prototyping this board is the only part of the project built so far.

## Status

Not currently in use on the cart. Prototyping this board cost way more than expected, and the rest of the cart, drive base, sprayer, controller, was never built as a result. Right now this is just the GNSS board sitting on its own.

## Overview

The ZED-F9P decodes both L1 and L2 bands at the same time, which speeds up how fast it locks a fixed RTK solution and helps under partial sky obstruction, useful near fences, trees, or the edge of a house. With a clear view of sky and a valid RTCM3 correction stream, this board is capable of about 10 mm 3D accuracy.

Two things it needs to actually hit that accuracy:
- Clear sky view. RTK does not work well indoors or under heavy tree cover.
- A correction stream, either from a second ZED-F9P set up as a base station, or from an NTRIP correction service.

## Features

- u-blox ZED-F9P module, L1/L2 dual band, multi constellation
- Rover mode for the cart, with base station mode available if running your own fixed reference
- About 10 mm 3D accuracy with valid RTK corrections
- Update rates up to 20 Hz in high precision RTK mode
- Five simultaneous communication interfaces, see below
- U.FL antenna connector for a low profile L1/L2 active antenna
- Rechargeable backup cell that keeps module config and almanac data through power off, so warm starts are faster. Holds state for roughly two weeks unpowered
- Status LEDs for power, PPS, and fix state

## Communication Interfaces

| Interface | Description |
|---|---|
| USB-C | Enumerates as a standard COM port, carries power and full UBX/NMEA data |
| UART1 | 3.3V TTL, general purpose NMEA/UBX I/O |
| UART2 | 3.3V TTL, dedicated to RTCM3 correction data. Input on a rover, output on a base |
| I2C DDC | u-blox's I2C implementation, shares physical pins with SPI |
| SPI | Alternate to I2C on the same pins, must be explicitly selected |

Interface select jumper: I2C and SPI share pins, so only one is active at a time.
- Jumper open, default: I2C/DDC enabled, UART1 and I2C both active
- Jumper closed: SPI enabled, this disables UART1 and I2C

Set this before wiring your host MCU. An unexpected jumper state is the most common reason I2C or SPI fails to come up.

## Hardware

| Item | Detail |
|---|---|
| GNSS module | u-blox ZED-F9P |
| Antenna connector | U.FL, active antenna with bias tee power |
| Antenna requirement | L1/L2 dual band active GNSS antenna |
| Logic level | 3.3V |
| Backup power | Rechargeable cell, retains config/almanac about 2 weeks unpowered |
| Board interconnect | 0.1" breadboard compatible headers |
| Input power | 5V or 3.3V |

## Pinout

| Pin | Function | Notes |
|---|---|---|
| 5V / 3V3 | Power in | Select per your supply |
| GND | Ground | |
| TX1 / RX1 | UART1, NMEA/UBX | 3.3V TTL, general data |
| TX2 / RX2 | UART2, RTCM3 | 3.3V TTL, correction data in/out |
| SDA / SCL | I2C, DDC | Shares pins with SPI, see jumper |
| MISO / MOSI / SCK / CS | SPI | Enabled only when jumper closed |
| PPS | Pulse per second | Precision timing output |
| SAFEBOOT | Firmware recovery | Pull low during boot to enter recovery |

## Getting Started

1. Connect an active L1/L2 GNSS antenna to the U.FL connector.
2. Set the I2C/SPI select jumper to match your intended interface. Default is open, I2C.
3. Power the board from 5V or 3.3V.
4. Connect to the module with a UBX capable configuration tool such as u-center over USB or UART1 to check satellite lock and fix status.
5. Once a valid 3D fix shows up, move to RTK setup below.

## RTK Setup

As a rover on the cart:
- Feed RTCM3 correction messages into UART2, or via I2C/SPI if you'd rather.
- Once corrections are flowing and satellite geometry cooperates, fix state moves from no fix to 3D to RTK float to RTK fixed.
- Fixed RTK is what you want before trusting the position for holding a line along the lawn edge.

If running your own base station instead of NTRIP:
- Configure survey in mode with a target accuracy and minimum observation time, for example 60 seconds and 2 meters.
- Once survey in completes, the module holds a fixed reference position and starts outputting RTCM3 corrections on UART2.
- Typical RTCM3 messages to enable: 1005 for station coordinates, 1077, 1087, 1097, 1127 for per constellation observations, 1230 for GLONASS bias.

Correction delivery options:
- A radio telemetry link, LoRa or 900 MHz, between a yard mounted base and the cart
- An NTRIP client and caster over the internet for networked corrections
