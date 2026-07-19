# odie_rerun.py — ODIE Mind Visualizer

Real-time telemetry dashboard for ODIE V3, powered by the [Rerun SDK](https://rerun.io). Streams live robot data to any browser — no install needed on the viewing device.

## What it does

Starts a gRPC server on the Pi and streams all robot data to the Rerun web viewer. Open the printed URL on any device on the same network to see ODIE's mind live.

## Data streams

| Path | Type | Description |
|------|------|-------------|
| `odie/servos/<name>` | Scalar | Individual joint angle (degrees) for all 12 servos |
| `odie/servos/all` | BarChart | All 12 joint angles as a bar chart |
| `odie/imu/pitch` | Scalar | Pitch angle from MPU6050 (degrees) |
| `odie/imu/roll` | Scalar | Roll angle from MPU6050 (degrees) |
| `odie/imu/fell` | Scalar | Fall detection — 1.0 if pitch or roll exceeds ±45° |
| `odie/behavior` | TextLog | Current behavior mode (stand, walk, sit, wave, etc.) |
| `odie/ball/detected` | Scalar | 1.0 if ball is currently tracked, 0.0 otherwise |
| `odie/ball/position` | Points2D | Ball position in 320×240 camera space |
| `odie/robot` | URDF | Full 3D robot model loaded from URDF + OBJ meshes |

## Serial inputs

Reads two serial ports simultaneously in background threads:

- **Servo2040** — `angles:<ch0>,...,<ch11>` lines for joint telemetry, `Mode:<mode>` for behavior state
- **IMU Pico** — `<pitch>,<roll>` CSV lines at 50Hz

## How to run

```bash
python3 odie_rerun.py
```

Then open the printed URL in any browser:

https://app.rerun.io/?url=rerun+http://<PI_IP>:9876/proxy

## Dependencies

```bash
sudo pip install rerun-sdk numpy pyserial opencv-python --break-system-packages
```

## Notes

- The URDF path and serial port IDs are hardcoded — update them if running on a different machine
- The gRPC server buffers up to 1GiB of data so late-connecting viewers get full history
- Run alongside `ball_detect_cv.py` or `controller.py` for full telemetry
