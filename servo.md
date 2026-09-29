# servo.md

## Servo Control & Hardware

Complete servo specification, calibration, and communication protocol.

### Servo Specifications

- **Type:** Dynamixel AX-12A or MG996R
- **Range:** 0–180°
- **Torque:** ~1.5–2.0 kg·cm
- **Speed:** 0.11–0.19 sec/60°
- **Voltage:** 4.5–5.5V (5V nominal)
- **Weight:** ~50g

### PWM to Angle Conversion

```python
def pwm_to_angle(pwm_us):
    """Convert PWM microseconds to servo angle."""
    return (pwm_us - 600) / (2400 - 600) * 180

def angle_to_pwm(angle_deg):
    """Convert servo angle to PWM microseconds."""
    return 600 + (angle_deg / 180) * (2400 - 600)
```

Conversion table:

| Angle (°) | PWM (µs) |
|-----------|----------|
| 0 | 600 |
| 45 | 1050 |
| 90 | 1500 |
| 135 | 1950 |
| 180 | 2400 |

### Channel Mapping

Front Left (FL) Front Right (FR) Rear Right (RR) Rear Left (RL)
Ch 0: Hip Ch 3: Hip Ch 6: Hip Ch 9: Hip
Ch 1: Knee Ch 4: Knee Ch 7: Knee Ch 10: Knee
Ch 2: Ankle Ch 5: Ankle Ch 8: Ankle Ch 11: Ankle


### Default Poses

**STAND:**
```python
STAND = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
```

| Ch | Angle | Joint | Desc |
|----|-------|-------|------|
| 0 | 90 | FL Hip | Center |
| 1 | 120 | FL Knee | Forward |
| 2 | 45 | FL Ankle | Down |
| 3 | 90 | FR Hip | Center |
| 4 | 60 | FR Knee | Fwd (mirror) |
| 5 | 135 | FR Ankle | Down |
| 6 | 90 | RR Hip | Center |
| 7 | 60 | RR Knee | Forward |
| 8 | 135 | RR Ankle | Down |
| 9 | 90 | RL Hip | Center |
| 10 | 120 | RL Knee | Fwd (mirror) |
| 11 | 45 | RL Ankle | Down |

**SIT:**
```python
SIT = [90, 120, 45, 90, 60, 135, 90, 60, 0, 90, 120, 180]
```
Rear ankles fully bent for compact posture.

### Serial Communication

**Servo2040:**
- Port: /dev/ttyACM0
- Baud: 115200
- Format: Comma-separated angles (0-180), newline-terminated
- Example: `90,120,45,90,60,135,90,60,135,90,120,45\n`

**MicroPython Code (servo2040_main.py):**

```python
import serial
from machine import Pin
from servo import ServoCluster

# Initialize servo controller
servo_cluster = ServoCluster(pio=0, sm=0, pins=[0,1,2,...,11])

# Default calibration
for servo in servo_cluster:
    servo.calibration(min_us=600, max_us=2400, min_angle=0, max_angle=180)

# Main loop
ser = serial.Serial()

while True:
    if ser.any():
        data = ser.readline().decode().strip()
        try:
            angles = list(map(int, data.split(',')))
            if len(angles) == 12:
                for i, angle in enumerate(angles):
                    servo_cluster[i].value(angle)
        except:
            pass
```

### Calibration Procedure

**Step 1: Zero All Servos**

```python
angles = [90] * 12
ser.write((','.join(map(str, angles)) + '\n').encode())
```

**Step 2: Visual Inspection**

Check that all servo horns are centered. If misaligned, loosen horn set screw, rotate, re-tighten.

**Step 3: Test Full Range**

```python
# Test 0°
angles = [0] * 12
ser.write((cmd + '\n').encode())
time.sleep(1)

# Test 180°
angles = [180] * 12
ser.write((cmd + '\n').encode())
time.sleep(1)

# Return to 90°
angles = [90] * 12
ser.write((cmd + '\n').encode())
```

**Step 4: Fine-Tune**

For any servo not properly centered at 90°:
1. Loosen horn set screw
2. Rotate horn slightly (~5° at a time)
3. Tighten set screw
4. Re-test

### MuJoCo Conversion

In simulation, servo angles (0-180°) convert to control values (radians around neutral):

```python
def deg_to_ctrl(servo_deg):
    return math.radians(servo_deg - 90)
```

| Servo (°) | Control (rad) | Description |
|-----------|---------------|-------------|
| 90 | 0 | Neutral |
| 0 | -π/2 ≈ -1.57 | Full reverse |
| 180 | π/2 ≈ +1.57 | Full forward |

### Inference Conversion

From PPO model output [-1, 1] to servo angles [0, 180]:

```python
def action_to_servo(action_value):
    # action_value in [-1, 1]
    return (action_value + 1.0) * 90.0  # Result in [0, 180]
```

### Power Requirements

- Per servo: ~1A max (stalled)
- All 12 servos: 12A peak
- Recommended supply: 5V, 15A regulated PSU
- Separate from Pi power

### Maintenance

- **Dust:** Keep servos dry; blow out dust periodically
- **Lubrication:** Apply light synthetic grease to gears (rare-earth servo grease)
- **Cables:** Use shielded serial cables; ferrite cores on USB
- **Connections:** Solder or use quality connectors; avoid loose crimps

### Troubleshooting

**Servo not moving:**
- Check power supply (5V ±0.5V)
- Verify serial cable connection
- Test command: `echo "90,90,90,..." > /dev/ttyACM0`
- Check horn is properly installed

**Servo jitter/noise:**
- Use shielded cables
- Add ferrite core to USB
- Ensure common ground
- Reduce cable length
- Check for electrical noise nearby

**Wrong angle:**
- Verify calibration (600–2400µs)
- Check horn mechanical position
- Recalibrate if horn slipped
