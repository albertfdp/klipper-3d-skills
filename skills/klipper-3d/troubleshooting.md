# Klipper Troubleshooting

Source: https://github.com/Klipper3d/klipper/blob/master/docs/FAQ.md

**First step:** Always check `klippy.log` (`/tmp/klippy.log`) before anything else.

```bash
tail -f /tmp/klippy.log
grep -i "error\|warning\|shutdown" /tmp/klippy.log | tail -30
```

## MCU / Serial Communication

### "Lost communication with MCU" (random, mid-print)

Most common causes (in order):
1. **Bad USB cable** — replace with a quality data-only cable; secure the plugs
2. **Pi undervoltage** — use a proper 5V/3A PSU; look for "under voltage" in OctoPrint
3. **Overloaded printer PSU** — power fluctuations reset the MCU's USB chip
4. **Stressed printer wiring** — crimped/frayed wires lose contact during movement
5. **ModemManager interference** — on some Linux distros, disable it:
   ```bash
   sudo systemctl disable ModemManager
   ```

### "Unable to open serial port" / "Unable to connect to MCU"

Wrong serial path, or Klipper not running:
```bash
# Find correct stable path (never use /dev/ttyUSB0)
ls /dev/serial/by-id/*
# If no unique ID (CH340 chips): ls /dev/serial/by-path/*

# Check Klipper service
sudo systemctl status klipper
sudo systemctl restart klipper

# View recent logs
journalctl -u klipper -n 50
```

Update `printer.cfg`:
```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32...  # exact path from ls above
```

### Serial port changes to /dev/ttyUSB1 on restart

Use the `/dev/serial/by-id/` path — it is stable across reboots. Never hardcode `/dev/ttyUSB0`.

## Homing Problems

### Z-axis makes screeching noise when homing (converted from Marlin)

`max_z_velocity` too high. Klipper runs steppers much faster than Marlin — the motor can't keep up.
```ini
[printer]
max_z_velocity: 5    # keep at 5mm/s or lower
```

### Stepper moves in wrong direction

Add `!` to invert `dir_pin`:
```ini
dir_pin: !PC2
```

### "Move out of range" errors

`position_endstop` doesn't match actual endstop trigger position, or `position_min`/`position_max` are misconfigured.

### "Cannot move before homing"

Intentional safety feature — prevents crashing into the bed. Run `G28` first.

For diagnostic movement without physical homing:
```
SET_KINEMATIC_POSITION X=0 Y=0 Z=10
```

Or add `[force_move]` to config:
```ini
[force_move]
enable_force_move: True
# Then: FORCE_MOVE STEPPER=stepper_z DISTANCE=10 VELOCITY=5
```

## TMC Driver Errors

### TMC reports OT (overtemperature)

Reduce `run_current` or improve driver cooling. Check if heatsinks are making contact.

### TMC reports GSTAT reset flag

Power supply problem or loose motor wiring. Check all connector seating.

### Stepper skipping / losing steps

1. Increase `run_current` (max ~85% of motor's rated current)
2. Check motor wiring — especially JST connectors
3. Reduce `max_accel`
4. Verify `rotation_distance` is correct (wrong value = excessive torque demand)

### TMC2208/TMC2224 driver shuts off mid-print (standalone mode)

Known bug — update Klipper:
```bash
cd ~/klipper && git pull
sudo service klipper restart
```

### Diagnose TMC state
```
DUMP_TMC STEPPER=stepper_x
```

## Temperature / Heating Errors

### "Heating failed" / Thermal runaway

Heater can't reach or hold target:
- Check heater cartridge wiring
- Check thermistor wiring and `sensor_type` in config — wrong type = wildly wrong readings
- Run `PID_CALIBRATE` to tune PID values
- Check for drafts hitting the hotend

### Wrong temperature reading

`sensor_type` in config doesn't match your actual thermistor. Check your printer's BOM.
Common confusion: `EPCOS 100K B57560G104F` vs `ATC Semitec 104GT-2` — they read differently.

### Heater turns off when Pi crashes

By design. The MCU requires a "heartbeat" confirmation from the host every 3 seconds. If the host dies, all heaters and steppers shut down safely.

## Bed Leveling / Probing

### BLTouch probe not triggering (or always triggered)

- Add `^` pull-up to `sensor_pin`: `sensor_pin: ^PB1`
- Test manually:
  ```
  BLTOUCH_DEBUG COMMAND=pin_down
  BLTOUCH_DEBUG COMMAND=touch_mode
  BLTOUCH_DEBUG COMMAND=pin_up
  ```

### PROBE_ACCURACY shows high variance (>0.025mm range)

Probe is not repeatable — don't use this Z offset:
- Check for mechanical looseness (probe mount, carriage)
- BLTouch: clean the pin; try replacing pin
- Increase `samples` and `sample_retract_dist` in `[probe]`/`[bltouch]`

### Bed mesh has large variation (>1mm)

Bed is severely unlevel — do manual screw leveling first:
```
SCREWS_TILT_CALCULATE
```

### Z offset drifts between prints

Always probe at print temperature — thermal expansion changes Z offset. Run `PROBE_CALIBRATE` hot.

## Firmware Flashing

### `make flash` fails

```bash
# Stop Klipper first
sudo service klipper stop

# Flash manually with explicit device path
make flash FLASH_DEVICE=/dev/serial/by-id/usb-...

# Verify port:
ls /dev/serial/by-id/*
```

For boards that don't support `make flash`, use DFU or the bootloader:
```bash
# DFU (STM32 with DFU bootloader)
dfu-util -d 0483:df11 -s 0x08000000:leave -D out/klipper.bin

# AVR via avrdude
avrdude -p atmega2560 -c stk500v2 -P /dev/ttyUSB0 -U flash:w:out/klipper.elf.hex:i
```

### Klipper version mismatch error

MCU firmware must match host software version exactly:
```bash
cd ~/klipper
git pull
~/klipper/scripts/install-octopi.sh  # update host dependencies
make menuconfig    # select your MCU
make clean
make
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/...
sudo service klipper start
```

### AVR device hangs on restart with `restart_method=command`

Known AVR bootloader bug. Change `restart_method` in `[mcu]` to `arduino` or `rpi_usb`, or flash an updated bootloader.

## Updating Klipper

Host software only (most common):
```bash
cd ~/klipper
git pull
sudo service klipper restart
```

If warned about MCU mismatch, also reflash:
```bash
make menuconfig && make clean && make
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/...
sudo service klipper start
```

Always check https://github.com/Klipper3d/klipper/blob/master/docs/Config_Changes.md before upgrading.

## Detecting Lost Steps

```
G28
GET_POSITION          # note mcu: step counts
# run your print or high-speed moves
G28
GET_POSITION          # compare mcu: step counts
```

A difference in `mcu:` values that's a multiple of 64 microsteps (for 16 microstep config) indicates a lost step.

## Quick Diagnostics Checklist

- [ ] `STATUS` — reports "ready"?
- [ ] `QUERY_ENDSTOPS` — all open when not pressed?
- [ ] Temperatures reading correctly? (Check `sensor_type` matches hardware)
- [ ] Serial port using stable `/dev/serial/by-id/` path?
- [ ] `max_z_velocity` ≤ 5 (Marlin converts)?
- [ ] Klipper and MCU firmware versions match? (`git log -1` vs `STATUS` MCU version)
- [ ] Pi power supply adequate? (5V/3A minimum)
- [ ] `klippy.log` checked for errors?
