# controller.md

## Gamepad Controller Logic

Pygame-based gamepad input handler for mode selection and control.

### Overview

Reads gamepad input (DPAD, buttons, sticks) at 100 Hz main loop rate. Maps button presses to robot modes with mode-switching logic and debouncing.

### Button Mapping

```python
MODEL_MAP = {
    'walk': 'walk',           # DPAD up/left/right
    'stand': 'walk',          # X button
    'sit': 'walk',            # Triangle button
    'wave': 'walk',           # Square button
    'dance': 'dance',         # O button
    'reset90': None,          # L1 - no model, direct servo
}

BUTTON_MAP = {
    'DPAD_UP': 'walk',
    'DPAD_LEFT': 'walk',
    'DPAD_RIGHT': 'walk',
    'X': 'stand',
    'TRIANGLE': 'sit',
    'SQUARE': 'wave',
    'CIRCLE': 'dance',
    'L1': 'reset90',
}
```

### Pygame Joystick Button Codes

PS4/Xbox-compatible mapping:

```python
BUTTON_CODES = {
    0: 'X',              # X / Square
    1: 'CIRCLE',         # O / B
    2: 'TRIANGLE',       # Triangle / Y
    3: 'SQUARE',         # Square / A
    4: 'L1',             # LB
    5: 'R1',             # RB
    6: 'L2',             # LT
    7: 'R2',             # RT
    8: 'SHARE',          # Back
    9: 'OPTIONS',        # Start
    10: 'L3',            # Left stick press
    11: 'R3',            # Right stick press
}

DPAD_MAP = {
    (0, -1): 'DPAD_UP',
    (0, 1): 'DPAD_DOWN',
    (-1, 0): 'DPAD_LEFT',
    (1, 0): 'DPAD_RIGHT',
}
```

### Controller Class

```python
class GamepadController:
    def __init__(self):
        pygame.init()
        pygame.joystick.init()
        
        joysticks = pygame.joystick.get_count()
        if joysticks == 0:
            raise RuntimeError("No gamepad detected")
        
        self.joystick = pygame.joystick.Joystick(0)
        self.joystick.init()
        
        self.current_mode = 'walk'
        self.last_mode_switch = 0
        self.mode_switch_delay = 0.5  # seconds
    
    def update(self):
        """Poll gamepad and return current mode."""
        pygame.event.pump()
        
        # Check DPAD
        hat = self.joystick.get_hat(0)
        if hat in DPAD_MAP:
            button = DPAD_MAP[hat]
            self._switch_mode(BUTTON_MAP[button])
        
        # Check buttons
        for button_code in range(12):
            if self.joystick.get_button(button_code):
                button = BUTTON_CODES.get(button_code)
                if button and button in BUTTON_MAP:
                    self._switch_mode(BUTTON_MAP[button])
        
        return self.current_mode
    
    def _switch_mode(self, new_mode):
        """Switch mode with debouncing."""
        now = time.time()
        if now - self.last_mode_switch < self.mode_switch_delay:
            return
        
        if new_mode != self.current_mode:
            self.current_mode = new_mode
            self.last_mode_switch = now
            print(f"Mode switched to: {new_mode}")
```

### Integration with demo.py

Main loop reads gamepad at 100 Hz:

```python
controller = GamepadController()

while True:
    mode = controller.update()
    
    if mode != current_mode:
        current_mode = mode
        # Kill old model process
        if model_process:
            model_process.terminate()
        
        # Start new model if needed
        if MODEL_MAP[mode] is not None:
            model_process = start_model_process(MODEL_MAP[mode])
    
    time.sleep(0.01)  # 100 Hz
```

### Stick Input (Optional)

For analog stick control:

```python
def get_stick_values(self):
    """Get left stick X/Y values."""
    lx = self.joystick.get_axis(0)  # -1.0 to 1.0
    ly = self.joystick.get_axis(1)  # -1.0 to 1.0
    
    # Apply deadzone
    deadzone = 0.1
    if abs(lx) < deadzone:
        lx = 0
    if abs(ly) < deadzone:
        ly = 0
    
    return lx, ly
```

### Debugging

Print joystick info:

```python
pygame.init()
pygame.joystick.init()
j = pygame.joystick.Joystick(0)
j.init()

print(f"Name: {j.get_name()}")
print(f"Buttons: {j.get_numbuttons()}")
print(f"Axes: {j.get_numaxes()}")
print(f"Hats: {j.get_numhats()}")
```

### Common Issues

- **No gamepad detected:** Check USB connection, run `lsusb`
- **Button codes wrong:** Different gamepads may vary; print pygame.JOYBUTTONDOWN events
- **Mode not switching:** Check mode_switch_delay (prevent rapid switching)
- **Stick drift:** Apply larger deadzone (0.15-0.2)
