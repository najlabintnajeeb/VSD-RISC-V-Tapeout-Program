# Ngspice Sky130 - Day 3 - CMOS Switching Threshold and Dynamic Simulations

## Table of Contents
1. [Voltage Transfer Characteristics – SPICE Simulations](#voltage-transfer-characteristics--spice-simulations)
    - [SPICE Deck Creation for CMOS Inverter](#spice-deck-creation-for-cmos-inverter)
    - [SPICE Simulation for CMOS Inverter](#spice-simulation-for-cmos-inverter)
    - [Labs Sky130 SPICE Simulation for CMOS](#labs-sky130-spice-simulation-for-cmos)
2. [Static Behavior Evaluation – CMOS Inverter Robustness and Switching Threshold](#static-behavior-evaluation--cmos-inverter-robustness-and-switching-threshold)
    - [Switching Threshold, Vm](#switching-threshold-vm)
    - [Analytical Expressions](#analytical-expressions)
    - [Static Simulation of CMOS Inverter](#static-simulation-of-cmos-inverter)
    - [Dynamic Simulation – Rise and Fall Delays](#dynamic-simulation--rise-and-fall-delays)
    - [CMOS Inverter with Increased PMOS Width](#cmos-inverter-with-increased-pmos-width)
    - [Applications of CMOS Inverter](#applications-of-cmos-inverter)


---

### Voltage Transfer Characteristics – SPICE Simulations

### SPICE Deck Creation for CMOS Inverter

**SPICE Deck Creation Steps**

- **Component Connectivity:** Define PMOS (M1), NMOS (M2), load capacitor (Cload), supply voltage (Vdd), ground (Vss), input (Vin), and output (Vout) connections.  
- **Component Values:** Assign transistor dimensions (W/L), supply voltage (e.g., 2.5 V), and load capacitance (Cload = 10 fF).  
- **Identify Nodes:** Recognize nodes such as in, out, vdd, vss, and transistor terminals (drain, gate, source, substrate).  
- **Name Nodes:** Assign meaningful names to all nodes for simulation clarity and plotting results.  

#### SPICE Deck Example

| **Command / Element** | **Explanation** | **Nodes / Details** |
|------------------------|----------------|---------------------|
| **M1 out in vdd vdd pmos W=0.375u L=0.25u** | PMOS transistor between VDD and output; gate connected to input; bulk tied to VDD. | Drain = out, Gate = in, Source = vdd, Bulk = vdd<br>W = 0.375 µm, L = 0.25 µm |
| **M2 out in 0 0 nmos W=0.375u L=0.25u** | NMOS transistor between output and ground; gate connected to input; bulk tied to ground. | Drain = out, Gate = in, Source = 0 (GND), Bulk = 0<br>W = 0.375 µm, L = 0.25 µm |
| **cload out 0 10f** | Load capacitor connected between output node and ground. | Positive = out, Negative = 0 (GND)<br>C = 10 fF |
| **Vdd vdd 0 2.5** | DC power supply providing 2.5 V. | Positive = vdd, Negative = 0 (GND)<br>V = 2.5 V |
| **Vin in 0 2.5** | Input voltage source used for sweep analysis. | Positive = in, Negative = 0 (GND)<br>Varies from 0–2.5 V |
| **.op** | Performs DC operating point (bias) analysis. | Finds DC node voltages and branch currents. |
| **.dc Vin 0 2.5 0.05** | Performs DC sweep of Vin from 0 V to 2.5 V in 0.05 V steps. | Used to generate VTC (Voltage Transfer Curve). |
| **.include tsmc_025um_model.mod** | Includes the MOSFET technology model file. | File defines parameters for NMOS and PMOS devices. |
| **.LIB "tsmc_025um_model.mod" CMOS_MODELS** | Alternate way to include the transistor model library. | Points to model section `CMOS_MODELS` inside the file. |
| **.end** | Marks the end of the SPICE netlist file. | Required termination statement. |

---

### SPICE Simulation for CMOS Inverter

| **Simulation Command** | **Purpose** | **Syntax / Notes** |
|------------------------|------------|------------------|
| DC Sweep | Calculate Voltage Transfer Characteristics (VTC) — Vout vs Vin | `.DC VIN 0 2.5 0.05` <br> Sweeps input voltage from 0 V to 2.5 V in 0.05 V steps |
| Transient Analysis (.TRAN) | Calculate rise/fall delays by applying a pulse input | `.TRAN step_time final_time` <br> Example: Pulse from 0 V → 1.8 V with defined rise/fall times, pulse width, and period |

---

### Labs Sky130 SPICE Simulation for CMOS

**Voltage Transfer Characteristics**

- **File:** `day3_inv_vtc_Wp084_Wn036.spice`
  
<img width="602" height="609" alt="Screenshot 2025-10-16 at 11 56 55 am" src="https://github.com/user-attachments/assets/9325bbbd-27fa-40c6-81cb-8758bf20d877" />


- Run and plot in Ngspice:
  
```spice
ngspice day3_inv_vtc_Wp084_Wn036.spice
plot out vs in
```

Below image shows the Voltage Transfer Characteristics (VTC) of a CMOS Inverter:

<img width="903" height="706" alt="Screenshot 2025-10-16 at 11 06 44 am" src="https://github.com/user-attachments/assets/faccddad-0112-4d15-bb62-0fb81e808386" />

**Finding Switching Threshold (V_M):**

The switching threshold is the point where ``V_in = V_out`` .
By zooming into the intersection point on the VTC curve, the value is found to be approximately **0.876 V** for the given sizing.


<img width="801" height="573" alt="Screenshot 2025-10-16 at 11 15 14 am" src="https://github.com/user-attachments/assets/02e0aff9-f3cd-40e9-94d3-d0dbefe86a9e" />






#### Transient Analysis and Delay Calculation

- **File:**: `day3_inv_tran_Wp084_Wn036.spice`
- 
  <img width="602" height="609" alt="Screenshot 2025-10-16 at 11 56 55 am" src="https://github.com/user-attachments/assets/d5078dc5-0eea-4a91-9b68-5403181ba914" />

- Run and plot in Ngspice:

``` spice
  ngspice day3_inv_tran_Wp084_Wn036.spice
plot out vs time in
```

- Transient Output Waveform:
Illustrates rise time delay and fall time delay.

<img width="953" height="654" alt="Screenshot 2025-10-16 at 11 22 23 am" src="https://github.com/user-attachments/assets/24aacfb5-17e4-425d-a0d0-893c3f59a395" />


- Propagation Delay Calculations:
Delays are measured at the 50% point (**0.9 V**) of the supply voltage.

**Rise Delay:** Time difference between input falling edge and output rising edge at 50%
Calculation: 2.482 ns − 2.142 ns = 0.34 ns

<img width="930" height="852" alt="Rise Delay" src="https://github.com/user-attachments/assets/b3b62250-8948-4359-aae1-9282d143b18b" />

**Fall Delay:** Time difference between input rising edge and output falling edge at 50%
Calculation: 4.335 ns − 4.05 ns = 0.285 ns

<img width="896" height="681" alt="Fall Delay" src="https://github.com/user-attachments/assets/d3afaff2-60df-4cc2-8784-44b0752fb02d" /> 

---

### Static Behavior Evaluation – CMOS Inverter Robustness – Switching Threshold

### Switching Threshold, Vm

the static behavior of a CMOS inverter through SPICE simulations, focusing on how the transistor sizes (specifically the Width-to-Length ratios, W/L) affect the **switching threshold voltage (V<sub>M</sub>)**, which is a critical parameter for robustness.

**1. Two Inverter Configurations:**

*   **Case 1-left graph in the below image (Symmetric Sizing):** 
   - Wn = Wp = 0.375 μm, Ln = Lp = 0.25 μm
- (W/L)n = (W/L)p = 1.5
  
*   **Case 2 - right graph in the below image (Asymmetric Sizing):** W/L of PMOS is 2.5 times larger than the NMOS.
  - Wn = 0.375 μm, Wp = 0.9375 μm, Ln = Lp = 0.25 μm
- (W/L)p = 3.75, (W/L)n = 1.5


<img width="1139" height="598" alt="Screenshot 2025-10-16 at 2 02 44 pm" src="https://github.com/user-attachments/assets/700dcdbc-6320-4d06-b019-ab10f7735ff1" />


**2. Key Observations from Waveforms:**

*   **Robustness:** In both cases, the fundamental CMOS characteristic holds: `VIN = 0 => VOUT = VDD` and `VIN = VDD => VOUT = 0`. This demonstrates the inherent robustness of CMOS logic.
*   **Switching Threshold (V<sub>M</sub>):** This is defined as the point where **V<sub>IN</sub> = V<sub>OUT</sub>**.
    *   For the symmetric inverter (Case 1), V<sub>M</sub> ≈ **0.98V**.
    *   For the asymmetric inverter with a larger PMOS (Case 2), V<sub>M</sub> ≈ **1.2V**.
      
<img width="1330" height="835" alt="Screenshot 2025-10-16 at 2 04 49 pm" src="https://github.com/user-attachments/assets/c4ffffdd-d198-4574-8e6f-2ffa9ac30296" />

      
*   **Region of High Current Flow:** The point V<sub>IN</sub> = V<sub>OUT</sub> is also the region where both the NMOS and PMOS transistors are in saturation, creating a direct path for current from VDD to GND (**short-circuit current**). This is a region of high power consumption.

### Analytical Expressions
** Analytical expression of Vm as a function of (W/L)p and (W/L)n:**

The goal is to find an equation for V<sub>M</sub> based on transistor parameters.For the Given the W/L ratios, calculating  V<sub>M</sub>

*   **Assumption:** The derivation uses the **velocity-saturated current model** and ignores channel-length modulation (λ ≈ 0) for simplicity.
*   **Condition at V<sub>M</sub>:** Since both transistors are ON and in saturation, the currents must be equal in magnitude but opposite in direction (Kirchhoff's Current Law at the output node):
    `I<sub>DSP</sub> + I<sub>DSN</sub> = 0` or `I<sub>DSP</sub> = -I<sub>DSN</sub>`.

*   **Current Equations:**
    *   **NMOS Current:** I<sub>DSN</sub> = k<sub>n</sub> * (V<sub>M</sub> - V<sub>Tn</sub>) * V<sub>DSATn</sub> / 2
        where k<sub>n</sub> = μ<sub>n</sub>C<sub>ox</sub> * (W/L)<sub>n</sub>
    *   **PMOS Current:** I<sub>DSP</sub> = k<sub>p</sub> * (V<sub>M</sub> - V<sub>DD</sub> - V<sub>Tp</sub>) * V<sub>DSATp</sub> / 2
        where k<sub>p</sub> = μ<sub>p</sub>C<sub>ox</sub> * (W/L)<sub>p</sub>
        *(Note: V<sub>Tp</sub> for PMOS is typically negative)*

<img width="637" height="354" alt="Screenshot 2025-10-16 at 2 07 23 pm" src="https://github.com/user-attachments/assets/cb0eaa68-66d9-408a-9c60-29c5ccb0d4aa" />

<img width="647" height="364" alt="Screenshot 2025-10-16 at 2 13 12 pm" src="https://github.com/user-attachments/assets/cb94948d-908f-49e8-992e-0c3fe0f5e120" />



*   **Solving for V<sub>M</sub>:**
    Setting `I<sub>DSP</sub> = -I<sub>DSN</sub>` and solving for V<sub>M</sub>.
A specific solution based on the velocity-saturation model:
    `V_M = (R * V_{DD}) / (1 + R)`
    where the constant `R` is defined as:
    `R = (k_p * V_{DSATp}) / (k_n * V_{DSATn})`

    By substituting the process parameters (μ, C<sub>ox</sub>, V<sub>DSAT</sub>, V<sub>T</sub>) and the W/L ratios into this equation, the calculated V<sub>M</sub> values match the SPICE simulation results (≈0.98V for Case 1 and ≈1.2V for Case 2).

<img width="637" height="357" alt="Screenshot 2025-10-16 at 2 15 52 pm" src="https://github.com/user-attachments/assets/02bba3a0-711c-445f-8abf-dc015b90e374" />


#### Analytical Derivation of (W/L)p and (W/L)n as a Function of Vm**

* The **switching threshold (Vm)** is the input voltage where ( V_{in} = V_{out} ).
* Analytical expressions relate Vm to transistor size ratios ((W/L)_p) and ((W/L)_n).
* By reversing the relation, ((W/L)) can be expressed as a function of Vm, allowing designers to **set a specific switching threshold** for desired performance.
* Example: For balanced operation at (V_m = V_{DD}/2), ((W/L)_p) must be appropriately scaled (typically 2–3× ((W/L)_n)) due to PMOS mobility differences.

 
<img width="616" height="314" alt="Screenshot 2025-10-16 at 3 53 51 pm" src="https://github.com/user-attachments/assets/89f0cdc8-4862-48cb-bc03-f4ef247c4c31" />

In this image the we can see the expression showing  how the required (Wp/Lp) / (Wn/Ln) ratio can be computed for a given Vm.

---

#### Static Simulation of CMOS Inverter**

* **DC Transfer Characteristics (VTC)** were obtained using Ngspice.
* The **switching threshold** was found by plotting a 45° line ((V_{in} = V_{out})).
* For equal transistor sizes ((W_p = W_n)), the **switching threshold ≈ 0.99 V** (for (V_{DD} = 2.5 V)).
* The inverter shows proper logic behavior with clear VOH and VOL regions.

<img width="1376" height="841" alt="Screenshot 2025-10-16 at 3 58 24 pm" src="https://github.com/user-attachments/assets/1ab3d8d0-2822-41b7-9fef-4b6afac0e724" />

---

#### Dynamic Simulation — Rise and Fall Delays**

* Transient analysis performed using a **pulse input** to evaluate **propagation delays**.
* **Rise Delay (t_r)** ≈ 148 ps and **Fall Delay (t_f)** ≈ 71 ps for equal transistor sizing.
* Delay calculated between 50% points of input and output waveforms.
* This reveals **asymmetric delay** due to mobility differences between PMOS and NMOS.
* 
<img width="647" height="358" alt="Screenshot 2025-10-16 at 4 04 39 pm" src="https://github.com/user-attachments/assets/006c6798-29c2-4a4c-88db-7149006a810d" />

---

#### CMOS Inverter with Increased PMOS Width**

* Increasing PMOS width ((W_p = 2W_n)):

  * **Switching threshold shifts right** to ≈ 1.2 V.
  * Rise and fall delays become more balanced (≈ 80 ps and 76 ps).
  * Stronger PMOS results in faster charging (reduced rise delay).
* Demonstrates how **transistor sizing impacts inverter symmetry and timing**.

<img width="649" height="362" alt="Screenshot 2025-10-16 at 4 02 35 pm" src="https://github.com/user-attachments/assets/dff260ba-7f37-4e48-9b1c-222c762fa257" />


---

### Applications of CMOS Inverter

- **Clock Networks:** Balanced inverters maintain V<sub>M</sub> ≈ V<sub>DD</sub>/2 for minimal clock skew.  
- **Logic Gates:** Used as building blocks in NAND, NOR, and buffer circuits.  
- **Digital Switching Circuits:** Provides rail-to-rail output for high noise margins.  

---




