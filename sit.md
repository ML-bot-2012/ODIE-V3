# sit.md

import mujoco, mujoco.viewer
import numpy as np
import time, math

m = mujoco.MjModel.from_xml_path('scene.xml')
d = mujoco.MjData(m)

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

mujoco.mj_resetData(m, d)
d.qpos[2] = -0.2
d.qpos[3] = 1.0
d.ctrl[:] = STAND
mujoco.mj_forward(m, d)

print("Sit. Close window to exit.")
with mujoco.viewer.launch_passive(m, d) as v:
    for i in range(60):
        t = i / 60
        d.ctrl[:] = STAND + (SIT - STAND) * t
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.02)
    while v.is_running():
        d.ctrl[:] = SIT
        mujoco.mj_step(m, d)
        v.sync()
        time.sleep(0.002)

## Motion Analysis
Type: Pose transition with hold
Interpolation: 60 frames @ 0.02s/frame = 1.0 second blend

STAND Baseline:
- FL: [90°, 120°, 45°]
- RR: [90°, 60°, 135°]
- FR: [90°, 60°, 135°]
- RL: [90°, 120°, 45°]

SIT Target:
- FL: [90°, 120°, 45°] (unchanged)
- RR: [90°, 60°, 0°] (rear-right ankle bends 135° → 0°)
- FR: [90°, 60°, 135°] (unchanged)
- RL: [90°, 120°, 180°] (rear-left ankle extends 45° → 180°)

Rear Legs: Both rear ankles flex to lower center of mass for sitting pose—RR pulls in (0°), RL extends out (180°), creating low-profile stable base.
Front Legs: Unchanged—remain in stand position for balance.

Control Flow:
1. Frames 0–59: Linear interpolation t ∈ [0, 1]
2. Frame 60+: Hold SIT indefinitely (0.002s loop matches physics)

Motor Gain: 15.0 provides smooth tracking without oscillation.
