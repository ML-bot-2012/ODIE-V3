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

### Pi5 ↔ Servo2040 (UART Serial)

Raspberry Pi 5 Pimoroni Servo2040
GPIO14 (TXD) ──yellow──→ GPIO0 (RX)
GPIO15 (RXD) ←──orange── GPIO1 (TX)
GND ──────black────────→ GND


**Baud**: 115200, 8N1

![Servo2040 Terminal Layout](https://github.com/user-attachments/assets/a0842b76-f98d-42ff-bf20-5bf6054a5b1b)

**Terminals 1–12**: Servo PWM + power connectors (no direct Pi5 connection—servos powered by PSU rail)

---

## Separate System: Pi Pico ↔ MPU6050 (I2C)

**Pi Pico and MPU6050 form an independent IMU subsystem** communicating via I2C

Pi Pico (RP2040)
├─ GPIO4 (SDA) ──blue──→ MPU6050 SDA
├─ GPIO5 (SCL) ──green─→ MPU6050 SCL
├─ +5V (VBUS) ──red───→ MPU6050 VCC
└─ GND ──────black─────→ MPU6050 GND


**I2C Address**: MPU6050 at 0x68 (AD0 = GND)

![Pico IMU Breakout Pinout](https://github.com/user-attachments/assets/b43fbf97-8600-4b40-8bda-c07da2d9e533)

---

## IMU PCB (Pico-based)

**Board**: Raspberry Pi Pico IMU breakout with soldered MPU6050

**Connections**:
- VCC → +5V (from Servo2040 PSU rail)
- GND → common ground
- GPIO4 (SDA) → I2C data line
- GPIO5 (SCL) → I2C clock line
- VBUS → 5V power input

**Components** (all onboard):
- Pico RP2040 (ARM Cortex-M0+ dual-core)
- MPU6050 sensor (3-axis accel + gyro)
- Voltage regulator (5V → 3.3V)
- Status LED (GPIO25)

---

## Architecture Summary

| Component | Communication | Purpose |
|-----------|---------------|---------|
| Pi5 ↔ Servo2040 | UART serial | Motion command protocol |
| Servo2040 ↔ Servos | PWM + power | Leg actuation (12 channels) |
| Pi Pico ↔ MPU6050 | I2C | IMU telemetry (accel, gyro) |
| All | +5V power | Fed from single PSU barrel jack |

---

## Connector Details

### Servo Header (Pimoroni Servo2040)

**3-pin JST connector per servo:**
- Pin 1: PWM signal (from Servo2040)
- Pin 2: +5V (from PSU rail)
- Pin 3: GND (common return)

Max cable length: 1m

### USB-C PSU to Barrel Jack

**Female barrel jack (5.5×2.1mm):**
- Soldered to 16AWG wires (short run, <20cm)
- Red to +5V, Black to GND
- Connector rated 10A

---

## Power-On Sequence

1. **PSU switched on** (USB-C → 5V DC)
2. **Barrel jack powers Servo2040** (VCC + GND rails energized)
3. **Pi5 boots** from GPIO 5V (or USB-C with separate charger)
4. **Pico boots** from VBUS (connected to Servo2040 VCC)
5. **UART handshake** (Pi5 → Servo2040 serial protocol)
6. **I2C scan** (Pico queries MPU6050 at 0x68)
7. **Ready** (accept motion commands)

---

## Shutdown

```bash
ssh pi5 sudo shutdown -h now
# Wait 10 seconds (Linux graceful halt)
# Manually disconnect USB-C PSU
```

**Critical**: Do NOT yank power (SD card corruption risk).
