# odie_train_setup.md – Running the PPO Walk Trainer

Setup, run steps, and what the trainer's observation, action, and reward mean. The code is in `odie_train.md`.

 
### Folder

Run from the folder that contains `scene.xml`. The script loads `scene.xml` by relative path.

```bash
mkdir -p models
```

### Setup

```bash
pip install mujoco gymnasium stable-baselines3
```

### Run

```bash
mjpython odie_train.py walk --view
```

Use `mjpython` on macOS when you want the viewer. Without `--view`, the trainer runs headless and is faster.

Checkpoints are saved every 100k steps to `models/odie_walk_v2_<steps>_steps.zip`. The final policy is `models/odie_walk_v2_final.zip`.

### Startup check

The first line of output is the settled standing height, for example `settled standing root z: -0.304 (fall below -0.554)`. The fall threshold is set 0.25 below that height. A robot that never settles will fall on every episode.

### Observation (45 values)

| Index | Contents |
|---|---|
| 0–11 | Joint angles (`qpos[7:19]`), hip/knee/ankle per leg |
| 12–23 | Joint velocities (`qvel[6:18]`) |
| 24–27 | Body orientation quaternion (`qpos[3:7]`) |
| 28–30 | Body angular velocity (`qvel[3:6]`) |
| 31–32 | Gait phase as sin and cos |
| 33–44 | Previous action |

### Action (12 values, −1 to 1)

Each action is a residual on top of the standing pose plus the trot reference:

```
ctrl = STAND_QPOS + gait_ramp * trot_ref(phase) + 0.10 * action
```

The gait ramps in over the first 50 steps (1 second) so the robot doesn't tip on the first step.

### Reward

| Term | Meaning |
|---|---|
| `2·exp(−2·|vx − 0.5|)` | Track forward speed of 0.5 |
| `0.5·upright` | Stay upright |
| `−2·|z − standing_z|` | Hold height |
| `−0.3·|vy|` | Don't drift sideways |
| `−0.001·Σaction²` | Small effort penalty |
| `−0.02·Σ(Δaction)²` | Smooth actions |
| `+0.3` per step | Alive bonus |
| `−50` on fall | Fall: height drop of 0.25 or tilt past 60° |

An episode ends on fall or after 500 steps.
