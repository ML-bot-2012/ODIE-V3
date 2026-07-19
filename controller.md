# controller.py — PS3 Gamepad Controller

PS3 gamepad input for ODIE with real-time IMU fall detection. Reads button presses via pygame and sends behavior commands to the Servo2040 over serial. Automatically commands ODIE to stand if a fall is detected.

## Usage

```bash
python3 controller.py
```

Requires a PS3 controller connected via USB or Bluetooth.

## Button Mapping

| Button | Action |
|--------|--------|
| D-Pad Up | Walk |
| D-Pad Left | Turn left |
| D-Pad Right | Turn right |
| X (Cross) | Stand |
| O (Circle) | Dance |
| Triangle | Sit |
| Square | Wave |
| L1 | Reset all servos to 90° |

## Fall Detection

Reads pitch and roll from the IMU Pico at ~50Hz in a background thread. If `abs(pitch) > 45°` or `abs(roll) > 45°`, ODIE is considered fallen and immediately commands `stand` regardless of current mode. Prints fall angles to console.

## Serial ports

| Port | Device |
|------|--------|
| Servo2040 | `usb-MicroPython_Board_in_FS_mode_e66598541b222033-if00` |
| IMU Pico | `usb-MicroPython_Board_in_FS_mode_e66538544728a330-if00` |

## Dependencies

```bash
pip install pygame pyserial
```

## Notes

- Commands only sent on mode change (not repeatedly) to avoid serial flooding
- Loop runs at 50Hz (`time.sleep(0.02)`)
- On exit: sends `stand`, closes serial ports, quits pygame
