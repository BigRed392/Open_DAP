# Chronos Licensing

This project uses a multi-layer licensing approach to ensure freedom,
repairability, and longevity for every part of the system.

## 1. Hardware Designs
**License:** CERN Open Hardware Licence Version 2 — Strongly Reciprocal (CERN-OHL-S v2)  
**Scope:** All hardware designs including schematics, PCB layouts, Gerber files,
mechanical CAD models, 3D prints, and Bill of Materials (BOM).

**What this means:**
- You are free to **manufacture, distribute, and modify** the hardware designs.
- **Strong Reciprocity:** If you manufacture a modified version of this hardware
  (e.g., a "Chronos Pro" with a different case or added features), you **MUST**
  release the complete design files for your modified version under the same
  CERN-OHL-S v2 license.
- **No Proprietary Lock-in:** You cannot take this design, make minor changes,
  and sell it as a closed-source commercial product. The "openness" travels
  with the hardware.
- **Patent Protection:** The license includes a patent grant from contributors
  to users, protecting you from patent litigation related to the design.

**Why this license?**
We chose CERN-OHL-S (Strongly Reciprocal) over the permissive version (CERN-OHL-P)
because our core mission is to prevent any entity from locking this design behind
a patent or price wall. We want to ensure that the "outlast me" philosophy cannot
be undermined by corporate enclosure.

**Full Text:**  
https://ohwr.org/cern_ohl_s_v2.txt

---

## 2. Software & Firmware
**License:** GNU General Public License Version 3 (GPL-3.0-or-later)  
**Scope:** All source code, including the bootloader, RTOS configuration,
device drivers, audio pipeline, user interface, and build tools.

**What this means:**
- You are free to **use, study, modify, and distribute** the software.
- **Copyleft:** If you distribute a modified version of the software (e.g.,
  a custom firmware build), you **MUST** release the complete corresponding
  source code under the same GPL-3.0 license.
- **No Tivoization:** The license ensures that users can run modified versions
  of the software on the hardware (no secure boot that blocks unsigned code).
- **Patent Grant:** Includes a patent grant to protect users from patent claims.

**Why this license?**
GPL-3.0 is the gold standard for ensuring software freedom. It guarantees that
the firmware remains auditable, modifiable, and free forever. It prevents the
creation of "closed" firmware variants that could restrict user control.

**Full Text:**  
https://www.gnu.org/licenses/gpl-3.0.txt

---

## 3. Documentation & Media
**License:** Creative Commons Attribution-ShareAlike 4.0 International (CC-BY-SA-4.0)  
**Scope:** All documentation, including guides, specifications, diagrams,
images, videos, and this README.

**What this means:**
- You are free to **share and adapt** the documentation for any purpose,
  even commercially.
- **Attribution:** You must give appropriate credit to the original authors.
- **ShareAlike:** If you remix, transform, or build upon the material, you
  must distribute your contributions under the same license as the original.
- **No Additional Restrictions:** You may not apply legal terms or technological
  measures that legally restrict others from doing anything the license permits.

**Why this license?**
Knowledge about how to build and repair this device must remain free and
accessible. CC-BY-SA ensures that improvements to the documentation (e.g.,
translated guides, expanded repair manuals) are shared back with the community.

**Full Text:**  
https://creativecommons.org/licenses/by-sa/4.0/legalcode

---

## 4. Trademarks
The name "Chronos" and associated logos are trademarks of the project.
While the designs are open source, you may not use the "Chronos" name
to imply endorsement by the original project team for your modified
versions unless explicitly authorized.

---

## 5. Disclaimer
THE HARDWARE AND SOFTWARE ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR
IN CONNECTION WITH THE HARDWARE, SOFTWARE OR THE USE OR OTHER DEALINGS IN THEM.

---

## Summary Table

| Component | License | Copyleft? | Commercial Use Allowed? |
| :--- | :--- | :--- | :--- |
| **Hardware (PCB, Schematics)** | CERN-OHL-S v2 | **Yes (Strong)** | Yes (Must share mods) |
| **Software (Firmware)** | GPL-3.0 | **Yes** | Yes (Must share mods) |
| **Documentation** | CC-BY-SA-4.0 | **Yes** | Yes (Must share mods) |
| **Trademarks** | Reserved | N/A | No (without permission) |
