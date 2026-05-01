# Klipper Config Reference

Full reference: https://www.klipper3d.org/Config_Reference.html  
Source: https://github.com/Klipper3d/klipper/blob/master/docs/Config_Reference.md

## Required Sections

### [mcu]
```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_...
# Find with: ls /dev/serial/by-id/*
# For CAN bus: canbus_uuid: abc123def456
#              canbus_interface: can0
#baud: 250000  # default, recommended
```

### [printer]
```ini
[printer]
kinematics: cartesian      # See kinematics list in SKILL.md
max_velocity: 300          # mm/s — toolhead max speed
max_accel: 3000            # mm/s²
#minimum_cruise_ratio: 0.5 # min fraction of move at cruise speed
#square_corner_velocity: 5.0  # mm/s through 90° corners

# Cartesian-specific:
max_z_velocity: 5          # KEEP LOW (≤5) to prevent Z screech on Marlin converts
max_z_accel: 100
```

### [stepper_x] / [stepper_y] / [stepper_z]
```ini
[stepper_x]
step_pin: PB9
dir_pin: PC2              # Add ! to invert direction
enable_pin: !PC3
microsteps: 16
rotation_distance: 40     # See calibration.md
full_steps_per_rotation: 200  # 200 for 1.8°, 400 for 0.9° motors
#gear_ratio: 80:16        # if gearbox present, e.g. "5:1" or "57:11, 2:1"
endstop_pin: PA5          # or probe:z_virtual_endstop for Z with probe
position_min: 0
position_max: 235
position_endstop: 0
homing_speed: 50
#homing_retract_dist: 5.0
#second_homing_speed: 25  # defaults to homing_speed/2
```

### [extruder]
```ini
[extruder]
step_pin: PB4
dir_pin: PB3
enable_pin: !PC3
microsteps: 16
rotation_distance: 22.6789511  # calibrate with measure-and-trim method
#gear_ratio: 50:17        # for BMG/Orbiter; omit for direct drive without gears
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PA1
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC5
min_temp: 0
max_temp: 250
#pressure_advance: 0.05   # see calibration.md
#pressure_advance_smooth_time: 0.040
```

### [heater_bed]
```ini
[heater_bed]
heater_pin: PA2
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC4
min_temp: 0
max_temp: 130
```

## Common Sensor Types

| Sensor | Config Name |
|--------|-------------|
| Generic NTC 100K | `NTC 100K MGB18-104F39050L32` |
| EPCOS 100K | `EPCOS 100K B57560G104F` |
| ATC Semitec 104GT | `ATC Semitec 104GT-2` |
| ATC Semitec 104NT | `ATC Semitec 104NT-4-R025H42G` |
| PT100 (via MAX31865) | `MAX31865` |
| PT1000 | `PT1000` |

## Bed Leveling & Probing

### [bltouch]
```ini
[bltouch]
sensor_pin: ^PB1      # ^ = pull-up required
control_pin: PB0
x_offset: -45.0       # physical offset from nozzle (negative = probe is left of nozzle)
y_offset: -10.0
z_offset: 2.0         # calibrate with PROBE_CALIBRATE
#speed: 5.0
#samples: 2
#sample_retract_dist: 3.0
```

### [probe] (generic inductive/optical)
```ini
[probe]
pin: ^PA5
x_offset: 0
y_offset: 25.0
z_offset: 0           # calibrate with PROBE_CALIBRATE
speed: 5.0
samples: 3
sample_retract_dist: 2.0
samples_tolerance: 0.100
```

### [safe_z_home]
```ini
[safe_z_home]
home_xy_position: 117, 117  # XY position to home Z from (center of bed)
speed: 50
z_hop: 10                   # raise Z before XY moves
z_hop_speed: 5
```

### [bed_mesh]
```ini
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 35
mesh_max: 200, 200
probe_count: 5, 5       # 5x5 = 25 probe points
#algorithm: lagrange    # lagrange (default ≤4x4) or bicubic (≥4x4)
```

### [z_tilt] (2 independent Z motors)
```ini
[z_tilt]
z_positions:
    0, 120
    250, 120
points:
    30, 120
    220, 120
speed: 100
horizontal_move_z: 10
retries: 5
retry_tolerance: 0.0075
```

### [quad_gantry_level] (4 Z motors, CoreXY)
```ini
[quad_gantry_level]
gantry_corners:
    -60,-10
    310,320
points:
    50,25
    50,275
    250,275
    250,25
speed: 100
horizontal_move_z: 10
retries: 5
retry_tolerance: 0.0075
max_adjust: 10
```

### [bed_screws] (manual leveling)
```ini
[bed_screws]
screw1: 30, 30
screw1_name: front left
screw2: 200, 30
screw2_name: front right
screw3: 200, 200
screw3_name: rear right
screw4: 30, 200
screw4_name: rear left
```

### [screws_tilt_adjust] (probe-assisted manual leveling)
```ini
[screws_tilt_adjust]
screw1: 75, 35
screw1_name: front left
screw2: 245, 35
screw2_name: front right
screw3: 245, 205
screw3_name: rear right
screw4: 75, 205
screw4_name: rear left
horizontal_move_z: 10
speed: 50
screw_thread: CW-M4    # CW-M3, CW-M4, CW-M5, CCW-M3, CCW-M4, CCW-M5
```

## Input Shaping

### [input_shaper]
```ini
[input_shaper]
shaper_freq_x: 45.0    # Hz — measure with resonance_tester or ADXL345
shaper_freq_y: 45.0
shaper_type: mzv       # zv, mzv, ei, 2hump_ei, 3hump_ei — see calibration.md
```

### [resonance_tester] (ADXL345 required)
```ini
[resonance_tester]
accel_chip: adxl345
probe_points: 117, 117, 20  # XYZ position to measure from

[adxl345]
cs_pin: rpi:None         # Raspberry Pi SPI (GPIO)
spi_bus: spi0
```

## TMC Stepper Drivers

### [tmc2209 stepper_x] (UART mode)
```ini
[tmc2209 stepper_x]
uart_pin: PC11
tx_pin: PC10             # omit if using single-wire UART
uart_address: 0          # 0–3 based on MS1/MS2 pins
run_current: 0.800       # Amps RMS — ~70% of motor rated current
#hold_current: 0.500     # defaults to run_current if omitted
stealthchop_threshold: 999999  # always stealthChop; use 0 for always spreadCycle
```

### [tmc2130 stepper_x] (SPI mode)
```ini
[tmc2130 stepper_x]
cs_pin: PE3
spi_bus: spi1
run_current: 0.800
stealthchop_threshold: 999999
```

### [tmc5160 stepper_x]
```ini
[tmc5160 stepper_x]
cs_pin: PE3
spi_bus: spi1
run_current: 1.200       # higher current capability than tmc2209
sense_resistor: 0.075    # check your board's R_sense
stealthchop_threshold: 0  # spreadCycle recommended for high-current motors
```

## Fans

```ini
[fan]                        # part cooling — M106/M107
pin: PA0

[heater_fan hotend_fan]
pin: PB0
heater: extruder
heater_temp: 50.0

[controller_fan board_fan]
pin: PB2
stepper: stepper_x, stepper_y  # fan runs while any of these steppers are enabled
```

## Macros

```ini
[gcode_macro PRINT_START]
gcode:
    {% set bed = params.BED|default(60)|float %}
    {% set nozzle = params.NOZZLE|default(200)|float %}
    M140 S{bed}               # start bed heating (no wait)
    M109 S{nozzle}            # wait for nozzle
    M190 S{bed}               # wait for bed
    G28                       # home all
    BED_MESH_CALIBRATE        # auto mesh (or LOAD profile instead)
    G90                       # absolute coords
    G1 Z5 F3000

[gcode_macro PRINT_END]
gcode:
    G91                       # relative
    G1 E-2 F2700              # retract
    G1 Z5 F3000               # raise Z
    G90                       # absolute
    G1 X0 Y{printer.toolhead.axis_maximum.y} F5000  # park at rear
    M106 S0                   # fan off
    M104 S0                   # hotend off
    M140 S0                   # bed off
    M84                       # motors off
```

## Filament Sensor

```ini
[filament_switch_sensor filament_sensor]
switch_pin: ^PA4
pause_on_runout: True
runout_gcode:
    PAUSE

[filament_motion_sensor filament_sensor]  # detects actual movement (not just presence)
switch_pin: ^PA4
detection_length: 7.0  # mm of filament per pulse
extruder: extruder
pause_on_runout: True
```

## Multiple MCUs

```ini
[mcu extra_mcu]
serial: /dev/serial/by-id/usb-Klipper_...second_board...
# Then reference pins as: extra_mcu:PA4
```
