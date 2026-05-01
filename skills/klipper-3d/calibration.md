# Klipper Calibration Guide

Source docs:
- https://github.com/Klipper3d/klipper/blob/master/docs/Rotation_Distance.md
- https://github.com/Klipper3d/klipper/blob/master/docs/Probe_Calibrate.md
- https://github.com/Klipper3d/klipper/blob/master/docs/Bed_Mesh.md
- https://github.com/Klipper3d/klipper/blob/master/docs/Resonance_Compensation.md
- https://github.com/Klipper3d/klipper/blob/master/docs/Pressure_Advance.md

## Rotation Distance

The most critical parameter — how far an axis moves per full stepper rotation.

### From Marlin steps_per_mm
```
rotation_distance = full_steps_per_rotation × microsteps / steps_per_mm
```
- `full_steps_per_rotation`: 200 for 1.8° motors, 400 for 0.9° motors
- `microsteps`: typically 16 (check your driver config)

Example: `200 × 16 / 80 = 40`

### From hardware (belts)
```
rotation_distance = belt_pitch_mm × teeth_on_pulley
```
Example: 2mm GT2 belt, 20-tooth pulley → `2 × 20 = 40`

### From hardware (lead screws)
```
rotation_distance = screw_pitch × number_of_threads
```
| Lead screw | rotation_distance |
|------------|------------------|
| T8 (2mm pitch, 4 threads) | `8` |
| T8 (2mm pitch, 2 threads) | `4` |
| M5 threaded rod | `0.8` |
| M6 threaded rod | `1.0` |
| M8 threaded rod | `1.25` |

### Common values
| Drive | rotation_distance |
|-------|------------------|
| GT2 belt, 20-tooth | `40` |
| GT2 belt, 16-tooth | `32` |
| BMG extruder (with gear_ratio) | `22.6789511` + `gear_ratio: 50:17` |
| Orbiter v2.0 | `4.637` + `gear_ratio: 7.5:1` |
| Titan extruder | `23.13` |

### Gearbox (`gear_ratio`)
```ini
[extruder]
rotation_distance: 22.6789511
gear_ratio: 50:17            # BMG
# or: gear_ratio: 57:11      # 5.18:1 planetary
# or: gear_ratio: 5:1, 80:16 # multiple stages
```

### Fine-Tune Extruder (Measure and Trim)

**Do not use this method for X/Y/Z axes — hardware calculation is more accurate for those.**

1. Heat nozzle to print temperature
2. Mark filament ~70mm from extruder intake with a marker
3. Measure exact distance from mark to intake: `<initial_mark_distance>`
4. Extrude slowly: `G91` then `G1 E50 F60` (use slow rate to avoid pressure error)
5. Re-measure distance from mark to intake: `<subsequent_mark_distance>`
6. `actual = initial - subsequent`
7. `new_rotation_distance = old_rotation_distance × actual / 50`
8. Repeat if actual differs from 50mm by more than ~2mm

## Z Offset (Probe Calibrate)

1. Home: `G28`
2. Start calibration: `PROBE_CALIBRATE`
3. Use paper test — move nozzle down until paper has light friction:
   ```
   TESTZ Z=-0.1    # move down 0.1mm
   TESTZ Z=+0.05   # move up 0.05mm
   ```
4. Accept position: `ACCEPT`
5. Save: `SAVE_CONFIG`

**Baby-step during print** (no restart needed):
```
SET_GCODE_OFFSET Z_ADJUST=-0.05  # lower nozzle into bed
SET_GCODE_OFFSET Z_ADJUST=+0.05  # raise nozzle from bed
```
To make permanent, add the net adjustment to `z_offset` in `[bltouch]`/`[probe]` and `SAVE_CONFIG`.

## Bed Leveling

### Manual bed screws
```
G28
SCREWS_TILT_CALCULATE
```
Shows which direction and how much to turn each screw. Repeat until all <0.05mm. Run `BED_SCREWS_ADJUST` for interactive paper-test version.

### Auto bed mesh
```
G28
BED_MESH_CALIBRATE
SAVE_CONFIG
```

Load in `PRINT_START` macro every print:
```
BED_MESH_CALIBRATE
# or load a saved profile:
BED_MESH_PROFILE LOAD=default
```

Use `ADAPTIVE=1` to probe only the area being printed (requires `[exclude_object]`):
```
BED_MESH_CALIBRATE ADAPTIVE=1
```

### Gantry leveling (CoreXY with multiple Z)
```
G28
QUAD_GANTRY_LEVEL    # 4 independent Z motors
# or
Z_TILT_ADJUST        # 2 independent Z motors
SAVE_CONFIG
```

## PID Tuning

Hotend:
```
PID_CALIBRATE HEATER=extruder TARGET=200
SAVE_CONFIG
```

Bed:
```
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```

Tune at the temperature you actually print at. Multiple extruders: `HEATER=extruder1`.

## Input Shaping (Resonance Compensation)

Reduces ghosting/ringing artifacts. Requires ADXL345 accelerometer.

### Config
```ini
[adxl345]
cs_pin: rpi:None    # Raspberry Pi native SPI
spi_bus: spi0

[resonance_tester]
accel_chip: adxl345
probe_points: 117, 117, 20
```

### Measure and auto-calibrate
```
G28
SHAPER_CALIBRATE        # measures both axes, picks best shaper
SAVE_CONFIG
```

Or measure manually and analyze output to pick values:
```
TEST_RESONANCES AXIS=X
TEST_RESONANCES AXIS=Y
# Generates CSV in /tmp/ — analyze with Klipper scripts or web UI
```

### Manual config (without accelerometer)
Print a ringing test and measure spacing visually, or use community-suggested defaults:
```ini
[input_shaper]
shaper_freq_x: 45.0
shaper_freq_y: 45.0
shaper_type: mzv
```

### Shaper type tradeoffs

| Shaper | Vibration reduction | Max accel | Use when |
|--------|-------------------|-----------|----------|
| `zv` | Moderate | Highest | Rigid printer, low ringing |
| `mzv` | Good | High | **Default choice** |
| `ei` | Better | Medium | Moderate ringing |
| `2hump_ei` | Best | Lower | Strong ringing |
| `3hump_ei` | Best | Lowest | Very strong ringing |

## Pressure Advance

Accounts for extruder pressure lag, reducing corner blobs and ooze.

### Tuning tower method (recommended)

1. Use high speed (100mm/s), coarse layer height (~75% nozzle diameter), zero infill
2. Disable dynamic acceleration control and scarf seams in slicer
3. Print `docs/prints/square_tower.stl` from Klipper repo
4. Before printing, send:
   ```
   SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=1 ACCEL=500
   TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.005
   # For long bowden: FACTOR=.020
   ```
5. Inspect tower — find height with sharpest corners, no blobs
6. Calculate: `pressure_advance = START + measured_height × FACTOR`
   - Example: `0 + 12.90 × .005 = 0.0645`

### Set in config
```ini
[extruder]
pressure_advance: 0.05          # typical range: 0.05–0.2 direct, 0.3–1.0 bowden
pressure_advance_smooth_time: 0.040  # default, usually no need to change
```

### Notes
- Tune per filament type — different pigments/brands need different values
- Tune after setting `rotation_distance` and nozzle temperature first
- If PA > ~0.200 causes extruder skipping, reduce acceleration instead
- Values >1.0 rarely help — disable PA if no improvement seen up to 1.0

## TMC Current Tuning

Start at 60–70% of motor's rated RMS current:
```ini
[tmc2209 stepper_x]
run_current: 0.800    # most NEMA17 (1A rated): 0.6–0.85A
```

Check for overheating after long prints. If losing steps, increase current (max ~85% rated).

Override at runtime for testing:
```
SET_TMC_CURRENT STEPPER=stepper_x CURRENT=0.9
```

Diagnose driver state:
```
DUMP_TMC STEPPER=stepper_x
```
