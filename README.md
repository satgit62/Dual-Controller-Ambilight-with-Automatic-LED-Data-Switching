# Dual-Controller-Ambilight-with-Automatic-LED-Data-Switching

A dual-controller Ambilight setup that allows an **SK6812 RGBW LED strip** to be shared between an **Adafruit Feather RP2040 SCORPIO running HyperSerialPico** and an **ESP32 running WLED**.

The LED strip does not need to be physically disconnected or rewired when switching between Ambilight and WLED.

A hardware relay automatically switches the **LED DATA signal** depending on whether the TV is powered on.

---

## ✨ Features

* 🎬 Dynamic Ambilight using **LG TV + PicCap + HyperHDR**
* 💡 **Adafruit Feather RP2040 SCORPIO** running HyperSerialPico
* 🌈 **SK6812 RGBW** LED strip
* 📱 **ESP32 + WLED** for standalone lighting
* 🎵 Music-reactive WLED effects using the ESP32 microphone
* 🔄 Automatic switching between Ambilight and WLED
* 🔌 No need to physically reconnect the LED strip
* ⚡ LED power remains permanently connected
* 🔀 Only the low-voltage **DATA signal** is switched
* 🛡️ Both controllers use level-shifted LED outputs

---

## 📖 Overview

The project combines two independently working LED controllers:

### TV ON

The LG TV powers the SCORPIO through USB.

The SCORPIO starts HyperSerialPico and activates the relay using **GPIO9**.

The relay connects the SCORPIO's LED DATA output to the LED strip.

```text
LG TV
  │
  │ USB
  ▼
SCORPIO
  │
  │ GPIO16 / OUT0
  ▼
Relay NO ── COM ── 330 Ω ──► SK6812 RGBW
```

The LEDs are therefore controlled by HyperHDR through HyperSerialPico.

### TV OFF

When the TV is switched off, its USB port no longer powers the SCORPIO.

GPIO9 becomes inactive and the relay returns to its normally closed position.

The WLED controller then controls the same LED strip.

```text
ESP32 / WLED
  │
  │ DATA
  ▼
Relay NC ── COM ── 330 Ω ──► SK6812 RGBW
```

This makes the transition between the two systems automatic.

---

## 🖼️ Wiring Diagram

The complete wiring diagram will be available here:

```text
docs/images/wiring-diagram.png
```

An SVG version is also provided for lossless scaling:

```text
docs/images/wiring-diagram.svg
```

> **Note:** The graphical wiring diagram is being finalized and will be added to the repository.

---

## 🔧 Hardware

### Main Components

| Component                        | Function                             |
| -------------------------------- | ------------------------------------ |
| Adafruit Feather RP2040 SCORPIO  | HyperSerialPico Ambilight controller |
| Adafruit Power Relay FeatherWing | LED DATA source switching            |
| ESP32 / WLED controller          | Standalone LED controller            |
| SK6812 RGBW LED strip            | Shared LED strip                     |
| External 5 V power supply        | LED and WLED power                   |
| 330 Ω resistor                   | LED DATA series resistor             |
| LG TV                            | Video source / USB power             |
| PicCap                           | Captures the TV video signal         |
| HyperHDR                         | Generates Ambilight data             |

The WLED controller can be a **Gledopto GL-C-016WL-D** or another compatible ESP32-based WLED controller with an appropriate level-shifted LED output.

---

# 🔌 Wiring

The relay switches **only the LED DATA line**.

The LED power supply remains permanently connected.

### DATA connections

| From                  | To              |
| --------------------- | --------------- |
| WLED DATA OUT         | Relay **NC**    |
| SCORPIO GPIO16 / OUT0 | Relay **NO**    |
| Relay **COM**         | 330 Ω resistor  |
| 330 Ω resistor        | SK6812 RGBW DIN |
| SCORPIO GPIO9 / D9    | Relay control   |

### Power connections

```text
External 5 V PSU
│
├── +5 V ─────────► SK6812 RGBW +5 V
│
├── +5 V ─────────► WLED controller*
│
└── GND ──────────► Common GND
                       │
                       ├── SK6812 GND
                       ├── WLED GND
                       └── SCORPIO GND
```

* Only if supported by the selected WLED controller.

### SCORPIO power

The SCORPIO is powered independently from the LG TV USB port:

```text
LG TV USB
   │
   └──► SCORPIO USB
```

### ⚠️ Important

**Do NOT connect the LG TV USB +5 V to the external LED PSU +5 V.**

The TV USB power is used to power the SCORPIO only.

The LED strip and WLED controller are powered from the dedicated 5 V LED power supply.

The grounds must be common.

---

# 📌 Pinout

Only two SCORPIO GPIOs are relevant to the relay-based switching system.

| SCORPIO        | Function         | Connection                 |
| -------------- | ---------------- | -------------------------- |
| **GPIO16**     | NEOPIXEL0 / OUT0 | Relay NO                   |
| **GPIO9 / D9** | Relay control    | Power Relay FeatherWing    |
| **GND**        | Common ground    | LED PSU / WLED / LED strip |

### GPIO16 — LED DATA

GPIO16 is the first dedicated SCORPIO NeoPixel output:

```text
GPIO16
  │
  └── OUT0 / NEOPIXEL0
```

This remains the HyperSerialPico LED DATA output.

### GPIO9 — Relay Control

GPIO9 is used exclusively to control the Power Relay FeatherWing.

It does **not** replace GPIO16 as the LED DATA output.

---

# 🔀 Relay Logic

The Power Relay FeatherWing is configured to use **D9 / GPIO9** as its control pin.

| TV state | GPIO9    | Relay    | LED DATA source |
| -------- | -------- | -------- | --------------- |
| TV OFF   | Inactive | NC → COM | WLED            |
| TV ON    | HIGH     | NO → COM | HyperSerialPico |

### TV OFF

```text
SCORPIO OFF
     │
     ▼
GPIO9 inactive
     │
     ▼
Relay de-energized
     │
     ▼
NC ── COM
     │
     ▼
WLED controls LEDs
```

### TV ON

```text
SCORPIO boots
     │
     ▼
GPIO9 = HIGH
     │
     ▼
Relay energized
     │
     ▼
NO ── COM
     │
     ▼
HyperSerialPico controls LEDs
```

---

# 💾 HyperSerialPico Modification

The HyperSerialPico SCORPIO firmware needs one additional startup initialization for GPIO9.

Add the following code during firmware initialization:

```cpp
// Configure GPIO9 as an output and activate the relay
gpio_init(9);
gpio_set_dir(9, GPIO_OUT);
gpio_put(9, 1);
```

The GPIO should be initialized once during startup rather than inside the LED update loop.

The roles remain:

```text
GPIO16 → LED DATA
GPIO9  → Relay control
```

> **Important:** Use the dedicated SCORPIO configuration/build of HyperSerialPico. Do not replace the SCORPIO LED output configuration with GPIO9.

The exact patch location can depend on the HyperSerialPico source version being used. The repository therefore documents the required code change rather than assuming a specific upstream source layout.

---

# 🧠 System Architecture

```text
                         ┌──────────────────┐
                         │      LG TV        │
                         │                  │
                         │ PicCap + USB     │
                         └────────┬─────────┘
                                  │
                         USB power│
                                  ▼
                    ┌─────────────────────────┐
                    │ Adafruit Feather        │
                    │ RP2040 SCORPIO          │
                    │                         │
                    │ HyperSerialPico         │
                    │                         │
                    │ GPIO16 ─────────────┐   │
                    │ GPIO9 ──────────┐  │   │
                    └─────────────────│──│───┘
                                      │  │
                                      │  │
                                      │  ▼
                                      │ ┌──────────────┐
                                      │ │ Power Relay  │
                                      │ │ FeatherWing  │
                                      │ │              │
                         WLED DATA ───┼─►│ NC          │
                                      │ │              │
                         GPIO16 ──────┼─►│ NO          │
                                        │              │
                                        │ COM          │
                                        └──────┬───────┘
                                               │
                                             330 Ω
                                               │
                                               ▼
                                      ┌────────────────┐
                                      │ SK6812 RGBW    │
                                      │ LED Strip      │
                                      └────────────────┘

          ┌───────────────────┐
          │ External 5 V PSU  │
          │                   │
          │ +5 V ─────────────┼────► LED strip
          │                   │
          │ GND ──────────────┼────► Common GND
          └───────────────────┘
                       │
                       └────────────► WLED / SCORPIO GND
```

---

# ⚡ Why Switch DATA Instead of LED Power?

The LED strip is permanently connected to the external 5 V supply.

Only the controller providing the DATA signal is switched.

This avoids switching the relatively high current required by the LED strip.

### Advantages

* No high-current relay switching
* LED power remains stable
* Both controllers can remain permanently connected
* Automatic controller selection
* Simple hardware implementation
* WLED remains available whenever the TV is off
* HyperSerialPico automatically takes over when the TV is powered on

---

# 🌈 WLED Mode

When the TV is off, WLED becomes the active LED controller.

This allows the same Ambilight installation to be used as a standalone lighting system.

Possible applications include:

* Static ambient lighting
* WLED effects
* Color presets
* Music-reactive effects
* ESP32 microphone-based audio visualization
* Remote control from a phone

---

# 🎬 Ambilight Mode

When the TV is on:

```text
LG TV
  │
  ▼
PicCap
  │
  ▼
HyperHDR
  │
  ▼
HyperSerialPico
  │
  ▼
SCORPIO GPIO16
  │
  ▼
Relay
  │
  ▼
SK6812 RGBW
```

The relay automatically connects the SCORPIO DATA output to the LED strip.

---

# 🧪 Testing

Both controller setups should first be tested independently.

## 1. Test WLED

Connect the LED strip directly to the WLED controller.

Verify:

* LED colors
* LED count
* RGBW configuration
* WLED effects
* music-reactive mode, if used

## 2. Test HyperSerialPico

Connect the LED strip directly to SCORPIO GPIO16 / OUT0.

Verify:

* HyperSerialPico communication
* HyperHDR output
* correct LED colors
* correct RGBW operation

## 3. Install the relay

Connect:

```text
WLED DATA OUT ─────► NC

SCORPIO GPIO16 ────► NO

COM ── 330 Ω ──────► SK6812 DIN
```

Leave the LED power wiring unchanged.

## 4. Test with TV OFF

Expected:

```text
SCORPIO = OFF
Relay   = NC
WLED    = active
```

## 5. Test with TV ON

Expected:

```text
SCORPIO = ON
GPIO9   = HIGH
Relay   = NO
HyperSerialPico = active
```

---

# ⚠️ Important Electrical Notes

* The relay is used only for the **low-voltage LED DATA signal**.
* Do not use this design to switch mains voltage.
* Use a suitable 5 V power supply for the total LED current.
* Ensure the LED PSU wiring is appropriately sized.
* Keep the 330 Ω DATA resistor close to the LED DATA input.
* All three systems must share a common ground:

  * SCORPIO
  * WLED
  * LED power supply
* Do not connect the LG TV USB +5 V to the LED PSU +5 V.
* No additional level shifter is required when using compatible level-shifted outputs from the two controllers.

---

# 📦 Repository Structure

```text
dual-controller-ambilight/
│
├── README.md
├── LICENSE
│
├── hardware/
│   └── BOM.md
│
├── docs/
│   ├── wiring.md
│   ├── pinout.md
│   ├── firmware.md
│   └── images/
│       ├── wiring-diagram.png
│       └── wiring-diagram.svg
│
└── firmware/
    └── hyperserialpico-gpio9.patch
```

---

# 📋 Bill of Materials

| Qty. | Component                          |
| ---: | ---------------------------------- |
|    1 | Adafruit Feather RP2040 SCORPIO    |
|    1 | Adafruit Power Relay FeatherWing   |
|    1 | ESP32 / compatible WLED controller |
|    1 | SK6812 RGBW LED strip              |
|    1 | Suitable 5 V LED power supply      |
|    1 | 330 Ω resistor                     |
|    1 | Set of suitable Feather headers    |
|    — | Suitable wiring/connectors         |

---

# 🔗 Related Projects

This project builds on several excellent open-source projects and hardware platforms:

* **HyperSerialPico** — high-speed LED control for Raspberry Pi Pico / RP2040
* **HyperHDR** — Ambilight processing
* **PicCap** — webOS video capture
* **WLED** — ESP32-based addressable LED controller
* **Adafruit Feather RP2040 SCORPIO** — multi-output RP2040 LED controller
* **Adafruit Power Relay FeatherWing** — relay switching

Please refer to the respective upstream projects for their documentation, licenses, and original source code.

---

# 📜 License

This repository contains original documentation, wiring information, and configuration created for this project.

Third-party software and hardware remain subject to their respective licenses and terms.

See [`LICENSE`](LICENSE) for the license covering the original material in this repository.

---

# 🤝 Contributing

Suggestions, corrections, improvements, additional controller compatibility, and tested configurations are welcome.

If you build this setup with another WLED-compatible controller or a different SK6812 RGBW configuration, feel free to document your results and share them with the project.
