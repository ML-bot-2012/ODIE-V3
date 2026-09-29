python
#!/usr/bin/env python3
"""
ODIE Rerun Dashboard - Real-time telemetry visualization
Logs robot state, IMU data, servo positions, mode, and performance metrics
"""

import rerun as rr
import numpy as np
from datetime import datetime

# ============================================================================
# RERUN INITIALIZATION
# ============================================================================

def init_dashboard():
    """Initialize Rerun dashboard."""
    rr.init("ODIE Demo", spawn=True)
    rr.log("world", rr.ViewCoordinates.RDF)
    print("[RERUN] Dashboard initialized")

# ============================================================================
# LOGGING FUNCTIONS
# ============================================================================

def log_imu_data(timestamp, ax, ay, az, gx, gy, gz):
    """Log IMU accelerometer and gyroscope data."""
    accel = np.array([ax, ay, az])
    gyro = np.array([gx, gy, gz])
    
    rr.log(f"imu/accel/magnitude", rr.Scalar(np.linalg.norm(accel)))
    rr.log(f"imu/accel/x", rr.Scalar(ax))
    rr.log(f"imu/accel/y", rr.Scalar(ay))
    rr.log(f"imu/accel/z", rr.Scalar(az))
    
    rr.log(f"imu/gyro/x", rr.Scalar(gx))
    rr.log(f"imu/gyro/y", rr.Scalar(gy))
    rr.log(f"imu/gyro/z", rr.Scalar(gz))

def log_servo_angles(angles):
    """Log all 12 servo angles."""
    rr.log("servos/all", rr.Bars(values=angles, label_values=[f"S{i}" for i in range(12)]))
    
    # Log by leg
    rr.log("servos/FL_hip", rr.Scalar(angles[0]))
    rr.log("servos/FL_knee", rr.Scalar(angles[1]))
    rr.log("servos/FR_hip", rr.Scalar(angles[3]))
    rr.log("servos/RR_hip", rr.Scalar(angles[6]))
    rr.log("servos/RL_hip", rr.Scalar(angles[9]))

def log_mode(mode_name):
    """Log current movement mode."""
    rr.log("status/mode", rr.TextLog(mode_name))

def log_fall_detected():
    """Log fall detection event."""
    rr.log("status/fall_alert", rr.TextLog("FALL DETECTED - Returning to STAND"))

def log_battery(voltage):
    """Log battery voltage (if available)."""
    rr.log("power/battery_v", rr.Scalar(voltage))

def log_fps(fps):
    """Log control loop FPS."""
    rr.log("performance/control_fps", rr.Scalar(fps))

def log_inference_time(ms):
    """Log model inference time."""
    rr.log("performance/inference_ms", rr.Scalar(ms))

# ============================================================================
# DASHBOARD LAYOUT
# ============================================================================

def setup_layout():
    """Configure Rerun dashboard layout."""
    rr.log(
        "world",
        rr.BarChart(
            values=[90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45],
        ),
    )

if __name__ == '__main__':
    init_dashboard()
    print("[RERUN] Use this module's functions in demo.py to log data")

odie_rerun.py summary: Rerun telemetry logging functions. Provides wrappers to log IMU data (accel/gyro), servo angles, current mode, fall alerts, battery voltage, and performance metrics. Called from demo.py during main loop. Creates live dashboards for monitoring robot state in real-time.
