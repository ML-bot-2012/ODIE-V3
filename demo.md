# demo.md

## Main Demo Launcher

Central orchestration for gamepad input, model switching, IMU feedback, servo control, and telemetry.

### Overview

100 Hz control loop integrating:
- Gamepad mode selection
- PPO model subprocess management
- Background IMU reader thread
- Fall detection logic
- Serial communication with Servo2040 and IMU Pico
- Rerun telemetry dashboard

### Architecture

demo.py (main loop @ 100 Hz)
├── GamepadController (poll input)
├── SerialManager (servo + IMU comms)
├── IMUReader (background thread)
├── ModelProcess (subprocess)
└── Rerun (telemetry)


### Key Classes

#### SerialManager

```python
class SerialManager:
    def __init__(self, servo_port, imu_port):
        self.servo_port = serial.Serial(servo_port, 115200, timeout=0.01)
        self.imu_port = serial.Serial(imu_port, 115200, timeout=0.01)
    
    def send_servo_angles(self, angles):
        """Send 12 servo angles to Servo2040."""
        cmd = ','.join(map(str, angles)) + '\n'
        self.servo_port.write(cmd.encode())
    
    def read_imu(self):
        """Read latest IMU data."""
        try:
            data = self.imu_port.readline().decode().strip()
            if data:
                parts = data.split(',')
                return {
                    'ax': float(parts[0]),
                    'ay': float(parts[1]),
                    'az': float(parts[2]),
                    'gx': float(parts[3]),
                    'gy': float(parts[4]),
                    'gz': float(parts[5]),
                }
        except:
            return None
```

#### IMUReader (Background Thread)

```python
class IMUReader(threading.Thread):
    def __init__(self, serial_manager):
        super().__init__(daemon=True)
        self.serial_manager = serial_manager
        self.latest_imu = None
        self.running = True
    
    def run(self):
        while self.running:
            imu_data = self.serial_manager.read_imu()
            if imu_data:
                self.latest_imu = imu_data
            time.sleep(0.01)  # 100 Hz
    
    def get_imu(self):
        return self.latest_imu
    
    def stop(self):
        self.running = False
```

#### ModelProcess

```python
class ModelProcess:
    def __init__(self, model_name):
        self.model_name = model_name
        self.process = subprocess.Popen(
            ['python3', 'onnx_to_mcu.py', model_name],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE
        )
    
    def is_running(self):
        return self.process.poll() is None
    
    def terminate(self):
        if self.is_running():
            self.process.terminate()
            self.process.wait(timeout=5)
```

### Main Loop

```python
def main():
    # Initialize
    controller = GamepadController()
    serial_mgr = SerialManager('/dev/ttyACM0', '/dev/serial/by-id/...')
    imu_reader = IMUReader(serial_mgr)
    imu_reader.start()
    
    model_process = None
    current_mode = 'stand'
    stand_pose = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
    sit_pose = [90, 120, 45, 90, 60, 135, 90, 60, 0, 90, 120, 180]
    
    fall_detected = False
    fall_threshold = 60  # degrees
    
    rerun.init("ODIE Demo")
    
    start_time = time.time()
    iteration = 0
    
    try:
        while True:
            iteration += 1
            loop_start = time.time()
            
            # 1. Read gamepad
            mode = controller.update()
            
            # 2. Mode switching logic
            if mode != current_mode:
                if model_process:
                    model_process.terminate()
                
                current_mode = mode
                
                if MODEL_MAP[mode] is not None:
                    model_process = ModelProcess(MODEL_MAP[mode])
                
                rerun.log_mode(mode)
            
            # 3. Read IMU
            imu = imu_reader.get_imu()
            if imu:
                rerun.log_imu_data(imu)
                
                # Fall detection
                pitch = math.degrees(math.atan2(imu['ay'], imu['az']))
                roll = math.degrees(math.atan2(imu['ax'], imu['az']))
                
                if abs(pitch) > fall_threshold or abs(roll) > fall_threshold:
                    if not fall_detected:
                        fall_detected = True
                        current_mode = 'stand'
                        serial_mgr.send_servo_angles(stand_pose)
                        rerun.log_fall_detected()
                else:
                    fall_detected = False
            
            # 4. Get servo angles
            if current_mode == 'reset90':
                angles = [90] * 12
                serial_mgr.send_servo_angles(angles)
            elif current_mode == 'sit':
                serial_mgr.send_servo_angles(sit_pose)
            elif current_mode == 'stand':
                serial_mgr.send_servo_angles(stand_pose)
            elif model_process and model_process.is_running():
                # Get from model process (via subprocess or pipe)
                angles = get_model_output(model_process)
                if angles:
                    serial_mgr.send_servo_angles(angles)
                    rerun.log_servo_angles(angles)
            
            # 5. Log telemetry
            elapsed = time.time() - loop_start
            if elapsed > 0:
                fps = 1.0 / elapsed
                rerun.log_fps(fps)
            
            # 6. Maintain 100 Hz
            sleep_time = 0.01 - (time.time() - loop_start)
            if sleep_time > 0:
                time.sleep(sleep_time)
    
    except KeyboardInterrupt:
        print("Shutdown...")
    finally:
        if model_process:
            model_process.terminate()
        imu_reader.stop()
        serial_mgr.servo_port.close()
        serial_mgr.imu_port.close()
```

### Configuration

```python
SERVO_PORT = '/dev/ttyACM0'
IMU_PORT = '/dev/serial/by-id/usb-Raspberry_Pi_Pico_...-if00'
BAUD_RATE = 115200

MODEL_MAP = {
    'walk': 'walk',
    'stand': 'walk',
    'sit': 'walk',
    'wave': 'walk',
    'dance': 'dance',
    'reset90': None,
}

STAND_POSE = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
SIT_POSE = [90, 120, 45, 90, 60, 135, 90, 60, 0, 90, 120, 180]

FALL_THRESHOLD = 60  # degrees
MODE_SWITCH_DELAY = 0.5  # seconds
CONTROL_RATE = 100  # Hz
```

### Runtime

```bash
python3 demo.py
```

Outputs:
- Rerun dashboard (http://localhost:8050)
- Console logs for mode switches, falls, FPS
- Servo angles to Servo2040 at 100 Hz
