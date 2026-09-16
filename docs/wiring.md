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

The Power Relay FeatherWing is mounted on the standard Feather header of the SCORPIO.

The relay control input is configured for:

**D9 / GPIO9**

The SCORPIO's dedicated LED output remains:

**GPIO16 / OUT0 / NEOPIXEL0**

These two GPIOs have different functions.

| SCORPIO pin | Function | Destination |
|---|---|---|
| **GPIO16 / OUT0 / NEOPIXEL0** | HyperSerialPico LED DATA | Relay NO |
| **GPIO9 / D9** | Relay control | Power Relay FeatherWing control input |
| **GND** | Common ground | Common GND |

### Relay control jumper

The Power Relay FeatherWing uses a solder jumper on the underside of the board to select the GPIO used for relay control.

For this project, configure that jumper for **D9 / GPIO9**.

The FeatherWing receives its control signal, 3.3 V supply, and ground through the Feather connection. The GPIO16 LED DATA connection is wired separately to the relay's **NO** contact.

### Important

GPIO16 and GPIO9 must not be swapped.

```text
GPIO16 → LED DATA
GPIO9  → Relay control
```

The Power Relay FeatherWing relay draws approximately **100 mA from the 3.3 V rail while energized**. This load should be considered when adding other 3.3 V accessories to the SCORPIO.

---

# 3. Relay Connections

The relay has three relevant contacts:

* **NC** — Normally Closed
* **NO** — Normally Open
* **COM** — Common

Connect them as follows:

| Relay | Connection |
|---|---|
| **NC** | WLED DATA OUT |
| **NO** | SCORPIO GPIO16 / OUT0 |
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

When the relay is **de-energized**, NC is connected to COM.

When the relay is **energized**, NO is connected to COM.

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

Before connecting an alternative WLED controller, verify that its DATA output is electrically suitable for the SK6812 strip and for the selected wiring.

---

# 5. SCORPIO DATA Connection

Connect:

```text
SCORPIO GPIO16 / OUT0 / NEOPIXEL0
          │
          ▼
       Relay NO
```

On the SCORPIO, GPIO16 is the first pin of the dedicated 8-pin NeoPixel/PIO header and is identified as **NEOPIXEL0**. The eight GPIO16–23 signals pass through the SCORPIO's onboard level shifter.

Do not connect the SCORPIO DATA signal to NC.

---

# 6. Relay COM to LED Strip

The relay COM contact is connected to the LED strip through a **330 Ω series resistor**.

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

Because the SCORPIO GPIO16 output is already level-shifted by the SCORPIO hardware, no additional level shifter is required on the SCORPIO DATA path. For the WLED side, use a controller with a suitable LED DATA output or provide appropriate level shifting where required.

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

Use a suitable 5 V supply and appropriately sized wiring for the actual LED load.

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

If using a controller such as the Gledopto GL-C-016WL-D, follow that controller's own power and wiring requirements.

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

The relay control ground is provided through the Feather connection between the SCORPIO and the Relay FeatherWing.

---

# 10. SCORPIO USB Power

The SCORPIO is powered from an LG TV USB port.

For the automatic switching logic used by this project, the selected TV USB port must **turn off when the TV is switched off**.

```text
LG TV USB
    │
    └────► SCORPIO USB
```

This has an important functional purpose.

When the TV is switched off and the selected USB port loses power:

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

When the TV is switched on and the USB port powers the SCORPIO:

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

The exact USB power behavior depends on the TV model and USB port. Verify this behavior on the actual TV used in the installation.

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

Do not use the relay contacts to switch the LED power in this project. The relay is used only for the low-voltage DATA signal.

---

# 12. Relay State Table

The complete switching logic is:

| Condition | SCORPIO | GPIO9 | Relay | DATA path | Controller |
|---|---|---|---|---|---|
| TV OFF | Off | Inactive | NC → COM | WLED → LED strip | WLED |
| TV ON | On | HIGH | NO → COM | SCORPIO GPIO16 → LED strip | HyperSerialPico |

The Power Relay FeatherWing uses a non-latching relay: the relay remains energized only while the control signal is active.

---

# 13. DATA Switching and Startup Behavior

The relay switches the DATA source only. It does not switch the LED power.

During the transition from TV OFF to TV ON, the SCORPIO must boot and initialize GPIO9 before the relay changes to the SCORPIO DATA path.

During the transition from TV ON to TV OFF, the SCORPIO loses power and the relay returns to its de-energized NC → COM state.

A brief visual glitch can occur while a mechanical relay changes contact state. This is normal for a mechanical relay used for DATA switching.

Do not rely on the relay for signal isolation or mains switching in this project.

---

# 14. Electrical and Safety Notes

* Use a suitable **5 V power supply** for the actual LED load.
* Size LED power wiring appropriately for the strip current.
* Keep the **330 Ω resistor** close to the LED DATA input where practical.
* Keep **SCORPIO GND, WLED GND, LED PSU GND, and LED strip GND** at a common reference.
* Verify the DATA voltage compatibility of any alternative WLED controller.
* Do not connect the LG TV USB +5 V to the external LED PSU +5 V.
* The relay switches **DATA only**; LED power remains permanently connected.
* The Power Relay FeatherWing relay is a non-latching relay and draws approximately **100 mA from the 3.3 V rail while energized**.
* Do not use this circuit to switch mains voltage. The relay contacts are used here only for the low-voltage LED DATA signal.
* Disconnect power before changing wiring or solder-jumper configuration.

---

# 15. Quick Wiring Checklist

Before powering the system, verify:

```text
[ ] SCORPIO GPIO16 / NEOPIXEL0 → Relay NO
[ ] SCORPIO GPIO9 / D9 → Relay control
[ ] Relay control jumper → D9 / GPIO9
[ ] WLED DATA OUT → Relay NC
[ ] Relay COM → 330 Ω → SK6812 DIN
[ ] External 5 V → LED strip +5 V
[ ] External GND → LED strip GND
[ ] External GND → WLED GND
[ ] External GND → SCORPIO GND
[ ] LG TV USB → SCORPIO USB
[ ] LG TV USB +5 V is NOT connected to external PSU +5 V
[ ] WLED controller power input is compatible with the selected PSU
[ ] TV USB port turns off when the TV is switched off
```

If all items are confirmed, test WLED and HyperSerialPico independently before relying on automatic relay switching.
