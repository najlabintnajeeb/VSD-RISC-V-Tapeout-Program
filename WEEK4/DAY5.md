
# NgspiceSky130 - Day 5  
## CMOS Power Supply and Device Variation Robustness Evaluation

This session focuses on testing the robustness of a CMOS inverter in the Sky130 technology node against two variations: **supply voltage scaling** and **transistor size variation**.

---

## Table of Contents
# Table of Contents

1. [SPICE Simulation for Power Supply Variations](#spice-simulation-for-power-supply-variations)
2. [Advantages and Disadvantages of Low Supply Voltage](#advantages-and-disadvantages-of-low-supply-voltage)
3. [Sky130 Supply Variation Lab](#sky130-supply-variation-lab)
4. [Device Variation and CMOS Inverter Robustness](#device-variation-and-cmos-inverter-robustness)
5. [Sky130 Device Variation Lab](#sky130-device-variation-lab)


---

## SPICE Simulation for Power Supply Variations

**Objective:** Verify that a CMOS inverter functions correctly when the supply voltage (VDD) is scaled down (e.g., 2.5V → 0.5V).

**Setup:**
- Single CMOS inverter: PMOS = 0.9375 µm, NMOS = 0.375 µm  
- Load capacitance: 10 fF  
- Supply voltage sweep: 2.5V → 0.5V in steps of 0.5V  
- SPICE `.control` loop:
  - Variable `power_supply` decremented in steps
  - `alter VDD` command updates the circuit
  - DC simulation generates VTC curves for each VDD

**Outcome:** Multiple VTC curves plotted in a single figure, showing inverter behavior across all voltages.

<img width="576" height="369" alt="Screenshot 2025-10-18 at 11 22 11 am" src="https://github.com/user-attachments/assets/6490ebac-de87-44e1-86b7-569353b91466" />

---

## Advantages and Disadvantages of Low Supply Voltage

**Advantages:**
1. **Increased Gain:** Voltage gain (dVout/dVin) improves. Example: 56% higher gain at 0.5V vs 2.5V.  
2. **Lower Energy Consumption:** Energy ∝ VDD². Reducing VDD from 2.5V → 0.5V gives ~96% energy reduction.

**Disadvantage:**
- **Performance Loss:** Rise/fall delay increases at low VDD.  
  - Example: 2.5V → ~70 ps, 1.0V → ~165 ps, 0.5V → may not fully charge load.  
- **Design Trade-off:** lower voltage = more efficient & higher gain, higher voltage = faster switching.

---

## Sky130 Supply Variation Lab

**File:** `day5_inv_supplyvariation_Wp1_Wn036.spice`

<img width="609" height="566" alt="Screenshot 2025-10-16 at 3 06 52 pm" src="https://github.com/user-attachments/assets/939c7994-ddd9-4827-b6b1-901e3ace4e70" />  

<img width="825" height="667" alt="Screenshot 2025-10-16 at 2 56 42 pm" src="https://github.com/user-attachments/assets/cca89960-a667-49da-bdbe-87523a54ccc3" />  

---

#### Gain Calculation from VTC Curves

Each colored curve (dc1.out → dc6.out) corresponds to a different VDD.
Steps:

- Identify two points (x0, y0) and (x1, y1) on the steepest part (transition region) of a single VTC curve.
- Compute the small-signal gain using the formula:
```

Gain = (y0 - y1) / (x0 - x1)

```

**Notes:**  
- A higher (steeper) gain means a sharper, more digital switch between logic levels.
- Gain typically increases as VDD decreases, but this comes at the cost of significantly reduced switching speed.
  
---






### Static behaviour evaluation-CMOS inverter robustness-Device variation

#### Device Variation and CMOS Inverter Robustness


**Device variation** is a key factor in defining the robustness of a CMOS inverter. Variations arise during fabrication and affect transistor dimensions and electrical characteristics, which in turn impact the inverter's performance.


#### **1. Etching Process Variations**

*   **What it is:** Etching is a critical fabrication step that defines the physical structures (shapes) of the transistors on a chip.
*   **The Impact:**
    *   **Ideal vs. Real:** In an ideal process, transistor gates have perfectly rectangular shapes with well-defined Width (W) and Length (L). In reality, etching can cause distorted, non-uniform edges.
    *   **Effect on Transistors:** This distortion leads to variations in the actual `W` and `L` dimensions from their designed values.
    *   **Pattern Dependence:** Transistors in the middle of a regular structure (like an inverter chain) tend to have similar, minimal variations. Transistors on the edges, adjacent to different structures (e.g., flip-flops), can have significantly different and more severe distortions.
*   **Why it Matters:** The drain current (`I_D`) of a MOSFET is directly proportional to the `W/L` ratio. Any variation in `W` or `L` directly alters the transistor's current drive capability.

  
<img width="554" height="321" alt="Screenshot 2025-10-18 at 11 37 57 am" src="https://github.com/user-attachments/assets/a1b151e6-a27d-49f7-98e9-d2f5c82a813d" />

#### **2. Oxide Thickness Variations**

*   **What it is:** The gate oxide is the thin insulating layer between the transistor's gate and the channel. In an ideal process, this thickness (`T_ox`) is uniform across the entire chip.
*   **The Impact:**
    *   **Ideal vs. Real:** In a real fabrication environment, the oxidation process can result in non-uniform oxide thickness across a single transistor and from one transistor to another.
    *   
 <img width="570" height="293" alt="Screenshot 2025-10-18 at 11 47 34 am" src="https://github.com/user-attachments/assets/cb121ef3-209b-46cb-8b50-c56a518bcef3" />

      
    *   **Effect on Transistors:** The oxide capacitance (`C_ox`) is inversely proportional to the oxide thickness (`T_ox`). `C_ox` is a key parameter in the drain current equation.
*   **Why it Matters:** Variations in `T_ox` cause variations in `C_ox`, which in turn directly impact the drain current (`I_D`).

#### **The Fundamental Consequence: Impact on Performance**

*   Both variation sources (Etching and Oxide Thickness) ultimately lead to **variations in the drain current (`I_D`)** of the transistors.
*   The current drive (`I_D`) of a transistor is a primary factor in determining the **charging and discharging speed** of the output load capacitance in a CMOS inverter.
*   Therefore, these fabrication variations directly **impact the delay** of the inverter and, by extension, the timing performance of the entire digital circuit (data paths, clock paths).

#### Device Variation Experiments in SPICE

| **Case**                    | **PMOS Width (µm)** | **NMOS Width (µm)** | **Strength / Resistance** | **Observations**                                                                                     |
| --------------------------- | ------------------- | ------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Strong PMOS / Weak NMOS** | 1.875               | 0.375               | PMOS: Low R, NMOS: High R | Strong pull-up → output stays high longer; switching threshold ~0.2 V; noise margin ~2.1 V           |
| **Weak PMOS / Strong NMOS** | 0.375               | 1.875               | PMOS: High R, NMOS: Low R | Strong pull-down → output goes low faster; switching threshold ~1.4 V; noise margin ~0.3 V           |
| **Intermediate Sweeps**     | 1.875 → 0.375       | 0.375 → 1.875       | Varying R                 | 5-step sweep in SPICE; DC transfer curves show minimal shift in switching threshold and noise margin |


**Key Observations**

- Switching Threshold (Vm)
Variation ~0.2 V → ~1.4 V (1.2 V range), still acceptable.

- Noise Margin
Strong PMOS / Weak NMOS: ~2.1 → 2.4 V (300 mV)
Weak PMOS / Strong NMOS: ~0.2 → 0.3 V (100 mV)
Remains within acceptable range to tolerate noise and voltage variation


## Sky130 Device Variation Lab

- File: day5_inv_supplyvariation_Wp1_Wn036.spice

<img width="636" height="642" alt="Screenshot 2025-10-16 at 2 58 37 pm" src="https://github.com/user-attachments/assets/a83549a0-65da-436c-bc7a-30dc6f15f73a" />  

- Run and plot the waveform in ngspice

<img width="982" height="744" alt="Screenshot 2025-10-16 at 3 05 15 pm" src="https://github.com/user-attachments/assets/eb119042-b806-49e5-94cd-10d31fc976ef" />  


- In the waveform, the larger PMOS width creates a stronger pull-up path, so the output remains high longer, while the NMOS pull-down is comparatively weaker, causing a faster transition to low
  

