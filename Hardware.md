# Hardware.md – Bill of Materials & Assembly

### FULL ASSEMBLED BOT:
<img width="3024" height="4032" alt="image" src="https://github.com/user-attachments/assets/7a7ff718-cbf8-492a-81a9-21e529e7995b" />


## Bill of Materials (BOM)

### Core Computing

| Part | Qty | Specs | Cost | Source |
|------|-----|-------|------|--------|
| Raspberry Pi 5 | 1 | 8GB RAM, BCM2712 | $80 | Adafruit/PiHut |
| Pimoroni Servo2040 | 1 | RP2040, 18 PWM channels | $25 | Pimoroni |
| Raspberry Pi Pico | 1 | RP2040, MicroPython | $5 | Official |
| **Subtotal** | | | **$110** | |

### Servo Actuators

| Part | Qty | Specs | Cost | Source |
|------|-----|-------|------|--------|
| MG996R servo | 12 | 55g, 13 kg-cm, metal gear | $12 ea | AliExpress/Amazon |
| Servo extension cable | 24 | 30cm, 3-pin JST | $0.50 ea | AliExpress |
| **Subtotal** | | | **$156** | |

### Sensors

| Part | Qty | Specs | Cost | Source |
|------|-----|-------|------|--------|
| MPU6050 IMU breakout | 1 | 3-axis accel/gyro, I2C | $5 | AliExpress |
| USB camera | 1 | 1080p 30fps, UVC | $15 | Amazon |
| **Subtotal** | | | **$20** | |

### Power

| Part | Qty | Specs | Cost | Source |
|------|-----|-------|------|--------|
| 5V/10A USB-C PSU | 1 | Pi 5 official or similar | $15 | Amazon |
| Female barrel jack adapter | 1 | 5.5×2.1mm to jumper wires | $3 | Amazon |
| Jumper wires (M-F) | 20 | 20cm, 22AWG | $4 | AliExpress |
| **Subtotal** | | | **$22** | |

### Structural & Mechanical

| Part | Qty | Type | Cost | Source |
|------|-----|------|------|--------|
| 3D printed chassis | 1 | PLA main body | $20 | Printed in-house |
| 3D printed legs | 4 | PLA femur/tibia | $15 | Printed in-house |
| 3D printed hip brackets | 4 | PLA servo mounts | $8 | Printed in-house |
| TPU feet grippers | 4 | Shoe-style grippers | $10 | 3D printed or purchased |
| M3 hardware | 100x | Bolts, nuts, washers | $8 | AliExpress |
| Zipties | 50x | Cable management | $2 | Amazon |
| **Subtotal** | | | **$63** | |

### Miscellaneous

| Part | Qty | Purpose | Cost | Source |
|------|-----|---------|------|--------|
| USB hub | 1 | Camera + controller | $12 | Amazon |
| HDMI micro cable | 1 | Display (dev) | $5 | Amazon |
| Pi active cooler | 1 | Optional (for YOLOv8 loads) | $12 | Amazon |
| **Subtotal** | | | **$29** | |

### **TOTAL BOM: ~$400**

---

## Assembly Tree

ODIE V3 Chassis
├── Frame (3D printed PLA)
│ ├── Main body (central plate)
│ └── Cable management clips
│
├── Legs (4×)
│ ├── FL (Front-Left)
│ │ ├── Hip servo (ch0, MG996R)
│ │ ├── Knee servo (ch1, MG996R)
│ │ ├── Ankle servo (ch2, MG996R)
│ │ ├── PLA femur segment
│ │ ├── PLA tibia segment
│ │ ├── TPU foot gripper
│ │ └── M3 hardware (bolts, nuts)
│ │
│ ├── FR (Front-Right)
│ │ └── [same as FL, ch3-5]
│ │
│ ├── RR (Rear-Right)
│ │ └── [same as FL, ch6-8]
│ │
│ └── RL (Rear-Left)
│ └── [same as FL, ch9-11]
│
├── Electronics Bay (central)
│ ├── Raspberry Pi 5 (8GB)
│ │ ├── USB hub (camera, input)
│ │ └── 5V power from PSU
│ │
│ ├── Pimoroni Servo2040
│ │ ├── 18 PWM channels → servo headers
│ │ ├── UART to Pi5 (jumper wires)
│ │ └── GND to Pi5
│ │
│ ├── Pico IMU Board
│ │ ├── RP2040 Pico
│ │ ├── MPU6050 sensor (I2C)
│ │ ├── I2C to Servo2040 (jumper wires)
│ │ └── 5V power
│ │
│ └── Power
│ ├── 5V/10A PSU
│ ├── Female barrel jack adapter
│ └── Jumper wires to Pimoroni VCC/GND rail
│
└── Sensors & I/O
├── USB camera (mounted on head)
├── Hailo-8L NPU (via USB)
└── PS3 controller (wireless)


---

## 3D Printed Parts

### Main Chassis (`chassis.stl`)
- **Material**: PLA, 15% infill
- **Dimensions**: 180mm × 120mm × 80mm
- **Print time**: ~6–8 hours
- **Weight**: ~150g
- **Features**:
  - Central electronics tray (Pi5 + Servo2040 mounting)
  - Servo header passthrough slots
  - Cable clips for harness management
  - Power barrel jack pocket

### Leg Segments (femur + tibia per leg)

#### Femur (`femur_l.stl`, `femur_r.stl`)
- **Material**: PLA, 20% infill
- **Length**: 80mm
- **Joint**: Servo horn adapter (M3 bolt)
- **Qty**: 4 (mirror pairs)
- **Print time**: ~2 hours each

#### Tibia (`tibia_l.stl`, `tibia_r.stl`)
- **Material**: PLA, 20% infill
- **Length**: 100mm
- **Joint**: Ankle servo mount
- **Qty**: 4 (mirror pairs)
- **Print time**: ~3 hours each

### Hip Brackets (`hip_bracket.stl`)
- **Material**: PLA, 25% infill (structural)
- **Qty**: 4
- **Dimensions**: 40mm × 30mm × 25mm
- **Features**:
  - Servo mount (MG996R spline adapter)
  - Femur link socket
  - Cable pass-through hole

---

## TPU Parts

### Feet Grippers (`foot_gripper.stl`)
- **Material**: TPU 90A (flexible, grippy)
- **Qty**: 4 (one per leg)
- **Purpose**: Traction on smooth/slippery surfaces
- **Mount**: Glued to tibia foot end

---

## Print Time Summary

| Part | Time | Qty | Total |
|------|------|-----|-------|
| Chassis | 7 hrs | 1 | 7 hrs |
| Femur | 2 hrs | 4 | 8 hrs |
| Tibia | 3 hrs | 4 | 12 hrs |
| Hip bracket | 1.5 hrs | 4 | 6 hrs |
| Foot gripper | 30 min | 4 | 2 hrs |
| **Total** | | | **~35 hrs** |

*Parallelize on multiple printers to reduce wall-clock time.*

---

## Assembly Checklist

- [ ] Print all PLA parts (chassis, legs, brackets)
- [ ] Print TPU foot grippers
- [ ] Clean printed parts (support removal, sanding)
- [ ] Assemble legs (femur + tibia + ankle servo)
- [ ] Glue TPU grippers to tibia feet
- [ ] Mount servos on brackets (M3 bolts, lock washers)
- [ ] Mount brackets on chassis frame
- [ ] Route servo cables through clips
- [ ] Mount Pi5, Servo2040, Pico on central tray
- [ ] Connect jumper wires (UART: Pi5 ↔ Servo2040)
- [ ] Connect jumper wires (I2C: Servo2040 ↔ Pico)
- [ ] Connect PSU power wires to Pimoroni VCC/GND rail
- [ ] Mount USB camera on head bracket
- [ ] Connect USB hub (camera, Hailo NPU)
- [ ] Test servo movement (one channel at a time)
- [ ] Calibrate servo centers (reset90 command)
- [ ] Verify IMU I2C communication
- [ ] Run stand.py to confirm all 12 servos respond

EOF
markdown
cat > electrical.md << 'EOF'
# electrical.md – Power & PCB Design

## External Power Supply

### Specifications

| Param | Value | Notes |
|-------|-------|-------|
| Input | 5V USB-C | Standard Pi charger |
| Output | 5V DC | Single rail |
| Rating | 10A continuous | 50W output |
| Connector | USB-C barrel jack | Female jack soldered to wires |

### Recommended PSU

- **Raspberry Pi Official 27W USB-C**: $15, reliable
- **Amazon Basic 5V/10A**: $12, budget alternative
- **RasTech 5V/10A USB-C**: $15, industrial spec

---

## Power Wiring (Simplified)

5V/10A USB-C PSU
↓
Female barrel jack (5.5×2.1mm)
├─→ Red wire (+5V) → Pimoroni Servo2040 VCC rail
└─→ Black wire (GND) → Pimoroni Servo2040 GND rail

Pimoroni distributes:
├─→ Servo header rail (12× MG996R)
├─→ Pico (5V GPIO pin)
└─→ Pi5 (5V GPIO header or USB-C)

[All GND connected at Servo2040]


### Current Budget

| Component | Idle | Active | Notes |
|-----------|------|--------|-------|
| 12× MG996R servos | 0.6 A | 6.0 A | Peak all swing simultaneously |
| Raspberry Pi 5 | 0.5 A | 2.5 A | Full CPU + GPU (YOLOv8) |
| Servo2040 + Pico | 0.15 A | 0.3 A | MCU overhead |
| USB hub + camera | 0.2 A | 0.5 A | Camera stream 30fps |
| **Total** | **1.45 A** | **9.35 A** | PSU 10A (5% headroom) |

---

## Signal Wiring (Jumper Wires)

### Pi5 ↔ Servo2040 (UART)

Raspberry Pi 5 Pimoroni Servo2040
GPIO14 (TXD) ──yellow──→ GPIO0 (RX)
GPIO15 (RXD) ←──orange── GPIO1 (TX)
GND ──────black────────→ GND


**Baud**: 115200, 8N1

### Servo2040 ↔ Pico (I2C)

Servo2040 Pico IMU Board
GPIO4 (SDA) ──blue──→ GPIO4 (SDA)
GPIO5 (SCL) ──green─→ GPIO5 (SCL)
GND ──────black─────→ GND
+5V ──────red──────→ VBUS


**I2C Address**: MPU6050 at 0x68 (AD0 = GND)

---

## IMU PCB (Pico-based)

### Layout (matches image above)

┌─────────────────────────────────┐
│ Raspberry Pi Pico RP2040 │
│ │
│ GPIO4 (SDA) ──┐ │
│ GPIO5 (SCL) ──┤ │
│ VBUS (+5V) │ │
│ GND │ │
│ │ │
│ ┌────┴──────┐ │
│ │ Pull-ups │ │
│ │ 4.7k each │ │
│ └────┬──────┘ │
│ │ │
│ ┌────▼──────┐ │
│ │ MPU6050 │ │
│ │ I2C Slave │ │
│ │ (0x68) │ │
│ │ │ │
│ │ 3-axis │ │
│ │ accel/ │ │
│ │ gyro │ │
│ └───────────┘ │
│ │
│ [Bypass caps: 10µF + 1µF] │
│ [Status LED on GPIO25] │
│ │
└─────────────────────────────────┘


### Schematic (Simple)

Pico VBUS (+5V) ──[10µF]──┬──[1µF]── GND
│
MPU6050 VCC
│
GPIO4 (SDA) ──[4.7k]──┬──────── MPU6050 SDA
│
GPIO5 (SCL) ──[4.7k]──┼──────── MPU6050 SCL
│
+5V


### Components (Minimal)

| Part | Qty | Purpose |
|------|-----|---------|
| MPU6050 breakout board | 1 | Pre-soldered, simplifies assembly |
| Pull-up resistors 4.7k | 2 | I2C pull-ups (some boards have built-in) |
| Bypass capacitors | 2 | 10µF + 1µF for power supply ripple |

**Note**: Use MPU6050 breakout board (Amazon $5) instead of raw IC. Eliminates soldering, includes voltage regulator.

---

## Connector Details

### Servo Header (Pimoroni Servo2040)

**3-pin JST connector per servo:**
- Pin 1: PWM signal (from Servo2040)
- Pin 2: +5V (from PSU rail)
- Pin 3: GND (common return)

Max cable length: 1m (capacitive loading on PWM ≤1µF OK)

### USB-C PSU to Barrel Jack

**Female barrel jack (5.5×2.1mm):**
- Soldered to 16AWG wires (short run, <20cm)
- Red to +5V, Black to GND
- Connector rated 10A (fused by PSU internal limit)

---

## Power-On Sequence

1. **PSU switched on** (USB-C → 5V DC)
2. **Barrel jack powers Servo2040** (VCC + GND rails energized)
3. **Pi5 boots** from GPIO 5V (or USB-C with separate charger)
4. **Pico boots** from VBUS (connected to Servo2040 VCC)
5. **UART handshake** (Pi5 → Servo2040)
6. **I2C scan** (Servo2040 queries MPU6050 at 0x68)
7. **Ready** (accept motion commands)

---

## Shutdown

```bash
ssh pi5 sudo shutdown -h now
# Wait 10 seconds (Linux graceful halt)
# Manually disconnect USB-C PSU
```

**Critical**: Do NOT yank power (SD card corruption risk).
