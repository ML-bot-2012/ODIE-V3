# wave.md – Wave Hand Gesture

```python
import mujoco, mujoco.viewer
import numpy as np
import time, math

m = mujoco.MjModel.from_xml_path('scene.xml')
d = mujoco.MjData(m)

# Motor parameters (higher gain for faster response)
for i in range(m.nu):
    m.actuator_gainprm[i, 0] = 100.0
    m.actuator_biasprm[i, 1] = -100.0
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

print("Wave. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    # Phase 1: Sit down
    for i in range(60):
        t = i / 60
        d.ctrl[:] = STAND + (SIT - STAND) * t
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.02)
    
    # Phase 2: Raise FL (Front-Left) leg
    WAVE = SIT.copy()
    WAVE[1] = deg_to_ctrl(60)   # FL hip forward (120° → 60°)
    WAVE[2] = deg_to_ctrl(5)    # FL knee up (45° → 5°)
    for i in range(30):
        t = i / 30
        d.ctrl[:] = SIT + (WAVE - SIT) * t
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.02)
    
    # Phase 3: Oscillate FL in wave motion
    wt = 0.0
    while v.is_running():
        ctrl = WAVE.copy()
        ctrl[1] = deg_to_ctrl(60 + 20 * math.sin(wt * 6))   # FL hip ±20° @ 6 rad/s
        ctrl[2] = deg_to_ctrl(5 + 5 * math.sin(wt * 6))     # FL knee ±5° @ 6 rad/s
        d.ctrl[:] = ctrl
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)
        wt += 0.01
```

### Motion Sequence

**Motor Gain:** 100.0 (10× higher than STAND/TURN for snappy response)

### Phase 1: Sit Down

- Duration: 60 frames, 1.2s
- Linear interpolation STAND → SIT
- Establishes stable base posture

### Phase 2: Raise FL Leg

- Duration: 30 frames, 0.6s
- FL hip: 120° → 60° (flex forward)
- FL knee: 45° → 5° (raise up)
- Creates "waving" position at chest height

### Phase 3: Wave Forever

- FL hip: 60° ± 20° × sin(6 × t) = [40°, 80°] peak-to-peak
- FL knee: 5° ± 5° × sin(6 × t) = [0°, 10°] peak-to-peak
- Frequency: 6 rad/s ≈ 0.95 Hz
- Period: ~6.3 seconds per full oscillation

### Waving Effect

Sine waves in-phase on both hip and knee create circular motion at paw—natural "hello" gesture.

### Time Step

0.002s per physics step (physics-ticked) vs 0.02s in sit phase (display-synced)

### Motor Response

Gain=100 allows servo to track sine wave smoothly without lag; bias=-100 centers neutral.

### Usage

```bash
python3 wave.py
```

Robot sits down, raises front-left leg, and continuously waves.

### Customization

**Faster wave (1.5 Hz):**
```python
ctrl[1] = deg_to_ctrl(60 + 20 * math.sin(wt * 9.42))  # 6 → 9.42 rad/s
```

**Slower wave (0.5 Hz):**
```python
ctrl[1] = deg_to_ctrl(60 + 20 * math.sin(wt * 3.14))  # 6 → 3.14 rad/s
```

**Larger wave amplitude:**
```python
ctrl[1] = deg_to_ctrl(60 + 30 * math.sin(wt * 6))     # 20 → 30
```

**Wave other leg (FR):**
```python
ctrl[4] = deg_to_ctrl(60 + 20 * math.sin(wt * 6))  # Use indices 3,4 instead of 1,2
```

### Integration with demo.py

```python
if mode == 'wave':
    # Sit
    for i in range(60):
        t = i / 60
        ctrl = STAND + (SIT - STAND) * t
        serial_mgr.send_servo_angles(ctrl)
        time.sleep(0.02)
    # Wave
    wt = 0.0
    while True:
        ctrl[1] = 60 + 20 * math.sin(wt * 6)
        serial_mgr.send_servo_angles(ctrl)
        time.sleep(0.002)
        wt += 0.01
```

### Physics

- High motor gain (100.0) required to track fast sine waves
- In-phase hip+knee oscillation creates circular paw motion
- No collision risk—purely gestural motion
- 6 rad/s frequency creates smooth, natural-looking wave
