# ball_detect_cv.py — OpenCV Ball Tracker

HSV-based orange/red ball tracker using OpenCV. ODIE's head follows the ball in real time via serial commands to the Servo2040. Includes a live Flask web dashboard streaming the annotated camera feed.

## What it does

Captures frames from a USB camera, runs HSV color segmentation to find an orange/red ball, then sends normalized (cx, cy) coordinates to the Servo2040 at 20Hz. The Servo2040 uses these to tilt ODIE's body toward the ball. A Flask server streams the annotated video feed and status to any browser on the network.

## Detection pipeline

1. Capture 320×240 MJPG frame from `/dev/video0`
2. Flip horizontally (mirror correction)
3. Convert to HSV
4. Threshold for orange/red (two HSV ranges to catch red wraparound)
5. Erode + dilate to remove noise
6. Find largest contour above 300px² area
7. Fit minimum enclosing circle → ball center + radius

## Serial protocol

- Ball detected: `track,<cx>,<cy>` — cx/cy normalized 0.0–1.0, cx is mirrored
- No ball: `noball`

## Web dashboard

Open in any browser on the same network:
