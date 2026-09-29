# dance.md – Synchronized Knee Oscillation

```python
import mujoco, mujoco.viewer
import numpy as np
import time, math

m = mujoco.MjModel.from_xml_path('scene.xml')
d = mujoco.MjData(m)

# Motor parameters (standard gain)
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

print("Dance. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    t = 0.0
    while v.is_running():
        ctrl = STAND.copy()
        
        # Front knees (FL, FR) in-phase
        offset_front = 15 * math.sin(t * 4.0) * math.radians(1.0)
        ctrl[2] += offset_front   # FL knee
        ctrl[5] += offset_front   # FR knee
        
        # Rear knees (RR, RL) opposite-phase
        offset_rear = 15 * math.sin(t * 4.0 + math.pi) * math.radians(1.0)
        ctrl[8] += offset_rear    # RR knee
        ctrl[11] += offset_rear   # RL knee
        
        d.ctrl[:] = ctrl
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)
        t += 0.002
```

### Oscillation Pattern

| Leg Pair | Phase | Frequency | Amplitude |
|----------|-------|-----------|-----------|
| Front (FL, FR) | 0 rad (in-phase) | 4.0 rad/s | ±15° |
| Rear (RR, RL) | π rad (opposite) | 4.0 rad/s | ±15° |

### Frequency

- 4.0 rad/s ÷ (2π) = 0.64 Hz oscillation
- Period: ~1.55 seconds per cycle
- Both front and rear complete 0.64 cycles/second

### Motion Sequence

1. **t = 0:** Front knees up, rear knees down
2. **t = 0.39s:** Front knees center, rear knees center
3. **t = 0.77s:** Front knees down, rear knees up
4. **t = 1.16s:** Front knees center, rear knees center
5. **t = 1.55s:** Cycle repeats

### Diagonal Energy Flow
FL   FR           FL ↑  FR ↑        FL ↓  FR ↓
↓    ↓   →    →    ↓    ↓     →       
RR   RL          RR ↑  RL ↑        RR ↓  RL ↓

Front/rear opposing phases create wave-like energy propagation (bouncing effect).

### Usage

```bash
python3 dance.py
```

Robot bounces with front and rear knees oscillating 180° out of phase.

### Customization

**Faster dance (1.0 Hz):**
```python
offset_front = 15 * math.sin(t * 6.28) * math.radians(1.0)  # 4.0 → 6.28 rad/s
```

**Slower dance (0.3 Hz):**
```python
offset_front = 15 * math.sin(t * 2.0) * math.radians(1.0)   # 4.0 → 2.0 rad/s
```

**Larger amplitude:**
```python
offset_front = 25 * math.sin(t * 4.0) * math.radians(1.0)   # 15 → 25
```

**Same-phase (all knees up/down together):**
```python
offset_rear = 15 * math.sin(t * 4.0) * math.radians(1.0)    # Remove π offset
```

### Integration with demo.py

```python
if mode == 'dance':
    t = 0.0
    while True:
        offset_f = 15 * math.sin(t * 4.0) * math.radians(1.0)
        offset_r = 15 * math.sin(t * 4.0 + math.pi) * math.radians(1.0)
        ctrl[2] = ctrl[5] = offset_f
        ctrl[8] = ctrl[11] = offset_r
        serial_mgr.send_servo_angles(ctrl)
        time.sleep(0.002)
        t += 0.002
```

### Physics

- Simultaneous front lift + rear press (or reverse) creates seesaw effect
- Diagonal oscillation mimics natural quadruped bounce
- Motor gain 15.0 tracks sine smoothly without lag
- No forward motion—purely vertical energy (stationary dancing)

### Parameters

| Parameter | Value | Effect |
|-----------|-------|--------|
| Frequency | 4.0 rad/s | 0.64 Hz bounce rate |
| Amplitude | ±15° | Knee range-of-motion |
| Phase offset | π rad (180°) | Front/rear opposition |
| Scaling | math.radians(1.0) | Fine-tune peak angle |
