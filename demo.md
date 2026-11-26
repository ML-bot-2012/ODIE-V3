# demo.md

## Main Demo Launcher

Central orchestration for gamepad input, model switching, IMU feedback, servo control, and telemetry.

### Overview

100 Hz control loop integrating:
- Gamepad mode selection
- PPO model subprocess management
- Background IMU reader thread
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

#### Imports and placeholders

```python
import math
import subprocess
import threading
import time
import serial
import queue
import sys
import rerun  # logging wrapper, see odie_rerun.md (placeholder import)
from controller import GamepadController  # see controller.md

def console_reader(cmds):
    # Reads typed commands: controller | ball | person | stop | quit
    for line in sys.stdin:
        cmds.put(line.strip())

def get_model_output(model_process):
    # Placeholder: read one line of 12 joint angles from the model's stdout.
    # Returns a list of 12 servo angles, or None if no output yet.
    return None
```

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
    person_process = None
    ball_process = None
    typed_mode = None
    console_cmds = queue.Queue()
    threading.Thread(target=console_reader, args=(console_cmds,), daemon=True).start()
    current_mode = 'stand'
    stand_pose = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
    sit_pose = [90, 120, 45, 90, 60, 135, 90, 60, 0, 90, 120, 180]
    
    
    rerun.init("ODIE Demo")
    
    start_time = time.time()
    iteration = 0
    
    try:
        while True:
            iteration += 1
            loop_start = time.time()
            
            # 1. Read the console (typed command) or the gamepad
            mode = controller.update()
            try:
                typed = console_cmds.get_nowait()
            except queue.Empty:
                typed = None
            if typed == 'quit':
                break
            if typed == 'controller':
                typed_mode = None
            elif typed in ('person', 'ball', 'stop'):
                typed_mode = typed
            if typed_mode:
                mode = typed_mode
            
            # 2. Mode switching logic
            if mode != current_mode:
                if model_process:
                    model_process.terminate()
                
                current_mode = mode
                
                if MODEL_MAP[mode] is not None:
                    model_process = ModelProcess(MODEL_MAP[mode])

                if mode == 'person':
                    # Stop the camera and serial users; hailo_odie.py takes both
                    subprocess.run(['pkill', '-f', 'ball_detect_cv.py'])
                    serial_mgr.servo_port.close()
                    person_process = subprocess.Popen(['bash', '-c', PERSON_CMD_TEXT])
                elif person_process:
                    person_process.terminate()
                    person_process.wait(timeout=5)
                    person_process = None
                    serial_mgr.servo_port = serial.Serial(SERVO_PORT, BAUD_RATE, timeout=0.01)

                if mode == 'ball':
                    subprocess.run(['pkill', '-f', 'hailo_odie.py'])
                    ball_process = subprocess.Popen(['bash', '-c', BALL_CMD_TEXT])
                elif ball_process:
                    ball_process.terminate()
                    ball_process.wait(timeout=5)
                    ball_process = None
                
                rerun.log_mode(mode)
            
            # 3. Read IMU
            imu = imu_reader.get_imu()
            if imu:
                rerun.log_imu_data(imu)
                
            
            # 4. Get servo angles
            if current_mode == 'reset90':
                angles = [90] * 12
                serial_mgr.send_servo_angles(angles)
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
        if person_process:
            person_process.terminate()
        if ball_process:
            ball_process.terminate()
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
    'sit': 'sit',
    'wave': 'wave',
    'dance': 'dance',
    'person': None,      # typed 'person' - runs hailo_odie.py, see person_detect.md
    'ball': None,        # typed 'ball' - runs ball_detect_cv.py, see ball_detect_cv.md
    'stop': None,        # typed 'stop' - no model, servos idle
    'reset90': None,
}

PERSON_CMD_TEXT = 'cd ~/hailo-rpi5-examples && source setup_env.sh && python3 hailo_odie.py --input /dev/video0'
BALL_CMD_TEXT = 'python3 ball_detect_cv.py'

STAND_POSE = [90, 120, 45, 90, 60, 135, 90, 60, 135, 90, 120, 45]
SIT_POSE = [90, 120, 45, 90, 60, 135, 90, 60, 0, 90, 120, 180]

MODE_SWITCH_DELAY = 0.5  # seconds
CONTROL_RATE = 100  # Hz
```

### Runtime

```bash
python3 demo.py
```

Outputs:
- Rerun dashboard (port 9876)
- Console logs for mode switches and FPS
- Servo angles to Servo2040 at 100 Hz
