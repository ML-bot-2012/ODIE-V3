# robot.xml.md

## MuJoCo Robot Model - ODIE Hardware Definition

Complete 12-DOF quadruped mechanical model for physics simulation training.

### Full Code

```xml
<?xml version="1.0"?>
<mujoco model="ODIE">
  <compiler angle="degree" coordinate="local"/>
  
  <default>
    <joint type="hinge" axis="0 0 1" range="-90 90" damping="0.5" frictionloss="0.01"/>
    <geom type="capsule" density="1000" friction="1.0 0.5 0.1"/>
    <inertial pos="0 0 0"/>
  </default>
  
  <worldbody>
    <body name="torso" pos="0 0 0.2">
      <!-- Torso (main body) -->
      <inertial mass="1.0" pos="0 0 0" 
                 diaginv="1.0 1.0 0.5"/>
      <geom name="torso_geom" type="box" size="0.1 0.05 0.05" 
             pos="0 0 0" rgba="0 0 1 1"/>
      
      <!-- ========== FRONT LEFT LEG ========== -->
      <body name="FL_hip" pos="0.08 0.08 0">
        <inertial mass="0.05" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
        <joint name="FL_hip" type="hinge" axis="0 0 1" range="-90 90" damping="0.5"/>
        <geom name="FL_hip_geom" type="capsule" fromto="0 0 0 0.02 0.02 0" size="0.01" 
               rgba="0.5 0 0 1"/>
        
        <body name="FL_knee" pos="0.02 0.02 0">
          <inertial mass="0.05" pos="0 0 -0.025" diaginv="1.0 1.0 1.0"/>
          <joint name="FL_knee" type="hinge" axis="0 1 0" range="-120 40" damping="0.5"/>
          <geom name="FL_knee_geom" type="capsule" fromto="0 0 0 0.03 0 -0.05" size="0.01" 
                 rgba="0.7 0 0 1"/>
          
          <body name="FL_ankle" pos="0.03 0 -0.05">
            <inertial mass="0.03" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
            <joint name="FL_ankle" type="hinge" axis="1 0 0" range="-60 60" damping="0.5"/>
            <geom name="FL_ankle_geom" type="sphere" pos="0 0 0" size="0.015" 
                   rgba="1 0 0 1"/>
          </body>
        </body>
      </body>
      
      <!-- ========== FRONT RIGHT LEG ========== -->
      <body name="FR_hip" pos="0.08 -0.08 0">
        <inertial mass="0.05" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
        <joint name="FR_hip" type="hinge" axis="0 0 1" range="-90 90" damping="0.5"/>
        <geom name="FR_hip_geom" type="capsule" fromto="0 0 0 0.02 -0.02 0" size="0.01" 
               rgba="0.5 0 0 1"/>
        
        <body name="FR_knee" pos="0.02 -0.02 0">
          <inertial mass="0.05" pos="0 0 -0.025" diaginv="1.0 1.0 1.0"/>
          <joint name="FR_knee" type="hinge" axis="0 1 0" range="-120 40" damping="0.5"/>
          <geom name="FR_knee_geom" type="capsule" fromto="0 0 0 0.03 0 -0.05" size="0.01" 
                 rgba="0.7 0 0 1"/>
          
          <body name="FR_ankle" pos="0.03 0 -0.05">
            <inertial mass="0.03" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
            <joint name="FR_ankle" type="hinge" axis="1 0 0" range="-60 60" damping="0.5"/>
            <geom name="FR_ankle_geom" type="sphere" pos="0 0 0" size="0.015" 
                   rgba="1 0 0 1"/>
          </body>
        </body>
      </body>
      
      <!-- ========== REAR RIGHT LEG ========== -->
      <body name="RR_hip" pos="-0.08 -0.08 0">
        <inertial mass="0.05" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
        <joint name="RR_hip" type="hinge" axis="0 0 1" range="-90 90" damping="0.5"/>
        <geom name="RR_hip_geom" type="capsule" fromto="0 0 0 -0.02 -0.02 0" size="0.01" 
               rgba="0.5 0 0 1"/>
        
        <body name="RR_knee" pos="-0.02 -0.02 0">
          <inertial mass="0.05" pos="0 0 -0.025" diaginv="1.0 1.0 1.0"/>
          <joint name="RR_knee" type="hinge" axis="0 1 0" range="-120 40" damping="0.5"/>
          <geom name="RR_knee_geom" type="capsule" fromto="0 0 0 -0.03 0 -0.05" size="0.01" 
                 rgba="0.7 0 0 1"/>
          
          <body name="RR_ankle" pos="-0.03 0 -0.05">
            <inertial mass="0.03" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
            <joint name="RR_ankle" type="hinge" axis="1 0 0" range="-60 60" damping="0.5"/>
            <geom name="RR_ankle_geom" type="sphere" pos="0 0 0" size="0.015" 
                   rgba="1 0 0 1"/>
          </body>
        </body>
      </body>
      
      <!-- ========== REAR LEFT LEG ========== -->
      <body name="RL_hip" pos="-0.08 0.08 0">
        <inertial mass="0.05" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
        <joint name="RL_hip" type="hinge" axis="0 0 1" range="-90 90" damping="0.5"/>
        <geom name="RL_hip_geom" type="capsule" fromto="0 0 0 -0.02 0.02 0" size="0.01" 
               rgba="0.5 0 0 1"/>
        
        <body name="RL_knee" pos="-0.02 0.02 0">
          <inertial mass="0.05" pos="0 0 -0.025" diaginv="1.0 1.0 1.0"/>
          <joint name="RL_knee" type="hinge" axis="0 1 0" range="-120 40" damping="0.5"/>
          <geom name="RL_knee_geom" type="capsule" fromto="0 0 0 -0.03 0 -0.05" size="0.01" 
                 rgba="0.7 0 0 1"/>
          
          <body name="RL_ankle" pos="-0.03 0 -0.05">
            <inertial mass="0.03" pos="0 0 0" diaginv="1.0 1.0 1.0"/>
            <joint name="RL_ankle" type="hinge" axis="1 0 0" range="-60 60" damping="0.5"/>
            <geom name="RL_ankle_geom" type="sphere" pos="0 0 0" size="0.015" 
                   rgba="1 0 0 1"/>
          </body>
        </body>
      </body>
    </body>
  </worldbody>
  
  <!-- ========== ACTUATORS (Motors) ========== -->
  <actuator>
    <!-- Front Left Leg -->
    <motor name="FL_hip_motor" joint="FL_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_knee_motor" joint="FL_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_ankle_motor" joint="FL_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    
    <!-- Front Right Leg -->
    <motor name="FR_hip_motor" joint="FR_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FR_knee_motor" joint="FR_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FR_ankle_motor" joint="FR_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    
    <!-- Rear Right Leg -->
    <motor name="RR_hip_motor" joint="RR_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="RR_knee_motor" joint="RR_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="RR_ankle_motor" joint="RR_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    
    <!-- Rear Left Leg -->
    <motor name="RL_hip_motor" joint="RL_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="RL_knee_motor" joint="RL_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="RL_ankle_motor" joint="RL_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
  </actuator>
</mujoco>
```

### Structure Overview

Torso (1.0 kg, blue box)
├── Front Left Hip → Knee → Ankle (foot)
├── Front Right Hip → Knee → Ankle (foot)
├── Rear Right Hip → Knee → Ankle (foot)
└── Rear Left Hip → Knee → Ankle (foot)

Total: 1 torso + 12 joints (3 per leg × 4 legs)


### Body Hierarchy

Each leg is a chain of 3 bodies:
1. **Hip** - Rotates side-to-side (abduction), mounted on torso
2. **Knee** - Bends forward/back, mounted on hip
3. **Ankle** - Foot tilt, mounted on knee

### Inertial Properties

| Part | Mass (kg) | Description |
|------|-----------|-------------|
| Torso | 1.0 | Main body |
| Hip | 0.05 | 4× per robot |
| Knee | 0.05 | 4× per robot |
| Ankle | 0.03 | 4× per robot (foot) |
| **Total** | **1.48 kg** | Real robot ≈ 2kg |

### Joint Ranges

| Joint | Type | Range | Axis |
|-------|------|-------|------|
| Hip | Hinge | ±90° | Z (vertical) |
| Knee | Hinge | -120° to +40° | Y (lateral) |
| Ankle | Hinge | ±60° | X (forward) |

Note: Knee range is asymmetric (more bend than extension) for realistic leg motion.

### Motor Specifications

All motors use `ctrllimited="true" ctrlrange="-1 1"` for normalized control.

```python
# In PPO training
action = model.predict(observation)  # Returns [-1, 1] for each DOF
# MuJoCo automatically scales to motor torque based on gear ratio
```

**Gear Ratios:**
- Hip: 10× (high torque needed for side swing)
- Knee: 5× (moderate torque)
- Ankle: 5× (moderate torque)

### Damping & Friction

damping="0.5" # Joint damping (prevents oscillation)
frictionloss="0.01" # Internal friction


Prevents servos from acting like pure springs; adds realistic resistance.

### Geometry

**Hip segment:** Capsule from torso attachment to knee
**Knee segment:** Capsule from hip to ankle  
**Ankle (foot):** Sphere (simple contact point)

Size `0.01` ≈ 1cm radius (proportional to real servos).

### Color Coding

- **Blue:** Torso
- **Red (0.5):** Hip motors
- **Red (0.7):** Knee segments
- **Bright red (1.0):** Ankle feet (contact points)

### Position Offsets

Front legs: `pos="0.08 ±0.08 0"` (forward, outward)
Rear legs: `pos="-0.08 ±0.08 0"` (back, outward)

Creates a 16cm wheelbase and 16cm track width.

### Training Loop Integration

```python
class ODIEEnv(gym.Env):
    def __init__(self):
        self.model = mujoco.MjModel.from_xml_path('mujoco/robot.xml')
        self.data = mujoco.MjData(self.model)
    
    def step(self, action):
        # action: [12] array, each element in [-1, 1]
        self.data.ctrl[:] = action
        mujoco.mj_step(self.model, self.data)
        
        obs = self._get_obs()
        reward = self._compute_reward()
        return obs, reward, done, {}
```

### Observation Space (13 dims)

```python
obs[0:3] = self.data.qpos[0:3]      # Position (x, y, z)
obs[3:6] = self.data.qvel[0:3]      # Velocity (vx, vy, vz)
obs[6] = self.data.qpos[2]          # Height (z)
obs[7:13] = zeros                   # Padding for future sensors
```

### Reward Function

```python
forward_reward = x_position * 0.5
energy_penalty = -np.sum(np.abs(action)) * 0.01
stability = 0.1 if height > 0.15 else -0.5

total_reward = forward_reward + energy_penalty + stability
```

Encourages forward progress, penalizes energy use, rewards stable posture.

### Troubleshooting

**Robot vibrates excessively:**
- Increase damping (0.5 → 1.0)
- Reduce gear ratios
- Reduce timestep

**Robot collapses:**
- Increase damping
- Check inertial properties
- Increase motor torque (gear ratio)

**Slow movement:**
- Reduce damping
- Increase gear ratios
- Increase control range

**Joint limits violated:**
- Ensure PPO output is [-1, 1]
- Check range definitions
- Add constraint penalties
