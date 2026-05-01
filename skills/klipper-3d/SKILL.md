---
name: klipper-3d
description: Use when working with Klipper 3D printer firmware — editing printer.cfg, writing gcode_macro blocks, calibrating motion systems (rotation_distance, Z offset, bed mesh), tuning input shaping or pressure advance, diagnosing MCU/TMC/serial errors, flashing firmware to microcontrollers, or integrating with OctoPrint, Mainsail, or Fluidd.
---

# Klipper 3D Printer Firmware

## Overview

Klipper runs on a general-purpose computer (Raspberry Pi) paired with one or more microcontrollers. The host handles motion planning in Python; MCUs execute step timing with ≤25µs precision. Config lives in `printer.cfg` — no reflash needed for config changes. `SAVE_CONFIG` writes calibration data back to the file.

**Docs:** https://www.klipper3d.org/ | **Source:** https://github.com/Klipper3d/klipper/

## Reference Files

| File | Contents |
|------|----------|
| `config-reference.md` | Config block syntax and parameters |
| `gcodes.md` | Standard G-codes and extended Klipper commands |
| `calibration.md` | Rotation distance, Z offset, bed mesh, input shaping, pressure advance |
| `troubleshooting.md` | MCU errors, TMC issues, homing problems, firmware flashing |

## Architecture

| Layer | Role | Tech |
|-------|------|------|
| Host | Motion planning, G-code parsing | Python on Raspberry Pi / SBC |
| MCU | Step timing, ADC, GPIO | C on ATmega / STM32 / RP2040 |
| Interface | User interaction | OctoPrint / Mainsail / Fluidd |

## Supported Kinematics

`cartesian`, `corexy`, `corexz`, `hybrid_corexy`, `hybrid_corexz`, `generic_cartesian`, `rotary_delta`, `delta`, `deltesian`, `polar`, `winch`, `none`

## Essential Commands

```
RESTART              # Reload printer.cfg (soft — no MCU reset)
FIRMWARE_RESTART     # Reset MCU communication
STATUS               # Current printer status
QUERY_ENDSTOPS       # Show endstop states (open/triggered)
GET_POSITION         # Current stepper and toolhead positions
SAVE_CONFIG          # Persist calibration data to printer.cfg
```

## First-Time Setup Checklist

1. Install Klipper on Raspberry Pi (use KIAUH or official install script)
2. Compile and flash MCU: `make menuconfig && make && make flash`
3. Find serial port: `ls /dev/serial/by-id/*` — use this stable path in `[mcu]`
4. Configure `printer.cfg`: `[mcu]`, `[printer]`, steppers, extruder, heater_bed
5. Run `Config_checks.md` steps, then home, calibrate, `SAVE_CONFIG`

## Pin Notation

- `PA4` — standard MCU hardware pin name
- `!PA4` — inverted (active low)
- `^PA4` — pull-up resistor enabled
- `~PA4` — pull-down resistor enabled
- `rpi:gpio17` — Raspberry Pi GPIO pin
- `extra_mcu:PA4` — pin on a secondary MCU named `extra_mcu`
