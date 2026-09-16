# Firmware

This project uses **HyperSerialPico v11.1.0** on the Adafruit Feather RP2040 SCORPIO.

The SCORPIO is used as the Ambilight LED controller when the TV is powered on.

The only modification required for the automatic DATA switching is the initialization of **GPIO9**, which controls the relay.

---

## HyperSerialPico Version

The project is based on:

**HyperSerialPico v11.1.0**

The official v11.1.0 release provides a dedicated:

```text
Adafruit_Feather_RP2040_Scorpio.zip
```

build for the SCORPIO.

Use the dedicated SCORPIO firmware rather than treating the board as a generic RP2040/Pico installation.

Official project:

[awawa-dev/HyperSerialPico on GitHub](https://github.com/awawa-dev/HyperSerialPico?utm_source=chatgpt.com)

Official releases:

[HyperSerialPico Releases](https://github.com/awawa-dev/HyperSerialPico/releases?utm_source=chatgpt.com)

---

# GPIO Assignment

The dual-controller setup uses two SCORPIO GPIOs for two completely different purposes.

| GPIO       | Function      | Connection                 |
| ---------- | ------------- | -------------------------- |
| **GPIO16** | LED DATA      | Relay NO                   |
| **GPIO9**  | Relay control | Power Relay FeatherWing    |
| GND        | Common ground | LED PSU / WLED / LED strip |

The important distinction is:

```text
GPIO16 → HyperSerialPico LED DATA
GPIO9  → Relay control
```

GPIO9 does **not** replace GPIO16 as the LED output.

---

# GPIO16 — LED DATA

The SCORPIO's first dedicated NeoPixel output is:

```text
GPIO16 / OUT0 / NEOPIXEL0
```

This remains the LED DATA output used by the SCORPIO HyperSerialPico configuration.

The signal path is:

```text
HyperSerialPico
      │
      ▼
GPIO16 / OUT0
      │
      ▼
Relay NO
      │
      ▼
Relay COM
      │
      ▼
330 Ω
      │
      ▼
SK6812 RGBW DIN
```

No firmware modification is required for GPIO16.

---

# GPIO9 — Relay Control

GPIO9 is added specifically for the automatic controller switching.

During firmware startup, configure GPIO9 as an output and set it HIGH:

```cpp
// Configure GPIO9 as an output and activate the relay
gpio_init(9);
gpio_set_dir(9, GPIO_OUT);
gpio_put(9, 1);
```

This activates the relay after the SCORPIO has started.

The relay therefore switches from:

```text
NC → COM
```

to:

```text
NO → COM
```

and the SCORPIO becomes the active LED DATA source.

---

# Where to Add the GPIO9 Initialization

The GPIO9 initialization should be executed **once during firmware startup**, as early as practical in the initialization sequence.

It should not be placed inside the LED rendering or update loop.

Conceptually:

```text
Firmware startup
      │
      ├── Initialize hardware
      │
      ├── Initialize GPIO9
      │       │
      │       └── GPIO9 = HIGH
      │
      ├── Initialize HyperSerialPico
      │
      └── Start LED processing
```

The exact source file and insertion point may change between upstream HyperSerialPico revisions.

This documentation intentionally does not assume a specific source path so that the instructions remain valid across source-tree changes.

---

# Relay Behavior

The relay is controlled by GPIO9.

| SCORPIO state |    GPIO9 | Relay        | Active DATA source |
| ------------- | -------: | ------------ | ------------------ |
| Powered off   | Inactive | De-energized | WLED via NC        |
| Powered on    |     HIGH | Energized    | SCORPIO via NO     |

### TV OFF

The SCORPIO is not powered because the LG TV USB port is off.

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
NC → COM
     │
     ▼
WLED controls
```

