# Klipper G-Codes Reference

Full reference: https://www.klipper3d.org/G-Codes.html  
Source: https://github.com/Klipper3d/klipper/blob/master/docs/G-Codes.md

## Standard G-Codes

| Command | Description |
|---------|-------------|
| `G0/G1 [X] [Y] [Z] [E] [F]` | Move (G0=travel, G1=print) |
| `G4 P<ms>` | Dwell (wait) for milliseconds |
| `G28 [X] [Y] [Z]` | Home axes |
| `G90` | Absolute positioning |
| `G91` | Relative positioning |
| `G92 [X] [Y] [Z] [E]` | Set position |
| `M18` / `M84` | Disable motors |
| `M82` | Absolute extrusion mode |
| `M83` | Relative extrusion mode |
| `M104 [T<idx>] S<temp>` | Set extruder temp (no wait) |
| `M109 [T<idx>] S<temp>` | Set extruder temp (wait for stable) |
| `M140 S<temp>` | Set bed temp (no wait) |
| `M190 S<temp>` | Set bed temp (wait for stable) |
| `M105` | Report temperatures |
| `M106 S<0-255>` | Set fan speed |
| `M107` | Fan off |
| `M112` | Emergency stop |
| `M114` | Get current position |
| `M115` | Get firmware version |
| `M204 S<mm/s²>` | Set acceleration |
| `M220 S<percent>` | Speed factor override |
| `M221 S<percent>` | Extrude factor override |
| `M400` | Wait for current moves to finish |

## System Commands

| Command | Description |
|---------|-------------|
| `RESTART` | Reload `printer.cfg` — no MCU reset |
| `FIRMWARE_RESTART` | Reset MCU communication and reload config |
| `STATUS` | Report printer ready/not-ready state |
| `HELP` | List available extended commands |
| `SAVE_CONFIG` | Write calibration results to `printer.cfg` and restart |
| `SET_GCODE_OFFSET Z=<val>` | Set absolute Z baby-step offset |
| `SET_GCODE_OFFSET Z_ADJUST=<val>` | Adjust Z offset relatively |
| `SET_VELOCITY_LIMIT [VELOCITY=] [ACCEL=] [SQUARE_CORNER_VELOCITY=]` | Override limits at runtime |

## Homing & Position

| Command | Description |
|---------|-------------|
| `G28` | Home all axes |
| `G28 X Y` | Home specific axes only |
| `QUERY_ENDSTOPS` | Report all endstop states (open/triggered) |
| `GET_POSITION` | Report stepper, toolhead, gcode, and kinematic positions |
| `SET_KINEMATIC_POSITION [X=] [Y=] [Z=]` | Force-set position without physical homing |

## Bed Leveling

| Command | Description |
|---------|-------------|
| `BED_MESH_CALIBRATE [PROFILE=name] [ADAPTIVE=1]` | Run auto bed mesh probing |
| `BED_MESH_PROFILE SAVE=name` | Save active mesh to named profile |
| `BED_MESH_PROFILE LOAD=name` | Load a saved mesh profile |
| `BED_MESH_PROFILE REMOVE=name` | Delete a saved mesh profile |
| `BED_MESH_CLEAR` | Disable active mesh compensation |
| `BED_MESH_OUTPUT` | Print current mesh values to terminal |
| `PROBE` | Run a single probe measurement |
| `PROBE_CALIBRATE` | Interactive Z offset calibration (paper test) |
| `PROBE_ACCURACY [SAMPLES=10]` | Measure probe repeatability |
| `Z_TILT_ADJUST` | Adjust tilt via 2 independent Z motors |
| `QUAD_GANTRY_LEVEL` | Level CoreXY gantry via 4 Z motors |
| `SCREWS_TILT_CALCULATE` | Guide manual bed screw adjustment |
| `BED_SCREWS_ADJUST` | Interactive manual bed leveling |

## PID & Temperature

| Command | Description |
|---------|-------------|
| `PID_CALIBRATE HEATER=extruder TARGET=200` | Run PID autotune for hotend |
| `PID_CALIBRATE HEATER=heater_bed TARGET=60` | Run PID autotune for bed |
| `SET_HEATER_TEMPERATURE HEATER=extruder TARGET=200` | Set heater target |
| `TEMPERATURE_WAIT SENSOR=extruder MINIMUM=200` | Wait for temperature |
| `QUERY_ADC NAME=<sensor>` | Read raw ADC value |

## Input Shaping & Resonance

| Command | Description |
|---------|-------------|
| `SHAPER_CALIBRATE [AXIS=X]` | Auto-calibrate shaper (requires ADXL345) |
| `TEST_RESONANCES AXIS=X` | Measure resonance on axis |
| `MEASURE_AXES_NOISE` | Measure accelerometer noise floor |
| `SET_INPUT_SHAPER [SHAPER_TYPE=mzv] [SHAPER_FREQ_X=45]` | Override shaper at runtime |
| `ACCELEROMETER_MEASURE [CHIP=adxl345]` | Start/stop accelerometer recording |
| `ACCELEROMETER_QUERY` | Query accelerometer current values |

## Pressure Advance

| Command | Description |
|---------|-------------|
| `SET_PRESSURE_ADVANCE ADVANCE=0.05` | Set pressure advance |
| `SET_PRESSURE_ADVANCE ADVANCE=0.05 SMOOTH_TIME=0.040` | Set PA with smooth time |
| `TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.005` | PA tuning tower |

## TMC Driver Commands

| Command | Description |
|---------|-------------|
| `DUMP_TMC STEPPER=stepper_x` | Dump all TMC registers to terminal |
| `SET_TMC_CURRENT STEPPER=stepper_x CURRENT=0.8` | Change run current at runtime |
| `SET_TMC_FIELD STEPPER=stepper_x FIELD=<name> VALUE=<val>` | Write specific register field |
| `INIT_TMC STEPPER=stepper_x` | Re-initialize TMC driver from config |
| `SET_STEPPER_ENABLE STEPPER=stepper_x ENABLE=0` | Enable or disable a stepper |

## Fans

| Command | Description |
|---------|-------------|
| `SET_FAN_SPEED FAN=fan SPEED=0.5` | Set named fan speed (0.0–1.0) |

## Macro & State Management

| Command | Description |
|---------|-------------|
| `PAUSE` | Pause print |
| `RESUME [VELOCITY=<val>]` | Resume print |
| `CANCEL_PRINT` | Cancel print |
| `SAVE_GCODE_STATE [NAME=state]` | Save toolhead position and settings |
| `RESTORE_GCODE_STATE [NAME=state] [MOVE=1] [MOVE_SPEED=<val>]` | Restore saved state |
| `SET_GCODE_VARIABLE MACRO=name VARIABLE=var VALUE=val` | Set macro variable at runtime |

## Jinja2 Macro Templating

```jinja
[gcode_macro EXAMPLE]
variable_my_var: 0        # accessible as printer["gcode_macro EXAMPLE"].my_var
gcode:
    # Parameters (always strings — cast explicitly)
    {% set temp = params.TEMP|default(200)|float %}
    {% set count = params.COUNT|default(3)|int %}
    {% set name = params.NAME|default("default")|string %}

    # Printer state
    {% set x = printer.toolhead.position.x %}
    {% set max_y = printer.toolhead.axis_maximum.y %}
    {% set fan = printer.fan.speed %}
    {% set extruder_temp = printer.extruder.temperature %}
    {% set homed = "x" in printer.toolhead.homed_axes %}

    # Conditional
    {% if temp > 240 %}
        M118 High temp warning
    {% elif temp < 150 %}
        M118 Low temp
    {% endif %}

    # Loop
    {% for i in range(count) %}
        G1 Z{i * 5} F3000
    {% endfor %}

    # Call another macro
    PRINT_START NOZZLE={temp}
```

### Useful Printer State Paths

```
printer.toolhead.position.x / .y / .z / .e
printer.toolhead.axis_maximum.x / .y / .z
printer.toolhead.max_velocity / max_accel
printer.toolhead.homed_axes             # string like "xyz"
printer.extruder.temperature / .target
printer.heater_bed.temperature / .target
printer.fan.speed                       # 0.0–1.0
printer.idle_timeout.state              # "Idle", "Printing", "Ready"
printer.pause_resume.is_paused
printer["gcode_macro MY_MACRO"].my_var  # macro variable
```

### Respond to Console
```jinja
M118 Message to terminal
{action_respond_info("Info message")}
{action_raise_error("Fatal error message")}
```
