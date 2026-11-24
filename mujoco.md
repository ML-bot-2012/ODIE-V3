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

### Training (odie_train.py)

PPO walk trainer. The code is in `odie_train.md`, and setup, run steps, observation, action, and reward are in `odie_train_setup.md`.

Run from the folder that contains `scene.xml`:

```bash
mjpython odie_train.py walk --view
```
