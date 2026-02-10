# Table of Contents: Circuit Design with NMOS and SPICE Simulation

## 1. Introduction to Circuit Design
- [1.1 Introduction to Basic Element in Circuit Design – NMOS](#11-introduction-to-basic-element-in-circuit-design--nmos)
- [1.2 Strong Inversion and Threshold Voltage](#12-strong-inversion-and-threshold-voltage)
- [1.3 Threshold Voltage with Positive Substrate Potential](#13-threshold-voltage-with-positive-substrate-potential)

## 2. NMOS Operating Regions
- [2.1 NMOS Resistive Region and Saturation Region of Operation](#21-nmos-resistive-region-and-saturation-region-of-operation)
- [2.2 Resistive Region of Operation with Small Drain-Source Voltage](#22-resistive-region-of-operation-with-small-drain-source-voltage)
- [2.3 Drift Current Theory](#23-drift-current-theory)
- [2.4 Drain Current Model for Linear Region of Operation](#24-drain-current-model-for-linear-region-of-operation)
- [2.5 SPICE Conclusion to Resistive Operation](#25-spice-conclusion-to-resistive-operation)
- [2.6 Pinch-off Region Condition](#26-pinch-off-region-condition)
- [2.7 Drain Current Model for Saturation Region of Operation](#27-drain-current-model-for-saturation-region-of-operation)

## 3. Introduction to SPICE Simulation for MOSFETs

- [3. Introduction to SPICE Simulation for MOSFETs](#3-introduction-to-spice-simulation-for-mosfets)
  - [3.1 Basic SPICE Setup](#31-basic-spice-setup)
  - [3.2 MOSFET Models in SPICE](#32-mosfet-models-in-spice)
  - [3.3 Inputs to SPICE Engine](#33-inputs-to-spice-engine)
  - [3.4 Circuit Description in SPICE Syntax](#34-circuit-description-in-spice-syntax)
  - [3.5 SPICE Lab with sky130 Models](#35-spice-lab-with-sky130-models)

---


### 1. Introduction to Circuit Design

#### 1.1 Introduction to Basic Element in Circuit Design – NMOS

Circuit design at the transistor level involves connecting PMOS and NMOS transistors in specific configurations to create logic gates.

**How it works:**  
- PMOS transistors are typically placed in the **pull-up network** (closer to VDD).  
- NMOS transistors are placed in the **pull-down network** (closer to GND).  
- The specific arrangement of these transistors defines the gate's function (e.g., AND, OR, NAND, NOR).

**Example: CMOS Inverter**

<img width="249" height="231" alt="CMOS Inverter Schematic" src="https://github.com/user-attachments/assets/b354188c-d51f-4c9f-92a1-05ac203adb64" />

**Circuit Description:**  
- One PMOS transistor connected to **VDD**.  
- One NMOS transistor connected to **GND**.  
- Gates of both transistors tied together as the **input**.  
- Drains of both transistors tied together as the **output**.  

**Function:** Logically inverts the input signal (Input=1 → Output=0, Input=0 → Output=1).

**W/L Ratio:**  
The **Width-to-Length ratio (W/L)** of the transistors determines current drive and the cell's delay. Adjusting W/L allows designers to tune the speed of the logic cell.

SPICE Curves for Inverter Analysis:

<img width="581" height="362" alt="SPICE Simulation Curves" src="https://github.com/user-attachments/assets/81895764-9026-4466-8490-4423668d4328" />


- The top graph indicates the current-voltage (I-V) characteristics of the NMOS and PMOS transistors within an inverter. It shows the relationship between the drain-source current (Ids) and the output voltage (Vout) for various input voltages (Vin).
- The bottom graph is the Voltage Transfer Curve (VTC) of the inverter, which plots the output voltage as a function of the input voltage

---

## 1.2 The Critical Role of SPICE Simulation

Even if higher-level flows (Physical Design, STA) don’t directly run SPICE, they depend on SPICE-generated data.

**Why SPICE is needed:**

Models transistor behavior accurately (I<sub>D</sub> − V<sub>DS</sub>).
Determines delay and slew for each logic cell.

**Delay Tables (Cell Characterization):**  

Cell characterization generates Delay Tables (often stored in Liberty files) that allow STA tools to quickly look up a cell's delay without running SPICE.


- **Structure:** Delay tables are 2D Lookup Tables (LUTs) indexed by two critical electrical parameters:

    --Input Slew: The transition time (rise/fall) of the signal arriving at the cell input.

    --Output Load: The total capacitance the cell's output must drive.
  
  <img width="642" height="197" alt="Screenshot 2025-10-15 at 12 25 29 pm" src="https://github.com/user-attachments/assets/4a4bd244-2ab2-4fe0-8883-f99e698fc999" />

  


- **Delay Value:** The intersection of a specific Input Slew row and Output Load column contains the pre-calculated delay value.

  <img width="576" height="314" alt="Screenshot 2025-10-15 at 11 28 37 am" src="https://github.com/user-attachments/assets/26962a6c-5bb2-4fc1-a728-0f4e85f0cee7" />


- **Cell Tuning:** Cells of the same logic function but with different drive strengths (e.g., Buffer Type 1 vs. Type 2) have different W/L ratios, resulting in unique delay values in their respective tables (e.g., X22 vs. Y24 for the same operating conditions).

- **Interpolation:** If an exact output capacitance value is not listed in the table, the STA tool uses interpolation between the nearest values to estimate the required delay (e.g., estimating a delay between X9 and X10).

<img width="579" height="298" alt="Screenshot 2025-10-15 at 11 28 28 am" src="https://github.com/user-attachments/assets/5f4a0fc0-daa9-4b08-aaf5-80f0e7d902fe" />

---

## 1.3 NMOS Transistor Structure and Threshold Voltage

**NMOS Overview:**  
- Four-terminal device built on a **P-type substrate** (N-channel MOS).  

**Terminals:**  
- **Gate (G):** Polysilicon/metal over gate oxide; controls conductivity.  
- **Source (S) & Drain (D):** N+ diffusion regions; source and drain are interchangeable until voltage applied.  
- **Body/Bulk (B):** P-type substrate; usually grounded. Applying voltage tunes **Threshold Voltage (V<sub>T</sub>)**.  

**Isolation:** Shallow trench isolation separates one transistor from its neighbor.

<img width="581" height="246" alt="NMOS Structure" src="https://github.com/user-attachments/assets/52c0ae21-5941-49db-88fd-7c2b31813cd3" />

---

### 1.3.1 Threshold Voltage (V<sub>T</sub>) and Channel Formation

- **Threshold Voltage (V<sub>T</sub>):** Minimum gate-to-source voltage required to form a conducting channel.  
- **Key to SPICE modeling.**

**Behavior with V<sub>GS</sub>:**  

| Step | Gate Voltage (V<sub>GS</sub>) | Effect | Result |
|------|-------------------------------|--------|--------|
| 1    | V<sub>GS</sub> = 0            | All terminals grounded. | High resistance between Source and Drain (junctions are off). |
| 2    | V<sub>GS</sub> is small & positive | Positive charge on the metal gate repels positive carriers (holes) in the P-substrate beneath the gate. | A Depletion Region forms, leaving immobile negative charges behind. The P-N junctions remain off. |
| 3    | V<sub>GS</sub> ≥ V<sub>T</sub> | Sufficiently strong positive potential pulls minority carriers (electrons) towards the gate-oxide interface. | An Inversion Layer (Channel) forms, connecting the N<sup>+</sup> Source and N<sup>+</sup> Drain. The transistor is now ON and can conduct current. |


**Illustration:**

<table>
<tr>
<td>
  <figure>
    <img src="https://github.com/user-attachments/assets/37fd98c7-e934-4398-9c1d-5aff6ba70a93" width="300">
    <figcaption>V<sub>GS</sub> = 0V</figcaption>
  </figure>
</td>
<td>
  <figure>
    <img src="https://github.com/user-attachments/assets/983cc8f9-1ece-4e6e-afee-15d39b1fe919" width="300">
    <figcaption>0V &lt; V<sub>GS</sub> &lt; V<sub>T</sub></figcaption>
  </figure>
</td>
<td>
  <figure>
    <img src="https://github.com/user-attachments/assets/b024e6e2-e00c-4d91-9ae1-4df6847343ca" width="300">
    <figcaption>V<sub>GS</sub> ≥ V<sub>T</sub></figcaption>
  </figure>
</td>
</tr>
</table>



#### 1.2 Strong Inversion and Threshold Voltage
- Concept of strong inversion in MOS devices
- Definition and significance of threshold voltage (V<sub>TH</sub>)
- Factors affecting threshold voltage
- Surface potential requirements

#### 1.3 Threshold Voltage with Positive Substrate Potential
- Body effect phenomenon
- Mathematical formulation of body effect
- V<sub>TH</sub> modification with substrate bias
- Practical implications in circuit design

### 2. NMOS Operating Regions

#### 2.1 NMOS Resistive Region and Saturation Region of Operation
- Overview of different operating regions
- Boundary conditions between regions
- I-V characteristics in each region

#### 2.2 Resistive Region of Operation with Small Drain-Source Voltage
- Linear operation at low V<sub>DS</sub>
- Channel formation and charge distribution
- Resistance characteristics

#### 2.3 Drift Current Theory
- Fundamental carrier transport mechanism
- Electron drift in electric fields
- Mobility considerations
- Current density equations

#### 2.4 Drain Current Model for Linear Region of Operation
- Derivation of linear region current equation
- I<sub>DS</sub> = f(V<sub>GS</sub>, V<sub>DS</sub>) relationship
- Approximation techniques
- Practical calculation methods

#### 2.5 SPICE Conclusion to Resistive Operation
- How SPICE models resistive region
- Accuracy considerations
- Model parameters for linear operation
- Simulation verification

#### 2.6 Pinch-off Region Condition
- Physical mechanism of channel pinch-off
- Voltage conditions for saturation
- Channel length modulation effect
- Transition from linear to saturation

#### 2.7 Drain Current Model for Saturation Region of Operation
- Saturation current derivation
- Square-law relationship
- Channel length modulation factor
- Advanced saturation models


----

### 3. Introduction to SPICE Simulation for MOSFETs

### What is SPICE and Why is it Used?
- **SPICE** (Simulation Program with Integrated Circuit Emphasis) is a software engine for circuit simulation
- Contains **predefined mathematical models** for semiconductor devices derived from physics
- **Purpose**: Calculates accurate voltage/current waveforms using circuit description (netlist) and technology parameters
- **Application**: Waveforms form foundation for calculating **cell delays** used in **Static Timing Analysis (STA)**


#### 3.1 Basic SPICE Setup
SPICE requires two main inputs:
1. **Circuit Netlist** - Text description of components and connections
2. **Technology Parameters** - Device model constants from foundry
   
#### 3.2 MOSFET Models in SPICE

SPICE uses complex device physics equations with full accuracy (unlike simplified hand calculations).

##### Threshold Voltage Equation
```
VT = VTO + γ * (√(2φF + VSB) - √(2φF))
```
Where:
- **VTO**: Threshold voltage when VBS = 0
- **γ (gamma)**: Body effect coefficient  
- **φF (phi)**: Fermi potential

##### Drain Current Equations
- **Linear Region**: Full equation including VDS²/2 term
- **Saturation Region**: Quadratic function with channel-length modulation factor (λ)

![MOSFET Equations](https://github.com/user-attachments/assets/fa374cd1-68c2-4bca-89f7-61f4f212ed8d)


#### 3.3 Inputs to SPICE Engine

##### A. SPICE Model Parameters (Technology File)
- **Constants**: VTO, γ, KP, λ, C<sub>ox</sub>, etc.
- **Technology-dependent**: Provided by foundry for each node (180nm, 20nm, etc.)
- **Crucial**: Correct model file for technology node is essential

##### B. SPICE Netlist
This is a text-based description of the circuit to be simulated.
It defines all components (transistors, resistors, voltage sources) and how they are connected via nodes.

#### 3.4 Circuit Description in SPICE Syntax

### Example Circuit Components

![Circuit Diagram](https://github.com/user-attachments/assets/8c1e2bdc-8a61-41ca-b8aa-c06ed4c943f5)

- NMOS transistor with gate protection resistor (R1 = 55Ω)
- Drain supply voltage (VDD = 2.5V)
- Variable gate voltage source (VIN)
- Source and Bulk grounded (VSS = 0V)
- NMOS dimensions: W = 1.8µ, L = 1.2µ

##### Step 1: Define Nodes
![Node Definition](https://github.com/user-attachments/assets/d76bce8d-c7c2-4592-ac8d-c3655a2b4b39)

Identify points in the circuit that are at the same potential (connected by a wire with no components).
Example Nodes: VDD, in, N1, 0 (ground).

##### Step 2: Component Syntax
![Netlist Syntax](https://github.com/user-attachments/assets/efe1f898-e5ac-43d5-90d2-09940fdcef34)

```spice
M1 vdd n1 0 0 nmos W=1.8u L=1.2u
R1 in n1 55
Vdd vdd 0 2.5
Vin in 0 2.5
```

| **Line in Netlist**                | **Definition / Description**                                                                                                                                                                                                                     |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `M1 vdd n1 0 0 nmos W=1.8u L=1.2u` | Defines the **NMOS transistor M1**. The four node connections are **Drain (vdd)**, **Gate (n1)**, **Source (0)**, **Bulk (0)**. The transistor uses the `nmos` model (from the technology file) with **Width = 1.8 µm** and **Length = 1.2 µm**. |
| `R1 in n1 55`                      | Defines a **resistor (R1)** of **55 Ω** connected between **input node (in)** and **gate node (n1)**. Acts as a **gate protection resistor**.                                                                                                    |
| `Vdd vdd 0 2.5`                    | Defines a **DC voltage source (VDD)** applying **2.5 V** between the **vdd node (positive)** and **ground (0)**. This is the **supply voltage** for the drain terminal.                                                                          |
| `Vin in 0 2.5`                     | Defines a **DC voltage source (VIN)** applying **2.5 V** between the **input node (in)** and **ground (0)**. This provides the **gate voltage** for the transistor.                                                                              |


#### 3.5 SPICE Lab with sky130 Models
### Setup sky130 Environment
```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
```

##### Important Files
![Repository Structure](https://github.com/user-attachments/assets/a32ad27e-2635-4167-8e70-05f8947e9c11)

- **sky130A.tech.lib** - Technology library
- **models/spectre/sky130.lib.spice** - SPICE models
- **day1_nfet_idvds_L2_W5.spice** - Example simulation file

##### Example SPICE File
![Example Netlist](https://github.com/user-attachments/assets/fa314d52-55d4-4ff3-946a-c9557e97794d)

##### Run Simulation and Plot Results
```bash
ngspice day1_nfet_idvds_L2_W5.spice
plot -vdd#branch
```

##### Simulation Results
![Simulation Waveform](https://github.com/user-attachments/assets/88d6c7e9-c4d3-404a-852c-ba233567b511)

![I-V Characteristics](https://github.com/user-attachments/assets/41b16a29-f1ff-4e09-91ba-ccc5d88afe87)

1. Linear (Triode) Region
- Occurs at low V<sub>DS</sub> (left side of the graph).

Characteristics:
- Curves are nearly straight with steep slope.
- Transistor behaves like a voltage-controlled resistor.
- I<sub>DS</sub> increases linearly with V<sub>DS</sub>.

2. Saturation Region
- Occurs as V<sub>DS</sub> increases past a certain point.

Characteristics:
- Curves flatten out.
- Transistor behaves like a voltage-controlled current source.
- I<sub>DS</sub> remains relatively constant, independent of V<sub>DS</sub>.

---














## Key Takeaways
- SPICE uses accurate device models for simulation
- Requires correct netlist syntax and technology parameters
- sky130 provides open-source PDK for practical learning
- Results include I-V characteristics for circuit analysis
