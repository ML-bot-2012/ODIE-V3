# turn_left.md – Left Steering Turn

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

# Initialize physics
mujoco.mj_resetData(m, d)
d.qpos[2] = -0.2
d.qpos[3] = 1.0
d.ctrl[:] = STAND
mujoco.mj_forward(m, d)

print("Turn left. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    step = 0
    while v.is_running():
        # Front-Right (FR) leg steering: indices 3, 5
        fr_s = 15 * math.sin(step * 0.003 * 2 * math.pi) * math.radians(3.5)
        fr_k = 40 * max(0, math.sin(step * 0.003 * 2 * math.pi)) * math.radians(3.5)
        ctrl = STAND.copy()
        ctrl[3] += fr_s   # FR hip oscillate
        ctrl[5] += fr_k   # FR knee oscillate (half-wave)
        d.ctrl[:] = ctrl
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)
        step += 1
```

### Turn Mechanics

**Steering:** Front-Right (FR) leg modulation while rear remains static in STAND

**Affected Channels:**
- ch[3]: FR hip
- ch[5]: FR knee

### Hip Oscillation

- fr_s = 15 × sin(step × 0.003 × 2π) × 3.5° = ±52.5° amplitude
- Frequency: step × 0.003 rad/sample ≈ 1 Hz oscillation
- Hip swings ±52.5° relative to STAND position

### Knee Oscillation (Half-Wave Rectified)

- fr_k = 40 × max(0, sin(...)) × 3.5° = 0 to 140° peak
- Only positive half of sine (leg lift only during swing)
- Creates asymmetric ground reaction force during swing

### Turning Physics

- Swinging FR leg while rear stays planted creates differential yaw torque
- Left turn: FR accelerates foot outward during swing, creating counter-clockwise torque
- Continuous steering until operator releases

### Cycle Time

~333 steps @ 0.002s/step = 0.67 seconds per oscillation period

### Integration with Walk

- Can combine with walk baseline by adding FR modulation to trot pattern
- Use fr_s and fr_k offsets during swing phases of walk cycle
- Maintains forward + rotational momentum

### Motor Response

Gain=15.0 tracks servo targets smoothly; half-wave rectification prevents ground collision during lift.

### Usage

```bash
python3 turn_left.py
```

Robot starts in STAND and continuously turns left by oscillating front-right leg.

### Customization

**Tighter turn (sharper steering):**
```python
fr_s = 25 * math.sin(...)  # Increase 15 → 25
```

**Faster rotation:**
```python
step * 0.005  # Increase 0.003 → 0.005
```

**Slower/smoother:**
```python
step * 0.001  # Decrease 0.003 → 0.001
```

### Comparison: Turn Left vs Turn Right

| Motion | Leg | Hip Index | Knee Index | Rotation |
|--------|-----|-----------|-----------|----------|
| Turn Left | Front-Right | 3 | 5 | Counter-clockwise |
| Turn Right | Rear-Left | 9 | 11 | Clockwise |
