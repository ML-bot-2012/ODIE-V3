# odie_rerun.md

## Rerun Telemetry Logging

Real-time visualization dashboard for monitoring robot state, sensor data, and performance.

### Overview

Rerun SDK for streaming telemetry to http://localhost:8050 during deployment.

### Key Functions

#### IMU Data

```python
def log_imu_data(imu_dict):
    """Log accelerometer and gyroscope data."""
    if not imu_dict:
        return
    
    # Acceleration
    accel = np.array([imu_dict['ax'], imu_dict['ay'], imu_dict['az']])
    rr.log("imu/accel", rr.Scalar(np.linalg.norm(accel)))
    
    # Gyroscope
    gyro = np.array([imu_dict['gx'], imu_dict['gy'], imu_dict['gz']])
    rr.log("imu/gyro", rr.Scalar(np.linalg.norm(gyro)))
    
    # Individual axes
    rr.log("imu/ax", rr.Scalar(imu_dict['ax']))
    rr.log("imu/ay", rr.Scalar(imu_dict['ay']))
    rr.log("imu/az", rr.Scalar(imu_dict['az']))
```

#### Servo Angles

```python
def log_servo_angles(angles):
    """Log 12 servo angles."""
    rr.log("servos/fl_hip", rr.Scalar(angles[0]))
    rr.log("servos/fl_knee", rr.Scalar(angles[1]))
    rr.log("servos/fl_ankle", rr.Scalar(angles[2]))
    
    rr.log("servos/fr_hip", rr.Scalar(angles[3]))
    rr.log("servos/fr_knee", rr.Scalar(angles[4]))
    rr.log("servos/fr_ankle", rr.Scalar(angles[5]))
    
    rr.log("servos/rr_hip", rr.Scalar(angles[6]))
    rr.log("servos/rr_knee", rr.Scalar(angles[7]))
    rr.log("servos/rr_ankle", rr.Scalar(angles[8]))
    
    rr.log("servos/rl_hip", rr.Scalar(angles[9]))
    rr.log("servos/rl_knee", rr.Scalar(angles[10]))
    rr.log("servos/rl_ankle", rr.Scalar(angles[11]))
```

#### Mode Logging

```python
def log_mode(mode):
    """Log current control mode."""
    rr.log("status/mode", rr.TextLog(mode))
```

#### Fall Detection

```python
def log_fall_detected():
    """Log fall event."""
    rr.log("status/fall", rr.TextLog("FALL DETECTED"))
```

#### Performance Metrics

```python
def log_fps(fps_value):
    """Log control loop FPS."""
    rr.log("performance/fps", rr.Scalar(fps_value))

def log_inference_time(ms):
    """Log model inference latency."""
    rr.log("performance/inference_ms", rr.Scalar(ms))

def log_battery(voltage):
    """Log battery voltage."""
    rr.log("power/battery_v", rr.Scalar(voltage))
```

### Integration in demo.py

```python
import rerun as rr

# Initialize
rr.init("ODIE Demo")

# In main loop
if imu:
    log_imu_data(imu)

if angles:
    log_servo_angles(angles)

log_fps(current_fps)
log_mode(current_mode)

if fall_detected:
    log_fall_detected()
```

### Dashboard Access

Open browser: http://localhost:8050

Displays real-time:
- IMU acceleration/gyro
- Servo angle values for all 12 joints
- Current mode (walk, dance, sit, etc)
- FPS and inference latency
- Fall detection events
- Battery voltage

### Logging Frequency

- IMU: 100 Hz (from sensor)
- Servos: 100 Hz (from control loop)
- Performance: 10 Hz (to avoid logging spam)
- Events: On-demand (fall, mode switch)

### Data Types

```python
rr.Scalar(float_value)      # Single scalar
rr.TextLog(string_value)    # Text message
rr.Tensor(np_array)         # Vector/matrix
rr.Image(cv2_image)         # Frame (optional)
```

### Memory Efficiency

Rerun buffers data locally; to reduce storage:

```python
# Log every 10th frame instead of every frame
if iteration % 10 == 0:
    log_imu_data(imu)
```

### Cleanup

Automatic on demo.py exit; manual flush:

```python
rr.flush()
```
