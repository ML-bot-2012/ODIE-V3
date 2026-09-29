# ball_detect.md

## Ball Detection with OpenCV

HSV color-based ball tracking for real-time detection and position estimation.

### Overview

Uses OpenCV to detect colored balls via HSV thresholding, morphological operations, and contour analysis. Outputs center coordinates and area of detected ball.

### Color Detection

Red ball uses two HSV ranges to handle the hue wrap-around at 0°/180°:

```python
# Lower red (0-10°)
lower_red1 = np.array([0, 100, 100])
upper_red1 = np.array([10, 255, 255])

# Upper red (170-180°)
lower_red2 = np.array([170, 100, 100])
upper_red2 = np.array([180, 255, 255])
```

### Processing Pipeline

1. Convert BGR to HSV color space
2. Create binary masks for both red ranges
3. Combine masks with OR operation
4. Apply morphological open (remove noise)
5. Apply morphological close (fill holes)
6. Find contours in processed image
7. Calculate moment (center of mass) for largest contour
8. Return center (cx, cy) and area

### Key Functions

```python
def detect_ball(frame):
    """
    Detect red ball in frame.
    
    Args:
        frame: BGR image from camera
        
    Returns:
        cx, cy: center coordinates (or None if not detected)
        area: contour area (or 0 if not detected)
    """
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    
    mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
    mask2 = cv2.inRange(hsv, lower_red2, upper_red2)
    mask = cv2.bitwise_or(mask1, mask2)
    
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)
    
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    if not contours:
        return None, None, 0
    
    largest = max(contours, key=cv2.contourArea)
    area = cv2.contourArea(largest)
    
    if area < 100:  # Minimum area threshold
        return None, None, 0
    
    M = cv2.moments(largest)
    if M["m00"] == 0:
        return None, None, 0
    
    cx = int(M["m10"] / M["m00"])
    cy = int(M["m01"] / M["m00"])
    
    return cx, cy, area
```

### Tuning Parameters

- **Hue range:** Adjust 0-10 and 170-180 for different red shades
- **Saturation:** 100-255 (adjust lower value for lighter colors)
- **Value:** 100-255 (adjust lower value for darker environments)
- **Kernel size:** (5, 5) for morphological operations
- **Min area:** 100 pixels minimum for valid detection

### Integration with demo.py

Ball detection runs in camera thread, updates ball position at ~30 Hz:

```python
cx, cy, area = detect_ball(frame)
if cx is not None and area > 100:
    log_ball_position(cx, cy, area)
```

### Debugging

Visualize detection with:

```python
cv2.imshow('HSV', hsv)
cv2.imshow('Mask', mask)
cv2.imshow('Detection', frame)
cv2.waitKey(1)
```

### Common Issues

- **False positives:** Tighten saturation/value thresholds
- **Missed detections:** Lower saturation/value thresholds
- **Jitter:** Increase minimum area threshold
- **Lighting sensitivity:** Adjust value (brightness) range
