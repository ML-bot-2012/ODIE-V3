# person_detect.py — Hailo YOLOv8 Person Detection

Uses the Hailo-8L NPU to run YOLOv8 person detection in real time. When a person is detected for the first time, ODIE waves for 10 seconds then returns to standing. Includes a live Flask web dashboard with annotated video feed.

## Usage

```bash
cd ~/hailo-rpi5-examples
source venv_hailo_rpi_examples/bin/activate
python3 person_detect.py --input /dev/video0
```

## Behavior

1. Starts standing, streams camera feed through Hailo YOLOv8
2. First person detected with confidence > 40% → sends `wave` to Servo2040
3. Waves for 10 seconds, counting down on screen
4. Returns to `stand` — never triggers again for the rest of the session (`done_forever` flag)

## Web dashboard

http://<PI_IP>:5000

Shows live annotated video with bounding boxes (green = person, orange = other objects), person count, and current status.

## Detection pipeline

Built on `GStreamerDetectionApp` from the Hailo RPi5 examples. Uses:
- Model: `yolov8s.hef` on Hailo-8L
- Confidence threshold: 0.4 for person class
- Input: USB camera via `--input /dev/video0`

## Dependencies

Requires the Hailo RPi5 examples venv:
```bash
source venv_hailo_rpi_examples/bin/activate
```

## Notes

- Only one greeting per session by design — reset by restarting the script
- Draws all COCO class detections but only reacts to `person`
- NPU must be free — kill any other Hailo processes before running
