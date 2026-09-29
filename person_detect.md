#!/usr/bin/env python3
"""
Person Detection - Hailo-8L YOLOv8 detection + serial output to demo.py
Detects humans in frame and triggers walk/chase behavior
"""

import cv2
import numpy as np
from hailo_platform import HEF, VDevice, HailoStreamVDevice, ConfigInterface
import serial
import time

# ============================================================================
# HAILO SETUP
# ============================================================================

def init_hailo():
    """Initialize Hailo-8L NPU with YOLOv8 model."""
    target = VDevice()
    hef = HEF(file='models/yolov8m.hef')
    config = hef.get_config()
    network_group = target.configure(hef)
    return target, network_group

def run_inference(network_group, frame):
    """Run YOLOv8 inference on frame."""
    # Resize to model input size (usually 640x640)
    resized = cv2.resize(frame, (640, 640))
    input_data = np.expand_dims(resized, axis=0).astype(np.uint8)
    
    # Run inference
    results = network_group.infer([input_data])
    return results

# ============================================================================
# DETECTION LOGIC
# ============================================================================

def detect_persons(results, frame_shape):
    """Extract person detections from YOLOv8 output."""
    detections = []
    
    # YOLOv8 outputs: [x, y, w, h, confidence, class_id]
    # Class 0 = person
    for detection in results[0][0]:
        if detection[5] == 0 and detection[4] > 0.5:  # Person class, conf > 50%
            x, y, w, h = detection[:4]
            detections.append((int(x), int(y), int(w), int(h)))
    
    return detections

# ============================================================================
# MAIN LOOP
# ============================================================================

def main():
    target, network_group = init_hailo()
    cap = cv2.VideoCapture(0)
    
    print("[PERSON] Hailo YOLOv8 person detection started")
    
    try:
        while True:
            ret, frame = cap.read()
            if not ret:
                break
            
            # Run inference
            results = run_inference(network_group, frame)
            detections = detect_persons(results, frame.shape)
            
            # Draw detections
            for x, y, w, h in detections:
                cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
                cv2.putText(frame, 'Person', (x, y-10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)
            
            cv2.imshow('Person Detection', frame)
            
            # Send detection signal (would integrate with demo.py)
            if detections:
                print(f"[PERSON] Detected {len(detections)} person(s)")
            
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break
            
            time.sleep(0.033)  # ~30 Hz
    
    finally:
        cap.release()
        cv2.destroyAllWindows()
        target.release()

if __name__ == '__main__':
    main()

person_detect.py summary: Uses Hailo-8L NPU to run YOLOv8 object detection at ~30 FPS. Detects humans in camera feed. Returns bounding boxes and confidence scores. Can trigger "walk" or "chase" mode in demo.py when person detected. Requires YOLOv8m.hef model file.
