# System Overview

## 1. Architecture Philosophy

Chronos is built on a **Modular Backplane Architecture**. Unlike modern consumer electronics where the CPU, storage, and audio components are soldered onto a single, monolithic PCB, Chronos separates every major function into an independent, interchangeable daughter module.

These modules connect to a passive **Main Board (Backplane)** via custom **FFC (Flexible Flat Cable)** connectors.

### Why This Architecture?

1.  **True Repairability:** If the SD card slot fails, you replace only the I/O module. If the DAC chip becomes obsolete, you design a new DAC module without touching the rest of the system. No soldering is required for component replacement.
2.  **Signal Integrity:** By physically separating the noisy digital logic (Compute, Radio) from the sensitive analog audio path (Clock, DAC, Amp), we minimize electromagnetic interference (EMI) and ground loops.
3.  **Future-Proofing:** When a component goes End-of-Life (EOL), only the affected module needs a redesign. The Main Board and other modules remain compatible.
4.  **Customization:** Users can mix and match modules (e.g., a "Pure Wired" edition without the Radio module, or a "High-End" edition with a superior DAC module).

## 2. System Block Diagram

```text
+-----------------------------------------------------------------------+
|                         MAIN BOARD (Backplane)                        |
|                                                                       |
|  [Power Distribution]  [Star Ground Point]  [FFC Connectors]          |
|                                                                       |
|      +-----------+    +-----------+    +-----------+                  |
|      |  Compute  |    |   Clock   |    |    DAC    |                  |
|      |  Module   |----|   Module  |----|   Module  |                  |
|      | (RP2350B) |    | (TCXO)    |    | (ESS/AKM) |                  |
|      +-----+-----+    +-----+-----+    +-----+-----+                  |
|            |                |                |                        |
|            |                |                |                        |
|      +-----v-----+    +-----v-----+    +-----v-----+                  |
|      |   Radio   |    |    Amp    |    |    I/O    |                  |
|      | (ESP32)   |    |  (Class A)|    | (SD/Jack) |                  |
|      +-----------+    +-----------+    +-----------+                  |
|                                                                       |
|  * All grounds meet at a single Star Point (Battery Negative)         |
+-----------------------------------------------------------------------+

3. The Six Core Modules

Each module is a self-contained PCB with its own power regulation, logic, and connectors. They plug into the Main Board via FFC cables.
Module	Primary Function	Key Components	Connector
Compute	System Brain	RP2350B, Dual 256MB Flash, Dual 64KB EEPROM	32-pin FFC
Clock	Timing Reference	Socketed TCXO/OCXO, Clock Buffer	10-pin FFC
DAC	Digital-to-Analog	High-end DAC Chip, Analog Filters, Op-Amps	24-pin FFC
Amp	Power Amplification	Power Amp Chip, Heat Sink, Volume Control	12-pin FFC
Radio	Bluetooth (Opt.)	ESP32-WROOM, Antenna, Kill Switch	12-pin FFC
I/O	User Interface	SD Card Slot, Buttons, Display, Headphone Jack	14-pin FFC
Module Interactions

    Data Flow: SD Card (I/O) → Compute Module (Decoding) → DAC Module (Conversion) → Amp Module (Amplification) → Headphones.
    Clock Flow: Clock Module (MCLK) → Compute Module (I2S Slave) & DAC Module (Timing).
    Control Flow: Compute Module (UI Logic) ↔ I/O Module (Buttons/Display) ↔ Radio Module (Bluetooth Pairing).
    Power Flow: Battery → Main Board (Distribution) → Local LDOs on each Module.

4. The Main Board (Backplane)

The Main Board is a passive board. It contains no active components (no CPUs, no chips). Its sole purpose is:

    Power Distribution: Routing 3.3V, 5V, and 12V rails from the battery/charger to each module.
    Signal Routing: Connecting the FFC pins between modules (e.g., passing MCLK from Clock to DAC).
    Star Grounding: Providing the single connection point where Digital, Analog, and Chassis grounds meet.
    Mechanical Mounting: Holding the modules in place with screws and standoffs.

Key Design Constraint

The Main Board must be designed to handle high current for the Amp module and clean power for the Clock/DAC modules. Trace widths and copper thickness are critical.
5. Connector Standard

All modules connect via FFC (Flexible Flat Cable) connectors.

    Pitch: 0.5mm (Standard for high-density, low-profile connections).
    Type: SMD (Surface Mount) sockets on the Main Board; matching FFC cables.
    Custom Pinouts: Each module has a unique pinout optimized for its specific needs (e.g., the Compute module has 32 pins for data/control, while the Clock module only needs 10).
    Durability: Gold-plated contacts to prevent oxidation over decades.

See FFC Connector Standard for detailed pinout definitions.
6. Signal Integrity & Grounding

Chronos employs a Three-Zone Star Grounding strategy to ensure class-leading audio quality.

    Digital Ground (DGND): For the Compute, Radio, and Clock logic. Carries high-frequency switching noise.
    Analog Ground (AGND): For the DAC, Op-Amps, and Amp. Must be silent.
    Chassis Ground (CGND): The metal case. Shields against external RF.

The Rule: These three grounds never touch except at a single Star Point located physically at the battery negative terminal on the Main Board. This prevents digital noise currents from flowing through the analog ground plane.

See Grounding Strategy for detailed implementation.
7. Software Architecture

The software runs entirely on the Compute Module (RP2350B).

    OS: Zephyr RTOS (Real-Time Operating System).
    File System: FAT32/exFAT (Native support for SD cards).
    Audio Pipeline:
        Decoder: libflac, libogg, libvorbis.
        Output: Hardware I2S engine (RP2350) synchronized to the external Clock Module.
    Updates: A/B Dual-Boot mechanism via SD card. No internet required.

See Firmware Architecture for details.
8. Next Steps

To proceed with the design, we must finalize the specifications for each module.

    Clock Module: Select specific TCXO/OCXO and define pinout.
    DAC Module: Select DAC chip and define analog output stage.
    Amp Module: Select amplifier topology and chip.
    I/O Module: Define SD card interface and display driver.
    Main Board: Draft the schematic for power distribution and grounding.

