# 12V-to-5V 3A Synchronous DC-DC Buck Converter (TPS54302)

**Author:** Torin Hamilton
**CAD Tool:** KiCad 10.0  
**Topology:** Fixed-Frequency ($400\text{ kHz}$) Peak-Current-Mode Synchronous Buck Converter  
**Board Dimensions:** $45\text{ mm} \times 30\text{ mm}$, 2-Layer FR-4 ($1.6\text{ mm}$ thickness, $1\text{ oz}$ copper)

![[12V_5V_Buck_Converter_3D_PCB.png]]

![[12V_5V_Buck_Converter_2D_PCB.png]]

---

## 1. Project Overview

This project implements a compact, high-efficiency **12V-to-5V, 3A Synchronous Step-Down (Buck) DC-DC Converter** based on the Texas Instruments **`TPS54302`** (`SOT-23-6`). It is designed for 12V/24V distributed power buses, embedded systems, and benchtop prototyping, with all surface-mount and through-hole footprints selected and spaced for reliable hand-soldering and oscilloscope characterization.

### Key Electrical Specifications

| Parameter | Symbol | Value | Notes |
| :--- | :---: | :---: | :--- |
| **Input Voltage Range** | $V_{\text{IN}}$ | $8\text{ V} \text{ to } 28\text{ V}$ | Optimized for $12\text{ V}$ nominal input |
| **Output Voltage** | $V_{\text{OUT}}$ | $5.08\text{ V}$ ($5.0\text{ V}$ nom) | Set by $1\%$ feedback divider ($R_2, R_3$) |
| **Max Continuous Output Current** | $I_{\text{OUT(MAX)}}$ | $3.0\text{ A}$ | Integrated $85\text{ m}\Omega$ HS / $40\text{ m}\Omega$ LS MOSFETs |
| **Switching Frequency** | $f_{\text{SW}}$ | $400\text{ kHz}$ | With $\pm 6\%$ frequency spread-spectrum for EMI reduction |
| **Peak Efficiency** | $\eta$ | $\approx 93\% \text{ -- } 95\%$ | At $V_{\text{IN}} = 12\text{ V}, V_{\text{OUT}} = 5\text{ V}$ |
| **Output Voltage Ripple** | $\Delta V_{\text{OUT}}$ | $< 30\text{ mV}_{\text{PP}}$ | Low-ESR MLCC output filter ($2 \times 22\,\mu\text{F}$) |
| **Transient Response ($1.5\text{ A}$ step)** | $\Delta V_{\text{TR}}$ | $< \pm 5\%$ ($\pm 250\text{ mV}$) | Phase-margin boosted via $75\text{ pF}$ feedforward cap |

### Repository Contents
- **[12V_5V_Buck_Converter.pdf](12V_5V_Buck_Converter.pdf)** — Exported PDF Schematic with engineering callouts
- **[12V_5V_Buck_Converter.kicad_sch](12V_5V_Buck_Converter.kicad_sch)** — KiCad 10 Schematic Capture (0 ERC violations)
- **[12V_5V_Buck_Converter.kicad_pcb](12V_5V_Buck_Converter.kicad_pcb)** — KiCad 10 2-Layer PCB Layout (0 DRC violations)
- **[BOM.csv](BOM.csv)** — Orderable Bill of Materials with Manufacturer Part Numbers (DigiKey / Mouser ready)
- **[gerbers/](gerbers/)** — Complete RS-274X Gerber & Excellon Drill package ready for PCB fabrication

---

## 2. Engineering Design & Component Calculations

All component values are derived from the [Texas Instruments TPS54302 Datasheet](tps54302.pdf) (Section 7.2, Pages 14–18).

### 2.1 Output Voltage Set Point ($R_2, R_3$)
The `TPS54302` regulates the feedback pin (`FB`, Pin 4) to an internal bandgap reference of $V_{\text{ref}} = 0.596\text{ V}$ (typical). Using $R_2 = 100\text{ k}\Omega$ ($1\%$) for the upper divider resistor and $R_3 = 13.3\text{ k}\Omega$ ($1\%$) for the lower divider resistor:

$$V_{\text{OUT}} = V_{\text{ref}} \times \left(1 + \frac{R_2}{R_3}\right) = 0.596\text{ V} \times \left(1 + \frac{100\text{ k}\Omega}{13.3\text{ k}\Omega}\right) = 5.077\text{ V}$$

A $49.9\,\Omega$ resistor ($R_1$) is placed in series between $V_{\text{OUT}}$ and the top of the feedback network to provide an injection point for control-loop Bode plot / stability measurements.

### 2.2 Programmable Input Undervoltage Lockout ($R_4, R_5$)
Resistors $R_4 = 511\text{ k}\Omega$ ($1\%$) and $R_5 = 105\text{ k}\Omega$ ($1\%$) form an external voltage divider on the `EN` pin (Pin 5) working with the internal pull-up current ($I_p = 0.7\,\mu\text{A}$) and hysteresis current ($I_h = 1.55\,\mu\text{A}$) to prevent brownout chatter during startup and shutdown:
- **Rising Turn-On Threshold ($V_{\text{EN(rising)}} = 1.23\text{ V}$):**
  $$V_{\text{START}} = V_{\text{EN(rising)}} \left(1 + \frac{R_4}{R_5}\right) - I_p R_4 \approx 6.86\text{ V}$$
- **Falling Turn-Off Threshold ($V_{\text{EN(falling)}} = 1.16\text{ V}$):**
  $$V_{\text{STOP}} = V_{\text{EN(falling)}} \left(1 + \frac{R_4}{R_5}\right) - (I_p + I_h) R_4 \approx 5.65\text{ V}$$

### 2.3 Output Inductor Selection ($L_1$)
For a target inductor ripple ratio $K_{\text{IND}} = 0.35$ up to $V_{\text{IN(MAX)}} = 28\text{ V}$ at $f_{\text{SW}} = 400\text{ kHz}$:

$$L_{\text{MIN}} = \frac{V_{\text{OUT}} \times (V_{\text{IN(MAX)}} - V_{\text{OUT}})}{V_{\text{IN(MAX)}} \times K_{\text{IND}} \times I_{\text{OUT}} \times f_{\text{SW}}} = \frac{5 \times (28 - 5)}{28 \times 0.35 \times 3 \times 400\times 10^3} = 9.78\,\mu\text{H}$$

Selecting the standard value **$L_1 = 10\,\mu\text{H}$** (`Bourns SRP7028A-100M`):
- **Inductor Ripple Current at $V_{\text{IN}} = 12\text{ V}$:**
  $$\Delta I_L = \frac{V_{\text{OUT}} \times (V_{\text{IN}} - V_{\text{OUT}})}{V_{\text{IN}} \times L_1 \times f_{\text{SW}}} = \frac{5 \times (12 - 5)}{12 \times 10\,\mu\text{H} \times 400\text{ kHz}} = 0.729\text{ A}_{\text{PP}}$$
- **Peak Inductor Current ($I_{L(\text{PK})}$):**
  $$I_{L(\text{PK})} = I_{\text{OUT(MAX)}} + \frac{\Delta I_L}{2} \approx 3.36\text{ A} \quad (\text{ at } 12\text{ V}) \quad / \quad 3.51\text{ A} \quad (\text{ at } 28\text{ V})$$
- **Selected Inductor Margins:** The `Bourns SRP7028A-100M` shielded power inductor is rated for **$I_{\text{RMS}} = 4.5\text{ A}$** and **$I_{\text{sat}} = 8.0\text{ A}$**, providing $>2\times$ saturation margin against core saturation during load transients.

### 2.4 Input & Output Capacitor Selection ($C_1\text{--}C_6$)
- **Input Decoupling ($C_1 = 10\,\mu\text{F}, 50\text{ V X7R}$ & $C_2 = 0.1\,\mu\text{F}, 50\text{ V X7R}$):**
  - Worst-case input RMS ripple current is $I_{\text{CIN(RMS)}} \approx I_{\text{OUT(MAX)}} / 2 = 1.5\text{ A}$.
  - $C_1$ ($1206$ package, $50\text{ V}$ X7R) provides bulk charge storage with minimal DC-bias derating at $12\text{ V}$, while $C_2$ ($0805$) is placed immediately adjacent to `U1` Pin 3 (`VIN`) and Pin 1 (`GND`) to shunt high-$di/dt$ switching currents.
- **Bootstrap Capacitor ($C_3 = 0.1\,\mu\text{F}, 50\text{ V X7R}$):**
  - Connected between `BOOT` (Pin 6) and `SW` (Pin 2) to supply the floating gate drive for the integrated high-side N-channel MOSFET.
- **Output Filter Capacitors ($C_4, C_5 = 2 \times 22\,\mu\text{F}, 25\text{ V X5R/X7R}$):**
  - To contain a $1.5\text{ A}$ load transient within $\Delta V_{\text{OUT}} = \pm 5\%$ ($0.25\text{ V}$), the minimum required effective output capacitance is:
    $$C_{\text{OUT(MIN)}} > \frac{2 \times \Delta I_{\text{OUT}}}{f_{\text{SW}} \times \Delta V_{\text{OUT}}} = \frac{2 \times 1.5\text{ A}}{400\text{ kHz} \times 0.25\text{ V}} = 30\,\mu\text{F}$$
  - Using two parallel $22\,\mu\text{F}$, $25\text{ V}$ `1206` ceramic capacitors ($44\,\mu\text{F}$ nominal) accounts for MLCC DC-bias derating at $5\text{ V}$ while halving equivalent series resistance (ESR) and ESL.
- **Feedforward Compensation Capacitor ($C_6 = 75\text{ pF}, 50\text{ V C0G/NP0}$):**
  - Placed in parallel with $R_2$ to introduce a phase-lead zero near the control loop crossover frequency ($f_0 \approx \frac{5.1}{V_{\text{OUT}} \times C_{\text{OUT}}} \approx 23\text{ kHz}$), boosting phase margin with low-ESR ceramic output capacitors:
    $$C_6 \approx \frac{1}{2\pi f_0 R_2} \approx 75\text{ pF}$$

---

## 3. PCB Layout & EMI Mitigation Architecture

Switching regulators involve high-$di/dt$ current loops ($400\text{ kHz}$ square-wave switching up to $3\text{ A}$). The 2-layer PCB ([12V_5V_Buck_Converter.kicad_pcb](12V_5V_Buck_Converter.kicad_pcb)) follows Texas Instruments' layout guidelines (Datasheet Section 7.4):

1. **Minimized High-$di/dt$ Input Hot Loop:** `C2` ($0.1\,\mu\text{F}$) and `C1` ($10\,\mu\text{F}$) are placed directly beside `U1` Pins 1 (`GND`) and 3 (`VIN`) with a dedicated top-layer `GND` tie and stitching via to the unbroken bottom `GND` plane.
2. **Compact Switching Node (`SW`):** The trace connecting `U1` Pin 2 (`SW`), `L1` Pad 1, and `C3` is kept short and direct on `F.Cu` to minimize parasitic ringing and capacitively coupled radiated EMI.
3. **High-Current Power Routing with Pin Neck-Down:** Power paths (`Vin`, `SW`, `Vout`) are routed with wide **$1.5\text{ mm}$** copper tracks between passives and connectors, necking down to **$0.5\text{ mm}$** only at the fine-pitch ($0.95\text{ mm}$) `SOT-23-6` pads.
4. **Noise-Isolated Feedback Network:** The high-impedance `FB` node (`R1`, `R2`, `R3`, `C6`) is grouped tightly next to `U1` Pin 4 on the opposite side of the IC from the inductor and `SW` node.
5. **Dual-Layer Ground Planes & Thermal Relief:** Both `F.Cu` and `B.Cu` are poured with `GND` zones (`0.3 mm` clearance and thermal-relief spokes for hand-solderability) stitched with through-hole vias.

---

## 4. Bill of Materials (BOM)

The full spreadsheet is available in [BOM.csv](BOM.csv).

| Ref | Qty | Value | Package | Manufacturer | MPN | Description |
| :--- | :-: | :--- | :--- | :--- | :--- | :--- |
| `U1` | 1 | `TPS54302` | `SOT-23-6` | Texas Instruments | `TPS54302DDCR` | 4.5V–28V In, 3A Sync Buck Converter |
| `L1` | 1 | `10uH` | `7.3x6.6mm` | Bourns | `SRP7028A-100M` | Shielded Power Inductor, $4.5\text{ A}_{\text{RMS}}, 8\text{ A}_{\text{sat}}$ |
| `C1` | 1 | `10uF` | `1206` | Samsung | `CL31B106KBHNNNE` | MLCC $10\,\mu\text{F}$, $50\text{ V}$, X7R, $\pm 10\%$ |
| `C2, C3` | 2 | `0.1uF` | `0805` | Yageo | `CC0805KRX7R9BB104` | MLCC $0.1\,\mu\text{F}$, $50\text{ V}$, X7R, $\pm 10\%$ |
| `C4, C5` | 2 | `22uF` | `1206` | Murata | `GRM31CR61E226KE15L` | MLCC $22\,\mu\text{F}$, $25\text{ V}$, X5R, $\pm 10\%$ |
| `C6` | 1 | `75pF` | `0805` | Murata | `GRM2165C1H750JA01D` | MLCC $75\text{ pF}$, $50\text{ V}$, C0G/NP0, $\pm 5\%$ |
| `R1` | 1 | `49.9` | `0805` | Yageo | `RC0805FR-0749R9L` | Thick Film Resistor $49.9\,\Omega$, $1\%$, $1/8\text{ W}$ |
| `R2` | 1 | `100k` | `0805` | Yageo | `RC0805FR-07100KL` | Thick Film Resistor $100\text{ k}\Omega$, $1\%$, $1/8\text{ W}$ |
| `R3` | 1 | `13.3k` | `0805` | Yageo | `RC0805FR-0713K3L` | Thick Film Resistor $13.3\text{ k}\Omega$, $1\%$, $1/8\text{ W}$ |
| `R4` | 1 | `511k` | `0805` | Yageo | `RC0805FR-07511KL` | Thick Film Resistor $511\text{ k}\Omega$, $1\%$, $1/8\text{ W}$ |
| `R5` | 1 | `105k` | `0805` | Yageo | `RC0805FR-07105KL` | Thick Film Resistor $105\text{ k}\Omega$, $1\%$, $1/8\text{ W}$ |
| `R6` | 1 | `1k` | `0805` | Yageo | `RC0805FR-071KL` | Thick Film Resistor $1\text{ k}\Omega$, $1\%$, $1/8\text{ W}$ |
| `D1` | 1 | `Green` | `0805` | Lite-On | `LTST-C170KGKT` | Green SMD LED ($571\text{ nm}, V_F = 2.0\text{ V}$) |
| `J1, J2` | 2 | `Screw Term` | `1x02 P5.08mm` | Phoenix Contact | `1715721` | 2-Pin Terminal Block, $5.08\text{ mm}$, $17.5\text{ A}$ |
| `TP1–TP4` | 4 | `Test Point` | `D1.5mm` PTH | Keystone | `5005` *(or DNP)* | Test Point Loop ($0.7\text{ mm}$ drill) or bare PTH |

---

## 5. Hand-Soldering & Bench Bring-Up Guide

### Recommended Soldering Order (Lowest to Highest Profile)
1. **Regulator IC (`U1` - `TPS54302`):** Verify the Pin 1 dot/bevel aligns with the top-left pad (`GND`, marked by the silkscreen triangle). Tack one corner pin with flux and fine solder, verify alignment, then solder the remaining 5 pins.
2. **0805 Passives & LED (`C2`, `C3`, `C6`, `R1`–`R6`, `D1`):** Note that `D1` is polarized—align the green cathode marking on the LED with **Pad 1 (`GND`, left pad)**.
3. **1206 Input/Output Capacitors (`C1`, `C4`, `C5`).**
4. **Power Inductor (`L1` - `SRP7028A-100M`):** Pre-tin one pad, reflow while sliding the inductor into place, then solder the second pad generously.
5. **Through-Hole Screw Terminals (`J1`, `J2`) & Test Points (`TP1`–`TP4`).**

### Safe First Power-Up Checklist
1. **Visual & Continuity Check (Power Off):**
   - Measure resistance across `J1` (`+12V` to `GND`) and `J2` (`+5V 3A` to `GND`) with a digital multimeter (DMM) to confirm there are no solder bridges or shorts to `GND`.
2. **Current-Limited No-Load Test:**
   - Set a bench DC power supply to **$12.0\text{ V}$** with a **$100\text{ mA}$ current limit** and connect to `J1`.
   - Turn on the output: the green indicator LED (`D1`) should illuminate, input current draw should be $< 10\text{ mA}$ at no load, and `TP3` (`Vout`) to `TP4` (`GND`) should read **$5.05\text{ V} \text{ -- } 5.10\text{ V}$**.
3. **Oscilloscope Switching & Ripple Verification:**
   - Increase the supply current limit to **$2.0\text{ A}$**, connect an electronic load (or power resistor) to `J2`, and probe `TP2` (`SW`) and `TP3` (`Vout` AC-coupled) to observe the $400\text{ kHz}$ switching waveform and $< 30\text{ mV}_{\text{PP}}$ output ripple.