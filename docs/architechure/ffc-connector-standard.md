# FFC Connector Standard

## 1. Overview

All modules in Chronos connect to the Main Board via **FFC (Flexible Flat Cable)** connectors. This standard defines the physical specifications, electrical characteristics, and pinout conventions for all connectors in the system.

### Why FFC?

1.  **Modularity:** Modules can be unplugged and replaced without soldering.
2.  **Low Profile:** FFC cables are thin and flexible, ideal for compact enclosures.
3.  **Reliability:** Gold-plated contacts resist oxidation over decades.
4.  **Serviceability:** Cables can be replaced if damaged without replacing the entire module.

## 2. Physical Specifications

### Connector Type

| Parameter | Specification |
| :--- | :--- |
| **Type** | SMD FFC/FPC Socket (Bottom Contact or Top Contact) |
| **Pitch** | 0.5mm (Standard for high-density, low-profile) |
| **Contact Plating** | Gold (≥0.76µm) for corrosion resistance |
| **Mating Cycles** | ≥50 cycles (should last for module swaps over decades) |
| **Cable Thickness** | 0.5mm - 1.0mm (depending on pin count) |
| **Cable Length** | 50mm - 150mm (optimized for case geometry) |

### Connector Manufacturers (Preferred)

| Manufacturer | Series | Notes |
| :--- | :--- | :--- |
| **Hirose** | DF40 Series | High reliability, widely available, expensive |
| **JST** | XH / PH Series | Good availability, moderate cost |
| **Molex** | Pico-Clasp | Robust, good for high-current |
| **Generic** | 0.5mm FFC Socket | Budget option, verify quality |

**Recommendation:** Use **Hirose DF40** for the Main Board (critical connections) and **generic 0.5mm** for module-to-module if cost is a concern. For DIY builders, generic sockets from LCSC or AliExpress are acceptable if verified for quality.

### Pin Count per Module

| Module | Pin Count | Connector Size |
| :--- | :--- | :--- |
| **Compute** | 32 pins | 16mm width |
| **Clock** | 10 pins | 5mm width |
| **DAC** | 24 pins | 12mm width |
| **Amp** | 12 pins | 6mm width |
| **Radio** | 12 pins | 6mm width |
| **I/O** | 14 pins | 7mm width |

## 3. Electrical Specifications

### Current Ratings

| Pin Type | Max Current per Pin | Notes |
| :--- | :--- | :--- |
| **Signal** | 100mA | Standard logic levels |
| **Power (3.3V/5V)** | 500mA | Use multiple pins in parallel |
| **Power (12V)** | 1A | Amp module requires heavy gauge |
| **Ground** | 1A | Multiple pins for low impedance |

### Voltage Levels

| Signal Type | Voltage | Logic Family |
| :--- | :--- | :--- |
| **Digital I/O** | 3.3V | CMOS / LVTTL |
| **I2C** | 3.3V | Open-drain with pull-ups |
| **SPI** | 3.3V | CMOS |
| **UART** | 3.3V | CMOS |
| **I2S** | 3.3V | CMOS |
| **Analog Audio** | ±2V RMS | Line-level (not digital) |

### Impedance Control

-   **High-Speed Signals (I2S, SPI):** 50Ω single-ended, 100Ω differential (if applicable).
-   **Clock Signals (MCLK):** 50Ω single-ended, matched to source impedance.
-   **Analog Signals:** No strict impedance control, but keep traces short and shielded.

## 4. Pinout Conventions

### General Rules

1.  **Power Pins:** Always placed at the edges of the connector for easy identification.
2.  **Ground Pins:** At least 3-4 ground pins per connector, distributed evenly.
3.  **Signal Grouping:** Related signals (e.g., I2S data + clocks) are placed adjacent to each other.
4.  **Keying:** Connectors are keyed (non-matable) to prevent accidental insertion of the wrong module.

### Pin Numbering

```text
┌─────────────────────────────────────┐
│  FFC Connector (View from Top)      │
│                                     │
│  1   3   5   7   9   11  13  15     │ ← Odd pins (front row)
│  2   4   6   8   10  12  14  16     │ ← Even pins (back row)
│                                     │
└─────────────────────────────────────┘

Pin 1 is marked with a silkscreen dot or square pad on the PCB.
Ground Pin Distribution
Module	Total Pins	Ground Pins	Ground Ratio
Compute	32	5	15.6%
Clock	10	2	20%
DAC	24	4	16.7%
Amp	12	4	33.3% (high current)
Radio	12	2	16.7%
I/O	14	3	21.4%
Signal Adjacency Rules

    Clock Signals: Place next to their corresponding ground pin (e.g., MCLK next to GND).
    Differential Pairs: Keep together with no other signals between them.
    High-Speed Signals: Surround with ground pins for shielding (e.g., GND-SPI-GND-SPI-GND).
    Analog Signals: Isolate from digital signals with ground pins on both sides.

5. Module-Specific Pinouts
5.1 Compute Module (32-pin)
Pin	Signal	Direction	Notes
1	3.3V	In	Power
2	GND	—	Ground
3	GND	—	Ground
4	I2S_DOUT	Out	Audio data to DAC
5	I2S_BCLK	Out	Bit clock
6	I2S_LRCK	Out	Word clock
7	MCLK_IN	In	From Clock Module
8	I2C_SDA	Bidir	DAC/EEPROM control
9	I2C_SCL	Out	DAC/EEPROM control
10	SPI_TX	Out	SD Card data
11	SPI_RX	In	SD Card data
12	SPI_CLK	Out	SD Card clock
13	SPI_CS	Out	SD Card select
14	UART_TX	Out	To Radio Module
15	UART_RX	In	From Radio Module
16	DISPLAY_CLK	Out	Display interface
17	DISPLAY_DATA	Out	Display interface
18	DISPLAY_CS	Out	Display chip select
19	GPIO_BTN_A	In	Play/Pause
20	GPIO_BTN_B	In	Next/Prev
21	GPIO_BTN_C	In	Volume Up
22	GPIO_BTN_D	In	Volume Down
23	GPIO_ENC_A	In	Rotary encoder A
24	GPIO_ENC_B	In	Rotary encoder B
25	RESET	In	System reset
26	GND	—	Ground
27	QSPI_A_CS	Out	Flash A chip select
28	QSPI_B_CS	Out	Flash B chip select
29	QSPI_CLK	Out	Shared QSPI clock
30	QSPI_MOSI	Out	Shared QSPI data out
31	QSPI_MISO	In	Shared QSPI data in
32	GND	—	Ground
5.2 Clock Module (10-pin)
Pin	Signal	Direction	Notes
1	3.3V	In	Power (from V_ANA rail)
2	GND	—	Ground (AGND)
3	GND	—	Ground (AGND)
4	MCLK_OUT	Out	To Compute & DAC
5	LRCK_OUT	Out	To DAC (if buffered)
6	SCK_OUT	Out	To DAC (if buffered)
7	I2C_SDA	Bidir	Clock config (optional)
8	I2C_SCL	Out	Clock config (optional)
9	ENABLE	In	Enable/disable clock
10	GND	—	Ground (AGND)
5.3 DAC Module (24-pin)
Pin	Signal	Direction	Notes
1	3.3V_DIG	In	Digital power
2	5V_ANA	In	Analog power
3	GND	—	Digital ground
4	GND	—	Digital ground
5	I2S_DIN	In	From Compute
6	I2S_BCLK	In	From Compute
7	I2S_LRCK	In	From Compute
8	MCLK_IN	In	From Clock Module
9	I2C_SDA	Bidir	DAC control
10	I2C_SCL	Out	DAC control
11	DAC_RESET	Out	Reset line
12	ANALOG_L_OUT	Out	Left channel to Amp
13	ANALOG_R_OUT	Out	Right channel to Amp
14	ANALOG_GND	—	Analog ground reference
15	GND	—	Analog ground
16	GND	—	Analog ground
17-24	NC	—	Reserved for future
5.4 Amp Module (12-pin)
Pin	Signal	Direction	Notes
1	12V_PWR	In	High-current power
2	12V_PWR	In	High-current power (parallel)
3	GND	—	Power ground
4	GND	—	Power ground
5	GND	—	Power ground
6	ANALOG_L_IN	In	From DAC
7	ANALOG_R_IN	In	From DAC
8	ANALOG_GND	—	Analog ground
9	VOLUME_I2C_SDA	Bidir	Digital volume control
10	VOLUME_I2C_SCL	Out	Digital volume control
11	MUTE	Out	Mute control
12	GND	—	Power ground
5.5 Radio Module (12-pin)
Pin	Signal	Direction	Notes
1	3.3V	In	Power
2	GND	—	Ground
3	GND	—	Ground
4	I2S_DIN	In	Audio from Compute
5	I2S_BCLK	In	Bit clock
6	I2S_LRCK	In	Word clock
7	UART_TX	Out	To Compute
8	UART_RX	In	From Compute
9	ENABLE	In	Power enable (kill switch)
10	STATUS	Out	Module status indicator
11	ANTENNA	—	Antenna trace (on module)
12	GND	—	Ground
5.6 I/O Module (14-pin)
Pin	Signal	Direction	Notes
1	3.3V	In	Power
2	5V	In	SD Card / Display backlight
3	GND	—	Ground
4	GND	—	Ground
5	SPI_TX	In	From Compute (SD)
6	SPI_RX	Out	To Compute (SD)
7	SPI_CLK	In	From Compute (SD)
8	SPI_CS	In	From Compute (SD)
9	DISPLAY_CLK	In	From Compute
10	DISPLAY_DATA	In	From Compute
11	DISPLAY_CS	In	From Compute
12	BTN_A	Out	To Compute (Play/Pause)
13	BTN_B	Out	To Compute (Next/Prev)
14	GND	—	Ground

Note: The headphone jack is physically on the I/O Module but connected directly to the Amp Module via a short internal cable (not through the Main Board FFC) to minimize analog signal path length.
6. USB-C Connector Specification
Purpose

The USB-C port serves three functions:

    Charging: 5V input for battery charging.
    File Transfer: Mass Storage Device (MSC) mode for direct music file transfer from computer.
    Firmware Updates: DFU (Device Firmware Upgrade) mode for flashing new firmware.

Connector Type
Parameter	Specification
Type	USB-C Receptacle (SMT, through-hole preferred for durability)
Orientation	Vertical (standing up) or Horizontal (lying flat)
Shielding	Metal shell connected to Chassis Ground
Mounting	Through-hole pins for mechanical strength
Pin Configuration
Pin	Function	Notes
VBUS	5V Power Input	Connected to charging circuit
GND	Ground	Multiple pins for current capacity
D+ / D-	USB Data	Connected to RP2350 USB PHY
CC1 / CC2	Configuration Channel	Pull-down resistors for device mode
SSTX / SSRX	SuperSpeed (Optional)	Not used; USB 2.0 only
USB Modes
Mode	Function	How to Activate
Mass Storage	SD card appears as USB drive	Hold "Play" button during boot
DFU (Firmware)	RP2350 enters bootloader mode	Hold "Volume Up" + "Volume Down" during boot
Normal Playback	Standard audio player	No special action
Important Note on Data Lines

The USB-C data lines (D+/D-) connect directly to the RP2350's USB PHY on the Compute Module. This means:

    The USB-C port is physically on the I/O Module (for user access).
    The data lines route through the Main Board FFC to the Compute Module.
    The charging circuit is on the Main Board (for power management).

This separation requires careful routing to avoid noise coupling between the high-speed USB data and the sensitive analog audio path.
7. Cable Assembly Guidelines
For Builders

    Cable Orientation: The white stripe on the FFC cable indicates Pin 1. Match this to the silkscreen marking on the connector.
    Insertion Depth: Push the cable fully into the socket until it seats firmly. Do not force.
    Locking Mechanism: Some connectors have a flip-lock. Engage this after insertion to prevent accidental disconnection.
    Cable Routing: Avoid sharp bends (minimum bend radius = 10x cable thickness).
    Strain Relief: Secure the cable near the connector with tape or a zip-tie to prevent stress on the solder joints.

For Manufacturers

    Impedance Testing: Verify 50Ω impedance on high-speed lines (I2S, SPI, USB).
    Continuity Testing: Verify all pins are correctly connected (no shorts or opens).
    Insulation Resistance: Verify >100MΩ between adjacent pins.
    Withstand Voltage: Verify >100V AC between signal and ground.

8. Open Questions

    Connector Availability: Are Hirose DF40 connectors available for hobbyist purchase, or should we specify a more accessible alternative (e.g., JST PH)?
    USB-C Placement: Should the USB-C port be on the I/O Module (with data routed to Compute) or on the Main Board (with data routed to Compute via FFC)?
    Cable Length: What is the optimal cable length for each module given the case dimensions? (Currently estimated at 50-150mm.)
    Keying Strategy: Should we use different connector sizes for each module (e.g., 32-pin vs 10-pin) to prevent accidental misinsertion, or rely on software detection?

