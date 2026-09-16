# Wiring Guide

This document describes the complete wiring of the dual-controller Ambilight system.

The system uses one **SK6812 RGBW LED strip** with two independent controllers:

1. **Adafruit Feather RP2040 SCORPIO + HyperSerialPico**
2. **ESP32 + WLED**

An **Adafruit Power Relay FeatherWing** automatically switches the LED DATA signal between the two controllers.

> **Important:** Only the LED DATA signal is switched. The LED power supply remains permanently connected.

---

# 1. System Wiring Overview

```text
                         LG TV
                    ┌──────────────┐
                    │              │
                    │ USB          │
                    └──────┬───────┘
                           │
                           │ USB power
                           ▼
              ┌─────────────────────────┐
              │ Feather RP2040 SCORPIO  │
              │                         │
              │ GPIO16 / OUT0 ──────────┼──────────────► Relay NO
              │                         │
              │ GPIO9 / D9 ─────────────┼──────────────► Relay CONTROL
              │                         │
              │ GND ────────────────────┼──────┐
              └─────────────────────────┘      │
                                               │
                                               │
                                    ┌──────────▼──────────┐
                                    │ Power Relay         │
                                    │ FeatherWing         │
                                    │                     │
                      WLED DATA ───►│ NC                  │
                                    │                     │
                   SCORPIO GPIO16 ─►│ NO                  │
                                    │                     │
                                    │ COM ──────┬─────────┘
                                    └───────────┼
                                                │
                                              330 Ω
                                                │
                                                ▼
                                     ┌────────────────────┐
                                     │ SK6812 RGBW        │
                                     │ LED Strip          │
                                     │                    │
                                     │ DIN ◄──────────────┘
                                     │ +5V ◄───────────────┐
                                     │ GND ◄──────────────┐│
                                     └────────────────────┘│
                                                          │
                       ┌────────────────────┐             │
                       │ External 5 V PSU   │             │
                       │                    │             │
                       │ +5 V ──────────────┼─────────────┘
                       │ GND ───────────────┼──────┐
                       └────────────────────┘      │
                                                   │
                    ┌────────────────────┐         │
                    │ ESP32 / WLED       │         │
                    │                    │         │
                    │ DATA OUT ──────────┼────────► Relay NC
                    │ GND ───────────────┼─────────┘
                    │ +5 V* ◄────────────┼────────── PSU +5 V
                    └────────────────────┘

* Only if supported by the selected WLED controller.
```

---

# 2. SCORPIO and Relay FeatherWing

The Power Relay FeatherWing is mounted on the Feather header of the SCORPIO.

The relay control input is configured for:

**D9 / GPIO9**

The SCORPIO's dedicated LED output remains:

**GPIO16 / OUT0 / NEOPIXEL0**

These two GPIOs have different functions.

| SCORPIO pin   | Function                 | Destination         |
| ------------- | ------------------------ | ------------------- |
| GPIO16 / OUT0 | HyperSerialPico LED DATA | Relay NO            |
| GPIO9 / D9    | Relay control            | Relay control input |
| GND           | Common ground            | Common GND          |

### Important

GPIO16 and GPIO9 must not be swapped.

```text
GPIO16 → LED DATA
GPIO9  → Relay control
```

The Relay FeatherWing can obtain the control signal, 3.3 V and GND through the Feather connection when configured for D9.

The GPIO16 LED DATA connection is wired separately to the relay's **NO** contact.

---

# 3. Relay Connections

The relay has three relevant contacts:

* **NC** — Normally Closed
* **NO** — Normally Open
* **COM** — Common

Connect them as follows:

| Relay   | Connection                  |
| ------- | --------------------------- |
| **NC**  | WLED DATA OUT               |
| **NO**  | SCORPIO GPIO16 / OUT0       |
| **COM** | 330 Ω resistor → SK6812 DIN |

The relay therefore acts as a DATA multiplexer:

```text
                 ┌─────────────┐
WLED DATA ──────►│ NC          │
                 │             │
SCORPIO DATA ───►│ NO       COM├────► 330 Ω ───► LED DIN
                 └─────────────┘
```

Only one DATA source is connected to COM at a time.

---

# 4. WLED DATA Connection

Connect the DATA output of the ESP32/WLED controller to:

```text
WLED DATA OUT
      │
      ▼
Relay NC
```

The exact WLED GPIO depends on the selected controller.

For this reason, this project documentation refers to the connection simply as:

**WLED DATA OUT**

Do not assume a particular GPIO when using a different WLED controller.

---

# 5. SCORPIO DATA Connection

Connect:

```text
SCORPIO GPIO16 / OUT0
          │
          ▼
       Relay NO
```

GPIO16 is the first dedicated SCORPIO NeoPixel output and is used by the SCORPIO HyperSerialPico configuration.

Do not connect the SCORPIO DATA signal to NC.

---

# 6. Relay COM to LED Strip

The relay COM contact is connected to the LED strip through a 330 Ω series resistor.

```text
Relay COM
    │
    ▼
  330 Ω
    │
    ▼
SK6812 RGBW DIN
```

The resistor should be placed as close to the LED DATA input as practical.

No additional level shifter is required when using the specified level-shifted controller outputs.

---

# 7. LED Power

The SK6812 RGBW strip is powered directly from the external 5 V power supply.

```text
External 5 V PSU
       │
       ├──── +5 V ────► LED strip +5 V
       │
       └──── GND ─────► LED strip GND
```

The LED strip power is **not switched by the relay**.

This means the LED strip remains permanently powered while the external LED power supply is switched on.

---

# 8. WLED Power

The WLED controller is powered from the external LED power supply where supported by the controller.

```text
External 5 V PSU
       │
       ├──── +5 V ────► WLED +5 V input
       │
       └──── GND ─────► WLED GND
```

Always follow the power requirements of the specific WLED controller being used.

Some WLED boards have their own power input or regulator and should not be connected directly to a 5 V rail in the same way as another board.

---

# 9. Common Ground

A common ground is required between the LED system and both controllers.

Connect:

```text
LED PSU GND
     │
     ├────► SK6812 GND
     │
     ├────► WLED GND
     │
     └────► SCORPIO GND
```

The common ground provides the electrical reference for the LED DATA signal.

Without a common ground, reliable DATA transmission cannot be guaranteed.

---

# 10. SCORPIO USB Power

The SCORPIO is powered from the LG TV USB port.

```text
LG TV USB
    │
    └────► SCORPIO USB
```

This has an important functional purpose.

When the TV is switched off, the LG TV USB port also switches off.

Therefore:

```text
TV OFF
  │
  ▼
SCORPIO loses power
  │
  ▼
GPIO9 inactive
  │
  ▼
Relay de-energized
  │
  ▼
NC → COM
  │
  ▼
WLED controls the LEDs
```

When the TV is switched on:

```text
TV ON
  │
  ▼
SCORPIO receives USB power
  │
  ▼
HyperSerialPico starts
  │
  ▼
GPIO9 = HIGH
  │
  ▼
Relay energized
  │
  ▼
NO → COM
  │
  ▼
HyperSerialPico controls the LEDs
```

---

# 11. ⚠️ Do Not Connect the Two +5 V Supplies

The LG TV USB +5 V and the external LED PSU +5 V are separate power sources.

**Do not connect them together.**

Correct:

```text
LG TV USB +5 V
      │
      └────► SCORPIO only


External LED PSU +5 V
      │
      ├────► SK6812 RGBW
      │
      └────► WLED*
```

The grounds are shared, but the two +5 V supplies remain separate.

* where supported by the WLED controller.

---

# 12. Relay State Table

The complete switching logic is:

| Condition | SCORPIO | GPIO9 | Relay | DATA path | Controller |
|---|---|
