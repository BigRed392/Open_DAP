# Grounding Strategy

## 1. The Problem: Ground Loops and Noise

In mixed-signal electronics (digital logic + analog audio), the biggest enemy of sound quality is **ground noise**.

-   **Digital Ground (DGND):** Carries high-frequency switching currents from the CPU, Flash, and Bluetooth. These currents create voltage fluctuations ($V = I \times Z$) on the ground plane.
-   **Analog Ground (AGND):** Must be a "quiet" reference point. Even microvolts of noise here manifest as audible hiss, hum, or distortion in the audio output.
-   **The Risk:** If DGND and AGND are connected at multiple points, digital noise currents will flow through the analog ground path, corrupting the audio signal. This is known as a **Ground Loop**.

## 2. The Solution: Three-Zone Star Grounding

Chronos uses a **Three-Zone Star Grounding** topology. This ensures that noise currents from the digital side never flow through the analog side.

### The Three Zones

1.  **Digital Ground (DGND)**
    -   **Scope:** Compute Module, Radio Module, Clock Module logic, SD Card interface.
    -   **Characteristics:** High-frequency, high-current switching noise.
    -   **Path:** Returns to the Star Point via dedicated digital traces.

2.  **Analog Ground (AGND)**
    -   **Scope:** DAC Module, Amplifier Module, Analog Filters, Headphone Jack.
    -   **Characteristics:** Must be extremely low impedance and noise-free.
    -   **Path:** Returns to the Star Point via dedicated analog traces (wide copper pours).

3.  **Chassis Ground (CGND)**
    -   **Scope:** The aluminum metal enclosure.
    -   **Function:** Electromagnetic shielding (RFI/EMI) and user safety.
    -   **Connection:** Connected to the Star Point via a **bleeder network** (Resistor + Capacitor) to prevent static buildup without creating a low-impedance loop.

### The Star Point

All three zones meet at **exactly one physical location**: the **Negative Terminal of the Battery** on the Main Board.

```text
       [Digital Ground Plane]       [Analog Ground Plane]       [Chassis]
              |                              |                      |
              |                              |                      |
              +--------------+---------------+----------------------+
                             |
                             |  (Single Connection)
                             v
                     [Battery Negative Terminal]
                             |
                             v
                     [System Ground Reference]

Why this works:

    Digital noise currents flow from the CPU -> DGND -> Star Point -> Battery. They cannot divert into the AGND plane because there is no other connection.
    Analog signals reference the AGND plane, which remains isolated from the digital noise.

3. Implementation Details
3.1 Main Board Layout

The Main Board is the hub for the Star Ground.

    Split Planes: The PCB will have distinct copper pours for DGND and AGND. They will not overlap.
    The Star Node: A dedicated copper pad or via cluster at the battery connector location.
    Trace Widths:
        AGND Traces: Minimum 20-30 mils (0.5-0.75mm) to minimize resistance and inductance.
        DGND Traces: Standard width, but kept away from analog signal traces.
    Via Stitching: In the analog zone, vias will stitch the top and bottom AGND planes together frequently to lower impedance. In the digital zone, stitching is also used but kept separate from analog vias.

3.2 Module-Level Grounding

Each daughter module has its own internal grounding strategy that respects the global Star topology.

    Compute Module:
        Has a split ground plane (DGND/AGND) if it contains any analog components (it doesn't, but the I2S lines are sensitive).
        Crucial: The I2S ground pins on the FFC connector are tied to DGND on the module, but the return path for the audio data is managed carefully.
    Clock Module:
        Entirely AGND (or a very quiet DGND if the buffer is digital).
        The FFC ground pins for the Clock Module are dedicated to the AGND rail on the Main Board.
    DAC Module:
        Split ground plane (DGND for the chip logic, AGND for the analog output).
        Star Point on Module: The DGND and AGND on the DAC module meet at a single point on the module itself, then exit to the Main Board via a single AGND pin on the FFC connector. This prevents digital noise from leaving the module.
    Amp Module:
        Heavy AGND copper pour to handle high current.
        Connected directly to the Main Board AGND rail.

3.3 FFC Connector Grounding

The FFC cables are the bridge between modules. Grounding is critical here.

    Multiple Ground Pins: Every FFC connector includes 3-4 dedicated ground pins (not just one).
        Example: In the 32-pin Compute connector, pins 2, 3, 26, 31, 32 are GND.
    Pin Assignment:
        Digital Modules (Compute, Radio): FFC ground pins connect to Main Board DGND.
        Analog Modules (Clock, DAC, Amp): FFC ground pins connect to Main Board AGND.
        Mixed Signals: If a signal crosses domains (e.g., I2S data), the return path is carefully managed to avoid ground loops.

3.4 Chassis Ground Bleeder

The metal case is connected to the Star Point via a Bleeder Network:

    10kΩ Resistor: Limits current in case of a fault.
    10nF Capacitor: Provides a low-impedance path for high-frequency RF interference to ground, while blocking low-frequency hum.
    Result: The case is shielded from RF noise but does not create a DC ground loop with the circuit.

4. Common Pitfalls to Avoid

    Don't "Star" on the Module: Do not try to create a star ground on every module. The global star point must be on the Main Board. Modules should use split planes that feed into the correct global rail.
    Don't Mix Grounds in FFC: Do not connect a DGND pin on a module to an AGND pin on the Main Board. The pinout must be strictly segregated.
    Don't Ignore Return Paths: Every signal trace must have a corresponding ground return path directly underneath it (microstrip/stripline) to minimize loop area.
    Don't Use Thin Traces for AGND: Analog ground traces must be wide. A thin trace has inductance, which creates voltage drops when current flows.

5. Verification

During prototyping, we will verify the grounding strategy using:

    Oscilloscope: Measure noise on the AGND rail relative to the Star Point while the CPU is under load. Target: < 1mV peak-to-peak noise.
    Audio Analyzer: Measure THD+N (Total Harmonic Distortion + Noise) with and without the Radio module active. The difference should be negligible (< 0.001%).

