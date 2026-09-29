# sit.md – Sit Pose Transition

```python
import mujoco, mujoco.viewer
import numpy as np
import time, math

m = mujoco.MjModel.from_xml_path('scene.xml')
d = mujoco.MjData(m)

# Motor parameters
for i in range(m.nu):
    m.actuator_gainprm[i, 0] = 15.0
    m.actuator_biasprm[i, 1] = -15.0
    m.actuator_biasprm[i, 2] = -1.0

def deg_to_ctrl(servo_deg):
    return math.radians(servo_deg - 90)

STAND = np.array([
    0.0, deg_to_ctrl(120), deg_to_ctrl(45),
    0.0, -deg_to_ctrl(60), deg_to_ctrl(135),
    0.0, deg_to_ctrl(60), deg_to_ctrl(135),
    0.0, -deg_to_ctrl(120), -deg_to_ctrl(45),
])

SIT = np.array([
    0.0, deg_to_ctrl(120), deg_to_ctrl(45),
    0.0, -deg_to_ctrl(60), deg_to_ctrl(0),
    0.0, deg_to_ctrl(60), deg_to_ctrl(135),
    0.0, -deg_to_ctrl(120), -deg_to_ctrl(180),
])

# Initialize physics
mujoco.mj_resetData(m, d)
d.qpos[2] = -0.2
d.qpos[3] = 1.0
d.ctrl[:] = STAND
mujoco.mj_forward(m, d)

print("Sit down. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    # Interpolate STAND → SIT over 60 frames
    for i in range(60):
        t = i / 60
        d.ctrl[:] = STAND + (SIT - STAND) * t
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.02)
    
    # Hold SIT position
    while v.is_running():
        d.ctrl[:] = SIT
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.02)
```

### Pose Transition

**STAND Pose:**

[90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]


**SIT Pose (rear ankles flex):**

[90, 120, 45, 90, 60, 0, 90, 60, 135, 90, 120, 180]
↑ FR ankle: 135° → 0° (forward flex)
↑ RL ankle: 45° → 180° (back flex)


### Interpolation

- Duration: 60 frames @ 0.02s/frame = 1.2 seconds
- Linear blend: `ctrl = STAND + (SIT - STAND) * t` where `t ∈ [0, 1]`
- Smooth deceleration to stable sit base

| Joint | STAND | SIT | Effect |
|-------|-------|-----|--------|
| FR ankle (5) | 135° | 0° | Flex forward for ground contact |
| RL ankle (11) | 45° | 180° | Flex backward for stability |
| Others | — | — | Hold steady |

### Physics

- Center-of-mass lowers as rear legs bend
- Front ankles engage ground earlier (FR: 135° → 0°)
- Rear ankles rotate backward (RL: 45° → 180°) creating tripod stability
- Motor gain 15.0 tracks interpolated targets smoothly without overshoot

### Usage

```bash
python3 sit.py
```

Robot smoothly transitions from STAND to stable SIT pose over ~1 second, then holds indefinitely.

### Customization

**Faster sit (0.5s):**
```python
for i in range(25):  # 25 frames @ 0.02s = 0.5s
```

**Slower sit (2s):**
```python
for i in range(100):  # 100 frames @ 0.02s = 2s
```

**Partial sit (mid-squat):**
```python
t = 0.5  # 50% interpolation
d.ctrl[:] = STAND + (SIT - STAND) * t
```

### Integration with demo.py

```python
if mode == 'sit':
    for i in range(60):
        t = i / 60
        ctrl = STAND + (SIT - STAND) * t
        serial_mgr.send_servo_angles(ctrl)
        time.sleep(0.02)
```

### Combined Motion

Sit before waving:
```python
# Sit transition
for i in range(60):
    t = i / 60
    ctrl = STAND + (SIT - STAND) * t
    d.ctrl[:] = ctrl
    mujoco.mj_step(m, d)
    v.sync()
    time.sleep(0.02)

# Then transition to wave
WAVE = SIT.copy()
WAVE[1] = deg_to_ctrl(60)
WAVE[2] = deg_to_ctrl(5)
# ... continue with wave motion
```
