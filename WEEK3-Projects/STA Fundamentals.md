## Static Timing Analysis (STA) 

This summary outlines the core components and concepts of Static Timing Analysis (STA), a method used to verify the timing performance of a digital circuit design.

### 1. The Three Pillars of STA

STA is built upon three fundamental components:

- **Timing Checks:** The core verification process. These are the "rules" that a design must meet, such as Setup, Hold, Recovery, and Removal.
- **Constraints:** The design specifications. They define the operational goals, like clock frequency (e.g., 1 GHz), which translate into required time limits for signals.
- **Libraries:** The database of cell characteristics. Libraries provide the delay models (e.g., LDM, Constant Current Source) for all logic gates and cells used in the design.


## Core STA Concepts

| Term | Definition | Key Role in STA |
|------|-------------|-----------------|
| **Timing Path** | A complete route for a signal from a start point to an end point | Crucial for identifying the specific circuit sections to analyze |
| **Start Point** | Location where a signal begins. Defined as: **Flop Clock Pin** or **Input Ports** | Defines the start of the timing path |
| **End Point** | Location where the signal must arrive. Defined as: **Flop D Pin** or **Output Ports** | Defines where timing requirements (arrival/required time) are measured |
| **Arrival Time** (T<sub>arrival</sub>) | The actual time required for a signal to travel from the start point to the end point | A measure of the design's actual performance |
| **Required Time** (T<sub>required</sub>) | The expected time (or time range) for a signal to arrive at the end point, as defined by constraints | Defines the system's timing specifications and requirements |
| **Slack** | The difference between the required time and the arrival time:<br>`Slack = T_required - T_arrival` | Determines if a timing requirement is met (positive slack) or violated (negative slack) |
| **Setup Slack** (Max Slack) | Related to the maximum expected arrival time (T<sub>required, max</sub>):<br>`T_required,max - T_arrival`<br>A negative value means the signal arrived too late (violates setup time) | Verifies maximum clock frequency and path speed |
| **Hold Slack** (Min Slack) | Related to the minimum expected arrival time (T<sub>required, min</sub>):<br>`T_arrival - T_required,min`<br>A negative value means the signal arrived too early (violates hold time) | Ensures data stability and prevents race conditions |

### Additional Timing Checks

Beyond basic setup/hold, STA involves several specialized checks:


- **Basic Setup/Hold Analysis:** Applied to all path categories (Reg2Reg, In2Reg, etc.) to ensure data stability around clock edges.

- **Clock Gating Checks:** Ensures the control signal of a clock gate (e.g., an AND gate) is stable to prevent glitches on the gated clock, which is critical for power saving.

- **Recovery & Removal Checks:** These are the setup/hold checks for asynchronous pins like reset or set. They ensure the release of an asynchronous signal is stable relative to the clock edge.

- **Data-to-Data Checks:** Special constraints applied between two data signals (neither is a clock). Used to enforce specific timing relationships, for example, between a control signal and a data pulse at a gate input.

### Timing Path Categories 

| Category           | Timing Path Description                               | Notes                                                                 |
|--------------------|-----------------------------------------------------|-----------------------------------------------------------------------|
| Reg-to-Reg         | Flop clock pin → Flop D pin (between two registers)  | Standard path analysis                                                |
| In-to-Reg          | Input Port → Flop D pin                             | Requires timing with external/virtual clock                           |
| Reg-to-Out         | Flop clock pin → Output Port                        | Part of I/O timing                                                    |
| In-to-Out          | Input Port → Output Port (pure combinational)       | Part of I/O timing                                                    |
| Clock Gating       | Clock → Gate Output (e.g., AND gate)                | Special setup/hold for power-saving logic                            |
| Recovery & Removal | Clock → Asynchronous Pin (e.g., Reset)              | Checks timing for async control signals                              |
| Data-to-Data       | Data signal → Data signal (e.g., A → Control)       | Checks timing between two data signals                               |

## Reg-to-Reg Setup Analysis

### 1. Overview
The analysis focuses on **Reg-to-Reg Setup Analysis** with a single clock, considering **real clock network delay**.  
This **maximum-delay analysis** ensures that the **Arrival Time (Tₐᵣᵣᵢᵥₐₗ)** is less than the **maximum allowable delay (Tᵣₑq,ₘₐₓ)**.

---

### 2. Analysis Setup
- **Circuit:** Launch Flop → Combinational Logic → Capture Flop (including clock network delay)
- **Clock Frequency:** 1 GHz → **Clock Period:** 1 ns
- **Goal:** To analyze all components—flops, combinational logic, and clock network—for a complete understanding of setup timing.
- **Delay Units:** Delays are expressed in abstract “units,” which correspond to picoseconds or nanoseconds in real hardware.

---

### 3. Timing Graph Construction (Graph-Based Analysis)
To simplify delay computation for the STA engine, the **combinational logic** is transformed into a **Timing Graph** or **Directed Acyclic Graph (DAG)**.

### 3.1 Graph Elements
- **Nodes:** Represent key circuit points such as input pins, gate terminals, and output pins. Each node is assigned a **cell delay**.
- **Source Node:** A virtual node connected to all primary inputs. Edges from this node model signal propagation delays.
- **Edges:** Directed arrows connecting nodes, each assigned a **wire delay** value.

### 3.2 Purpose
The timing graph enables efficient computation of **arrival** and **required** times at every node, not just endpoints, facilitating detailed timing diagnostics.

---

#### 4. Timing Computation

### 4.1 Actual Arrival Time (AAT)
- **Definition:** The time when the latest transition occurs at any node after the clock edge at the launch flop.
- **Direction:** Computed **forward** (Launch → Capture).
- **Computation Rule:**  
  At nodes with multiple inputs, the **worst (maximum)** arrival time is taken to account for the longest possible delay path.

`` AAT_node = Worst(AAT_input + Delay_wire + Delay_cell) ``



---

#### 4.2 Required Arrival Time (RAT)
- **Definition:** The latest time a transition is allowed to arrive at a node within a clock cycle, based on design constraints.
- **Direction:** Computed **backward** (Capture → Launch).
- **Computation Rule:**  
At nodes with multiple fan-out paths, the **best (minimum)** required time is taken to ensure the most stringent timing constraint is met.

`` RAT_node = Best(RAT_fanout - Delay_wire - Delay_cell)`` 


---

#### 5. Slack Computation
**Slack** represents the timing margin at each node.

`` Slack = RAT - AAT``


#### Key Points
- **Positive Slack:** Timing is met.
- **Zero Slack:** Path operates at its timing limit.
- **Negative Slack:** Timing violation exists.

- Node-level slack computation helps locate the exact source of violations, guiding **ECO (Engineering Change Order)** actions for timing closure.

#### Advanced Timing Graph: Pin-Node Convention
The Pin-Node Convention is a technique for modeling a circuit's timing graph where every significant point (pin) is converted into a node. This contrasts with simpler methods where entire cells (like gates) are single nodes.

**Key Concepts**

- Conversion: Every input/output pin of a cell (e.g., A1,A0,B1,B0) becomes a separate node in the timing graph.

- Cell Delay Abstraction: The logic cell itself disappears from the graph, but its cell delay is retained as the delay on the directed edge connecting its input pin node to its output pin node (e.g., the edge from A1 to A0 carries the 2-unit cell delay).

- Wire Delay: Wire delays remain as the delay on the edges connecting the output pin of one gate to the input pin of another (e.g., A0 to B2 has a wire delay of 0.2 units).

- Analysis Continuity: Once the graph is built, the calculation of Actual Arrival Time (AAT), Required Arrival Time (RAT), and Slack at each node follows the same rules as in the Graph-Based Analysis:

Transistor-Level Analysis for Library Parameters
This section explains the internal structure of a flip-flop and uses it to analytically derive the crucial timing parameters found in technology libraries.

#### A. Flip-Flop Structure (Master-Slave)

A standard Positive Edge-Triggered Flip-Flop is constructed using a Master-Slave configuration:

Master Latch (Negative Latch): Operates and is transparent when the clock signal (CLK) is LOW (negative level). It captures the D input to its internal output (QM).

Slave Latch (Positive Latch): Operates and is transparent when the CLK is HIGH (positive level). It captures the data from the Master Latch (QM) to the final output (Q).

Mechanism: When CLK is low, the Master captures D. When CLK transitions high (the positive edge), the Master becomes opaque (holds its value), and the Slave opens to pass the captured QM to Q.

#### B. Derivation of Setup Time

Definition: T setup is the time the D input must be stable before the rising edge of the clock to ensure the data is reliably latched internally.

Transistor Justification: For the Master Latch to reliably store the new D value, the signal must propagate through its internal circuitry before the clock edge arrives and closes the latch.

#### Clock-to-Q Delay (Tcq) and Hold Time (Th)

##### Tclk-to-Q Delay:

Definition: The time required for the data to transition from the clock pin to the Q output after the active clock edge arrives.

Transistor Justification: Once the clock goes HIGH, the Slave Latch opens, and the latched data (QM) travels through the Slave Latch to the final Q output. This travel time is the Tclk-to-Q.

##### Hold Time (Thold): (Discussion was incomplete, but the concept was set up)

Definition: The time the D input must remain stable after the active clock edge.

Transistor Justification: Ensures the new data doesn't corrupt the value being transferred out of the Master Latch during the clock edge transition.

### On-Chip Variation (OCV) and Its Impact on Timing

#### 1. Purpose
OCV analysis aims to account for real-world fabrication variations that cause timing deviations in chips. It studies how microscopic process differences affect transistor performance and overall circuit delay.
#### 2. Key Sources of Variation

- (a) Etching Process
Etching defines the shape and dimensions (width W and length L) of transistor structures.
During fabrication, imperfections such as irregular edges or distorted patterns occur due to physical and chemical effects in the fab.

As a result:

The actual gate dimensions (W′, L′) differ from the ideal (W, L).
This alters the W/L ratio, which directly affects the drain current (ID).
Since ID controls how fast a transistor switches, this impacts propagation delay.

Observation:
Even minor W/L variations in a single inverter get amplified when many inverters are chained in a circuit.

#### (b) Oxide Thickness Variation

Each transistor has a gate oxide layer (tox) between the gate and channel.
Ideally, tox is uniform, but in practice, oxide thickness fluctuates due to non-uniform oxidation during fabrication.

The oxide capacitance (Cox) depends on this thickness.
→ Variation in tox changes Cox → changes ID → alters cell delay.
Pattern:
Middle transistors (surrounded by others) show less variation than edge transistors (exposed to different neighboring structures).

#### 3. Relationship Between Drain Current and Delay

During switching, the inverter behaves like an RC circuit:
PMOS (when ON) acts as a resistor (R) charging the load capacitance (C).

The propagation delay (tpd) ≈ R × C.
Since R ∝ 1/ID,
→ Lower ID ⇒ Higher resistance ⇒ Longer delay.

Thus, any variation that changes W/L or tox leads to drain current fluctuation, which directly affects circuit timing.

#### Relationship Between Resistance, Drain Current, and On-Chip Variation (OCV)
##### 1. Propagation Delay and Resistance

The propagation delay (tpd) is the time difference between the 50% point of input and output waveforms in a CMOS inverter.

Delay depends on resistance (R) and capacitance (C).

Higher resistance ⇒ slower current flow ⇒ longer capacitor charging time ⇒ increased delay and distorted output waveform.

##### 2. Resistance and Drain Current (ID)
In MOSFETs, resistance is not constant — it varies with drain current (ID) and drain voltage (VDS).

Ohm’s law (V = IR) implies constant R for linear systems, but MOSFETs are nonlinear devices.

When plotting ID vs VDS, current increases and then saturates, making resistance R = V/I vary across different operating points.

##### 3. Linking Delay, Resistance, and Device Parameters
Variations in gate width (W), length (L), or oxide thickness (tₒₓ) → change in drain current (ID) → change in resistance (R) → change in propagation delay (tpd).

##### 4. Delay Variations Across Inverters
Even if all inverters are designed for the same delay (e.g., 100 ps), manufacturing variations cause each inverter to behave slightly differently.These variations come from differences in W, L, tox, and proximity effects during fabrication.

##### 5. Delay Distribution and OCV
   
Plotting the number of inverters vs. delay gives a distribution curve centered around the nominal delay (e.g., 100 ps).

Most inverters show delays close to nominal, while fewer show higher or lower delays.

The spread of this distribution represents On-Chip Variation (OCV).

Example:
+8% D-rate → slower cells
–9% D-rate → faster cells
These OCV D-rates are used in Static Timing Analysis (STA) to model timing margins due to process variations.
