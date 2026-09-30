# 4.2 Channel Active Audio Amplifier (TDA2030A + TDA2050)

A hobby-grade, bench-tested **4.2 channel active amplifier** built around four **TDA2030A** chips (mid/full-range and treble) and two **TDA2050** chips (bass/sub), with an onboard **JRC4558 pre-amp, bass low-pass filter and treble high-pass filter**, a dual-rail unregulated power supply, a regulated ±12 V supply for the analog front end, and a 5 V rail for a Bluetooth audio card.

Designed in **EasyEDA**. Schematic revision **3.7**, dated 27-09-2026. Author: **Subhajit Halder**.

> **Status:** Schematic is final and was tested on veroboard. PCB layout is in progress.

---

## Table of Contents

1. [Features](#features)
2. [System Overview](#system-overview)
3. [Schematic Sheets](#schematic-sheets)
4. [Sheet 1: Pre-Amp, Filters and Volume Control](#sheet-1-pre-amp-filters-and-volume-control)
5. [Sheet 2: Power Amplifiers](#sheet-2-power-amplifiers)
6. [Sheet 3: Power Supply and Grounding](#sheet-3-power-supply-and-grounding)
7. [Speaker Configuration](#speaker-configuration)
8. [Transformer and Power Budget](#transformer-and-power-budget)
9. [Grounding Scheme](#grounding-scheme)
10. [Controls and Connectors](#controls-and-connectors)
11. [Bill of Materials (Summary)](#bill-of-materials-summary)
12. [Build and Bring-Up Guide](#build-and-bring-up-guide)
13. [PCB Layout Guidelines](#pcb-layout-guidelines)
14. [Known Limitations](#known-limitations)
15. [Roadmap](#roadmap)
16. [Safety Warning](#safety-warning)
17. [License](#license)

---

## Features

- 6 power amplifier channels: 4x TDA2030A + 2x TDA2050
- Active frequency split done in the pre-amp stage (no passive crossover on the speaker side)
- Separate **Master**, **Bass** and **Treble** volume controls, with provision for a future **Mid** control
- Bass channel is summed to mono for deeper low-frequency output, with an extra gain stage
- Dual-rail (+16 V / 0 V / -16 V) supply from a 12-0-12 V transformer
- Regulated ±12 V split supply for the JRC4558 op-amps
- 5 V rail (7805) for a Bluetooth audio card
- Star grounding with a separate power ground (PwrGND) and signal ground (SigGND), joined through a chassis network
- Mains input with fuse, MOV surge protection and indicator LED
- LED indicators for mains, each raw rail, each regulated rail and the 5 V rail
- Per-chip supply decoupling, output clamp diodes and Zobel (RC snubber) networks on every amplifier

---

## System Overview

```
                 +------------------------------ Sheet 1 (Pre-Amp) -------------------------------+
                 |                                                                                |
 Ch_A / Ch_B --> |  Main Pre-Amp (U2)  --> Master Vol (POT2, POT3)                                |
 (H1 input)      |        |                                                                       |
                 |        +--> L_O / R_O (mid / full-range)  via POT4, POT5 (future mid control)  |
                 |        |                                                                       |
                 |        +--> Low Pass Filter (U1)  -> Bass Vol (POT1) -> Bass_O/P               |
                 |        |                                                                       |
                 |        +--> High Pass Filter (U3) -> Treble Vol (POT6, POT7) -> Trb_O/P_A, B   |
                 +--------------------------------------------------------------------------------+
                          |            |                |
                          v            v                v
                 +------------------ Sheet 2 (Power Amps) ---------------------+
                 |  TDA2030A x2 (U4, U5)  Mid / Full range  -> SPK1, SPK2      |
                 |  TDA2030A x2 (U6, U7)  Treble            -> SPK3, SPK4      |
                 |  TDA2050  x2 (U8, U9)  Bass / Sub        -> SW1, SW2        |
                 +--------------------------------------------------------------+
                          ^
                          |  +/-16 V
                 +------------------ Sheet 3 (Power) --------------------------+
                 |  AC 220 V -> Fuse -> MOV -> 12-0-12 transformer -> DB1       |
                 |  -> 8x 2200 uF -> +16 V / GND / -16 V                        |
                 |  -> 7812 / 7912 -> +/-12 V for JRC4558                       |
                 |  -> 7805 -> 5 V for Bluetooth card                           |
                 +--------------------------------------------------------------+
```

---

## Schematic Sheets

| Sheet | File / Title | Contents |
|-------|--------------|----------|
| 1/3 | Pre Amp, Audio Freq based Splitting and Vol control | Main pre-amp, low-pass filter, high-pass filter, all volume pots, input header |
| 2/3 | Amplifier Schematics | 4x TDA2030A and 2x TDA2050 power amplifiers |
| 3/3 | Power Supply and Grounding | Mains input, rectifier, reservoir caps, ±12 V and 5 V regulators, star ground network |

> Add exported images of the three sheets to a `docs/` or `schematics/` folder and reference them here:
>
> ```md
> ![Sheet 1](schematics/01_PreAmp.png)
> ![Sheet 2](schematics/02_Amplifier.png)
> ![Sheet 3](schematics/03_PowerSupply.png)
> ```

---

## Sheet 1: Pre-Amp, Filters and Volume Control

All op-amps are **JRC4558** dual op-amps powered from **±12 V**.

### Main Pre-Amp (U2)

- Provides additional gain in each channel for louder output.
- Input comes from header **H1** (Ch_A, Ch_B, SigGND).
- **POT2 / POT3** (47 k) are the **Master volume** for Ch_A and Ch_B.
- Output feeds:
  - the mid/full-range outputs **L_O** and **R_O** (through **POT4 / POT5**, 47 k, reserved for a future mid volume control)
  - the **low-pass filter** (bass path)
  - the **high-pass filter** (treble path)

### Low Pass Filter (U1) - Bass

- Converts Ch_A and Ch_B to **mono** (summed through R3 and R8) for deeper bass.
- An additional op-amp stage provides extra gain for "punchy" bass.
- **POT1** (47 k) is the **Bass volume** control.
- Output: `Bass_O/P_for_TDA2050`, which feeds both TDA2050 amplifiers.
- Works well for both woofer and subwoofer use.

### High Pass Filter (U3) - Treble

- Used on both Ch_A and Ch_B for better treble in individual channels.
- **POT6 / POT7** (47 k) are the **Treble volume** controls.
- Outputs: `Trb_O/P_A_for_TDA2030` and `Trb_O/P_B_for_TDA2030`.

### Notes from the schematic

1. POT2 and POT3 control master volume for Ch_A and Ch_B.
2. POT6 and POT7 control treble volume.
3. POT1 controls bass volume.
4. Due to hardware constraints the **mid-pass filter is not implemented**.
5. The high-pass filter output currently drives a full-range speaker.
6. The full-range output provides the mids of the audio for now.
7. POT4 and POT5 are reserved for future mid volume control.
8. An additional mid-pass filter schematic has not been made or tested yet.

---

## Sheet 2: Power Amplifiers

| Block | ICs | Input signal | Output |
|-------|-----|--------------|--------|
| Mid / Full-range amplifier | U4, U5 (TDA2030A) | `R_O`, `L_O` | SPK1, SPK2 (P3, P1) |
| Treble amplifier | U6, U7 (TDA2030A) | `Trb_O/P_A`, `Trb_O/P_B` | SPK3, SPK4 (P4, P2) |
| Woofer / Sub amplifier | U8, U9 (TDA2050TB) | `Bass_O/P_for_TDA2050` | SW1, SW2 (P5, P6) |

### Per-amplifier circuit elements

- Non-inverting configuration with a resistor feedback network (TDA2030A: 22 k feedback with 330 Ω to ground and 47 µF; TDA2050: 33 k feedback with 560 Ω and 22 µF).
- Input bias resistor and coupling capacitor at the input.
- Local supply decoupling: 470 µF electrolytic plus 100 nF ceramic on each rail.
- **Output Zobel network**: 3.3 Ω (or 2.2 Ω on the TDA2050) in series with 470 nF.
- **Output clamp diodes** (1N5408) to the supply rails for protection against inductive kickback.
- A supply-side diode (D11 / D12) on the negative rail of each TDA2050 block.

### Design notes from the schematic

1. The TDA2050 amplifiers are made solely for amplifying the bass input.
2. Mid channels currently use a full-range input with full-range output speakers.
3. For simplicity, the treble and mid amplifiers use the same design, although mid- and treble-specific improvements can be made.
4. **No crossover** is used or implemented at the speaker level; the band split happens in the pre-amp.

---

## Sheet 3: Power Supply and Grounding

### Mains input and protection

- **CN1**: AC 220 V input with chassis earth connection
- **F1**: 1 A fuse (use slow-blow / time-delay type because of transformer inrush)
- **MV1**: MOV-10D varistor for surge protection
- **LED1 + R58 (120 k)**: Yellow mains indicator
- **J1 / J2**: Transformer primary and secondary connectors

### Rectifier and reservoir

- **DB1**: Bridge rectifier (choose 10 A or higher)
- **C75 to C82**: 8x 2200 µF, 4 per rail. Multiple smaller caps are used to handle ripple current better than a single 8800 µF or 2x 4400 µF.
- **LED4 / LED5** (red, with 4.7 k): indicate that the bridge rectification is working on each rail
- **R66 / R67** (100 k, 1 W): bleeder resistors
- Output: **+16 V / 0 V (GND) / -16 V**

### Transformer selection

| Usage | Recommendation |
|-------|----------------|
| Low to moderate usage | 12-0-12 V, 5 A per rail |
| Prolonged, high-volume usage | 12-0-12 V, 7 A per rail |

### ±12 V regulated split supply (for JRC4558)

- **U11 (7812)** for +12 V, **U10 (7912)** for -12 V, fed from the +16 V and -16 V rails
- Input and output capacitors (220 µF + 100 nF) on each regulator
- **LED2 / LED3** (green, 1 k): indicate regulated split supply
- Output header **J3**: -12 V / 0 V / +12 V

### 5 V supply for Bluetooth card

- **U12 (7805)** fed from +16 V
- Filtering: 100 nF, 220 µF, 470 µF
- **LED6** (blue, 1 k): 5 V rail indicator
- Output header **J4**: GND / 5 V

---

## Speaker Configuration

Current test bench configuration:

| Role | Speaker | Amplifier |
|------|---------|-----------|
| Bass | 4 Ω, 40 W woofers | TDA2050 (U8, U9) |
| Mid / full range | 4 Ω, 25 W speakers | TDA2030A (U4, U5) |
| Treble | 4 Ω, 25 W speakers | TDA2030A (U6, U7) |

Use **4 Ω or higher** loads only. Do not go below 4 Ω.

---

## Transformer and Power Budget

The supply runs at roughly **±16 V** under load. Realistic output at this rail voltage is approximately:

| Amplifier | Channels | Approx. output per channel (4 Ω) |
|-----------|----------|----------------------------------|
| TDA2030A | 4 | about 18-20 W |
| TDA2050 | 2 | about 32-35 W at this rail voltage |

Total is roughly **90-115 W peak music power**. The TDA2050 can deliver considerably more with a higher rail voltage, but the TDA2030A has an absolute maximum of ±22 V, so a single shared rail has to stay within the TDA2030A limit.

**Important:** A 12 V transformer produces a noticeably higher voltage unloaded (typically 13.5-14 V AC, so about ±18-19 V DC) and higher again with mains at +10%. Always **measure the unloaded rail voltage** and confirm it stays below ±20 V. All reservoir capacitors should be rated **35 V or more**.

---

## Grounding Scheme

The design uses a star-ground approach to minimize hum and avoid multiple ground loops.

```
 PwrGND --[10R 1W]--[100 nF]--+--[100 nF]--[10R 1W]-- SigGND
                              |
                        Chassis GND (point C)
                              |
                       Household earth
```

Rules followed in the design:

- PwrGND and SigGND are kept isolated everywhere except through this network at the final grounding point.
- Chassis ground **must** be connected to household earth.
- The amplifier-side grounds (input bias, feedback, decoupling, Zobel) go to PwrGND; the pre-amp and filter sections use SigGND.
- Do not create additional ground connections between PwrGND and SigGND, for example through the Bluetooth card, USB audio sources or a shared chassis mounting.

This grounding arrangement was tested on veroboard and performed well.

---

## Controls and Connectors

| Ref | Function |
|-----|----------|
| POT1 | Bass volume |
| POT2, POT3 | Master volume (Ch_A, Ch_B) |
| POT4, POT5 | Mid / full-range volume (reserved for future use) |
| POT6, POT7 | Treble volume (A and B) |
| H1 | Audio input: Ch_A, Ch_B, SigGND |
| CN1 | AC 220 V mains input with chassis earth |
| J1 | Transformer primary |
| J2 | Transformer secondary (12-0-12) |
| J3 | Regulated -12 V / 0 V / +12 V output |
| J4 | 5 V output (GND / 5 V) for Bluetooth card |
| P1, P2, P3, P4 | Speaker outputs SPK1 to SPK4 |
| P5, P6 (SW1, SW2) | Woofer / subwoofer outputs |

---

## Bill of Materials (Summary)

Full BOM should be exported from EasyEDA. Key parts:

| Qty | Part | Notes |
|-----|------|-------|
| 4 | TDA2030A | Mid and treble amplifiers |
| 2 | TDA2050TB | Bass amplifiers |
| 3 | JRC4558 | Pre-amp, LPF, HPF |
| 1 | 7812 | +12 V regulator |
| 1 | 7912 | -12 V regulator |
| 1 | 7805 | 5 V regulator for Bluetooth card |
| 1 | Bridge rectifier, 10 A or higher | DB1 |
| 8 | 2200 µF electrolytic, 35 V or higher | Reservoir caps |
| 1 | 12-0-12 V transformer, 5-7 A per rail | About 120-200 VA |
| 1 | Fuse, 1 A slow-blow | F1 |
| 1 | MOV-10D | MV1 |
| 12 | 1N5408 | Output clamp diodes and supply diodes |
| 7 | 47 k potentiometer | POT1 to POT7 |
| 6 | Heatsink + insulating pads | Chip tabs are connected to the negative supply |
| Misc | LEDs (yellow, red, green, blue), resistors, film and ceramic capacitors, connectors | See schematic |

---

## Build and Bring-Up Guide

1. **Build the power section first** with nothing else connected. Use a fuse, and ideally a series bulb limiter for the first power-up.
2. Check the rectifier LEDs (red) light up and measure **+16 V / 0 V / -16 V**. Confirm the **unloaded** rail is below ±20 V.
3. Check the regulated outputs: **±12 V** at J3 and **5 V** at J4.
4. Power down and let the capacitors discharge fully before touching the board.
5. Connect the pre-amp board and verify ±12 V on each JRC4558 supply pin.
6. Connect **one amplifier** and one speaker at a time. Turn all volume pots to minimum first.
7. Check for DC offset at the speaker terminals (should be near 0 V) before connecting a speaker for the first time.
8. Bring up the remaining channels one at a time, checking heatsink temperature.
9. Mount all chips on heatsinks using insulating pads, or isolate the heatsink from the chassis, because the TDA2030A and TDA2050 metal tab is connected to the **negative supply**.

---

## PCB Layout Guidelines

If you are laying this out as a PCB, preserve the tested grounding behaviour:

- Keep **mains circuitry** (CN1, F1, MV1, LED1 and R58) in an isolated corner with at least 3 mm clearance, or a board slot, from low-voltage circuitry. Use wide mains traces.
- Keep the **rectifier to reservoir capacitor loop** short and away from the amplifier and pre-amp traces.
- Use a **single star-ground point** where the centre tap (GND), PwrGND and the chassis lug meet.
- Give the amplifier section a thick PwrGND trace (2 mm or wider) back to the star point.
- Keep SigGND for the pre-amp section only, connected to the rest only through the point C network.
- Use trace widths of **2-3 mm or more** for supply and speaker traces (1 oz copper); more if possible.
- Place each chip's decoupling capacitors immediately next to its supply pins.
- Put the power chips at the board edge so they can bolt to a heatsink.
- Consider splitting into two boards (power + amplifiers, and pre-amp + filters), connected by ±12 V, SigGND and the five audio lines.
- Leave footprints for optional parts (for example a low-value resistor between SigGND and PwrGND) if you want room to tune hum later.

---

## Known Limitations

- **No mid-pass filter.** Mid-range content is whatever the full-range channel carries; there is no dedicated mid band.
- **No crossover on the power amplifier side.** Full-range speakers receive the full-range signal, so heavy bass content can overdrive small speakers at high volume.
- **No speaker protection or turn-on mute.** The TDA2030A and TDA2050 have no mute pin, so a turn-on or turn-off thump may be audible. A speaker protection relay with delay and DC detection can be added if needed.
- **Shared rail voltage.** The TDA2050 amplifiers are limited by the lower ±16 V rail chosen to keep the TDA2030A within its safe limits.
- **Pre-amp and power amp gain.** Combined gain is high, so noise floor and useful volume pot range depend on the input source level.
- **Sensitive to grounding and wiring.** Hum behaviour depends heavily on physical layout and wiring.

---

## Roadmap

- [ ] Finish PCB layout (single or two-board version)
- [ ] Design and test the mid-pass filter
- [ ] Enable POT4 / POT5 as mid volume control
- [ ] Optional: speaker protection relay and delay circuit
- [ ] Optional: mid and treble specific improvements to the amplifier stage
- [ ] Add exported schematic images, BOM and Gerbers to the repository

---

## Safety Warning

This project connects directly to **220 V mains**. Mains voltage can kill.

- Only work on the mains section if you are qualified and comfortable doing so.
- Always connect the chassis to a proper household earth.
- Use a fuse and keep all mains wiring insulated and strain-relieved.
- Discharge the reservoir capacitors before touching the board.
- Never work on a powered board.

The authors take no responsibility for damage or injury resulting from use of this design. Build and use it at your own risk.

---

## License

Add a license of your choice (for example MIT for software or CERN-OHL-P / CC BY-SA 4.0 for hardware) as a `LICENSE` file in the repository.

---

## Author

**Subhajit Halder**
Schematic revision 3.7, designed in EasyEDA.
