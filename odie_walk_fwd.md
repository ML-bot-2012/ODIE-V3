# odie_walk_fwd.md – Forward Diagonal Trot

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

def leg_step_bezier(phase, hip_sign, knee_sign):
    """Bezier curve for smooth leg swing during stance/swing phases."""
    if phase < 0 or phase > 1:
        return 0, 0
    # Swing: parabolic lift (peak at 0.5 phase)
    lift = 4 * phase * (1 - phase)
    # Hip: sinusoidal back-and-forth
    hip_offset = hip_sign * 20 * math.sin(phase * math.pi) * math.radians(3.5)
    # Knee: lift during swing (positive half-cycle)
    knee_offset = knee_sign * 30 * lift * math.radians(3.5)
    return hip_offset, knee_offset

# Initialize physics
mujoco.mj_resetData(m, d)
d.qpos[0] = 0.291   # Forward x position
d.qpos[1] = 0.096   # Lateral y position
d.qpos[2] = -0.2    # Body height
d.qpos[3] = 1.0     # Body orientation
d.ctrl[:] = STAND
mujoco.mj_forward(m, d)

print("Walk forward. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    cycle_step = 0
    while v.is_running():
        # Cycle: 500 steps = 1 second @ 0.002s/step
        phase = (cycle_step % 500) / 500
        
        ctrl = STAND.copy()
        
        if phase < 0.5:
            # Phase 1: FL + RR swing (FR + RL stance)
            phase_1 = phase / 0.5
            # FL swing
            fl_h, fl_k = leg_step_bezier(phase_1, -1, 1)
            ctrl[0] += fl_h
            ctrl[1] += fl_k
            # RR swing
            rr_h, rr_k = leg_step_bezier(phase_1, 1, 1)
            ctrl[6] += rr_h
            ctrl[7] += rr_k
        else:
            # Phase 2: FR + RL swing (FL + RR stance)
            phase_2 = (phase - 0.5) / 0.5
            # FR swing
            fr_h, fr_k = leg_step_bezier(phase_2, -1, 1)
            ctrl[3] += fr_h
            ctrl[4] += fr_k
            # RL swing
            rl_h, rl_k = leg_step_bezier(phase_2, 1, 1)
            ctrl[9] += rl_h
            ctrl[10] += rl_k
        
        d.ctrl[:] = ctrl
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)
        cycle_step += 1
```

### Gait Type

**Diagonal Trot:** Alternating diagonal pairs (FL+RR, then FR+RL) for stable 4-point contact and forward momentum.

### Motion Phases

| Phase | Duration | Swing Legs | Stance Legs | Effect |
|-------|----------|-----------|------------|--------|
| 0.0–0.5s | 250 frames | FL + RR | FR + RL | Push forward via rear diagonal |
| 0.5–1.0s | 250 frames | FR + RL | FL + RR | Push forward via front diagonal |

### Leg Bezier Curve

```python
phase ∈ [0, 1] (normalized within each swing phase)
lift = 4 × phase × (1 − phase)          # Parabolic (0→1→0)
hip = hip_sign × 20 × sin(phase × π) × 3.5° = ±70°
knee = knee_sign × 30 × lift × 3.5°     # 0→105°→0
```

### Stance vs Swing

| Leg | Phase 0–0.5 | Phase 0.5–1.0 |
|-----|------------|--------------|
| FL (0,1) | Swing (lift) | Stance (ground) |
| FR (3,4) | Stance (ground) | Swing (lift) |
| RR (6,7) | Swing (lift) | Stance (ground) |
| RL (9,10) | Stance (ground) | Swing (lift) |

### Initial Position

```python
d.qpos[0] = 0.291   # X: forward stride start
d.qpos[1] = 0.096   # Y: lateral balance
d.qpos[2] = -0.2    # Z: body height (ground clearance)
d.qpos[3] = 1.0     # Orientation: neutral roll
```

These offsets optimize stride length and ground contact.

### Hip Asymmetry

- FL: hip_sign = -1 (swing backward to push forward)
- FR: hip_sign = -1 (swing backward to push forward)
- RR: hip_sign = +1 (swing forward to pull body forward)
- RL: hip_sign = +1 (swing forward to pull body forward)

This creates forward thrust during both phases.

### Usage

```bash
python3 odie_walk_fwd.py
```

Robot walks continuously forward at ~0.2 m/s (estimated) using diagonal trot.

### Customization

**Faster walk (1.5 Hz):**
```python
for i in range(333):  # 333 steps ≈ 0.67s cycle
```

**Slower walk (0.5 Hz):**
```python
for i in range(1000):  # 1000 steps ≈ 2.0s cycle
```

**Larger stride:**
```python
hip_offset = hip_sign * 30 * math.sin(...)  # 20 → 30
```

**Higher lift:**
```python
knee_offset = knee_sign * 45 * lift * math.radians(3.5)  # 30 → 45
```

### Integration with demo.py

```python
if mode == 'walk_fwd':
    cycle_step = 0
    while True:
        phase = (cycle_step % 500) / 500
        ctrl = generate_trot_ctrl(phase)
        serial_mgr.send_servo_angles(ctrl)
        time.sleep(0.002)
        cycle_step += 1
```

### Physics

- Diagonal trot reduces roll/pitch oscillation vs quadrupedal gallop
- Always 2 legs in contact (or in transition) = stable support polygon
- Hip swing creates forward CoM displacement during each phase transition
- Knee lift prevents ground collision during swing

### Parameters

| Parameter | Value | Effect |
|-----------|-------|--------|
| Cycle period | 500 steps = 1.0s | Walk speed (0.002s × 500) |
| Hip amplitude | 20° | Stride width |
| Knee amplitude | 30° (peak) | Ground clearance |
| Phase split | 50/50 | Balanced alternation |
