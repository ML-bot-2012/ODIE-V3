# mujoco.md

## MuJoCo Simulation Files

Physics simulation environment for training PPO gaits. The sim is used to learn a walking policy, which is then mapped to the servos on the real robot.

### scene.xml - World Definition

Defines the simulation environment: ground plane, lighting, camera, and the robot.

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
- **timestep:** 0.002 s (500 Hz simulation)
- **ground:** large plane at z = -0.5
- **robot:** imported from `robot.xml`

### robot.xml - Robot Model

12-DOF quadruped with 4 legs (hip, knee, ankle on each leg).

Structure:

torso
├── FL hip → FL knee → FL ankle
├── FR hip → FR knee → FR ankle
├── RR hip → RR knee → RR ankle
└── RL hip → RL knee → RL ankle


Key specs:
- **Torso inertia:** 1.0 kg
- **Hip/Knee inertia:** 0.05 kg each
- **Ankle inertia:** 0.03 kg each
- **Joint ranges:** hip ±90°, knee -120° to 40°, ankle ±60°
- **Motor gear ratios:** hip 10, knee 5, ankle 5
- **Control range:** [-1, 1]

Truncated example (FR, RR, and RL legs follow the FL pattern):

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

      <!-- Repeat for FR, RR, RL legs -->
    </body>
  </worldbody>

  <actuator>
    <!-- Motors for each joint -->
    <motor name="FL_hip_motor" joint="FL_hip" gear="10" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_knee_motor" joint="FL_knee" gear="5" ctrllimited="true" ctrlrange="-1 1"/>
    <motor name="FL_ankle_motor" joint="FL_ankle" gear="5" ctrllimited="true" ctrlrange="-1 1"/>

    <!-- Repeat for FR, RR, RL legs -->
  </actuator>
</mujoco>
```

### Training Environment (odie_train.py)

> **Status:** `odie_train.py` is not in this repo yet. The sections below describe the design from the project notes. Replace them with the real observation, reward, and parameters from the training script before publishing.

The training script wraps the MuJoCo model as a Gymnasium environment and trains it with PPO from stable-baselines3. It uses domain randomization and a curriculum, so the policy has to handle variation rather than one fixed sim.

### Observation Space

52 dimensions (per project notes).

### Action Space

12 continuous values in [-1, 1], one per joint:
- 0-2: FL leg (hip, knee, ankle)
- 3-5: FR leg
- 6-8: RR leg
- 9-11: RL leg

Actions map to servo angles around the stand pose.

### Reward Function

Reward terms from the project notes:
- posture gate
- forward velocity
- lateral penalty
- smoothness
- fall penalty

Exact weights are set in `odie_train.py`.

### Training Runs

Logged runs from the training machine:
- Sanity run: 500k steps, about 2 minutes
- v4 run: 5M steps, about 57 minutes (about 1,400 fps)
- v3 run: 10M steps, peak reward around 4–5M steps

Use the checkpoint from the run chosen for deployment, and record which one in the README.

## Setup

```bash
pip install mujoco gymnasium stable-baselines3
python3 odie_train.py
```

Run from the directory that contains `scene.xml` and `robot.xml`.
