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

### Via [skills.sh](https://skills.sh/)

```bash
npx skillsadd <your-github-username>/klipper-3d-skills/klipper-3d
```

### Via Claude Code plugin system

Inside a Claude Code session, add this repo as a marketplace:

```
/plugin marketplace add /path/to/klipper-3d-skills
```

Then install the plugin from the Discover tab, or:

```
/plugin install klipper-3d@klipper-3d-skills
```
