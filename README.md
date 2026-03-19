# AC6328_PWM

BLE-controlled dual-motor PWM firmware for AC6328 (Jieli JLAI SDK).

This firmware implements a BLE GATT server to control two motors (port & starboard) using:
- RC ESC pulse-width (µs)
- Direct BLDC duty cycle (0–100%)

Designed for low-latency real-time control from an Android app.

---

## Features

- BLE GATT service (ae00)
- Dual motor control (port / starboard)
- ESC PWM mode (50 Hz)
- BLDC PWM mode (1 kHz)
- Command echo via notify
- Runtime mode switching
- Battery + uptime telemetry
- Safe stop on disconnect
- STOP command

---

## BLE Protocol

Service UUID:
ae00

---

### ae03 — Motor Command (Write Without Response)

5-byte packet:

Byte 0: CMD
Byte 1-2: Port value (uint16 LE)
Byte 3-4: Starboard value (uint16 LE)

Commands:

0x01 = ESC PWM (µs)
0x02 = BLDC DUTY (0–10000)
0xFF = STOP

---

### ae02 — Command Echo (Notify)

Returns the same 5-byte packet after execution.

---

### ae10 — Status + Mode Control

Write (1 byte):
0x01 = ESC mode
0x02 = BLDC mode

Mode switch will stop motors.

Read response (ASCII):
M<mode>A<vbat_mv>T<uptime_min>

Example:
M1A3712T5

M = mode (1=ESC, 2=BLDC)
A = battery voltage (mV)
T = uptime (minutes)

---

## Motor Control

### ESC Mode
- Frequency: 50 Hz
- Input: pulse width (µs)

Range:
ESC_US_MIN = 500
ESC_US_MAX = 1000

Note: This is non-standard (typical ESC is 1000–2000 µs)

---

### BLDC Mode
- Frequency: 1 kHz
- Input: 0–10000

Conversion:
duty (%) = value / 100

---

## Hardware Mapping

Port motor:
  Pin: IO_PORT_DM
  Timer: JL_TIMER3

Starboard motor:
  Pin: IO_PORT_DP
  Timer: JL_TIMER2

---

## Safety

- STOP command always works
- Mode switch stops motors
- BLE disconnect stops motors

Watchdog (disabled in code):
WATCHDOG_TIMEOUT_MS = 2000

Enable with:
sys_timer_add(NULL, pwm_watchdog, 200);

---

## Example Commands

ESC 1500 µs:
[0x01, 0xDC, 0x05, 0xDC, 0x05]

BLDC 50%:
[0x02, 0x88, 0x13, 0x88, 0x13]

STOP:
[0xFF, 0x00, 0x00, 0x00, 0x00]

---

## Initialization

Entry point:
bt_ble_init()

Steps:
1. Init BLE stack
2. Set device name ("PWM")
3. Configure advertising (ae00)
4. Enable BLE

---

## Notes

- Uses Write Without Response for low latency
- Notify echo confirms execution
- No floating point (efficient + deterministic)

---

## Future Improvements

- Enable watchdog
- Add ramp / slew rate limiting
- Add CRC or sequence ID
- Add telemetry (RPM, current)
- Add deadband handling
