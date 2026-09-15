# Energy-Efficient Logic-in-Memory Enabled by a Physics-Based RRAM Compact Model


---

## Overview

This project implements **Logic-in-Memory (LiM)** computing using **Resistive RAM (RRAM)** to address the Von Neumann bottleneck — the energy and latency cost of constantly shuttling data between CPU and memory. Conventional Flash memory access costs ~50 µJ and ~0.5 ms per operation, creating a ~10⁹× energy gap between compute and memory access. RRAM's non-volatility, sub-300 ps switching speed, and ability to perform both storage and logic within the same device make it a strong candidate for ultra-low-power IoT and edge computing applications.

We built a physics-based RRAM compact model in Verilog-A, validated it against literature, and used it to implement **Material IMPLY logic** — demonstrating a complete 1-bit full adder entirely within a memory crossbar array.

## Motivation

- Von Neumann architecture separates memory and compute, forcing constant data transfer and high energy waste
- IoT/edge devices require ultra-low power operation that conventional Flash/DRAM cannot efficiently support
- RRAM offers non-volatility, ultra-fast switching (<300 ps, <100 ps for nitride-based devices), dual memory+logic functionality, and a compact ≤4F² cell area

## RRAM Compact Model

Implemented in **Verilog-A**, the model:
- Reproduces AC and DC device behavior, valid down to ultra-short (10 ns) pulses
- Includes intrinsic resistance variability (Gaussian distribution)
- Captures temperature dynamics and conductive filament (CF) evolution

**Key equations:**
```
R = R_LRS · [(tox - x)/tox + exp(x/k - 1) · e^(EG/kBT)]
I = (V₀/R) · sinh(V/V₀)
dx/dt = c₀ · exp[(ED - (g₀ - αx)^β · T_ox) / kBT]   (reset)
dx/dt = -xc₀ · exp[(EG - f·V/x) / kBT]              (set)
dT/dt = Cp⁻¹ · [V·I - kT(T - T₀)]
```

**Device states:**

| State | Meaning | Resistance | Switching Condition |
|---|---|---|---|
| HRS (Logic 0) | Broken filament | 30–70 kΩ | VRESET = −1.3 V |
| LRS (Logic 1) | Intact filament (CF) | 3–7 kΩ | IC = 100 µA |

**Model validation (Cadence Virtuoso):**

| Parameter | Paper | Our Model |
|---|---|---|
| Reset Voltage | −1.3 V | −1.255 V |
| Set Voltage | 0.6 V | 0.546 V |

## IMPLY Logic — The Computational Primitive

Material implication (**P IMPLY Q = ¬P ∨ Q**) combined with a FALSE (unconditional RESET) operation forms a functionally complete Boolean logic set. Each gate requires two RRAM devices sharing a common ground resistor (R_G).

**Truth Table:**

| P | Q | Q' (Output) |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

**Optimized operating point:** VSET = 1.16 V, VCOND = 1.05 V, R_G = 4 kΩ

**Operation (10 ns pulse):** P (R_P) is held at its stored resistance while Q (R_Q) transitions RESET → SET depending on the logical condition, read out against a 3–7 kΩ (Logic 1) / 30–70 kΩ (Logic 0) resistance band.

## Reliability: Logic State Degradation

Repeated IMPLY operations cause progressive resistance drift in Q when P=1, Q=0, eventually flipping the stored bit.

| Configuration | Voltages | Endurance |
|---|---|---|
| Non-optimized | VSET = 1.12 V, VCOND = 0.88 V | Fails after **~67 cycles** |
| Optimized | VSET = 1.16 V, VCOND = 1.05 V | Stable for **>3000 cycles** |

*(Worst-case variability scenario, initial R_Q = 60 kΩ)*

## 1-Bit Full Adder Implementation

A complete 1-bit full adder was built using a 9-device RRAM crossbar array:

- **Inputs:** A, B, Cin (logic states preserved in-memory)
- **Outputs:** Sum (S) at node M1, Carry-out (Cout) at node M5
- **Auxiliary nodes:** M2, M3, M4, M6 (intermediate/temporary states)
- **Total operation count:** 43 steps (17 FALSE + 26 IMPLY operations)

This demonstrates a fully functional combinational logic circuit computed entirely within memory, with no separate ALU or data movement to external logic.

## Future Directions

| Direction | Description | Expected Impact |
|---|---|---|
| Reduce FALSE Operations | New logic sequences minimizing reset steps | ~2–3× energy reduction |
| Adaptive Voltage Control | Dynamic VSET/VCOND tuning per cycle | Actively counters degradation |
| Hybrid CMOS + RRAM | CMOS for control, RRAM for data ops | Best-of-both-worlds architecture |
| Parallel IMPLY Execution | Simultaneous multi-gate computation | Reduces 43-step overhead |
| Error Correction Layer | ECC to mask variability-induced flips | Extends reliable operation cycles |
| 3D Crossbar Stacking | Vertical integration for higher density | Sub-1 fJ/op target |

## Tools Used

- **Verilog-A** — physics-based RRAM device modeling
- **Cadence Virtuoso** (Spectre) — transient/DC simulation and model validation


## References

Based on physics-based RRAM modeling and IMPLY logic literature; device parameters cross-validated against published Set/Reset characteristics.

---

*Course project for EE658: Memory Device Technology and Applications, IIT Gandhinagar.*
