# Chronos — Open-Source Repairable Hi-Fi Player

## What Is This?

Chronos is a fully open-source, modular, repairable digital audio player
designed to function for 30+ years. Every active electronic component is
socketed or modular — no soldering is required to replace any chip. There
is no cloud dependency, no DRM, no Wi-Fi, and no proprietary firmware.

The device is air-gapped by design. Music lives on a standard SD card.
Bluetooth is optional and removable. The hardware physically cannot
connect to the internet.

**Key Feature:** Fast file transfers via **USB 3.0** (up to 90MB/s) using
a dedicated bridge chip, while maintaining a low-power, socketable microcontroller
core for playback.

## Why Does This Exist?

Most consumer audio players are designed to be replaced, not repaired.
When a single component fails — a jack, a battery, a flash chip — the
entire device becomes e-waste. Proprietary firmware, glued-in batteries,
and soldered-down SoCs ensure that nothing survives the manufacturer's
planned obsolescence cycle.

Chronos is the opposite: a device designed to outlast its creator.

Every design decision prioritises:

- **Longevity over convenience.** Socketed chips, modular boards,
  documented alternatives.
- **Fidelity over features.** Independent clocking, isolated analog
  paths, star grounding.
- **Freedom over ecosystem.** Open licenses, no cloud, no DRM,
  no phone app required for basic operation.

## Design Principles

1. **Zero Lock-In** — No proprietary firmware, no DRM, no cloud
   dependency, no phone app required for basic operation.
2. **Full Repairability** — Every active component is socketed or
   modular. Replacement requires no soldering. Every PCB design is
   owned by the project.
3. **Audio Fidelity** — Class-leading signal integrity via independent
   clocking, isolated analog paths, and star grounding.
4. **Privacy First** — Air-gapped by design. No Wi-Fi. Bluetooth is
   optional, removable, and has a hardware kill switch.
5. **Community Sustainability** — All hardware, software, and
   documentation released under strong copyleft licenses. Anyone can
   build, modify, and repair.

## Architecture Summary

Chronos uses a **modular backplane architecture**. A passive Main Board
distributes power and routes signals between **ten interchangeable daughter
modules**, each connected via custom FFC (Flexible Flat Cable) connectors.

| Module | Function | Connector | Key Components |
|:---|:---|:---|:---|
| **Compute** | System Brain | 32-pin FFC | RP2350B, Dual 256MB Flash, Dual 64KB EEPROM |
| **Clock** | Timing Reference | 10-pin FFC | Socketed TCXO/OCXO, Clock Buffer |
| **DAC** | D/A Conversion | 24-pin FFC | High-end DAC, Analog Filters, Op-Amps |
| **Amp** | Power Amplification | 12-pin FFC | Power Amp, Heat Sink, Volume Control |
| **Radio** | Bluetooth (Opt.) | 12-pin FFC | ESP32-WROOM, Antenna, Kill Switch |
| **Storage/USB** | Data Transfer | 12-pin FFC | SD Slot, **USB 3.0 Bridge Chip**, USB-C Port |
| **Charging** | Power Management | 8-pin FFC | Charger IC, Buck/Boost, Battery Connector |
| **Buttons** | Transport Control | 8-pin FFC | Tactile Switches, Rotary Encoder |
| **Display** | Visual Output | 8-pin FFC | OLED/LCD/E-Ink Panel |
| **Jack** | Audio Output | 6-pin FFC | 3.5mm / 4.4mm Headphone Jack |

### The USB 3.0 Solution
To achieve fast file transfers (>40MB/s) without sacrificing the RP2350's
benefits (socketability, low power, open boot), the **Storage/USB Module**
includes a dedicated **USB 3.0-to-SD Bridge Chip** (e.g., VIA VL700).

- **Playback Mode:** RP2350 reads SD card via SDIO (~25MB/s). Sufficient for high-res audio.
- **Transfer Mode:** User activates "Transfer Mode" (button hold). RP2350 releases SD lines.
  The Bridge Chip takes over, connecting the SD card directly to the USB-C port at **USB 3.0 speeds (~90MB/s)**.
- **Result:** Fast transfers to/from a PC, without needing a full SoC.

## Supported Audio Formats

- FLAC (lossless, all sample rates up to 192kHz / 32-bit)
- WAV / AIFF (uncompressed PCM)
- Ogg Vorbis (lossy, open codec)
- Opus (lossy, open codec)
- MP3 (lossy, widely compatible)

No proprietary codecs. No DRM. If you can put it on an SD card,
Chronos can play it.

## Licensing

| Layer | License | Why |
|:---|:---|:---|
| Hardware (schematics, PCBs, mechanical) | CERN-OHL-S v2 | Prevents proprietary lock-in of the physical design |
| Software (firmware, bootloader, tools) | GPL-3.0-or-later | Ensures firmware remains free and auditable |
| Documentation (guides, specs, images) | CC-BY-SA-4.0 | Ensures knowledge stays shared and attributed |

The **CERN-OHL-S (Strongly Reciprocal)** license is the hardware
equivalent of the GPL. It requires anyone who manufactures a modified
version to release their hardware designs under the same terms. This
means no company can take this design, make a proprietary variant, and
lock it behind a patent or price wall.

## Documentation Index

### Architecture
- [System Overview](docs/architecture/system-overview.md)
- [Grounding Strategy](docs/architecture/grounding-strategy.md)
- [Signal Flow](docs/architecture/signal-flow.md)
- [Power Distribution](docs/architecture/power-distribution.md)
- [FFC Connector Standard](docs/architecture/ffc-connector-standard.md)
- [Privacy & Security](docs/architecture/privacy-and-security.md)
- [Update Mechanism](docs/architecture/update-mechanism.md)

### Modules
- [Compute Module](docs/modules/compute-module.md)
- [Clock Module](docs/modules/clock-module.md)
- [DAC Module](docs/modules/dac-module.md)
- [Amp Module](docs/modules/amp-module.md)
- [Radio Module](docs/modules/radio-module.md)
- [Storage/USB Module](docs/modules/storage-usb-module.md)
- [Charging Module](docs/modules/charging-module.md)
- [Buttons Module](docs/modules/buttons-module.md)
- [Display Module](docs/modules/display-module.md)
- [Jack Module](docs/modules/jack-module.md)

### Hardware
- [Adapter Board Library](docs/hardware/adapter-board-library.md)
- [Socketing Guide](docs/hardware/socketing-guide.md)
- [BOM Strategy](docs/hardware/bom-strategy.md)
- [Case Design](docs/hardware/case-design.md)

### Software
- [Firmware Architecture](docs/software/firmware-architecture.md)
- [Audio Pipeline](docs/software/audio-pipeline.md)
- [Bluetooth Stack](docs/software/bluetooth-stack.md)

### Community
- [Build Guide](docs/community/build-guide.md)
- [Repair Manual](docs/community/repair-manual.md)
- [Contribution Guide](docs/community/contribution-guide.md)

## Project Status

**Design Specification Phase** — No hardware has been fabricated.
All specifications are draft and subject to change. The project is
currently writing its foundational documentation before beginning
hardware design.

## Getting Involved

*To be determined as the project matures.*

## Name

"Chronos" (χρόνος) — Greek personification of time. A device built
to endure it.

