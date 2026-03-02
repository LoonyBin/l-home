# L-Home

A modular, wired home automation system built around a central ESP32-C5 controller communicating with peripheral boards over I2C (daisy-chained) and RS-485. Designed to control lights, fans, and other AC/DC loads in a home, with multiple wireless protocols for remote control and integration.

All hardware designs are KiCad 9.0 projects.

## System Architecture

```
[5-24V PSU] → [Controller] ←I2C→ [Fan Regulator] (ceiling fan + light)
                   ↕ I2C          [Relay Board]    (additional lights/appliances)
                   ↕ I2C          [PWM Board]      (LED strips, dimmable lights)
                   ↕ RS-485       (long-distance peripheral boards)
                   ↕ Ethernet     (home network / MQTT / Home Assistant)
                   ↕ 433MHz/IR    (remote controls, legacy appliance control)
```

The controller connects to your home network via Ethernet or Wi-Fi, receives commands (e.g. via MQTT), and dispatches them over I2C/RS-485 to the peripheral boards. Physical wall switches are supported alongside digital control through the screw terminal inputs on the fan regulator.

## Boards

### Controller (Brain)

- **MCU**: ESP32-C5-DevKitC-1 (RISC-V, Wi-Fi 6, Bluetooth 5)
- **Wired interfaces**: Ethernet (LAN8720A RMII PHY), RS-485 (isolated via MAX13432EESD), I2C (level-shifted via PCA9306 with Stemma QT/QWIIC connector), USB-C (programming via CH340C)
- **Wireless interfaces**: 433 MHz TX/RX modules (via 74AHCT125 level shifter), NRF24L01 2.4 GHz, IR blaster (7 LEDs, multi-directional) + IR receiver
- **Power**: 5-24V input → XL1509 (5V) → TLV62565 (3.3V)
- **Extras**: 1x onboard SPDT relay, buzzer, 3 status LEDs, auto-programming circuit with boot/reset buttons

### PWM Board (Dimmable Lighting / Motor Control)

- **16 PWM channels** (4 sub-sheets x 4 channels) driven by a PCA9685 12-bit I2C PWM controller
- **Galvanically isolated**: 6N137 high-speed optocouplers between logic and power
- **Power switching**: IRLZ44N logic-level MOSFETs + SPDT relays per channel
- **Power**: 24V input per driver stage → TPS5430 (5V isolated) on each sub-sheet
- **Manual override** switches per channel, 6-bit DIP switch for I2C address (up to 64 boards)
- **Daisy-chainable** I2C connectors (J1 in, J2 out)

### Relay Board (On/Off Switching)

- **4 relay channels**, each with 2x SPDT relays
- **I2C controlled** via PCF8574P 8-bit I/O expander
- **Galvanically isolated**: EL817 optocouplers between logic and mains
- **Supports 220V AC or 12-24V DC loads** — each channel has its own MB6S bridge rectifier
- **5-pin screw terminals** per channel (NC, NO, L, N, BYPASS)
- **Manual bypass switch** and LED indicator per channel
- 3-bit DIP switch for I2C address, daisy-chainable

### Fan Regulator (Ceiling Fan Speed Control + Light)

- **1 fan channel** with capacitive speed regulation (2.4uF + 4.2uF switched by 4 SPDT relays, ~4-5 discrete speed levels)
- **1 light channel** (on/off relay control)
- **I2C controlled** via PCF8574P, same isolation approach (EL817 + 74AHCT125 + SS8550)
- **7-pin screw terminal** for fan (L, N, FAN, BYPASS, SW_ON, SW_UP, SW_DOWN — supports physical wall switches)
- **5-pin screw terminal** for light (NC, NO, L, N, BYPASS)
- Daisy-chainable, address-configurable

## Design Principles

1. **Safety first**: Every board uses galvanic isolation (optocouplers) between low-voltage logic and mains/high-voltage power. TVS diodes, resettable fuses, and flyback diodes throughout.
2. **Modularity**: Peripheral boards are interchangeable I2C slaves with daisy-chain connectors and DIP-switch addressing. Mix and match boards per room.
3. **Manual override**: Every output channel has a physical bypass switch — the system degrades gracefully if the controller goes down.
4. **Multi-protocol wireless**: 433 MHz (cheap RF remotes/sensors), NRF24L01 (custom sensor nodes), IR (existing appliances like ACs/TVs), plus native Wi-Fi 6 and BLE.
5. **Wired backbone**: RS-485 for long-distance reliable communication; Ethernet for network connectivity; I2C for short-range board-to-board within an enclosure.
6. **AC/DC flexibility**: The relay board and fan regulator work with both 220V AC and 12-24V DC loads.
7. **Capacitive fan regulation**: Traditional capacitor-based speed control (not TRIAC/phase-cut) for ceiling fans, targeting the Indian/Asian home market.

## License

This project is licensed under the [CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S v2.0)](LICENSE).
