# NgspiceSky130 - Day 2
## Velocity Saturation and Basics of CMOS Inverter VTC

## Table of Contents
- [SPICE Simulation for Lower Nodes and Velocity Saturation Effect](#spice-simulation-for-lower-nodes-and-velocity-saturation-effect)
  - [MOSFET IV Characteristics & Short-Channel Effects](#mosfet-iv-characteristics--short-channel-effects)
  - [Drain Current vs Gate Voltage Analysis](#drain-current-vs-gate-voltage-analysis)
  - [Velocity Saturation at Lower and Higher Electric Fields](#velocity-saturation-at-lower-and-higher-electric-fields)
  - [Velocity Saturation Drain Current Model](#velocity-saturation-drain-current-model)
  - [Sky130 Lab Simulations](#sky130-lab-simulations)
- [CMOS Voltage Transfer Characteristics (VTC)](#cmos-voltage-transfer-characteristics-vtc)
  - [MOSFET Switching Operation](#mosfet-switching-operation)
  - [VTC Analysis Steps](#vtc-analysis-steps)

---

## SPICE Simulation for Lower Nodes and Velocity Saturation Effect

### MOSFET IV Characteristics & Short-Channel Effects

**Objective:**
Analyze MOSFET behavior (drain current I<sub>D</sub> vs drain-source voltage V<sub>DS</sub> and gate-source voltage V<sub>GS</sub>) for different device sizes:
- Long channel: 1.2 µm 
- Short channel: 0.25 µm

<img width="583" height="318" alt="MOSFET IV Characteristics" src="https://github.com/user-attachments/assets/350a7c1e-53ac-4101-9459-594892e743c8" />

#### **MOSFET Regions of Operation**

*   **Cut-off Region (V<sub>GS</sub> ≤ V<sub>T</sub>):**
    *   **Observation:** Curve along x-axis (I<sub>D</sub> ≈ 0)
    *   **Reason:** MOSFET turned off; no channel formed

*   **Linear/Triode Region (Low V<sub>DS</sub>):**
    *   **Observation:** Initial rising, curved part where I<sub>D</sub> increases with V<sub>DS</sub>
    *   **Equation:** I<sub>D</sub> = K<sub>N</sub>'[(V<sub>GS</sub> - V<sub>T</sub>)V<sub>DS</sub> - V<sub>DS</sub>²/2]
    *   **Behavior:** Approximately linear function of V<sub>DS</sub> at low voltages

*   **Saturation Region (High V<sub>DS</sub> > V<sub>GS</sub> - V<sub>T</sub>):**
    *   **Observation:** Flatter region with slight positive slope
    *   **Equation:** I<sub>D</sub> = (1/2)K<sub>N</sub>'(V<sub>GS</sub> - V<sub>T</sub>)²(1 + λV<sub>DS</sub>)
    *   **Behavior:** Current mostly constant with slight increase due to **Channel Length Modulation (λ)**

### Drain Current vs Gate Voltage Analysis

<img width="585" height="260" alt="Long vs Short Channel Comparison" src="https://github.com/user-attachments/assets/0c35959d-5418-4173-a53e-8bef4a10d8a3" />

#### **Long-Channel MOSFET (L=1.2µm, W=1.8µm)**
*   **Observation:** Quadratic I<sub>D</sub> vs V<sub>GS</sub> relationship at constant V<sub>DS</sub>
*   **Theory Match:** Perfectly follows I<sub>D</sub> ∝ (V<sub>GS</sub> - V<sub>T</sub>)²

#### **Short-Channel MOSFET (L=0.25µm, W=0.375µm)**
*   **Objective:** Test W/L ratio scaling consistency
*   **Finding:** Behavior differs despite same W/L ratio
*   **Critical Observation:** 
    - Linear I<sub>D</sub> vs V<sub>GS</sub> at high V<sub>GS</sub> (not quadratic)
    - Caused by **Velocity Saturation**

<img width="580" height="264" alt="Short Channel Effects" src="https://github.com/user-attachments/assets/be44936c-2b64-4604-b049-5fce7a5c4421" />

### Velocity Saturation at Lower and Higher Electric Fields

#### **Carrier Velocity Behavior**
- **Low electric fields:** v = μ<sub>n</sub>·E (linear increase)
- **Beyond critical field (E<sub>c</sub>):** v → v<sub>sat</sub> (velocity saturation)

<img width="579" height="323" alt="Velocity vs Electric Field" src="https://github.com/user-attachments/assets/2be01933-3090-4ae4-91ee-e274d592cf86" />

#### **SPICE Simulation Evidence**
<img width="590" height="273" alt="I_D vs V_GS Comparison" src="https://github.com/user-attachments/assets/bfb44d2f-c896-4a84-ad9a-571b22168c64" />

<details>
<summary>📊 Simulation Graph Explanation</summary>

#### **Long-Channel Device (Left Plot)**
- **Dimensions:** L=1.2μm
- **Behavior:** Entirely quadratic curve
- **Physics:** Classical MOSFET behavior with I<sub>D</sub> ∝ (V<sub>GS</sub> - V<sub>T</sub>)²

#### **Short-Channel Device (Right Plot)**
- **Dimensions:** L=0.25μm (same W/L ratio)
- **Behavior:** Two distinct regions:
  - **Quadratic** at low V<sub>GS</sub>
  - **Linear** at high V<sub>GS</sub>
- **Physics:** **Velocity saturation** limits carrier speed
</details>

### Velocity Saturation Drain Current Model

**Single equation for all operating regions:**

\[
I_D = K_n \left[ V_{GT} \cdot V_{min} - \frac{V_{min}^2}{2} \right] (1 + \lambda V_{DS})
\]

**Where:**
- V<sub>GT</sub> = V<sub>GS</sub> - V<sub>T</sub>
- V<sub>min</sub> = min(V<sub>GT</sub>, V<sub>DS</sub>, V<sub>DSAT</sub>)
- V<sub>DSAT</sub>: Technology-defined velocity saturation voltage
- λ: Channel-length modulation factor

<img width="576" height="269" alt="Drain Current Derivation" src="https://github.com/user-attachments/assets/e49b2b01-e677-455e-bea0-2b60125eb698" />

#### **Peak Current Comparison**
<img width="578" height="293" alt="Peak Current Comparison" src="https://github.com/user-attachments/assets/de12f9fe-baca-4b22-bda6-e790af2ef39f" />

<details>
<summary>📈 Device Performance Analysis</summary>

| Parameter | Long-Channel Device | Short-Channel Device |
|-----------|---------------------|----------------------|
| **Dimensions** | W=1.8µm, L=1.2µm | W=0.375µm, L=0.25µm |
| **Peak I<sub>D</sub>** | ≈ 410 µA | ≈ 210 µA |
| **Behavior** | Clear saturation | Early velocity saturation |
| **Cause** | Mobility-dominated | Velocity-limited |
</details>

#### **MOSFET Operating Regions**

| Region | Condition | Behavior |
|--------|-----------|----------|
| Cutoff | V<sub>GS</sub> < V<sub>T</sub> | Device OFF |
| Linear | V<sub>DS</sub> < V<sub>GS</sub> - V<sub>T</sub> | Resistive |
| Saturation | V<sub>DS</sub> ≥ V<sub>GS</sub> - V<sub>T</sub> | Quadratic |
| **Velocity Saturation** | V<sub>DS</sub> > V<sub>DSAT</sub> | Linear |

### Sky130 Lab Simulations

#### **I<sub>D</sub>-V<sub>DS</sub> Characterization**

```
ngspice day2_nfet_idvds_L015_W039.spice
plot -vdd#branch
```
<img width="500" height="500" alt="I_D-V_DS Lab" src="https://github.com/user-attachments/assets/133122a3-75ce-4f79-a453-4db7bd67ce7c" />

#### **I<sub>D</sub>-V<sub>GS</sub> Characterization**

```
ngspice day2_nfet_idvgs_L015_W039.spice
plot -vdd#branch
```

<img width="500" height="500" alt="I_D-V_GS Lab" src="https://github.com/user-attachments/assets/7ed0ed33-fc7f-4676-a3f8-e1f3a1818c4e" />

---

## CMOS Voltage Transfer Characteristics (VTC)



#### MOSFET Switching Operation
##### 1. MOSFET as a Switch

#### Key Concept
- **MOSFET operates as a voltage-controlled switch**
- **Turn-on condition:** |V<sub>GS</sub>| > |V<sub>T</sub>|
  - NMOS: Positive V<sub>GS</sub> > V<sub>T</sub>
  - PMOS: Negative V<sub>GS</sub> < -V<sub>T</sub> (|V<sub>GS</sub| > |V<sub>T</sub>|)

<img width="553" height="340" alt="Screenshot 2025-10-15 at 4 11 00 pm" src="https://github.com/user-attachments/assets/106d9e6a-d506-4c32-95cf-30922c014929" />


#### Switch Behavior
- **OFF State (V<sub>GS</sub> < V<sub>T</sub>):** Infinite resistance → Open circuit
- **ON State (V<sub>GS</sub> > V<sub>T</sub>):** Finite resistance → Closed circuit

#### 2. CMOS Inverter Structure

<img width="582" height="318" alt="Screenshot 2025-10-15 at 5 38 32 pm" src="https://github.com/user-attachments/assets/2da21026-28b5-4f69-a775-8905eb5d9f31" />


### Key Components
- **PMOS (Top):** Source connected to V<sub>DD</sub>
- **NMOS (Bottom):** Source connected to GND
- **Gates:** Tied together as V<sub>IN</sub>
- **Drains:** Tied together as V<sub>OUT</sub>
- **C<sub>L</sub>:** Load capacitance (wires + next stage input)

#### 3. Operating Conditions Analysis

##### Case 1: V<sub>IN</sub> = V<sub>DD</sub> (High Input)
The middle diagram in the above image shows  the switch model when Vin = Vdd:
#### Voltage Analysis:
- **NMOS V<sub>GS</sub> = V<sub>IN</sub> - GND = V<sub>DD</sub>** → **Turns ON**
- **PMOS V<sub>GS</sub> = V<sub>IN</sub> - V<sub>DD</sub> = 0** → **Turns OFF**

#### Result:
- **Discharge path** from C<sub>L</sub> to GND through R<sub>n</sub>
- **V<sub>OUT</sub> discharges to 0V**

#### Case 2: V<sub>IN</sub> = 0V (Low Input)
The right diagram in the above image shows the switch model when Vin = 0
#### Voltage Analysis:
- **NMOS V<sub>GS</sub> = 0 - GND = 0** → **Turns OFF**
- **PMOS V<sub>GS</sub> = 0 - V<sub>DD</sub> = -V<sub>DD</sub>** → **Turns ON**

#### Result:
- **Charge path** from V<sub>DD</sub> to C<sub>L</sub> through R<sub>p</sub>
- **V<sub>OUT</sub> charges to V<sub>DD</sub>**

#### 4. Current Flow and Naming Conventions

##### Current Directions
- **V<sub>IN</sub> = V<sub>DD</sub>:** Discharge current from C<sub>L</sub> to GND
- **V<sub>IN</sub> = 0V:** Charge current from V<sub>DD</sub> to C<sub>L</sub>
- `IDSP = -IDSN` due to inversion of current direction in PMOS.

#### Naming Conventions
| Parameter | NMOS | PMOS |
|-----------|------|------|
| Gate-Source Voltage | V<sub>GSN</sub> = V<sub>IN</sub> - GND | V<sub>GSP</sub> = V<sub>IN</sub> - V<sub>DD</sub> |
| Drain-Source Voltage | V<sub>DSN</sub> | V<sub>DSP</sub> |
| Drain Current | I<sub>DSN</sub> | I<sub>DSP</sub> |
| On Resistance | R<sub>n</sub> | R<sub>p</sub> |


**Voltage Relationships**
   - NMOS: `VGSN = VIN - VSS ≈ VIN`, `VDSN = VOUT`
   - PMOS: `VGSP = VIN - VDD`, `VDSP = VOUT - VDD`
   - PMOS requires careful calculation since the gate-to-source voltage is referenced to `VDD`.



**NMOS Load Curve**
   - Drain current `IDSN` increases with `VGSN`.
   - Below threshold: `IDSN ≈ 0` (device off)
   - Above threshold: current increases with `VGSN` and saturates at higher gate voltages.

**PMOS Load Curve**
   - Inverted version of NMOS curve: `IDSP = -IDSN`.
   - Negative gate-to-source voltage: `VGSP < -VTP` turns PMOS on.
   - Output current direction opposite to NMOS.
  These load curves are building blocks to derive the **CMOS voltage transfer characteristic (VTC)**.

**What is VTC?**

Relationship: V<sub>OUT</sub> vs V<sub>IN</sub>
Purpose: Understand inverter switching behavior
Importance: Foundation for delay calculations

From Internal Voltages to Logic Voltages


- In a real digital design, only `VIN` (input) and `VOUT` (output) are visible.
- Internal node voltages (`VGSP`, `VGSN`, `VDSP`, `VDSN`) are abstracted away.
- Goal: **model drain currents and transistor behavior purely as functions of `VIN` and `VOUT`.**

  

### VTC Analysis Steps

#### **Step 1: Convert PMOS Gate-Source Voltage to V<sub>in</sub>**

- V<sub>GSP</sub> = V<sub>IN</sub> - V<sub>DD</sub>
- V<sub>DSP</sub> = V<sub>OUT</sub> - V<sub>DD</sub>

<img width="1439" height="859" alt="Screenshot 2025-10-16 at 8 23 56 am" src="https://github.com/user-attachments/assets/39987134-5913-44c1-8ffb-9aca90668e8f" />




#### **Step 2 & 3: Convert NMOS/PMOS  Drain-Source Voltages to V<sub>out</sub>**

##### Load Line Curves for PMOS
Voltage Transformation

VDSP = V<sub>OUT</sub> - V<sub>DD</sub>
V<sub>OUT</sub> = V<sub>DD</sub> + V<sub>DSP</sub>
Graphical Interpretation

Shift PMOS curves left by V<sub>DD</sub>
VDSP = -2V → V<sub>OUT</sub> = 0V (fully discharged capacitor)
VDSP = 0V → V<sub>OUT</sub> = V<sub>DD</sub> (fully charged capacitor)

<img width="1335" height="656" alt="Screenshot 2025-10-16 at 8 24 53 am" src="https://github.com/user-attachments/assets/d2546598-4e87-451d-9043-6b5c8e9c072a" />

Physical Interpretation

V<sub>OUT</sub> = 0V: Capacitor discharged, maximum charging current needed
V<sub>OUT</sub> = V<sub>DD</sub>: Capacitor fully charged, zero current flow
Intermediate V<sub>OUT</sub>: Partial charging, finite current

Load Line Curves for NMOS

voltage  Conversion

V<sub>GSN</sub> = V<sub>IN</sub> (since V<sub>SS</sub> = 0V)
V<sub>DSN</sub> = V<sub>OUT</sub>


- No complex voltage shifts needed
- Direct replacement of axis labels
- NMOS analysis is straightforward compared to PMOS
- 
<img width="1320" height="658" alt="Screenshot 2025-10-16 at 8 25 20 am" src="https://github.com/user-attachments/assets/1779c609-5dc4-4815-84b3-8918d6123b8b" />



#### **Step 4: Merge Load Curves and Plot VTC**

Superimpose the NMOS and PMOS load curves on a common VIN vs VOUT graph.
Intersection points of the load curves determine VOUT for a given VIN:

Stepwise example:
- VIN = 0V: PMOS in linear region, NMOS cutoff → VOUT = VDD
- VIN = 0.5V: NMOS in saturation, PMOS in linear → VOUT between 1.5–2 V
- VIN = 1V: Both NMOS and PMOS in saturation → VOUT ≈ 0.5–1.5 V (high-gain transition region)
- VIN = 1.5V: NMOS linear, PMOS saturation → VOUT ≈ 0–0.5 V
- VIN = 2V: NMOS linear, PMOS cutoff → VOUT = 0V


<img width="1325" height="660" alt="Screenshot 2025-10-16 at 8 26 28 am" src="https://github.com/user-attachments/assets/fd044f8e-662c-41b6-a479-d26899812b04" />



### Summary – Day 2
- Analyzed NMOS and PMOS IV characteristics for long and short channels
- Observed velocity saturation effects
- Derived load curves for NMOS and PMOS
- Introduced CMOS inverter VTC and explained how to reduce internal voltages to VIN and VOUT




---
