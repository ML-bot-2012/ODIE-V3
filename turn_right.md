# turn_right.md – Right Steering Turn

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

print("Turn right. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    step = 0
    while v.is_running():
        # Rear-Left (RL) leg steering: indices 9, 11
        rl_s = 15 * math.sin(step * 0.003 * 2 * math.pi) * math.radians(3.5)
        rl_k = 40 * max(0, math.sin(step * 0.003 * 2 * math.pi)) * math.radians(3.5)
        ctrl = STAND.copy()
        ctrl[9] += rl_s   # RL hip oscillate
        ctrl[11] += rl_k  # RL knee oscillate (half-wave)
        d.ctrl[:] = ctrl
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)
        step += 1
```

### Turn Mechanics

**Steering:** Rear-Left (RL) leg modulation while front remains static in STAND

**Affected Channels:**
- ch[9]: RL hip
- ch[11]: RL knee

### Hip Oscillation

- rl_s = 15 × sin(step × 0.003 × 2π) × 3.5° = ±52.5° amplitude
- Frequency: step × 0.003 rad/sample ≈ 1 Hz oscillation
- Hip swings ±52.5° relative to STAND position

### Knee Oscillation (Half-Wave Rectified)

- rl_k = 40 × max(0, sin(...)) × 3.5° = 0 to 140° peak
- Only positive half of sine (leg lift only during swing)
- Creates asymmetric ground reaction force during swing

### Turning Physics

- Swinging RL leg while front stays planted creates differential yaw torque
- Right turn: RL accelerates foot inward during swing, creating clockwise torque
- Continuous steering until operator releases

### Cycle Time

~333 steps @ 0.002s/step = 0.67 seconds per oscillation period

### Differential Steering

- Left turn uses FR (front-right) modulation → counter-clockwise yaw
- Right turn uses RL (rear-left) modulation → clockwise yaw
- Mirror strategies balance turning on opposite diagonal pairs

### Integration with Walk

- Can combine with walk baseline by adding RL modulation to trot pattern
- Use rl_s and rl_k offsets during swing phases of walk cycle
- Maintains forward + rotational momentum

### Motor Response

Gain=15.0 tracks servo targets smoothly; half-wave rectification prevents ground collision during lift.

### Usage

```bash
python3 turn_right.py
```

Robot starts in STAND and continuously turns right by oscillating rear-left leg.

### Customization

**Tighter turn (sharper steering):**
```python
rl_s = 25 * math.sin(...)  # Increase 15 → 25
```

**Faster rotation:**
```python
step * 0.005  # Increase 0.003 → 0.005
```

**Slower/smoother:**
```python
step * 0.001  # Decrease 0.003 → 0.001
```
