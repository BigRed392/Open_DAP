# Signal Flow

## 1. Overview

The audio signal in Chronos travels through four distinct stages, each
handled by a separate module. The design ensures that **digital noise
never contaminates the analog signal** and that **timing precision is
maintained from clock to DAC without software-induced jitter**.

```text
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  SD Card │───▶│ Compute  │───▶│   DAC    │───▶│   Amp    │───▶ 🎧
│ (I/O Mod)│    │ (RP2350B)│    │  Module  │    │  Module  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                     ▲               ▲
                     │               │
                     │    ┌──────────┘
                     │    │
                ┌────┴────┐
                │  Clock  │
                │  Module │
                │ (TCXO)  │
                └─────────┘

2. Stage 1: Storage → Compute (Digital)
Source: SD Card on I/O Module

The SD Card is physically located on the I/O Module, connected to the Compute Module via the Main Board FFC routing.
Protocol: SPI (SDIO fallback available)
Signal	Direction	Description
SPI_CLK	Compute → I/O	Clock for SD communication
SPI_CS	Compute → I/O	Chip select (active low)
SPI_TX	Compute → I/O	Data from CPU to SD (commands)
SPI_RX	I/O → Compute	Data from SD to CPU (audio data)
Process

    The RP2350 reads the FAT32/exFAT file system on the SD card.
    It locates the requested audio file (e.g., track01.flac).
    It reads raw file data into the RP2350's 520KB SRAM buffer.
    The buffer is sized to hold at least 1 second of decoded audio at the highest supported sample rate (192kHz / 32-bit stereo ≈ 1.5MB). Note: 520KB is insufficient for a full second at max rate. The firmware will use a double-buffer strategy: while one buffer plays, the other fills from SD. This requires careful DMA management.

Key Constraint

The SD card read speed must exceed the audio playback data rate.

    Max audio data rate: 192kHz × 32-bit × 2ch = 12.3 Mbps
    Typical SD card SPI speed: 25-50 Mbps (sufficient)
    SDIO mode (fallback): 100-200 Mbps (overkill but available)

3. Stage 2: Compute → Clock Synchronization (Timing)
The Critical Design Decision

The RP2350 does NOT use its internal oscillator for audio timing. It operates as an I2S Slave to the external Clock Module.
Clock Distribution

┌─────────────┐
│ Clock Module │
│  (TCXO)     │
│             │── MCLK (22.5792MHz or 24.576MHz) ──▶ RP2350 PLL
│             │                                              │
│             │── MCLK ──────────────────────────────────▶ DAC Chip
│             │                                              │
│             │── LRCK (generated from MCLK via buffer) ──▶ DAC Chip
└─────────────┘

How It Works

    The Clock Module generates a ultra-low-jitter MCLK (Master Clock) signal. Typical frequencies:
        22.5792 MHz for 44.1kHz family (44.1, 88.2, 176.4 kHz)
        24.576 MHz for 48kHz family (48, 96, 192 kHz)
    MCLK is sent to two destinations simultaneously:
        RP2350: The MCLK feeds into the RP2350's PLL, which locks the hardware I2S engine to the external clock. The RP2350 then generates BCLK and LRCK in perfect sync with MCLK.
        DAC Chip: The MCLK drives the DAC's internal oversampling filter and delta-sigma modulator directly, ensuring the conversion timing is as precise as the oscillator allows.
    The RP2350 pushes I2S Data (the decoded audio bits) to the DAC, timed by the clock-locked I2S engine.

Why This Matters

    Without external clock: The RP2350's internal oscillator has jitter measured in hundreds of picoseconds. This causes time-domain errors in the DAC, manifesting as harshness and loss of spatial detail.
    With external clock: The TCXO has jitter measured in sub-picoseconds. The DAC receives timing that is orders of magnitude more precise, resulting in cleaner, more natural sound.

Dual Clock Frequency Support

The Clock Module may need to support both 22.5792 MHz and 24.576 MHz to handle both the 44.1kHz and 48kHz sample rate families without asynchronous sample rate conversion (SRC), which degrades audio quality.

Options:

    Dual Oscillators: Two TCXOs on the Clock Module, selected by the Compute Module via I2C command. Best audio quality, higher cost.
    Single Oscillator + SRC: One TCXO (e.g., 24.576 MHz) with software sample rate conversion for 44.1kHz family. Lower cost, slight quality compromise.
    Fractional-N PLL: A single TCXO feeding a fractional PLL that can generate both frequencies. Good compromise, moderate complexity.

Decision pending. See Clock Module for resolution.
4. Stage 3: Compute → DAC (Digital-to-Analog)
Protocol: I2S (Inter-IC Sound)

The RP2350's hardware I2S engine outputs three signals to the DAC Module via the Main Board FFC routing:
Signal	Direction	Description
I2S_DOUT	Compute → DAC	Serial audio data (left + right channel multiplexed)
I2S_BCLK	Compute → DAC	Bit clock (derived from MCLK via RP2350 PLL)
I2S_LRCK	Compute → DAC	Word clock / Left-Right clock (marks channel boundaries)
I2S Timing Example (44.1kHz / 24-bit stereo)
Parameter	Value	Calculation
Sample Rate	44,100 Hz	—
Bits per Sample	24	—
Channels	2	—
LRCK Frequency	44,100 Hz	Equals sample rate
BCLK Frequency	2,822,400 Hz	44,100 × 32 × 2 (32-bit frame incl. padding)
MCLK Frequency	22,579,200 Hz	44,100 × 512 (standard MCLK/LRCK ratio)
DAC Module Internal Flow

I2S_DOUT ──▶ [DAC Chip Digital Section] ──▶ [Delta-Sigma Modulator]
                                                        │
MCLK ──────▶ [DAC Chip Clock Section] ──────────────────┘
                                                        │
                                               [Internal DAC]
                                                        │
                                                        ▼
                                              [Analog Filter]
                                                        │
                                                        ▼
                                              [Op-Amp Buffer]
                                             (DIP-8 Socketed)
                                                        │
                                                        ▼
                                           Analog L/R Output → Amp Module

DAC Control Interface

The Compute Module configures the DAC chip (volume, filter mode, de-emphasis, mute) via I2C:
Signal	Direction	Description
I2C_SDA	Bidirectional	Data line (shared with EEPROM bus)
I2C_SCL	Compute → DAC	Clock line (shared with EEPROM bus)

Note: The DAC and EEPROM share the I2C bus but have different addresses. This is standard practice and avoids pin duplication.
5. Stage 4: DAC → Amp → Headphones (Analog)
The Analog Signal Path

Once the DAC produces an analog voltage, the signal is entirely in the analog domain. It must be amplified to drive headphones.

DAC Analog Output (≈2V RMS)
        │
        ▼
  [Low-Pass Filter]     ← Removes residual high-frequency quantization noise
        │
        ▼
  [Op-Amp Buffer]       ← DIP-8 socketed, user-swappable
        │                 (e.g., NE5532, OPA1612, LME49720)
        ▼
  [Volume Attenuator]   ← Potentiometer or digital attenuator (I2C)
        │
        ▼
  [Power Amplifier]     ← Class AB or Class D (module-specific)
        │
        ▼
  [Output Jack]         ← 3.5mm SE or 4.4mm Balanced (on I/O Module)
        │
        ▼
     🎧 Headphones

Signal Level Standards
Stage	Typical Level	Impedance
DAC Output	2V RMS (max)	100Ω - 1kΩ
Op-Amp Buffer	2V RMS (gain = 1)	< 10Ω output
Amp Output (SE)	Up to 2V RMS	< 1Ω output
Amp Output (Balanced)	Up to 4V RMS	< 2Ω output
Headphone Compatibility

The Amp Module must drive a wide range of headphones:
Type	Impedance	Sensitivity	Power Required
IEMs	16-32Ω	100-120 dB/mW	< 1 mW
Low-Z Over-Ear	32-64Ω	95-110 dB/mW	1-10 mW
High-Z Over-Ear	250-600Ω	85-100 dB/mW	10-100 mW

The Amp Module design must ensure low noise with IEMs (no audible hiss) while providing sufficient voltage swing for high-impedance headphones.
6. Bluetooth Alternate Path (Optional)

When the Radio Module is installed and active, an alternate audio output path exists:

SD Card → Compute (Decode) → I2S → ESP32 (Radio Module)
                                         │
                                         ▼
                                   Bluetooth Codec (SBC/AAC)
                                         │
                                         ▼
                                   Bluetooth Headphones

Key Differences from Wired Path

    The DAC Module is bypassed. Audio goes directly from the RP2350's I2S output to the ESP32's Bluetooth transmitter.
    Audio quality is limited by the Bluetooth codec. SBC is lossy; AAC is better but still lossy. The full fidelity of the wired path is not available over Bluetooth.
    Clock Module is still used. The I2S stream sent to the ESP32 is still timed by the external clock, ensuring the data fed to the Bluetooth codec is jitter-free (though the codec itself introduces lossy compression artifacts).

Bluetooth Kill Switch

A hardware switch on the Main Board physically disconnects power to the Radio Module's FFC connector. When the switch is off:

    The ESP32 receives no power.
    No RF emissions.
    No Bluetooth scanning or pairing.
    The device is functionally identical to a "Pure Wired" edition.

7. Signal Integrity Checklist

For each signal crossing between modules via FFC:
Signal	Domain Crossing	Mitigation
SPI (SD Card)	DGND → DGND	None needed (same domain)
I2S Data	DGND → AGND	Return path managed; ground pins adjacent to signal pins
I2S BCLK/LRCK	DGND → AGND	Same as above
MCLK	AGND → DGND + AGND	Low-jitter buffer on Clock Module; short FFC path
I2C	DGND → AGND	Slow bus; minimal noise risk
Analog Audio	AGND → AGND	Shielded FFC or twisted pair; short path
UART (Radio)	DGND → DGND	None needed (same domain)
8. Open Questions

    Dual Clock Frequencies: Should the Clock Module support both 22.5792 MHz and 24.576 MHz natively (dual TCXO), or rely on software SRC? (Affects audio quality for 44.1kHz family.)
    Analog FFC Routing: Should the analog audio signal between the DAC Module and Amp Module travel via the Main Board FFC, or via a dedicated shielded cable? (FFC may pick up noise from digital traces.)
    I2S Ground Return: The I2S signals cross from DGND (Compute) to AGND (DAC). The return current must flow through the Star Point, which adds loop area. Is this acceptable, or should we add a dedicated "bridge" ground near the I2S connector?

These questions must be resolved before finalizing the FFC pinouts.
