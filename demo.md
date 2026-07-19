# demo.py — Unified ODIE Launcher

Single entry point for all ODIE demo modes. Starts the Rerun telemetry dashboard automatically on launch, then lets you switch between control modes by typing a command. Switching modes kills all running processes and restarts cleanly.

## Usage

```bash
python3 demo.py
```

## Commands

| Command | What it runs |
|---------|-------------|
| `controller` | PS3 gamepad control via `controller.py` |
| `ball` | OpenCV ball tracking via `ball_detect_cv.py` |
| `person` | Hailo YOLOv8 person detection via `person_detect.py` |
| `stop` | Kills everything, restarts dashboard only |
| `quit` | Kills everything and exits |

## Behavior

- On startup: kills any existing processes on ports 9876/5000, starts `odie_rerun.py`, waits for it to be ready
- On every command: kills all running subprocesses, restarts dashboard, launches new mode
- All subprocesses run in their own process groups so `kill_all()` cleanly terminates them and any children
- `person` mode activates the Hailo venv before running

## Notes

- Hardcoded PI IP `172.20.10.3` in the dashboard URL — update if your IP changes
- Person detection kills any lingering Hailo processes first to free the NPU
- Rerun dashboard takes ~3 seconds to start before the mode script launches
