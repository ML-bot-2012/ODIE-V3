# MuJoCo Simulation — ODIE V3

Physics simulation of ODIE V3 using MuJoCo. All gaits and behaviors were developed and tuned in simulation before being transferred to the real robot. The MJCF model was exported from the Onshape CAD assembly.

## Files

| File | Description |
|------|-------------|
| `robot.xml` | MJCF robot model — bodies, joints, inertia, meshes, actuators |
| `scene.xml` | Scene wrapper — ground plane, lighting, sky, includes robot.xml |
| `odie_walk_fwd.py` | Diagonal trot gait — forward walking |
| `turn_left.py` | Left turn gait — RR leg stepping |
| `turn_right.py` | Right turn gait — RL leg stepping |
| `sit.py` | Smooth stand → sit transition |
| `wave.py` | Sit → weight shift → FL leg wave |
| `dance.py` | Rhythmic knee oscillation |
| `stand.py` | Servo2040 stand script (MicroPython, runs on real hardware) |

## Running

```bash
cd odie_irl
python3 odie_walk_fwd.py   # forward walk
python3 turn_left.py        # turn left
python3 turn_right.py       # turn right
python3 sit.py              # sit
python3 wave.py             # wave
python3 dance.py            # dance
```

Requires MuJoCo and the `assets/` folder with STL meshes.

```bash
pip install mujoco
```

## Robot model

- **12 actuators** — position-controlled hinges, one per joint
- **Actuator gains** — kp=15, dampratio=1 (wave uses kp=100)
- **Leg order in sim** — FL (0-2), RR (3-5), FR (6-7-8), RL (9-10-11)
- **Meshes** — STL files in `assets/` exported from Onshape

## Coordinate mapping

Servo angles (degrees) are converted to MuJoCo radians relative to 90°:

```python
def deg_to_ctrl(servo_deg):
    return math.radians(servo_deg - 90)
```

## Gait — diagonal trot

Two diagonal pairs alternate at 50% duty cycle each:
- Phase 0.0–0.5: FL + RR swing
- Phase 0.5–1.0: FR + RL swing

Hip and knee offsets computed via sine wave, mirrored per leg side. Same logic runs on the real Servo2040 in `main.py`.

## Sim-to-real

Gait parameters (hip amplitude, knee lift, phase timing) were tuned in MuJoCo then directly transferred to `main.py` on the Servo2040 with no modification — the sine wave gait is identical in both.
