# person_detect.md – Person Detection (YOLOv8n)

Person detection runs on the Hailo-8L AI HAT+. The robot's greeting is `hailo_odie.py`. Nothing else in this file runs on the robot.

## Install (once, on the Pi 5)

Clone and install Hailo's example repo, then copy `hailo_odie.py` into it:

```bash
cd ~
git clone https://github.com/hailo-ai/hailo-rpi5-examples.git
cd hailo-rpi5-examples
./install.sh
./download_resources.sh
```

Run `hailo_odie.py` from this folder, after `source setup_env.sh` (see Run below).

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
- Kill all Python processes first so nothing else holds the camera, the Servo2040 serial port, or port 5000:

```bash
pkill -f python3
```

Run this only when you start this script by hand, not from `demo.py`. `demo.py` is also a Python process, so `pkill -f python3` stops it too.
