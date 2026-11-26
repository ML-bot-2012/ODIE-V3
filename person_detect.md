# person_detect.md – Person Detection (YOLOv8n)


## Robot script (`hailo_odie.py`)

This script runs on the Pi 5. It uses the Hailo-8L AI HAT+ for YOLOv8 person detection and sends commands to the Servo2040 over USB serial.

- On startup it sends `stand`.
- On the first person detection (confidence above 0.4), it sends `wave` and shows "Sitting & waving" for 10 seconds. It sends only `wave`; the "sit" in the comments is not sent.
- After 10 seconds it sends `stand` and waits 3 seconds.
- If someone is still in view, the cycle repeats. If no one is in view, it sends `stand` and stays standing.
- It serves a status page on port 5000 with the camera feed and a status line.

Run on the Pi:

```bash
cd ~/hailo-rpi5-examples && source setup_env.sh && python3 hailo_odie.py --input /dev/video0
```

Before running, check that:
- `ls /dev/serial/by-id` shows the Servo2040 at `usb-MicroPython_Board_in_FS_mode_e66598541b222033-if00`.
- `hailortcli fw-control identify` reports the Hailo-8L.
- Nothing else is on port 5000. `ball_detect_cv.py` uses the same port, so run only one at a time.

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

Open `http://localhost:5001` (or the Pi's IP on that port). 
