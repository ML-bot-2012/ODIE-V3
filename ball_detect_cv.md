python
#!/usr/bin/env python3
"""
Ball Detection - OpenCV HSV color tracking
Detects colored ball and returns center position + area
"""

import cv2
import numpy as np

# ============================================================================
# COLOR RANGE (adjust for your ball color)
# ============================================================================

# Red ball example
LOWER_RED = np.array([0, 100, 100])
UPPER_RED = np.array([10, 255, 255])

LOWER_RED2 = np.array([170, 100, 100])
UPPER_RED2 = np.array([180, 255, 255])

# ============================================================================
# DETECTION LOGIC
# ============================================================================

def detect_ball(frame):
    """Detect colored ball using HSV threshold."""
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    
    # Create masks for red (two ranges due to wrap-around)
    mask1 = cv2.inRange(hsv, LOWER_RED, UPPER_RED)
    mask2 = cv2.inRange(hsv, LOWER_RED2, UPPER_RED2)
    mask = mask1 | mask2
    
    # Morphological operations
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)
    
    # Find contours
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    if not contours:
        return None, None
    
    # Find largest contour
    largest = max(contours, key=cv2.contourArea)
    area = cv2.contourArea(largest)
    
    if area < 100:  # Min pixel area
        return None, None
    
    # Get circle (moments)
    M = cv2.moments(largest)
    if M['m00'] > 0:
        cx = int(M['m10'] / M['m00'])
        cy = int(M['m01'] / M['m00'])
        return (cx, cy), area
    
    return None, None

# ============================================================================
# MAIN LOOP
# ============================================================================

def main():
    cap = cv2.VideoCapture(0)
    
    print("[BALL] Ball detection started (red color)")
    
    try:
        while True:
            ret, frame = cap.read()
            if not ret:
                break
            
            center, area = detect_ball(frame)
            
            if center:
                cx, cy = center
                cv2.circle(frame, (cx, cy), int(np.sqrt(area)/np.pi), (0, 255, 0), 2)
                cv2.putText(frame, f'Ball: ({cx}, {cy})', (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
                print(f"[BALL] Center: {center}, Area: {area}")
            
            cv2.imshow('Ball Detection', frame)
            
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break
            
            cv2.waitKey(33)  # ~30 Hz
    
    finally:
        cap.release()
        cv2.destroyAllWindows()

if __name__ == '__main__':
    main()

ball_detect.py summary: OpenCV-based color tracking using HSV. Converts camera frame to HSV, creates mask for target color (red), finds largest contour, returns center point and area. Adjustable color range for different ball colors. Runs at ~30 FPS. Can trigger "chase" or "kick" behaviors in demo.py.
