# mujoco.md

## MuJoCo Simulation Files

Physics simulation environment for training PPO gaits.

### scene.xml - World Definition

Defines the simulation environment: ground plane, gravity, timestep, and robot placement.

```xml
<?xml version="1.0"?>
<mujoco model="ODIE Scene">
  <compiler angle="degree" coordinate="local"/>
  
  <option timestep="0.002">
    <flag energy="enable"/>
  </option>
  
  <default>
    <geom conaffinity="1" condim="3"/>
    <body pos="0 0 0"/>
  </default>
  
  <worldbody>
    <!-- Ground plane -->
    <geom name="ground" type="plane" size="10 10 0.1" pos="0 0 -0.5" rgba="0.9 0.9 0.9 1"/>
    
    <!-- Lighting -->
    <light pos="0 0 1" dir="0 0 -1"/>
    
    <!-- Camera -->
    <camera name="fixed" pos="0 -1 0.5" xyaxes="1 0 0 0 1 0"/>
    
    <!-- Include robot -->
    <include file="robot.xml"/>
  </worldbody>
</mujoco>
```

Key elements:
- **timestep:** 0.002s (500 Hz simulation)
- **ground:** Large plane at z=-0.5
- **robot:** Imported from robot.xml

### robot.xml - Robot Model

12-DOF quadruped with 4 legs (hip, knee, ankle each).

Structure:

torso (1kg)
├── FL hip → FL knee → FL ankle
├── FR hip → FR knee → FR ankle
├── RR hip → RR knee → RR ankle
└── RL hip → RL knee → RL ankle


Key specs:
- **Torso inertia:** 1.0 kg
- **Hip/Knee inertia:** 0.05 kg each
- **Ankle inertia:** 0.03 kg each
- **Joint ranges:** Hip ±90°, Knee -120 to 40°, Ankle ±60°
- **Motor gear ratios:** Hip 10, Knee 5, Ankle 5
- **Control range:** [-1, 1]

Truncated example:

```xml
<?xml version="1.0"?>
<mujoco model="ODIE">
  <worldbody>
    <!-- Torso -->
    <body name="torso" pos="0 0 0.2">
      <inertial mass="1.0" diaginv="1 1 0.5"/>
      <geom type="box" size="0.1 0.05 0.05" rgba="0 0 1 1"/>
      
      <!-- Front Left Leg -->
      <body name="FL_hip" pos="0.08 0.08 0">
        <joint name="FL_hip" type="hinge" axis="0 0 1" range="-90 90"/>
        <inertial mass="0.05" diaginv="1 1 1"/>
        <geom type="capsule" fromto="0 0 0 0.02 0.02 0" size="0.01"/>
        
        <body name="FL_knee" pos="0.02 0.02 0">
          <joint name="FL_knee" type="hinge" axis="0 1 0" range="-120 40"/>
          <inertial mass="0.05" diaginv="1 1 1"/>
          <geom type="capsule" fromto="0 0 0 0.03 0 -0.05" size="0.01"/>
          
          <body name="FL_ankle" pos="0.03 0 -0.05">
            <joint name="FL_ankle" type="hinge" axis="1 0 0" range="-60 60"/>
            <inertial mass="0.03" diaginv="1 1 1"/>
            <geom type="sphere" pos="0 0 0" size="0.015" rgba="1 0 0 1"/>
          </body>
        </body>
      </body>
      
      <!-- Repeat for FR, RR, RL legs... -->
    </body>
  </worldbody>
  
  <actuator>
    <!-- Motors for each joint -->
    <motor name="FL_hip_motor" joint="FL_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_knee_motor" joint="FL_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_ankle_motor" joint="FL_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    
    <!-- Repeat for FR, RR, RL legs... -->
  </actuator>
</mujoco>
```

### Training Environment (odie_train.py)

Gymnasium wrapper around MuJoCo:

```python
class ODIEEnv(gym.Env):
    def __init__(self):
        self.model = mujoco.MjModel.from_xml_path('mujoco/scene.xml')
        self.data = mujoco.MjData(self.model)
        self.viewer = None
        
        self.observation_space = gym.spaces.Box(
            low=-np.inf, high=np.inf, shape=(13,)
        )
        self.action_space = gym.spaces.Box(
            low=-1, high=1, shape=(12,)
        )
    
    def reset(self):
        mujoco.mj_resetData(self.model, self.data)
        return self._get_obs()
    
    def step(self, action):
        # Convert action [-1, 1] to servo angles [0, 180]
        servo_angles = (action + 1.0) * 90.0
        
        # Convert servo angles to MuJoCo control
        ctrl = [math.radians(angle - 90) for angle in servo_angles]
        self.data.ctrl[:] = ctrl
        
        # Simulate one step
        mujoco.mj_step(self.model, self.data)
        
        # Calculate reward
        forward_reward = self.data.qpos[0] * 0.5
        energy_penalty = -np.sum(np.abs(action)) * 0.01
        height = self.data.qpos[2]
        stability_bonus = 0.1 if height > 0.15 else -0.5
        
        reward = forward_reward + energy_penalty + stability_bonus
        
        # Check termination (fell over)
        done = height < 0.1 or height > 1.0
        
        return self._get_obs(), reward, done, {}
    
    def _get_obs(self):
        obs = np.zeros(13)
        obs[0:3] = self.data.qpos[0:3]  # Position
        obs[3:6] = self.data.qvel[0:3]  # Velocity
        obs[6] = self.data.qpos[2]      # Height
        return obs
```

### Observation Space

13 dimensions:
- 0-2: Position (x, y, z)
- 3-5: Velocity (vx, vy, vz)
- 6: Height (z)
- 7-12: Padding (zeros for future sensor data)

### Action Space

12 continuous values [-1, 1]:
- 0-2: FL leg (hip, knee, ankle)
- 3-5: FR leg
- 6-8: RR leg
- 9-11: RL leg

Converted to servo angles: `angle = (action + 1) * 90`

### Reward Function

```python
forward_reward = x_position * 0.5       # 50% weight on progress
energy_penalty = -sum(|actions|) * 0.01 # Penalize energy use
stability = +0.1 if height > 0.15 else -0.5

total = forward_reward + energy_penalty + stability
```

### Training Parameters

```python
model = PPO(
    'MlpPolicy',
    env,
    learning_rate=3e-4,
    n_steps=2048,
    batch_size=64,
    n_epochs=10,
    verbose=1,
)

model.learn(total_timesteps=50000)
```

Training on GPU: ~2-4 hours for 50k steps
