## Servo 2040 Pinout
<img width="956" height="717" alt="Screenshot 2026-09-30 at 10 52 24 PM" src="https://github.com/user-attachments/assets/1ce7bcb1-a121-465b-813d-95aced68bcfc" />
## Servo Channel Mapping

### Channel Layout (Servo2040 Pins 1-12)

```
FRONT-LEFT (FL)          FRONT-RIGHT (FR)
  ch1: hip                 ch4: hip
  ch2: knee                ch5: knee
  ch3: ankle               ch6: ankle

REAR-RIGHT (RR)          REAR-LEFT (RL)
  ch7: hip                 ch10: hip
  ch8: knee                ch11: knee
  ch9: ankle               ch12: ankle
```

---
## Detailed Servo Connections
 
### Front-Left (FL) - Channels 1, 2, 3
 
| Joint | Servo2040 Pin |
|-------|-----------------|
| FL Hip | Pin 1 |
| FL Knee | Pin 2 |
| FL Ankle | Pin 3 |
 
**Physical Layout**: FL leg servos arranged vertically on robot's front-left
 
### Front-Right (FR) - Channels 4, 5, 6
 
| Joint | Servo2040 Pin |
|-------|-----------------|
| FR Hip | Pin 4 |
| FR Knee | Pin 5 |
| FR Ankle | Pin 6 |
 
**Physical Layout**: FR leg servos arranged vertically on robot's front-right
 
### Rear-Right (RR) - Channels 7, 8, 9
 
| Joint | Servo2040 Pin |
|-------|-----------------|
| RR Hip | Pin 7 |
| RR Knee | Pin 8 |
| RR Ankle | Pin 9 |
 
**Physical Layout**: RR leg servos arranged vertically on robot's rear-right
 
### Rear-Left (RL) - Channels 10, 11, 12
 
| Joint | Servo2040 Pin |
|-------|-----------------|
| RL Hip | Pin 10 |
| RL Knee | Pin 11 |
| RL Ankle | Pin 12 |
 
**Physical Layout**: RL leg servos arranged vertically on robot's rear-left
 
---
 
## Servo Connections Breakdown
 
### What Each Joint Does
 
**Hip (Channels 1, 4, 7, 10)**
- Controls left-right leg rotation
- Allows forward/backward swing motion
- Range: Typically 0–180° (adjust per calibration)
**Knee (Channels 2, 5, 8, 11)**
- Controls leg lift and fold
- Enables walking gait leg raise
- Range: Typically 0–180° (full extension to full fold)
**Ankle (Channels 3, 6, 9, 12)**
- Controls foot angle/ground contact
- Fine-tunes gait stability
- Range: Typically 0–180° (toe-up to flat to toe-down)
---

## Servo Connections Breakdown

### What Each Joint Does

**Hip (Channels 1, 4, 7, 10)**
- Controls left-right leg rotation
- Allows forward/backward swing motion
- Range: Typically 0–180° (adjust per calibration)

**Knee (Channels 2, 5, 8, 11)**
- Controls leg lift and fold
- Enables walking gait leg raise
- Range: Typically 0–180° (full extension to full fold)

**Ankle (Channels 3, 6, 9, 12)**
- Controls foot angle/ground contact
- Fine-tunes gait stability
- Range: Typically 0–180° (toe-up to flat to toe-down)

---

## Physical Wiring Steps

### 1. Servo2040 Power Rails

Connect **all 12 servos** to:
- **VCC** (red): +5V from barrel jack
- **GND** (black): Ground from barrel jack
- **Signal** (white/orange): Pin 0-11 on Servo2040

### 2. Per-Leg Connection Example (FL)

```
FL Hip Servo
  ├─ Red (VCC)    → Servo2040 +5V rail
  ├─ Black (GND)  → Servo2040 GND rail
  └─ Signal       → Servo2040 Pin 1

FL Knee Servo
  ├─ Red (VCC)    → Servo2040 +5V rail
  ├─ Black (GND)  → Servo2040 GND rail
  └─ Signal       → Servo2040 Pin 2

FL Ankle Servo
  ├─ Red (VCC)    → Servo2040 +5V rail
  ├─ Black (GND)  → Servo2040 GND rail
  └─ Signal       → Servo2040 Pin 3
```

Repeat for FR (pins 4-6), RR (pins 7-9), RL (pins 10-12)

### 3. Cable Routing

- **Power cables** (red/black): Bundle together, route through center of chassis
- **Signal cables** (white/orange): Label each with leg + joint name before soldering
- **Strain relief**: Use cable ties at Servo2040 connector area
- **Heat shrink**: Cover all solder joints on servo connectors

---

## Servo Calibration Values

After wiring, each servo needs calibration. Standard values:

| Servo | Min (µs) | Center (µs) | Max (µs) | Notes |
|-------|----------|------------|----------|-------|
| All MG996R | 600 | 1500 | 2400 | Standard PWM range |

**Mapping**: 
- 600µs = 0°
- 1500µs = 90°
- 2400µs = 180°

---

## Servo2040 Pin Layout Reference

```
Servo2040 Terminal Block (Top View)

Pin 1  [FL Hip]      Pin 2  [FL Knee]     Pin 3  [FL Ankle]
Pin 4  [FR Hip]      Pin 5  [FR Knee]     Pin 6  [FR Ankle]
Pin 7  [RR Hip]      Pin 8  [RR Knee]     Pin 9  [RR Ankle]
Pin 10 [RL Hip]      Pin 11 [RL Knee]     Pin 12 [RL Ankle]

+5V Rail (Red)  ←  USB-C PSU
GND Rail (Black) ← USB-C PSU
```

---

## Quick Reference: Servo-to-Channel Lookup

**If you need servo signal for a specific joint, find the channel:**

- FL Hip? → Channel 1
- FR Knee? → Channel 5
- RR Ankle? → Channel 9
- RL Hip? → Channel 10

**Formula**: 
- Leg offset: FL=1, FR=4, RR=7, RL=10
- Joint offset: Hip=0, Knee=1, Ankle=2
- **Channel = Leg Offset + Joint Offset**

Example: FR Ankle = (FR=4) + (Ankle=2) = Channel 6 ✓

---

## Testing After Wiring

1. **Power check**: Servo2040 powers on, LED lights
2. **USB connection**: Pi5 detects Servo2040 over USB serial
3. **Individual servo test**: Run `stand.py` to move all servos to home position
4. **Visual inspection**: All servos respond, no jitter or buzzing
5. **Range test**: Verify 0–180° motion on each servo

---

## Troubleshooting

**Servo doesn't move on a channel:**
- Check signal wire connection to that pin
- Verify servo power (red/black wires) at Servo2040 rails
- Test with a different servo on same channel (isolate problem)

**Servo moves wrong direction:**
- Swap signal wire polarity (some servos have reversed logic)
- Or invert channel in code: `angle = 180 - angle`

**Servo jitters or chatters:**
- Check power supply stability (5V under load)
- Ensure solid ground connection across all servos
- Reduce cable length if possible

**All servos unresponsive:**
- Check USB connection Pi5 ↔ Servo2040
- Verify barrel jack connection (5V/GND from PSU)
- Check Servo2040 fuse (if present)

---

## Color Coding Convention

**Recommended wire colors for clarity:**
- **Red** = Positive/VCC (+5V)
- **Black** = Ground/GND (0V)
- **White/Orange** = Signal (PWM)

Heat shrink each connector and label with leg+joint (e.g., "FL Hip").

---

## Summary Table

| Leg | Hip Ch | Knee Ch | Ankle Ch |
|-----|--------|---------|----------|
| FL  | 1      | 2       | 3        |
| FR  | 4      | 5       | 6        |
| RR  | 7      | 8       | 9        |
| RL  | 10     | 11      | 12       |

Print this and keep it handy during wiring and debugging.
