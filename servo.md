# main.py — Servo2040 MicroPython Firmware

MicroPython firmware for the Pimoroni Servo2040, controlling all 12 of ODIE's servos. Runs the gait engine, behavior state machine, and ball tracking head movement. Receives commands over USB serial from the Pi and streams joint angles back.

## Upload

Flash to the Servo2040 via Thonny IDE — save as `main.py` on the device. Runs automatically on boot.

## Behaviors

| Command | Description |
|---------|-------------|
| `stand` | Upright pose with breathing idle animation |
| `walk` | Diagonal trot gait (FL+RR / FR+RL alternating) |
| `left` | Left turning gait |
| `right` | Right turning gait |
| `sit` | Smooth transition to sitting pose |
| `wave` | Weight shift + front left leg wave |
| `dance` | Rhythmic body oscillation |
| `reset90` | All servos to 90° (calibration) |

## Serial input

Receives newline-terminated commands from the Pi at 115200 baud. Parsed via `select.select` for non-blocking reads inside the main loop.

Ball tracking commands:
- `track,<cx>,<cy>` — normalized ball position (0.0–1.0), tilts body toward ball
- `noball` — returns tilt/hip offsets to zero

## Serial output

Streams current joint angles after every `send_angles()` call:
angles:90,120,45,90,60,135,90,60,135,90,120,45

Also prints mode changes:
Mode: walk

## Servo layout

| Channel | Joint |
|---------|-------|
| 0 | FL shoulder |
| 1 | FL hip |
| 2 | FL knee |
| 3 | FR shoulder |
| 4 | FR hip |
| 5 | FR knee |
| 6 | RR shoulder |
| 7 | RR hip |
| 8 | RR knee |
| 9 | RL shoulder |
| 10 | RL hip |
| 11 | RL knee |

## Key poses (degrees)

```python
STAND = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
SIT   = [90, 120, 45, 90, 60, 135, 90, 60,   0, 90, 120, 180]
```

## Notes

- Calibration: 600–2400µs pulse width mapped to 0–180°
- Ball tracking uses exponential smoothing (α=0.05) for smooth head movement
- Breathing animation uses sine wave on knee servos during idle stand
