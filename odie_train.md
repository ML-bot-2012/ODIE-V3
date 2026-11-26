# odie_train.md – PPO Walk Training

Trains a walking policy in MuJoCo with stable-baselines3 PPO. Run from the folder that contains `scene.xml`.


Setup:

```bash
pip install mujoco gymnasium stable-baselines3
```

Run:

```bash
mjpython odie_train.py walk --view
```

Checkpoints go to `models/` as `odie_walk_v2_<steps>_steps.zip`. The final policy is `models/odie_walk_v2_final.zip`.

```python
import os, sys, math, time
os.environ['KMP_DUPLICATE_LIB_OK']='TRUE'
import numpy as np
import mujoco, mujoco.viewer
import gymnasium as gym
from gymnasium import spaces
from stable_baselines3 import PPO
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.callbacks import CheckpointCallback

GAIT_FREQ = 1.0
HIP_AMP = 0.25
KNEE_LIFT = 0.15
RESID = 0.10
TARGET_VX = 0.5
CTRL_DT = 0.02
SETTLE_STEPS = 25
GAIT_RAMP_STEPS = 50
OBS_DIM = 45

def deg_to_ctrl(deg):
    return math.radians(deg - 90)

STAND_QPOS = np.array([
    0.0, deg_to_ctrl(120), deg_to_ctrl(45),
    0.0, -deg_to_ctrl(60), deg_to_ctrl(135),
    0.0, deg_to_ctrl(60), deg_to_ctrl(135),
    0.0, -deg_to_ctrl(120), -deg_to_ctrl(45),
])

def cycloid_lift(p):
    p = p % (2*math.pi)
    return math.sin(p) if p < math.pi else 0.0

def trot_ref(phase):
    # Joint order per leg: hip (0), knee (1), ankle (2). Legs: FL, RR, FR, RL.
    pa, pb = phase, phase + math.pi
    ha = -HIP_AMP*math.sin(pa)
    ka = KNEE_LIFT*cycloid_lift(pa)
    hb = HIP_AMP*math.sin(pb)
    kb = KNEE_LIFT*cycloid_lift(pb)
    return np.array([ha, ka, 0, ha, ka, 0, hb, kb, 0, hb, kb, 0], dtype=np.float64)

class ODIETrainEnv(gym.Env):
    def __init__(self, gait_type="walk", show_viewer=False):
        self.m = mujoco.MjModel.from_xml_path('scene.xml')
        self.d = mujoco.MjData(self.m)
        self.show_viewer = show_viewer
        self.viewer = None
        self.target_z = None
        self.printed = False
        for i in range(self.m.nu):
            self.m.actuator_gainprm[i, 0] = 15.0
            self.m.actuator_biasprm[i, 1] = -15.0
            self.m.actuator_biasprm[i, 2] = -1.0
        self.observation_space = spaces.Box(-np.inf, np.inf, shape=(OBS_DIM,), dtype=np.float32)
        self.action_space = spaces.Box(-1.0, 1.0, shape=(12,), dtype=np.float32)
        self.max_steps = 500
        self.n_sub = max(1, int(round(CTRL_DT / self.m.opt.timestep)))

    def _obs(self):
        return np.concatenate([
            self.d.qpos[7:19], self.d.qvel[6:18],
            self.d.qpos[3:7], self.d.qvel[3:6],
            [math.sin(self.phase), math.cos(self.phase)],
            self.prev_action,
        ]).astype(np.float32)

    def _upright(self):
        w, x, y, z = self.d.qpos[3:7]
        return 1.0 - 2.0*(x*x + y*y)

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)
        mujoco.mj_resetData(self.m, self.d)
        self.d.qpos[0:3] = [0.291, 0.096, -0.35]
        self.d.qpos[3:7] = [1, 0, 0, 0]
        self.d.qpos[7:19] = STAND_QPOS
        self.d.ctrl[:] = STAND_QPOS
        mujoco.mj_forward(self.m, self.d)
        for _ in range(SETTLE_STEPS):
            for _ in range(self.n_sub):
                mujoco.mj_step(self.m, self.d)
        self.target_z = float(self.d.qpos[2])
        if not self.printed:
            print(f"settled standing root z: {self.target_z:.3f}  (fall below {self.target_z - 0.25:.3f})")
            self.printed = True
        self.step_count = 0
        self.phase = 0.0
        self.prev_action = np.zeros(12)
        if self.show_viewer and self.viewer is None:
            self.viewer = mujoco.viewer.launch_passive(self.m, self.d)
        return self._obs(), {}

    def step(self, action):
        action = np.clip(action, -1.0, 1.0)
        ramp = min(1.0, self.step_count / GAIT_RAMP_STEPS)
        self.d.ctrl[:] = STAND_QPOS + ramp*trot_ref(self.phase) + RESID*action
        for _ in range(self.n_sub):
            mujoco.mj_step(self.m, self.d)
        self.phase += 2*math.pi*GAIT_FREQ*CTRL_DT
        self.step_count += 1

        vx = self.d.qvel[0]
        vy = self.d.qvel[1]
        z = self.d.qpos[2]
        upright = self._upright()

        reward = 2.0*math.exp(-2.0*abs(vx - TARGET_VX))
        reward += 0.5*upright
        reward += -2.0*abs(z - self.target_z)
        reward += -0.3*abs(vy)
        reward += -0.001*float(np.sum(np.square(action)))
        reward += -0.02*float(np.sum(np.square(action - self.prev_action)))
        reward += 0.3

        fallen = z < self.target_z - 0.25 or upright < 0.5
        if fallen:
            reward -= 50.0
        self.prev_action = action.copy()

        if self.show_viewer and self.viewer and self.viewer.is_running():
            self.viewer.sync()
            time.sleep(0.01)

        truncated = self.step_count >= self.max_steps
        return self._obs(), float(reward), bool(fallen), bool(truncated), {}

if __name__ == "__main__":
    gait = sys.argv[1] if len(sys.argv) > 1 else "walk"
    show = "--view" in sys.argv
    os.makedirs("models", exist_ok=True)
    env = make_vec_env(ODIETrainEnv, n_envs=1, env_kwargs={"gait_type": gait, "show_viewer": show})
    model = PPO("MlpPolicy", env, learning_rate=3e-4, n_steps=2048, batch_size=256,
                n_epochs=10, gamma=0.99, gae_lambda=0.95, clip_range=0.2, ent_coef=0.005,
                policy_kwargs=dict(net_arch=[256, 256]), verbose=1)
    callback = CheckpointCallback(save_freq=100000, save_path="./models/", name_prefix=f"odie_{gait}_v2")
    model.learn(total_timesteps=2000000, callback=callback)
    model.save(f"models/odie_{gait}_v2_final")
    print(f"Trained {gait}. Saved to models/odie_{gait}_v2_final.zip")
```
