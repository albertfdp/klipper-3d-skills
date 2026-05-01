# Klipper 3D Skills

Agent skills for working with [Klipper 3D printer firmware](https://www.klipper3d.org/).

## Skills

### klipper-3d

Reference skill for Klipper firmware covering:

- **Configuration** (`printer.cfg`) — config blocks, syntax, and common parameters
- **G-codes** — standard and extended Klipper commands with macro templating
- **Calibration** — rotation distance, Z offset, bed leveling, input shaping, pressure advance
- **Troubleshooting** — MCU errors, TMC issues, homing problems, firmware flashing

## Installation

Inside a Claude Code session, add this repo as a marketplace:

```
/plugin marketplace add /path/to/klipper-3d-skills
```

Then install the plugin from the Discover tab, or:

```
/plugin install klipper-3d@klipper-3d-skills
```
