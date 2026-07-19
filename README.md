# ODIE V3 — Open Quadruped Robot

ODIE (Omni Directional Intelligent Explorer) is my 12-DOF servo-driven quadruped robot. I'm Malhar Labade, founder of 5K Robotics, and ODIE V3 is my third iteration — featuring real-time telemetry visualization, computer vision, fall detection, and autonomous behaviors.

## Hardware

- **Brain**: Raspberry Pi 5
- **Servo Controller**: Pimoroni Servo2040 (12 servos, MicroPython)
- **IMU**: MPU6050 on a separate Raspberry Pi Pico (pitch/roll/fall detection)
- **Vision**: USB camera + Hailo-8L NPU for YOLOv8 inference
- **Legs**: 12× MG996R servos (3 per leg: shoulder, hip, knee)
- **Power**: 2S LiPo

## Software Stack

| Component | Description |
|-----------|-------------|
| `main.py` | MicroPython firmware for Servo2040 — gaits, behaviors, serial protocol |
| `odie_rerun.py` | Live telemetry dashboard via Rerun SDK (servos, IMU, URDF, ball detection) |
| `ball_detect_cv.py` | OpenCV HSV ball tracker with Flask web dashboard |
| `person_detect.py` | Hailo YOLOv8 person detection — ODIE waves on first detection |
| `controller.py` | PS3 controller input via pygame with fall detection |
| `demo.py` | Unified launcher — switch between modes from one terminal |

## Behaviors

- **Stand** — breathing idle animation
- **Walk** — diagonal trot gait
- **Left / Right** — turning gaits
- **Sit** — smooth transition to sitting pose
- **Wave** — weight shift + front leg wave
- **Dance** — rhythmic body oscillation
- **Ball tracking** — HSV detection, head follows ball
- **Person greeting** — YOLOv8 detects person, ODIE sits and waves

## Telemetry (Rerun)

Live mind visualizer accessible from any browser:

https://app.rerun.io/?url=rerun+http://<PI_IP>:9876/proxy

Streams: 12 joint angles, IMU pitch/roll, fall detection, ball position, URDF 3D model, behavior state.

## Serial Protocol

Pi → Servo2040:
- `stand`, `walk`, `left`, `right`, `sit`, `wave`, `dance`, `reset90`
- `track,<cx>,<cy>` — normalized ball position
- `noball` — no ball detected

Servo2040 → Pi:
- `angles:<ch0>,<ch1>,...,<ch11>` — current joint angles
- `Mode:<mode>` — current behavior

IMU Pico → Pi:
- `<pitch>,<roll>` — degrees

## Running

```bash
python3 demo.py
# Then type: controller | ball | person | stop | quit
```

## Built by

## Built by

**Malhar Labade** — 5K Robotics | [@MalharLabade](https://youtube.com/@MalharLabade)
