# person_detect.md

## Person Detection with Hailo-8L

YOLOv8 inference on Hailo-8L NPU for real-time person detection.

### Overview

Uses Hailo-8L AI accelerator to run YOLOv8 model for detecting people in video frames. Outputs bounding boxes filtered for person class (ID 0) with confidence threshold.

### Setup

Install Hailo runtime and model:

```bash
pip install hailo-sdk
# Download YOLOv8n model compiled for Hailo
# Place in: models/yolov8n_hailo.hef
```

### Person Detection Function

```python
def detect_persons(frame, confidence_threshold=0.5):
    """
    Detect persons in frame using YOLOv8 on Hailo-8L.
    
    Args:
        frame: BGR image from camera
        confidence_threshold: Min confidence (0.0-1.0)
        
    Returns:
        List of dicts with:
            - x, y: Top-left corner
            - w, h: Width, height
            - confidence: Detection confidence
    """
    # Preprocess for model
    h, w = frame.shape[:2]
    blob = cv2.dnn.blobFromImage(frame, 1/255.0, (640, 640))
    
    # Run inference
    net = cv2.dnn.readNetFromHailo('models/yolov8n_hailo.hef')
    net.setInput(blob)
    detections = net.forward()
    
    persons = []
    
    for detection in detections:
        # detection: [x_center, y_center, w, h, confidence, class_probs...]
        confidence = detection[4]
        class_id = np.argmax(detection[5:])
        class_conf = detection[5 + class_id]
        
        # Filter for person class (0) and confidence
        if class_id != 0 or confidence * class_conf < confidence_threshold:
            continue
        
        # Convert to bounding box
        x_center = int(detection[0] * w)
        y_center = int(detection[1] * h)
        box_w = int(detection[2] * w)
        box_h = int(detection[3] * h)
        
        x = x_center - box_w // 2
        y = y_center - box_h // 2
        
        persons.append({
            'x': x,
            'y': y,
            'w': box_w,
            'h': box_h,
            'confidence': float(confidence * class_conf),
        })
    
    return persons
```

### Model Details

**YOLOv8n (Nano):**
- Input: 640×640 RGB
- Output: 8400 detections (x_center, y_center, w, h, conf, 80 class probs)
- Latency: ~20-30ms on Hailo-8L
- Memory: ~100MB

**COCO Classes:**
- Class 0: person
- Class 1-79: other objects

### Integration with demo.py

```python
import person_detect

def vision_thread():
    """Background thread for person detection."""
    cap = cv2.VideoCapture(0)
    
    while True:
        ret, frame = cap.read()
        if not ret:
            continue
        
        persons = person_detect.detect_persons(frame, confidence_threshold=0.5)
        
        # Log detections
        for person in persons:
            rr.log(
                f"vision/person_{person['x']}",
                rr.BoundingBox(
                    x=person['x'], y=person['y'],
                    w=person['w'], h=person['h']
                )
            )
        
        time.sleep(0.03)  # ~30 Hz
```

### Post-Processing

```python
def filter_persons(persons, nms_threshold=0.5):
    """
    Apply Non-Maximum Suppression to remove duplicate detections.
    """
    if not persons:
        return []
    
    boxes = np.array([[p['x'], p['y'], p['x'] + p['w'], p['y'] + p['h']] 
                      for p in persons])
    confidences = np.array([p['confidence'] for p in persons])
    
    indices = cv2.dnn.NMSBoxes(
        boxes.tolist(),
        confidences.tolist(),
        score_threshold=0.5,
        nms_threshold=nms_threshold
    )
    
    return [persons[i] for i in indices.flatten()]
```

### Performance

- **Throughput:** ~30 FPS on Hailo-8L
- **Latency:** 20-30ms per frame
- **Power:** <2W on Hailo-8L
- **Model size:** 100MB
- **Memory:** ~500MB working set

### Tuning Parameters

```python
confidence_threshold = 0.5      # Min person confidence
nms_threshold = 0.4             # NMS overlap threshold
input_size = 640                # Model input resolution
inference_backend = 'hailo'     # 'hailo' or 'cpu'
```

### Common Issues

- **No detections:** Lower confidence_threshold (0.3-0.4)
- **False positives:** Raise confidence_threshold (0.6-0.7)
- **Slow inference:** Use Hailo backend instead of CPU
- **Memory error:** Reduce frame size or batch size

### References

- Hailo documentation: https://www.hailo.ai/
- YOLOv8: https://docs.ultralytics.com/models/yolov8/
- COCO dataset: https://cocodataset.org/
