# 📅 Day 2 – Summary: RTL, `.lib`, Flip-Flops & Synthesis Optimizations

## Objective
Gain practical understanding of **Liberty library files, Verilog flip-flops, hierarchical vs. flat synthesis**, and **common synthesis optimizations** using Yosys and the Sky130 standard cell library.

---

## 1. Understanding the `.lib` (Library) File
- `.lib` files define **timing, power, and area** characteristics of standard cells.
- **Key Elements:**
  - **Library name:** e.g., `sky130_fd_sc_hd__tt_025C_1v80.lib` → library, process corner, temperature, voltage
  - **PVT corners:** Process, Voltage, Temperature variations ensure reliable operation
  - **Units:** ns, V, nW, mA, kΩ, pF
  - **Cells:** Logic gates, flip-flops, multiple **drive strengths**
- **Synthesis implication:** Tools select appropriate cell flavors automatically to meet timing and power requirements
#### [Understanding the `.lib` (Library) File](#1understanding-the-lib-library-file-in-sky130)
---

## 2. Understanding `synth -top`
- **Hierarchical Synthesis:** Preserves submodules (e.g., U1, U2), useful for modular design
- **Flat Synthesis:** Flattens all submodules into a single-level netlist, optimized for area/timing
- **Lab Workflow:** Load library → read Verilog → `synth -top` → `abc -liberty` → `show`
- **Observation:** Submodules can be synthesized separately for **divide-and-conquer optimization**
#### [2. Understanding synth -top](#2understanding-synth--top)
---

## 3. Verilog Flip-Flop Implementation: Simulation & Hardware Analysis
- **Purpose:** Flip-flops stabilize combinational logic and prevent glitches
- **Controls:**
  - **Asynchronous Reset:** Q goes low immediately, independent of clock
  - **Asynchronous Set:** Q goes high immediately, independent of clock
  - **Synchronous Reset:** Q changes only on the clock edge
- **Simulation:** iVerilog + GTKWave confirms functional behavior
- **Synthesis:** Async controls → direct inputs to library DFF cells; Sync controls → combinational logic feeding D input
- **Insight:** RTL coding style affects synthesized hardware
#### [Coding & Types of Flops in Verilog](#3-coding--types-of-flops-in-verilog)
---

## 4. Synthesis Optimizations
- **Power-of-2 Multipliers:** Converted to **bit-shifting** (wiring only, no cells inferred)
- **Composite Constants:** Broken into simpler operations (e.g., `9*A = 8*A + A`)
- **Tool Insight:** Yosys detects and optimizes patterns to **minimize cell usage**
- **Lab Example:** `mult_2.v` → `{A, 1'b0}`, `mult_8.v` → `{A, 3'b0} + A`
#### [Synthesis Optimisations](#4-synthesis-optimisations)

---

##  summary
1. `.lib` files are essential for timing, power, and area characterization
2. Hierarchical synthesis preserves clarity; flat synthesis optimizes for area/timing
3. Flip-flop RTL behavior maps differently depending on **async vs. sync controls**
4. Synthesis tools optimize arithmetic operations to reduce hardware overhead







  
 ## 1. Understanding the `.lib` (Library) File in Sky130

### 🔹 Objective
Gain a practical understanding of the Liberty file (`.lib`), which defines the **timing, power, and area characteristics** of standard cells in digital IC design.

---

### 🔹 What is a `.lib` File?
- A `.lib` file (Liberty format) is a **characterization file** for standard cells.  
- Defines the **timing, power, and functional characteristics** of each cell.  
- Acts as a **bridge** between RTL → synthesis → physical design.

---

### 🔹 Key Elements of the Library File

#### 1. Library Name
- Example: `sky130_fd_sc_hd__tt_025C_1v80.lib`  
- Breakdown:
  - `sky130_fd_sc_hd` → SkyWater 130nm Standard Cell HD library  
  - `tt` → Typical process corner  
  - `025C` → Temperature (25°C)  
  - `1v80` → Supply Voltage (1.8V)  

---

#### 2. PVT Corners
- **P → Process:** Variations in fabrication affect transistor speed  
  - TT (Typical-Typical): Nominal/typical corner  
  - FF (Fast-Fast), SS (Slow-Slow), SF (Slow-Fast), FS (Fast-Slow)  
- **V → Voltage:** Supply voltage changes impact speed/power  
  - Examples: 1.62V, 1.80V (nominal), 1.95V  
- **T → Temperature:** High temperatures slow transistors, increase leakage  
  - Examples: -40°C, 25°C, 125°C  

📌 **Why it matters:** Libraries are characterized across all PVT corners to ensure reliable operation.  

⚡ **Analogy:** Like baking bread—oven heat/method causes variation; process variations are similar in IC fabrication.

---

#### 3. Units
Defines units for tool accuracy:  
- Time → ns  
- Voltage → V  
- Power → nW  
- Current → mA  
- Resistance → kΩ  
- Capacitance → pF  

---

#### 4. Cells
- `.lib` contains **all standard cells**: logic gates, flip-flops, etc.  
- Each **cell block** begins with the keyword `cell`.  

Examples: `AND2_X0`, `AND2_X1`, `AND2_X2`, `AND2_X4` → different **drive strengths** of a 2-input AND gate.

---

#### 5. Cell Characteristics

##### Area
- The **physical size** of the cell in μm²  
- Larger cells → faster, but higher power/area  

##### Leakage Power
- Power consumed **when the cell is idle**  
- Specified for all input combinations (e.g., 2-input AND → 4 combinations)  

##### Pin Information
- **Input Capacitance:** Load presented to the driving cell  
- **Power/Timing Data:** Transition times, power consumption, timing arcs  

##### Timing Information
- Propagation delay of the cell  
- Usually provided as **lookup tables** indexed by:  
  - Input slew (transition time)  
  - Output load capacitance  

---

### 🔹 Cell Flavors (Drive Strength)
- Single logic function often provided in **multiple drive strengths**  
- Example suffixes: `_0`, `_2`, `_4`  

#### Example – AND2 Gate Variants

| Cell Variant | Transistor Size | Area    | Propagation Delay | Power Consumption |
|:------------:|:---------------:|:------:|:----------------:|:----------------:|
| and2_0       | Smallest        | Lowest | Highest (Slowest)| Lowest           |
| and2_1       | Medium          | Medium | Medium           | Medium           |
| and2_4       | Largest         | Highest| Lowest (Fastest) | Highest          |

📌 **Design Implication:** Synthesis tools automatically select the **appropriate drive strength** for each cell to meet timing constraints efficiently.

---
## 2.Understanding synth -top 

  🔹 Objective
- To understand hierarchical vs flat synthesis.
- To explore how Yosys handles synth-top and how netlists differ.
- To see submodule-level synthesis and its practical use.



We used the multiple_modules.v file containing:
Submodule 1: AND gate
Submodule 2: OR gate
Top module (multiple_modules): Instantiates Submodule 1 (U1) and Submodule 2 (U2)




### Hierarchical vs. Flat Synthesis

| Aspect              | Hierarchical Synthesis        | Flat Synthesis             |
|--------------------|------------------------------|----------------------------|
| Command            | Default synthesis            | Use flatten command        |
| Netlist Structure  | Preserves module hierarchies | Removes all hierarchies    |
| Instance Names     | Shows U1, U2 (module instances) | Shows direct gate instances |
| Use Case           | Modular design, IP reuse,clarity     | Optimized single-level design |
| Example view      |    <img width="200" height="300" alt="heir" src="https://github.com/user-attachments/assets/22dc9600-50e2-4645-811e-586a24724170" />| <img width="200" height="300" alt="Screenshot 2025-09-25 at 9 18 10 am" src="https://github.com/user-attachments/assets/e64469b2-606a-4d54-b065-52b8fb03a7af" /> <img width="200" height="700" alt="flat" src="https://github.com/user-attachments/assets/7a7aa5f4-6300-48ae-9767-35d2eeb8904c" />
 
#### Lab Workflow & Results
#####  Example Design – multiple_modules.v
<img width="500" height="300" alt="mutiple modules verilog file" src="https://github.com/user-attachments/assets/548d6732-e2d0-44b5-b6ee-c683ce6bc42e" />

The design contains two submodules and a top module:

######  Steps Followed in Yosys

#####  Launch yosys and load the library

```
cd yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

#####  Read the Verilog file with multiple modules
``read_verilog multiple_modules.v`` 

#####  Run synthesis with top module
```synth -top multiple_modules```

#####  Map synthesized design to standard cell library
```abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```
# Optional: flatten the hierarchy (skip for hierarchical synthesis)
``flatten``

##### Visualize the synthesized netlist
```show```

Opens a graphical view of the design:
- Hierarchical synthesis: U1 and U2 submodules visible
- Flat synthesis: All gates appear inside the top module
** Useful for verifying module hierarchy and gate connections before exporting netlist **
  
#### Hierarchical Synthesis Results
<img width="600" height="154" alt="Screenshot 2025-09-25 at 12 24 29 pm" src="https://github.com/user-attachments/assets/cf1d1d53-60e9-4502-b943-79c9abf2679b" />

#####  Netlist preserves hierarchies:
```
module multiple_modules(a, b, c, y);
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  output y;
  wire y;
  wire net1;
  sub_module1 u1 (
    .a(a),
    .b(b),
    .y(net1)
  );
  sub_module2 u2 (
    .a(net1),
    .b(c),
    .y(y)
  );
endmodule

module sub_module1(a, b, y);
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  wire _0_;
  wire _1_;
  wire _2_;
  sky130_fd_sc_hd__and2_0 _3_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  assign _1_ = b;
  assign _0_ = a;
  assign y = _2_;
endmodule

module sub_module2(a, b, y);
  input a;
  wire a;
  input b;
  wire b;
  output y;
  wire y;
  wire _0_;
  wire _1_;
  wire _2_;
  sky130_fd_sc_hd__or2_0 _3_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  assign _1_ = b;
  assign _0_ = a;
  assign y = _2_;
endmodule
```

#####  Flat Synthesis Results
<img width="300" height="159" alt="Screenshot 2025-09-25 at 12 33 06 pm" src="https://github.com/user-attachments/assets/e3264ac3-0335-41e1-9448-6092d6e625fa" />

#####  Netlist flattens hierarchies:
```

module multiple_modules(a, b, c, y);
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  output y;
  wire y;
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  wire _4_;
  wire _5_;
  wire net1;
  wire \u1.a ;
  wire \u1.b ;
  wire \u1.y ;
  wire \u2.a ;
  wire \u2.b ;
  wire \u2.y ;
  sky130_fd_sc_hd__and2_0 _6_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  sky130_fd_sc_hd__or2_0 _7_ (
    .A(_4_),
    .B(_3_),
    .X(_5_)
  );
  assign _4_ = \u2.b ;
  assign _3_ = \u2.a ;
  assign \u2.y  = _5_;
  assign \u2.a  = net1;
  assign \u2.b  = c;
  assign y = \u2.y ;
  assign _1_ = \u1.b ;
  assign _0_ = \u1.a ;
  assign \u1.y  = _2_;
  assign \u1.a  = a;
  assign \u1.b  = b;
  assign net1 = \u1.y ;
endmodule
```


### Interesting Observation: OR Gate Optimization

#####  Expected vs. Actual Implementation

```
Expected OR gate:
Y = A + B

Actual implementation (De Morgan's Theorem):
Y = (A' · B')' = A + B  (NAND gate with inverted inputs)
```




### Why Submodule Synthesis?
```
synth_top <module name>  # Synthesize only submodule1
show                  # Shows only AND gate, ignores other hierarchy
```

#### Multiple Instances 
- Multiple Instances: Synthesize once, replicate multiple times
- Instead of synthesizing all 10 → synthesize once → reuse netlist.
- Saves time + ensures consistent optimization.
<img width="300" height="343" alt="Screenshot 2025-09-25 at 12 33 58 pm" src="https://github.com/user-attachments/assets/b82a380e-615a-4467-86e1-3ea13c739991" />
<img width="300" height="351" alt="image" src="https://github.com/user-attachments/assets/c51453a8-67a8-466b-bfe9-24b5ce0cfe5a" />

  

#### Divide-and-Conquer
- Synthesize submodules separately.
- Optimize each block → stitch them together.
- Improves tool performance and design manageability.
<img width="300" height="488" alt="Screenshot 2025-09-25 at 1 13 50 pm" src="https://github.com/user-attachments/assets/e288f3dd-eddb-41fc-a2c1-174bfc5f0bbc" />

<img width="300" height="158" alt="Screenshot 2025-09-25 at 1 11 15 pm" src="https://github.com/user-attachments/assets/3ed2f2df-c8f8-4ac2-8c1a-73905e6e0f21" />
<img width="300" height="170" alt="Screenshot 2025-09-25 at 1 13 22 pm" src="https://github.com/user-attachments/assets/5159a1cc-4bd2-420b-8f2e-a3ebc06ea642" />

---

### Observations
The synth -top option defines the top-level module for synthesis.
Hierarchical netlist keeps module boundaries, good for debugging and IP reuse.
Flat netlist removes boundaries, may lead to better optimization but harder readability.
Designers choose based on trade-off between optimization vs modularity.

## 3. Verilog Flip-Flop Implementation: Simulation Behavior and Synthesized Hardware Analysis

### 1.The Purpose of a Flip-Flop (Flop)

#### Problem: Glitches in Combinational Logic

- Combinational circuits have propagation delays.
- When inputs change at different times, the output can momentarily glitch (produce an incorrect value) before settling to the correct final value.
- Cascading multiple combinational stages causes glitches to propagate, making the output unstable and never truly settled.

#### Solution: Flops as Storage Elements

- Flops (D Flip-Flops) are used to "shield" or isolate combinational stages.
- A flop's output (Q) only changes on a specific clock edge (e.g., the rising edge).
- Even if the flop's input (D) is glitching, the output (Q) remains stable between clock edges.
- This ensures that the next combinational stage receives a stable input, allowing its output to settle correctly.

### 2. The Need for Reset and Set

- Initialization: A flop must have a known starting value when a circuit powers on. An unknown initial state (X) would cause the downstream logic to produce garbage values.
- Control Pins: Reset (forces output to 0) and Set (forces output to 1) are used for this initialization.

#### Synchronous vs. Asynchronous:

**Asynchronous:** The action (reset/set) happens immediately, independent of the clock signal.
**Synchronous:** The action (reset/set) only takes effect on the next active clock edge. It "waits" for the clock.



### 2. Simulation Results (Behavioral Verification)

Toolchain: iVerilog (simulator) -> GTKWave (waveform viewer).
The key differentiator is the **sensitivity list (`always @(...)`)** and the **priority of `if/else` statements**.


## Lab 2
This session is a practical lab demonstrating the simulation and synthesis of three fundamental types of D Flip-Flops (DFFs) in Verilog:

- DFF with Asynchronous Reset: The reset signal takes immediate effect, independent of the clock.
- DFF with Asynchronous Set: The set signal takes immediate effect, independent of the clock.
- DFF with Synchronous Reset: The reset signal only takes effect on the active clock edge.
The lab confirms that the simulated behavior matches the theoretical expectations for each flop type. It then proceeds to synthesize the designs, showing how the RTL code maps to actual standard cells (gates and flip-flops) from the Sky130 library, highlighting key differences in the resulting hardware.

### 1. Simulation Results (Behavioral Verification)

Toolchain: iVerilog (simulator) -> GTKWave (waveform viewer).
#### Summary of D-Flip-Flop Reset/Set Behaviors




| Flop Type | Key Observed Behavior |  Waveform |
| :--- | :--- | :--- |
| **Asynchronous Reset** | When **reset** is asserted, the output (**Q**) goes low **immediately**, without waiting for the clock edge. When reset is de-asserted, Q follows D only at the next clock edge. | Reset pulse causes an **immediate Q change mid-cycle**. <br> <img width="992" height="151" alt="asyncreset" src="https://github.com/user-attachments/assets/39490ea3-6629-4a99-9251-a25061f8172c" /> |
| **Asynchronous Set** | When **set** is asserted, the output (**Q**) goes high **immediately**. While set is active, Q ignores the D input. After set is de-asserted, Q follows D on clock edges. | Set pulse causes an **immediate Q change mid-cycle**. D changes are ignored while set is active. <br> <img width="706" height="151" alt="async set" src="https://github.com/user-attachments/assets/9f675e95-5670-45c5-b2b1-2828acf52c9c" /> |
| **Synchronous Reset** | When **reset** is asserted, the output (**Q**) only changes to low on the **next clock edge**. It does not change immediately. The reset signal has priority over the D input on the clock edge. | Reset is asserted, but **Q waits for the clock edge** to change. <br> <img width="706" height="171" alt="syncset" src="https://github.com/user-attachments/assets/6dbee3f2-a68a-4f4c-8467-03365c88482e" /> |
| **Async + Sync Reset** | Async reset → immediate low; sync reset → low on next clock edge if async inactive. |<img width="706" height="171" alt="async_sync_reset" src="https://github.com/user-attachments/assets/cd34cc21-fe99-40b7-b508-1bf135e17362" />|


----

### 2. Synthesis Results (Physical Implementation)

Toolchain: Yosys (synthesis tool) with Sky130 standard cell library.

#### Comparison of Synthesized D-Flip-Flop Circuits

| Flop Type | Synthesized Circuit | Important Observations |
| :--- | :--- | :--- |
| **Asynchronous Reset** | A single **DFF cell with an asynchronous reset pin** (e.g., `DFF_RSTN`). | The library DFF uses an **active-low** reset (`RSTN`). Yosys inserted an **inverter** on the reset line to convert the RTL's **active-high** reset: <code>reset → inverter → reset_n → DFF</code>. |
| **Asynchronous Set** | A single **DFF cell with an asynchronous set pin** (e.g., `DFF_SETN`). | Similarly, the library DFF uses an **active-low** set (`SETN`). An **inverter** converts the RTL's active-high set: <code>set → inverter → set_n → DFF</code>. |
| **Synchronous Reset** | A **standard DFF** (without a dedicated reset pin) combined with **combinational logic** (MUX/AND gate) on the D input. | The synchronous reset is implemented as logic *before* the D input: <code>D_effective = (sync_reset) ? 1'b0 : D</code>. Yosys realized this using an **AND gate**: <code>D_effective = ~sync_reset & D</code>. |

<img width="500" height="165" alt="Screenshot 2025-09-25 at 5 28 00 pm" src="https://github.com/user-attachments/assets/88313bb4-00b4-445d-b9c8-dc912146a742" />
<img width="500" height="134" alt="Screenshot 2025-09-25 at 5 29 10 pm" src="https://github.com/user-attachments/assets/61be3019-641e-4864-8c95-d82937e8ec96" />
<img width="500" height="174" alt="Screenshot 2025-09-25 at 5 30 01 pm" src="https://github.com/user-attachments/assets/9b15bd52-98df-4aa7-a1ba-db079a548c55" />
<img width="500" height="186" alt="Screenshot 2025-09-25 at 5 31 13 pm" src="https://github.com/user-attachments/assets/f7eabb45-76da-4df4-acf4-934b4ef57776" />



**Critical Synthesis Insight**

Asynchronous controls are direct inputs to the flip-flop cell itself.
Synchronous controls are implemented using the existing combinational logic that feeds the D-pin of a standard flip-flop (with no dedicated reset/set pin). This is why the synthesized circuit for the synchronous reset looks different from the asynchronous ones.


### 3.Commands Used 

#### Simulation:
```
iverilog -o a.out tb_<filename>.v
./a.out (generates a .vcd file)
gtkwave tb_<filename>.vcd
```

#### Synthesis (Yosys):
```
cd yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog <filename>.v
synth -top <module_name>
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib (Maps to flip-flop cells)
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib (Maps logic to standard cells)
show (Displays the synthesized schematic)
```
[Terminal Snapshots](/snapshots)

# 4. Synthesis Optimisations  

## 🔑 Key Concepts
- **Special Case Optimisations**: When multiplying by powers of 2 (e.g., 2, 4, 8), the operation simplifies to **bit-shifting** (appending zeros to the input).  
- **Hardware Efficiency**: No actual hardware (e.g., multipliers) is needed for these cases — just **rewiring of signals**.  
- **Synthesis Tools**: Tools like **Yosys** automatically detect and optimise such patterns, resulting in **zero standard cells inferred**.  

---

###  Example 1: Multiplication by 2 (`mult_2.v`)
- **Input**: 3-bit signal `a`.  
- **Output**: 4-bit signal `Y = 2 * a`.  

**Optimisation:**  
Y = {A, 1'b0}; // append a 0 to A

**Synthesis Result:**  
- No cells inferred → pure wiring.
  <img width="631" height="236" alt="Screenshot 2025-09-26 at 7 59 28 am" src="https://github.com/user-attachments/assets/badccb2e-1a52-4c5c-a3d3-909865507388" />


### Example 2: Multiplication by 9 (mult_8.v)
Input: 3-bit signal A.
Output: 6-bit signal Y = 9 * A.

#### Optimisation:
 9 * A = 8 * A + A
       = {A, 3'b0} + A
       = {A, A};   // concatenate A with itself

**Synthesis Result:**
No cells inferred → pure wiring.
<img width="631" height="236" alt="Screenshot 2025-09-26 at 7 59 28 am" src="https://github.com/user-attachments/assets/ad0672d4-688a-4fd7-9743-67d5e8868ed7" />




## Lab Workflow
**Read Design Files**
```
cd yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog mult_2.v (mult_8.v for example 2)
synth -top mult2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib (Maps logic to standard cells)
show (Displays the synthesized schematic)
```

Check Cell Usage → Confirm no cells are inferred.
Generate Netlist → Observe optimised wiring ({A, 1'b0}).

### Terminal snaps:
Mult2.v
<img width="808" height="575" alt="Screenshot 2025-09-26 at 8 11 50 am" src="https://github.com/user-attachments/assets/dbd6fe2c-f360-4158-8e91-95b92cce3463" />

<img width="606" height="221" alt="Screenshot 2025-09-26 at 7 58 10 am" src="https://github.com/user-attachments/assets/74987686-4a88-49e0-8e00-35a59ddd84b8" />
- **Netlist:**  

<img width="364" height="306" alt="Screenshot 2025-09-26 at 8 05 35 am" src="https://github.com/user-attachments/assets/4c94b9a8-7c79-4bcc-9b62-61edff157e33" />

Mult8.v
<img width="801" height="568" alt="Screenshot 2025-09-26 at 8 12 13 am" src="https://github.com/user-attachments/assets/3943e5fa-6d7b-4bf0-8ae4-24bcd1956188" />


<img width="605" height="176" alt="Screenshot 2025-09-26 at 8 00 45 am" src="https://github.com/user-attachments/assets/0c4c0b39-8763-4892-9daf-40e7b81862fd" />
**Netlist:**
<img width="227" height="137" alt="Screenshot 2025-09-26 at 8 04 28 am" src="https://github.com/user-attachments/assets/1ed94a52-12f7-4d9a-a519-755f5a825831" />

## Takeaways
- Power-of-2 Multipliers → Replace with bit-shifting (append zeros).
- Composite Constants → Break into smaller operations (e.g., 9*A = 8*A + A).
- Synthesis Insight → Tools optimise away multipliers when logic reduces to wiring.
