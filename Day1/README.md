# ⚡ NMOS Transistor Characterization: Id vs Vds Analysis

## 🎯 SKY130 PDK SPICE Simulation Workshop

---

<div align="center">

```
╔════════════════════════════════════════════════════════════╗
║                                                            ║
║     NMOS DRAIN CURRENT CHARACTERIZATION                   ║
║     Using SKY130 Open-Source PDK                          ║
║                                                            ║
║     Understanding Id-Vds Relationships Through            ║
║     SPICE Simulation and Analysis                         ║
║                                                            ║
╚════════════════════════════════════════════════════════════╝
```

</div>

---

## 📋 Table of Contents

```
┌──────────────────────────────────────────────────────┐
│ 1. 🎯 Simulation Objectives                          │
│ 2. 🔬 NMOS Device Physics Fundamentals               │
│ 3. 📊 Operating Regions Overview                     │
│ 4. 🔧 SPICE Netlist Construction                     │
│ 5. ⚡ Simulation Execution and Analysis              │
│ 6. 📈 Interpreting Id-Vds Characteristics            │
│ 7. 💡 Practical Design Insights                      │
│ 8. 🛠️ SKY130 Technology Parameters                   │
└──────────────────────────────────────────────────────┘
```

---

## 🎯 Section 1: Simulation Objectives

### Why Characterize NMOS Transistors?

Understanding the Id-Vds relationship is fundamental to digital and analog circuit design. This characterization reveals:

- **Operating region boundaries** (Linear vs Saturation)
- **Current drive capability** at different bias conditions
- **Output resistance** for analog applications
- **Switching behavior** for digital circuits
- **Process variation impact** on performance

### 🎓 Learning Outcomes

```
╔══════════════════════════════════════════════════════════╗
║  After completing this simulation, you will understand:  ║
║                                                          ║
║  ✓ How to set up SPICE simulations for MOSFET analysis ║
║  ✓ Relationship between Vgs, Vds, and drain current    ║
║  ✓ Distinguishing linear and saturation regions         ║
║  ✓ Using SKY130 PDK models in circuit simulation        ║
║  ✓ Extracting key transistor parameters from curves     ║
╚══════════════════════════════════════════════════════════╝
```

---

## 🔬 Section 2: NMOS Device Physics Fundamentals

### Transistor Structure

```
                    Gate (G)
                       │
              ┌────────┴────────┐
              │   Gate Oxide    │  ← SiO₂ dielectric
              └─────────────────┘
    ┌─────────────────────────────────────┐
    │  N⁺                           N⁺    │
    │ Source  ←──── Channel ────→  Drain  │
    │  (S)         Region           (D)   │
    └─────────────────────────────────────┘
              P-Substrate (Body)
                       │
                    Bulk (B)
```

### Terminal Definitions

| Terminal | Symbol | Role | Typical Voltage |
|----------|--------|------|-----------------|
| **Gate** | G | Controls channel conductivity | 0V to VDD |
| **Drain** | D | Current collector | 0V to VDD |
| **Source** | S | Current emitter | 0V (reference) |
| **Bulk** | B | Substrate connection | 0V (tied to source) |

---

## 📊 Section 3: Operating Regions Overview

### Three Operational Modes

An NMOS transistor operates in three distinct regions depending on the gate-source voltage (Vgs) and drain-source voltage (Vds).

#### 1️⃣ **Cut-off Region**

```
Condition: Vgs < Vt
Behavior:  Id ≈ 0 (only leakage current)
Channel:   Not formed
Use:       Digital OFF state
```

#### 2️⃣ **Linear (Triode) Region**

```
Condition: Vgs > Vt  AND  Vds < (Vgs - Vt)
Behavior:  Id increases with Vds
Channel:   Fully formed, uniform depth
Use:       Switches, resistive loads, transmission gates

Equation:
  Id = μn·Cox·(W/L)·[(Vgs - Vt)·Vds - Vds²/2]
```

#### 3️⃣ **Saturation Region**

```
Condition: Vgs > Vt  AND  Vds ≥ (Vgs - Vt)
Behavior:  Id relatively constant with Vds
Channel:   Pinched-off at drain end
Use:       Amplifiers, current sources, digital ON state

Equation:
  Id = (1/2)·μn·Cox·(W/L)·(Vgs - Vt)²·(1 + λ·Vds)
```

### Transition Point: Vdsat

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║  Vdsat = Vgs - Vt                                   ║
║                                                      ║
║  This voltage marks the boundary between            ║
║  linear and saturation regions                      ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## 🔧 Section 4: SPICE Netlist Construction

### Complete SPICE Netlist Breakdown

Let's analyze the simulation setup shown in your screenshot:

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description

XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
display
setplot dc1
.endc

.end
```

### Netlist Component Analysis

#### 📌 **Header and Model Inclusion**

```spice
*Model Description
.param temp=27
```
- Sets simulation temperature to 27°C (room temperature)
- Temperature affects mobility and threshold voltage

```spice
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
```
- Includes SKY130 process models
- `tt` = typical-typical corner (nominal process)
- Other corners: `ff` (fast-fast), `ss` (slow-slow), `fs`, `sf`

#### 🔌 **Device Instantiation**

```spice
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
```

Breaking down the MOSFET declaration:
- `XM1` - Instance name (X prefix for subcircuit)
- `Vdd` - Drain node
- `n1` - Gate node
- `0` - Source node (ground)
- `0` - Bulk node (ground)
- `sky130_fd_pr__nfet_01v8` - Model name (1.8V NMOS)
- `w=5` - Width = 5 µm
- `l=2` - Length = 2 µm

#### 🔋 **Voltage Sources**

```spice
Vdd vdd 0 1.8V    → Supply voltage (Drain voltage)
Vin in 0 1.8V     → Input voltage (Gate voltage)
R1 n1 in 55       → 55Ω resistor between gate and input
```

#### ⚙️ **Analysis Commands**

```spice
.op                              → Operating point analysis
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2 → DC sweep
```

**DC Sweep Parameters:**
- Primary sweep: `Vdd` from 0V to 1.8V in 0.1V steps (Vds)
- Secondary sweep: `Vin` from 0V to 1.8V in 0.2V steps (Vgs)
- This generates a family of Id-Vds curves

---

## ⚡ Section 5: Simulation Execution and Analysis

### Running the Simulation

Based on your terminal screenshot, here's the simulation workflow:

```bash
# Navigate to project directory
cd /sky130CircuitDesignWorkshop/design

# Run SPICE simulation
ngspice day1_nfet_idvds_L2_W5.spice

# ngspice will execute and display results
```

### Expected Terminal Output

```
******
** ngspice-36 : Circuit level simulation program
** The U. C. Berkeley CAD Group
** Copyright 1985-1994, Regents of the University of California.
** Please get your ngspice manual from http://ngspice.sourceforge.net
******

No compatibility mode selected!

Circuit: *model description

Doing analysis at TEMP = 27.000000 and TNOM = 27.000000

Reference value : 0.00000e+00
No. of Data Rows : 190

Title: *model description
Name: dc1 (DC Analysis)
Date: Sun Oct 19 19:54:27 2025
```

### Plotting the Results

```bash
ngspice 1 -> plot -vdd#branch
ngspice 2 -> █
```

The `-vdd#branch` command plots the current flowing through the Vdd source, which equals the drain current (Id).

---

## 📈 Section 6: Interpreting Id-Vds Characteristics

### Simulation Output Analysis

<p align="center">
  <img src="day1_img1.png" alt="NMOS Id-Vds Characteristics" width="90%">
</p>

### Understanding the Curve Family

The graph shows **drain current (Id)** vs **drain-source voltage (Vds)** for multiple gate voltages:

```
Id (µA)
  │
400│                                         ╱────────
   │                                    ╱────  Vgs = 1.8V
350│                               ╱────
   │                          ╱────
300│                     ╱────          Vgs = 1.6V
   │                ╱────         ╱────
250│           ╱────         ╱────
   │      ╱────         ╱────      Vgs = 1.4V
200│ ╱────         ╱────      ╱────
   │          ╱────      ╱────
150│      ╱────      ╱────           Vgs = 1.2V
   │  ╱────      ╱────          ╱────
100│ ────      ╱────       ╱────
   │      ╱────       ╱────          Vgs = 1.0V
50 │  ╱────      ╱────          ╱────
   │ ────   ╱────          ╱────     Vgs = 0.8V
0  ├──────┬──────┬──────┬──────┬──────────→ Vds (V)
   0     0.4    0.8    1.2    1.6    1.8
   
   └─ Linear ─┘└─── Saturation ───────┘
```

### Key Observations

#### 1. **Linear Region** (0V < Vds < ~0.6V)

```
Characteristics:
- Id increases linearly with Vds
- Slope depends on Vgs
- Transistor behaves like voltage-controlled resistor
- Both linear and quadratic terms matter
```

#### 2. **Saturation Region** (Vds > Vdsat)

```
Characteristics:
- Id becomes nearly constant (flat curves)
- Slight upward slope due to channel length modulation
- Current determined primarily by Vgs
- Transistor acts as current source
```

#### 3. **Current Levels**

From the simulation output (Vds = 1.8V):

| Vgs | Id (µA) | Region | Notes |
|-----|---------|--------|-------|
| 0.0V | ~0 | Cut-off | Below threshold |
| 0.2V | ~0 | Cut-off | Below threshold |
| 0.4V | ~10 | Weak inversion | Near threshold |
| 0.6V | ~50 | Saturation | Above threshold |
| 0.8V | ~100 | Saturation | Quadratic increase |
| 1.0V | ~150 | Saturation | Strong inversion |
| 1.2V | ~200 | Saturation | Maximum drive |
| 1.4V | ~260 | Saturation | Approaching VDD |
| 1.6V | ~330 | Saturation | Near VDD |
| 1.8V | ~400 | Saturation | At VDD |

---

## 💡 Section 7: Practical Design Insights

### Extracting Design Parameters

#### 🎯 **Threshold Voltage (Vt)**

From the curves, Vt can be estimated where Id starts to increase significantly:

```
Vt ≈ 0.4V to 0.5V (typical for SKY130 NMOS)
```

#### 🎯 **Transconductance (gm)**

In saturation region:
```
gm = ∂Id/∂Vgs = μn·Cox·(W/L)·(Vgs - Vt)

At Vgs = 1.8V, Vds = 1.8V:
gm ≈ (400µA - 260µA) / (1.8V - 1.4V) ≈ 350 µS
```

#### 🎯 **Output Resistance (ro)**

Slope of Id-Vds in saturation:
```
ro = ∂Vds/∂Id ≈ 1/(λ·Id)

From curve: ro ≈ 100-200 kΩ (typical for long channel)
```

### Design Trade-offs

```
╔════════════════════════════════════════════════════════╗
║  Parameter Impact on Performance:                      ║
║                                                        ║
║  W/L Ratio ↑  →  Id ↑, gm ↑, Area ↑                  ║
║  Length ↑    →  Id ↓, ro ↑, matching ↑               ║
║  Vgs ↑       →  Id ↑, speed ↑, power ↑               ║
║  Vds ↑       →  Slight Id ↑, headroom ↓              ║
║                                                        ║
╚════════════════════════════════════════════════════════╝
```

---

## 🛠️ Section 8: SKY130 Technology Parameters

### NMOS Model: `sky130_fd_pr__nfet_01v8`

Key specifications for the 1.8V NMOS device:

```
┌────────────────────────────────────────────────────┐
│ Technology: 130nm CMOS                             │
│ Gate Oxide: Thin oxide (tox ≈ 4nm)                │
│ Supply Voltage: 1.8V nominal                       │
│ Threshold Voltage: 0.4-0.5V (typical corner)      │
│ Minimum Length: 0.15µm                             │
│ Mobility (μn): 400-450 cm²/V·s                    │
│ Cox: ~7 fF/µm²                                     │
└────────────────────────────────────────────────────┘
```

### Device Sizing Guidelines

For your simulation: **W = 5µm, L = 2µm**

```
Aspect Ratio = W/L = 5/2 = 2.5

This is a relatively small ratio, resulting in:
✓ Moderate drive current (~400µA at Vgs=Vds=1.8V)
✓ High output resistance
✓ Good matching characteristics
✓ Long channel behavior (less short-channel effects)
```

### Process Corners

```
╔═══════════════════════════════════════════════════╗
║  Corner    Speed    Use Case                      ║
║  ──────────────────────────────────────────────   ║
║  tt        Typical  Nominal analysis              ║
║  ff        Fast     Best-case performance         ║
║  ss        Slow     Worst-case performance        ║
║  fs        Mixed    Slow NMOS, fast PMOS          ║
║  sf        Mixed    Fast NMOS, slow PMOS          ║
╚═══════════════════════════════════════════════════╝
```

---

## 🎓 Conclusion and Next Steps

### Key Takeaways

```
✓ Successfully characterized NMOS Id-Vds relationships
✓ Identified linear and saturation operating regions
✓ Extracted key parameters: Vt, gm, ro
✓ Understood SKY130 PDK model usage in SPICE
✓ Interpreted family of curves for different Vgs values
```

### Further Exploration

1. **Vary W/L ratios** - Observe impact on drive current
2. **Process corners** - Simulate ff, ss corners
3. **Temperature sweep** - Analyze thermal effects
4. **AC analysis** - Find transit frequency (fT)
5. **Transient analysis** - Measure switching delays

### Design Applications

- **Digital gates:** Size transistors for target delay
- **Current mirrors:** Match devices for accuracy
- **Amplifiers:** Bias in saturation for gain
- **Switches:** Operate in linear region for low Ron

---

## 📚 References

- SKY130 PDK Documentation: https://skywater-pdk.readthedocs.io
- Ngspice Manual: http://ngspice.sourceforge.net
- MOSFET Theory: Razavi, "Design of Analog CMOS Integrated Circuits"

---

<div align="center">

```
╔════════════════════════════════════════════════════════╗
║                                                        ║
║  🎉 Lab Complete!                                     ║
║                                                        ║
║  You've successfully analyzed NMOS characteristics    ║
║  using industry-standard SPICE simulation tools       ║
║                                                        ║
╚════════════════════════════════════════════════════════╝
```

</div>
