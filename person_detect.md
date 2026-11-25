# person_detect.md – Person Detection (YOLOv8n)

Detects people in the camera feed and reports the count and a wave trigger. Runs on a Mac or the Pi 5 CPU with Ultralytics. No Hailo required.

## Setup

```bash
pip install ultralytics opencv-python flask
```

The first run downloads `yolov8n.pt` (about 6 MB) automatically.

## Script (`odie_vision_flask.py`)

```python
import os, threading, time, cv2
from flask import Flask, Response, jsonify
from ultralytics import YOLO

WAVE_SECS = 5
COOLDOWN = 10

model = YOLO("yolov8n.pt")
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
lock = threading.Lock()
state = {"jpg": None, "msg": "Standing"}
wave_until = 0
next_wave_at = 0

def detect_loop():
    global wave_until, next_wave_at
    while True:
        ok, frame = cap.read()
        if not ok:
            time.sleep(0.05)
            continue
        r = model(frame, classes=[0], conf=0.4, verbose=False)[0]
        n = 0
        for b in r.boxes:
            x1, y1, x2, y2 = map(int, b.xyxy[0])
            score = float(b.conf[0])
            n += 1
            cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
            cv2.putText(frame, f"person {score:.2f}", (x1, max(y1 - 5, 12)),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
        now = time.time()
        if n >= 1 and now >= next_wave_at:
            wave_until = now + WAVE_SECS
            next_wave_at = wave_until + COOLDOWN
        msg = "Waving..." if now < wave_until else "Standing"
        cv2.putText(frame, f"People: {n}", (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 255), 2)
        cv2.putText(frame, msg, (10, 60),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 255, 0), 2)
        _, buf = cv2.imencode(".jpg", frame, [cv2.IMWRITE_JPEG_QUALITY, 70])
        with lock:
            state["jpg"] = buf.tobytes()
            state["msg"] = msg

app = Flask(__name__)

@app.route("/video_feed")
def video_feed():
    def gen():
        while True:
            with lock:
                jpg = state["jpg"]
            if jpg:
                yield b"--frame\r\nContent-Type: image/jpeg\r\n\r\n" + jpg + b"\r\n"
            time.sleep(0.03)
    return Response(gen(), mimetype="multipart/x-mixed-replace; boundary=frame")

@app.route("/status")
def status():
    with lock:
        return jsonify({"msg": state["msg"]})

threading.Thread(target=detect_loop, daemon=True).start()
app.run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)), threaded=True)
```

## Run

```bash
PORT=5001 python3 odie_vision_flask.py
```

Open `http://localhost:5001` (or the Pi's IP on that port). Port 5000 is used by AirPlay Receiver on macOS, so use another port there.

## Notes

- This script only updates the on-screen status. It does not send any command to the Servo2040.
- On the robot, a person detection triggers the sit action. That control code is not in this script yet, so it needs to be documented here before the repo matches the robot.
- `WAVE_SECS` and `COOLDOWN` control how long the wave shows and how long before it can trigger again.
