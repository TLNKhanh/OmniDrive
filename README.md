# OmniDrive

Dual H-bridge motor driver + ESP32 + USB-C PD, squeezed into a 45×59.5mm 2-layer board.

I wanted a board that I could plug a single USB-C cable into — no bench supply, no programmer, no extra wiring — and have it drive two motors while reporting current over WiFi. This is the result.

![PCB](hardware/photos/assembled.jpg)

## What it does

- Drives 2 DC motors (or 1 stepper) with two DRV8833 H-bridges, up to 1.5A per channel
- Negotiates power over USB-C via CH224K — any PD charger from 5V to 20V works
- Measures total motor current with an INA219 over I2C
- Lets you program the ESP32 over USB through the CH340C, with automatic boot/reset
- Breaks out 14 GPIOs + I2C on a header for sensors, encoders, whatever

## Why I built it

I'm working on a small robot arm and got tired of having a separate motor driver board, an ESP32 dev board, a USB-UART dongle, and a bench power supply all connected with jumper wires. OmniDrive replaces most of that with one board.

## Specs

| Part | What |
|------|------|
| MCU | ESP32-WROOM-32E |
| Motor drivers | 2× DRV8833 (HTSSOP-16, 1.5A/ch) |
| USB-C PD | CH224K sink, negotiates 5–20V |
| Current sense | INA219, 0.1Ω shunt, I2C |
| USB-UART | CH340C with auto DTR/RTS |
| Logic power | ME6211C50 (5V) → ME6211C33 (3.3V) |
| Breakout | 14 GPIO, I2C, WS2812 on 1×8 header |
| PCB | 2-layer, 1.6mm FR4, 45×59.5mm |

## Power flow

USB-C VBUS → CH224K negotiates voltage → +BATT rail (5–20V depending on charger)

+BATT splits three ways:
- Directly to the DRV8833 motor drivers
- Through a 5V LDO (ME6211C50) for the CH340C and ESP32's 5V input
- 5V then goes through a 3.3V LDO (ME6211C33) for the ESP32, INA219, WS2812

Motor current goes through a shared 0.1Ω sense resistor before reaching the drivers, so the INA219 sees total bus current. Per-motor would need two INA219s — maybe rev 2.

## Known issues / things to fix

This is rev 1, so there are a few things I'll change next time:

**GPIO12 strapping** — I used GPIO12 as a motor control pin, but it's also the MTDI strapping pin for flash voltage. It'll probably work with the right pull resistors, but it's not ideal. Would move to a different pin in rev 2.

**LDO dropout** — The ME6211C33 is rated for 500mA and ESP32 can spike past that during WiFi TX. On a long USB-C cable with some voltage drop, the 5V rail might sag enough to make the 3.3V regulator drop out. An AP2112 or similar low-dropout part would be safer.

**No ESD protection** — Didn't add TVS diodes on the USB-C CC/D+/D- lines or the motor outputs. Fine for a desk prototype, not fine for anything that gets touched a lot.

**Shared current sense** — As mentioned, only total motor current. Can't tell which motor is drawing what.

## Files in hardware/

```
hardware/
├── schematic.pdf          — full schematic
├── PCB_Front.pdf          — front copper render
├── PCB_Bottom.pdf         — bottom copper render
├── gerbers.zip            — Gerbers + drill files
├── bom.csv                — BOM for JLCPCB assembly
├── pick-place.csv         — centroid file
├── kicad/                 — source KiCad project
└── photos/                — board photos (TBD)
```

## License

Hardware: CERN-OHL-S-2.0  
Firmware: MIT  
Docs: CC-BY-SA-4.0
