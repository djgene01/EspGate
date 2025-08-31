# ESP32 RF Gate Controller (ESPHome + IBT-2)

An **ESPHome-based smart gate controller** for ESP32 that supports:

- 📡 **RF Remote Learning** (KeeLoq remotes, 2 buttons stored in flash)
- 🔄 **Forward / Reverse / Stop Sequence** (reverse → stop → forward cycle)
- 🚪 **Auto-Close** (configurable 0–240 s, cancelable by remote stop/manual action)
- ⚡ **Soft Start & Stop** (reduced speed during first & last second)
- 🛡 **Obstacle Detection** using BTS7960/IBT-2 current sense pins
- 🌐 **Web & Home Assistant Integration** (via ESPHome API)

---

## Hardware

- **ESP32 dev board**
- **BTS7960 / IBT-2 H-bridge driver**
  - Drives the DC gate motor with PWM
  - Provides current sense outputs (`R_IS` / `L_IS`)
- **RF Receiver module** (433/315 MHz with KeeLoq decoding)
- DC motor gate hardware

### Wiring Overview

| ESP32 Pin | BTS7960 Pin | Function                 |
|-----------|-------------|--------------------------|
| GPIO25    | RPWM        | Forward PWM              |
| GPIO26    | LPWM        | Reverse PWM              |
| GPIO34    | R_IS        | Current sense (forward)  |
| GPIO35    | L_IS        | Current sense (reverse)  |
| 5V        | VCC (logic) | Logic supply             |
| GND       | GND         | Common ground            |

Additional:
- Tie **R_EN** and **L_EN** HIGH (5 V) to permanently enable the driver  
- `M+`/`M-` → Gate motor terminals  
- `VCC`/`GND` (power side) → Motor power supply (e.g. 12/24 V DC)

---

## Features

### 🔑 RF Remote Learning
- Two buttons can be enrolled (`Learn Button #1`, `Learn Button #2`)
- Stored persistently in ESP32 flash
- Sequence logic:
  - Press → Reverse
  - Press again while reversing → Stop
  - Press again while stopped after reversing → Forward
  - Press while forwarding → Stop

### 🚦 Auto-Close
- Toggle switch: **Auto Close Enabled**
- Adjustable timeout: **0–240 seconds**
- Arms automatically after a full reverse (open) cycle
- Cancels on:
  - Remote stop
  - Manual forward/reverse press
  - Obstacle trigger

### ⚡ Soft Start / Stop
- First 1 s and last 1 s run at reduced duty cycle (default 40%)
- Middle section runs full speed
- Duty can be adjusted via `Soft Start/Stop %`

### 🛡 Obstacle Detection
- Uses IBT-2 current sense pins → ADC on ESP32
- Configurable threshold (A) and debounce (ms)
- If current exceeds threshold for > debounce time:
  - Motor stops immediately
  - Auto-close is cancelled

---

## ESPHome Configuration

The YAML config defines:
- **Globals**: motor state, RF learned codes, auto-close flags
- **Outputs**: PWM for RPWM/LPWM
- **Scripts**: `run_forward`, `run_reverse`, `motor_stop`
- **Sensors**: ADC → current sense → amps
- **UI Controls**:
  - Auto-close toggle + timeout
  - Motor run times
  - Soft-start % duty
  - Obstacle detection threshold & debounce

---

## Setup

1. Clone this repo and copy the YAML into your ESPHome configs folder.
2. Adjust GPIO pins in `output:` / `sensor:` if your wiring differs.
3. Flash to ESP32 with ESPHome.
4. In Home Assistant (or ESPHome dashboard):
   - Enroll your RF remote buttons.
   - Tune `Auto Close Timeout`, `Soft Start %`, and `Obstacle Threshold`.

---

## Safety Notes

- ⚠️ Always test obstacle detection carefully — start with a **high threshold** and lower until reliable.
- ⚠️ Never PWM or switch relays directly on **AC mains motors**. This config is **only for low-voltage DC motors**.
- ⚠️ Add fuses or breakers on the motor supply for protection.

---

## License

MIT — free to use, modify, and share.

---
