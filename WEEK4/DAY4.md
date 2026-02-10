
# NgspiceSky130 - Day 4 - CMOS Noise Margin robustness evaluation


- [Introduction to Noise Margin](#introduction-to-noise-margin)
- [Noise Margin Voltage Parameters](#noise-margin-voltage-parameters)
- [Noise Margin Equation and Summary](#noise-margin-equation-and-summary)
- [Noise Margin Variation with Respect to PMOS Width](#noise-margin-variation-with-respect-to-pmos-width)
- [Sky130 Noise Margin Labs](#sky130-noise-margin-labs)



## Introduction to Noise Margin

Noise Margin defines the **robustness** of a logic gate (like a CMOS inverter) against unwanted disturbances such as **crosstalk** and **glitches**, which are significant in lower technology nodes. It determines how much **noise voltage** a gate can tolerate at its input while still maintaining correct logical output.

The **Voltage Transfer Characteristic (VTC)** of an inverter shows how the **output voltage (Vout)** changes with **input voltage (Vin)**:
- **X-axis:** Input voltage (Vin)
- **Y-axis:** Output voltage (Vout)
- At **Vin = 0**, output = **VDD**
- As **Vin increases**, **Vout decreases**, dropping sharply near **Vin = VDD/2** (switching threshold)

### Ideal vs. Practical Inverter

**Ideal Case:**
- The transition around **VDD/2** is instantaneous
- **Slope (dVout/dVin)** → **Infinite**

**Practical Case:**
- Due to **resistances and capacitances** in PMOS/NMOS, the transition is **gradual**, not vertical
- The slope becomes **finite**, typically close to **–1** in the transition region
- The output does not reach exact 0 V or VDD, but near those values

![Ideal vs Practical VTC](https://github.com/user-attachments/assets/0710d8a0-65b0-4c62-b8d5-1b364e50a636)

*Left side shows ideal case and right side graph shows practical case*

- For input **0 – VIL** → Output ≈ **VOH** (logic 1)
- For input **VIH – VDD** → Output ≈ **VOL** (logic 0)
- The mid-region between VIL and VIH is the **transition zone**

## Noise Margin Voltage Parameters

The critical voltage parameters for noise margin analysis are:
- **VIL (Input Low Voltage)**: Maximum input voltage recognized as logic 0
- **VIH (Input High Voltage)**: Minimum input voltage recognized as logic 1  
- **VOL (Output Low Voltage)**: Maximum output voltage for logic 0
- **VOH (Output High Voltage)**: Minimum output voltage for logic 1

![Noise Margin from VTC](https://github.com/user-attachments/assets/6eb2253c-63ae-413e-b29e-94b81be01b54)

*This image shows how to find noise margin from VTC*

## Noise Margin Equation and Summary

To ensure reliable logic operation between cascaded stages:

```
Noise Margin (High): NM_H = VOH - VIH
Noise Margin (Low): NM_L = VIL - VOL
```

- **NMH** indicates tolerance against noise in logic 1 state
- **NML** indicates tolerance against noise in logic 0 state
- Larger noise margins → **better inverter robustness**

**Key Design Principles:**
- **VOL < VIL** ensures the next stage correctly detects logic 0
- **VOH > VIH** ensures the next stage correctly detects logic 1
- The slope near the switching point (≈ –1) reflects the **gain** and influences noise immunity
- By adjusting **transistor sizing (Wp/Wn ratio)**, designers can **shift the switching threshold**, optimizing noise margins and speed

**Design Implications:**
- **Digital Design:** Uses flat regions of VTC curve for noise immunity
  ![Digital Design](https://github.com/user-attachments/assets/93cf2da7-bba1-42af-8c8a-a94af57af6d1)
- **Analog Design:** Uses steep transition region for amplification
  ![Analog Design](https://github.com/user-attachments/assets/b16418e3-892f-4b3d-96d5-d0f5c7409c80)

## Noise Margin Variation with Respect to PMOS Width

This analysis quantifies how **increasing PMOS width relative to NMOS** affects CMOS inverter characteristics:

![PMOS Width Variation Table](https://github.com/user-attachments/assets/d29d7164-8570-49f0-9c6f-fab025640a15)

As PMOS width scales from 1x to 5x NMOS width:
- **Noise Margin High (NMH)** improves significantly from 0.3V to 0.42V, showing better noise immunity for logic '1'
- **Noise Margin Low (NML)** remains stable at 0.3V until 4x width, then slightly degrades to 0.27V
- **Switching Threshold (Vm)** consistently shifts rightward from 0.99V to 1.4V

**Key Findings:**
- **Optimal performance at 2x-3x PMOS width**, where NMH improves by 33% with no NML degradation
- Beyond 3x, diminishing returns set in with minimal NMH gains and slight NML reduction
- Demonstrates the CMOS inverter's inherent robustness - it maintains good noise immunity across wide transistor sizing variations, making it tolerant to fabrication process variations

## Sky130 Noise Margin Labs

### Simulation Setup and Execution

To analyze the noise margins of the inverter circuit, run the following ngspice command:

```spice
ngspice day4_inv_noisemargin_wp1_wn036.spice
plot out vs in
```

### Voltage Transfer Characteristic (VTC) Curve

The voltage transfer characteristic shows the relationship between input voltage (in) and output voltage (out):

![VTC Curve](https://github.com/user-attachments/assets/d8218fe0-356e-43b3-afbf-f73fefbc409b)

### Critical Voltage Points Extraction

#### Procedure:
1. **Click on PMOS slope (top)** → Terminal displays: `x0 = VIL, y0 = VOH`
2. **Click on NMOS slope (bottom)** → Terminal displays: `x1 = VIH, y1 = VOL`

#### Extracted Values:
![Terminal Output](https://github.com/user-attachments/assets/c4c071db-260b-4430-a9ee-0bbc7c52bc75)

From the terminal output, we obtain:
- VIL (Input Low Voltage) = 0.7395V
- VOH (Output High Voltage) = 1.737V  
- VIH (Input High Voltage) = 0.9720V
- VOL (Output Low Voltage) = 0.1125V

### Noise Margin Calculations

#### Noise Margin High (NMH)
```
NMH = VOH − VIH = y0 − x1
NMH = 1.737V - 0.9720V = 0.765V
```

#### Noise Margin Low (NML)
```
NML = VIL − VOL = x0 − y1  
NML = 0.7395V - 0.1125V = 0.627V
```

These noise margins indicate the inverter's robustness against voltage fluctuations, with higher values providing better noise immunity for maintaining correct logic states.
