# scene.xml.md

## MuJoCo Scene - World Definition

Defines the simulation environment, physics parameters, ground plane, and robot placement for training.

### Full Code

```xml
<?xml version="1.0"?>
<mujoco model="ODIE Scene">
  <compiler angle="degree" coordinate="local"/>
  
  <option timestep="0.002" apirate="500">
    <flag energy="enable" contact="enable"/>
  </option>
  
  <default>
    <geom conaffinity="1" condim="3" friction="1 0.5 0.1"/>
    <body pos="0 0 0"/>
  </default>
  
  <asset>
    <texture type="skybox" builtin="gradient" rgb1="0.3 0.3 0.3" rgb2="0.7 0.7 0.7"/>
  </asset>
  
  <worldbody>
    <!-- Ground plane -->
    <geom name="ground" type="plane" size="10 10 0.1" pos="0 0 -0.5" 
          rgba="0.9 0.9 0.9 1" friction="1.0 0.5 0.1"/>
    
    <!-- Lighting -->
    <light pos="0 0 1" dir="0 0 -1" diffuse="0.8 0.8 0.8"/>
    <light pos="0 -2 1" dir="0 1 -1" diffuse="0.4 0.4 0.4"/>
    
    <!-- Fixed camera -->
    <camera name="fixed" pos="0 -1.5 0.5" xyaxes="1 0 0 0 1 0" fovy="45"/>
    
    <!-- Free camera (manual control) -->
    <camera name="free" pos="0 -2 1" xyaxes="1 0 0 0 1 0" fovy="45" mode="free"/>
    
    <!-- Include ODIE robot model -->
    <include file="robot.xml"/>
  </worldbody>
</mujoco>
```

### Parameters

#### Compiler
- `angle="degree"` - Use degrees for joint ranges (vs radians)
- `coordinate="local"` - Use local coordinates for body frames

#### Physics (option)
- `timestep="0.002"` - 0.002s per step = 500 Hz simulation
- `apirate="500"` - Python API runs at 500 Hz
- `energy="enable"` - Track kinetic/potential energy (optional)
- `contact="enable"` - Enable contact dynamics

#### Ground Plane (geom)
- `type="plane"` - Infinite plane
- `size="10 10 0.1"` - 10m × 10m, 0.1m thickness
- `pos="0 0 -0.5"` - Positioned 0.5m below origin
- `friction="1.0 0.5 0.1"` - Sliding, rolling, torsional friction
- `rgba="0.9 0.9 0.9 1"` - Light gray color

#### Lighting
- Main light: directly overhead
- Fill light: from behind, reduces shadows

#### Cameras
- `fixed` - Bird's eye view for training visualization
- `free` - User-controlled camera for interactive simulation

### Integration with Training

```python
# In odie_train.py
import mujoco

model = mujoco.MjModel.from_xml_path('mujoco/scene.xml')
data = mujoco.MjData(model)

# Simulation loop
for step in range(num_steps):
    mujoco.mj_step(model, data)
    # step takes timestep=0.002s
    # 500 steps = 1 second real time
```

### Timestep Selection

| Timestep | Frequency | Use Case |
|----------|-----------|----------|
| 0.001s | 1000 Hz | Very accurate but slow |
| 0.002s | 500 Hz | Balanced (recommended) |
| 0.005s | 200 Hz | Faster, less accurate |
| 0.01s | 100 Hz | Very fast, lower precision |

For ODIE: 0.002s provides good stability for servo control while remaining fast for training.

### Friction Model

friction=[sliding, rolling, torsional]
= [1.0, 0.5, 0.1]


- **Sliding:** Friction parallel to contact surface
- **Rolling:** Resistance to rolling motion
- **Torsional:** Twisting friction

These values prevent servos from slipping and keep the robot stable on the ground plane.

### Camera Usage

**Fixed camera (training visualization):**
```python
viewer = mujoco.viewer.launch_passive(model, data)
# Automatically uses 'fixed' camera
viewer.sync()
```

**Free camera (interactive inspection):**
```python
viewer = mujoco.viewer.launch(model, data)
# User can pan/zoom with mouse
```

### Sky/Environment

Gradient skybox with dark bottom (0.3) to light top (0.7) provides realistic lighting context.

### Troubleshooting

**Robot vibrates/explodes:**
- Reduce timestep (0.002 → 0.001)
- Increase contact damping in robot.xml
- Check motor control limits

**Robot moves slowly:**
- Increase timestep (0.002 → 0.005)
- Reduce contact friction
- Increase motor torque

**Camera view is dark:**
- Adjust light intensity in `diffuse` attribute
- Reposition lights
- Use viewer's light controls
