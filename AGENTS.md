# Chronos — Agent Context & Handover Document

## Purpose
This document provides immediate context for AI assistants or new human
contributors joining the Chronos project. It captures the **locked design
decisions**, **current status**, **pending questions**, and **known risks**
so that work can continue seamlessly without re-discussing settled matters.

## Project Snapshot
- **Name:** Chronos (Greek: Time)
- **Goal:** A fully open-source, repairable, modular Hi-Fi player designed
  to last 30+ years.
- **Philosophy:** "Outlast the creator." No soldering for repairs. No cloud.
  No DRM. No proprietary lock-in.
- **Current Phase:** Design Specification (Writing docs before hardware fabrication).

## 🔒 Locked Design Decisions (Do Not Revisit Without Discussion)

These decisions have been finalized. Do not suggest changing them unless
a critical flaw is discovered.

1.  **Microcontroller:** **RP2350B** (Dual ARM Cortex-M33 + RISC-V Hazard3).
    *   *Reason:* Hardware I2S engine, 520KB SRAM, dual PLLs, FPU, future-proof RISC-V option.
2.  **Storage Architecture:** **Dual 256MB QSPI Flash (SOP-8)** + **Dual 64KB EEPROM (SOP-8)**.
    *   *Reason:* True A/B dual-boot for unbrickable updates; redundant settings storage.
    *   *Constraint:* Must be **SOP-8** package to allow socketing via SOP-to-DIP adapters.
3.  **Modular Architecture:** **6 Daughter Modules** connected via **Custom FFC Connectors** (0.5mm pitch).
    *   *Modules:* Compute, Clock, DAC, Amp, Radio, I/O.
    *   *Constraint:* Each module has a unique pinout and connector size.
4.  **Clocking Strategy:** **Independent Clock Module** with socketed TCXO/OCXO.
    *   *Reason:* Eliminates jitter from the CPU. The RP2350 acts as an I2S Slave to this external clock.
5.  **Grounding Topology:** **Star Grounding** with 3 isolated domains (Digital, Analog, Chassis).
    *   *Constraint:* All grounds meet only at the battery negative terminal on the Main Board.
6.  **Connectivity:** **Air-Gapped** (No Wi-Fi). **Bluetooth is Optional** (ESP32 module, removable, hardware kill switch).
    *   *Constraint:* No persistent pairing data stored in non-volatile memory.
7.  **Repairability Strategy:** **SOP-to-DIP Adapter Boards**.
    *   *Method:* Every SMD chip (Flash, EEPROM, DAC, Amp, etc.) is soldered to a tiny adapter board, which plugs into a standard DIP socket on the module.
    *   *Result:* No chip is ever soldered directly to a module PCB.
8.  **Licensing:**
    *   Hardware: **CERN-OHL-S v2** (Strongly Reciprocal).
    *   Software: **GPL-3.0-or-later**.
    *   Docs: **CC-BY-SA-4.0**.

## 🚧 Pending Decisions (Needs Resolution)

These items are currently open for discussion or require research.

1.  **Clock Chip Selection:** Specific TCXO vs. OCXO model. (Need to verify SOP-8 availability).
2.  **DAC Chip Selection:** ESS Sabre (ES9038PRO) vs. AKM Velvet Sound (AK4499EX). (Need to verify adapter complexity).
3.  **Amp Topology:** Class AB (warmth) vs. Class D (efficiency). Specific chip TBD.
4.  **Battery Capacity:** Target playtime (e.g., 10h vs 20h) determines case size.
5.  **Display:** OLED vs. LCD. Resolution and interface (SPI vs. I2C).
6.  **FFC Pinouts:** Only the Compute Module (32-pin) is drafted. Clock, DAC, Amp, Radio, and I/O pinouts are pending.
7.  **Connector Sourcing:** Confirming 0.5mm pitch FFC sockets are available for DIY builders (Hirose/JST vs. generic).

## ⚠️ Known Risks & Mitigations

| Risk | Impact | Proposed Mitigation |
| :--- | :--- | :--- |
| **FFC Connector Availability** | High | If 0.5mm is too hard to source, switch to 1.0mm pitch or standard IDC headers. |
| **Clock Jitter via FFC** | Medium | Keep Clock Module physically close to DAC; use twisted pairs in FFC if possible. |
| **RP2350 Driver Maturity** | Medium | Start with "Hello World" I2S test immediately. Fallback to PIO if needed. |
| **Thermal Management** | Medium | Design case to act as heatsink for Amp module. |
| **Component Obsolescence** | High | Strict BOM strategy: Industrial grade, SOP-8 preferred, multiple supplier options. |

## 📂 Document Navigation

- **Start Here:** [README.md](../README.md)
- **Architecture:** [docs/architecture/](../docs/architecture/)
- **Module Specs:** [docs/modules/](../docs/modules/)
- **Hardware Guides:** [docs/hardware/](../docs/hardware/)
- **Software:** [docs/software/](../docs/software/)

## 🤖 Instructions for AI Agents

1.  **Respect Locked Decisions:** Do not suggest changing the RP2350, the dual-flash strategy, or the socketing method unless explicitly asked to review a "Locked Decision."
2.  **Focus on Pending Items:** When asked for help, prioritize resolving the "Pending Decisions" list.
3.  **Maintain Tone:** Be analytical, precise, and respectful of the "outlast me" philosophy. Avoid marketing fluff.
4.  **Cite Sources:** When suggesting components, always check for SOP-8 availability and industrial temperature ratings.
5.  **Update Context:** If a new decision is made, update this document immediately.

## Current Immediate Task
**Drafting Module Documentation.**
Next step: Define the **Clock Module** specification (Chip selection, pinout, adapter design).
