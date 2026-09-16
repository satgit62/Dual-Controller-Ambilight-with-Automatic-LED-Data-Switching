# Bill of Materials (BOM)

This document lists the hardware required to build the dual-controller Ambilight setup.

The system uses a shared **SK6812 RGBW LED strip** with two independent LED controllers:

* Adafruit Feather RP2040 SCORPIO running HyperSerialPico
* ESP32-based WLED controller

A relay switches the LED DATA signal between the two controllers.

---

## Required Hardware

| Qty. | Component                            | Purpose / Notes                                                       |
| ---: | ------------------------------------ | --------------------------------------------------------------------- |
|    1 | **Adafruit Feather RP2040 SCORPIO**  | HyperSerialPico Ambilight controller                                  |
|    1 | **Adafruit Power Relay FeatherWing** | Switches the LED DATA signal                                          |
|    1 | **ESP32 / WLED controller**          | Standalone WLED controller                                            |
|    1 | **SK6812 RGBW LED strip**            | Shared LED strip                                                      |
|    1 | **5 V DC power supply**              | Powers the LED strip and WLED controller                              |
|    1 | **330 Ω resistor**                   | Series resistor in the LED DATA line                                  |
|    1 | **USB cable for SCORPIO**            | Connects SCORPIO to the LG TV                                         |
|    1 | **Feather headers**                  | Required for stacking the Relay FeatherWing, if not already installed |
|    — | **Suitable wires/connectors**        | Power, DATA and GND connections                                       |

---

## WLED Controller

The WLED controller can be an ESP32-based board with an appropriate level-shifted LED DATA output.

The tested configuration uses a controller compatible with the:

**Gledopto GL-C-016WL-D**

Other ESP32/WLED controllers can be used if they provide:

* compatible SK6812 RGBW support
* a suitable 5 V LED DATA output
* a common GND connection
* suitable power input for the selected board

The exact DATA output pin is therefore controller-dependent and is referred to as **WLED DATA OUT** throughout this documentation.

---

## LED Strip

The project is designed for:

**SK6812 RGBW**

The LED strip requires:

* `+5 V`
* `GND`
* `DIN`

The LED strip is powered directly from the external 5 V power supply.

The DATA signal is supplied by either:

* SCORPIO GPIO16 / OUT0, or
* WLED DATA OUT

The relay selects which controller is connected to `DIN`.

---

## Power Supply

The external 5 V power supply must be appropriately sized for the total LED load.

The required current depends primarily on:

* LED strip length
* number of LEDs
* brightness
* RGBW usage
* maximum expected power consumption

The power supply should therefore be selected for the actual LED installation rather than using a fixed wattage recommendation.

### Power distribution

```text
External 5 V PSU
│
├── +5 V ───────► SK6812 RGBW LED strip
│
├── +5 V ───────► WLED controller*
│
└── GND ────────► Common GND
                     │
                     ├── SK6812 GND
                     ├── WLED GND
                     └── SCORPIO GND
```

* Only where supported by the selected WLED controller.

---

## SCORPIO Power

The SCORPIO is powered independently through the LG TV USB port.

```text
LG TV USB
    │
    └────► SCORPIO USB
```

The LG TV USB power is **not** used to power the LED strip.

### ⚠️ Important

**Do not connect the LG TV USB +5 V directly to the external LED PSU +5 V.**

The two +5 V supplies remain separate.

Only the **GND reference** is shared between the LED system and SCORPIO.

---

## DATA Signal Components

Only one additional component is required in the LED DATA path:

### 330 Ω Series Resistor

The resistor is installed between the relay COM contact and the LED strip DATA input.

```text
Relay COM
   │
   ▼
330 Ω
   │
   ▼
SK6812 RGBW DIN
```

The resistor should be located close to the LED strip DATA input where practical.

---

## Relay

The **Adafruit Power Relay FeatherWing** is used exclusively to switch the low-voltage LED DATA signal.

Relay connections:

| Relay terminal | Connected to            |
| -------------- | ----------------------- |
| **NC**         | WLED DATA OUT           |
| **NO**         | SCORPIO GPIO16 / OUT0   |
| **COM**        | 330 Ω → SK6812 RGBW DIN |

The relay control input is configured for:

**D9 / GPIO9**

---

## No Additional Level Shifter

No additional level shifter is required in this design.

Both the SCORPIO and the selected WLED controller provide suitable level-shifted LED DATA outputs.

The signal path is therefore:

```text
SCORPIO GPIO16 ──► Relay NO
                       │
WLED DATA OUT ────► Relay NC
                       │
                    Relay COM
                       │
                     330 Ω
                       │
                       ▼
                 SK6812 RGBW DIN
```

---

## Optional / Installation-Specific Hardware

Depending on the physical installation, the following may also be required:

* additional 5 V power injection wiring
* suitable connectors
* cable clips or mounting hardware
* inline fuse or appropriate power protection
* suitable wire gauge for LED power
* enclosure or mounting hardware

These items depend on the individual LED installation and are not specific to the controller switching circuit.

---

## Hardware Summary

The essential switching architecture is:

```text
                 ┌─────────────────┐
                 │ SCORPIO         │
                 │ GPIO16 / OUT0   │
                 └────────┬────────┘
                          │
                          ▼
                        NO
                    ┌──────────┐
                    │  RELAY   │
                    │          │
 WLED DATA ───────► │ NC   COM ├──► 330 Ω ──► SK6812 DIN
                    └──────────┘
                          ▲
                          │
                       GPIO9
                    Relay control
```

The LED power remains permanently connected to the external 5 V supply.

Only the DATA source is switched.

