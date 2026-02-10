# 📅 Day 3 
Contents:
- [Topic: Introduction](#topic--introduction)
- [Lab 6 Combinational Optimization](#lab-6)
- [Lab 7 Sequential Optimization](#lab-7)

# Topic : Introduction

## 1. Combinational Logic Optimization

### 1. Introduction & Purpose
- **Goal:** Simplify ("squeeze") combinational logic circuits.
- **Why Optimize?**
  - **Area:** Fewer transistors/gates required.
  - **Power:** Reduced switching activity and static power consumption.

---

### 2. Key Optimization Techniques

#### A. Constant Propagation
- **Concept:** Propagate fixed logic values (0 or 1) through a circuit to simplify downstream logic.
- **Example:**
  ```verilog
  // Original Function
  Y = (A & B | C)'   // A tied to 0

  // Constant Propagation
  // A = 0 -> (0 & B | C) = C -> Y = ~C
  ```
- **Impact:**
Before: Required 6 transistors (complex CMOS gate).
After: Requires only 2 transistors (a simple inverter).

- **Result:** Major savings in area and power.
B. Boolean Logic Optimization

Concept: Using algebraic laws or algorithms (like Karnaugh Maps or the Quine-McCluskey algorithm) to simplify complex Boolean expressions into their most efficient form.
Example:

Original Complex Expression (from a nested multiplexer):
```
Y = A'·C' + A·( B·C + B'·(C·A + C'·0) )
Step-by-Step Simplification:

Inner Mux: (C·A + C'·0) = A·C
Middle Mux: (B·C + B'·(A·C))
Full Expression: Y = A'·C' + A·B·C + A·B'·C
Factor: Y = A'·C' + A·C·(B + B')
Simplify (B + B' = 1): Y = A'·C' + A·C
```

Final Optimized Result:
```
Y = A ⊕ C (An XNOR gate).
```
A very complex logic block is reduced to a single, fundamental gate.

### summary
- Synthesis Tools: Modern Electronic Design Automation (EDA) tools automatically perform these optimizations during the synthesis process.
- Fundamental Principles: The underlying principles are constant propagation and Boolean algebra simplification.
- Significant Impact: Effective optimization is crucial for achieving high-performance, low-power, and area-efficient digital integrated circuits (ICs).

---
## 2. sequential optimization techniques


### 1. Basic Technique: Sequential Constant Propagation
- Definition:
A sequential element (flop) whose output Q always takes a constant value regardless of clock, reset, or other inputs.
The constant value can then propagate through downstream logic, potentially simplifying or removing combinational gates.

Example:

A flop has D = 0 and reset input.

#### Behavior:
- When reset is applied: Q = 0
- When reset is not applied: Q = 0 (since D = 0)
- Downstream logic sees a constant value (0 or 1) and can be simplified.

**Result:** Gates connected to Q can often be removed or replaced with constants (e.g., Y = 1).

**Important Note:**

Not all flops with D tied to a constant are sequential constants.

Example: Flop with asynchronous set and D = 0:

- When set = 1: Q = 1 immediately (asynchronous)
- When set = 0: Q waits for the next clock edge to follow D → Q = 0

Key Insight: Q can take both 0 and 1 depending on asynchronous inputs and clock timing.
Such flops cannot be optimized away.

### Advanced Sequential Logic Optimization Techniques

#### 1. State Optimization
- Optimizes state machines by condensing or removing unused states  
- Creates more efficient state machine implementations  

#### 2. Logic Cloning
- **Problem:** Single logic block driving distant flip-flops causes large routing delays  
- **Solution:** Duplicate the logic block when there's positive timing slack  
- **Benefit:** Places copies closer to destination flip-flops to meet timing constraints  
- **Use Case:** Physical-aware synthesis during floorplanning  

#### 3. Retiming
- **Problem:** Uneven logic delays between flip-flops limit overall clock frequency  
- **Solution:** Redistribute combinational logic across pipeline stages  
- **Example:** Moving logic from a critical path (5ns) to a path with slack (2ns)  
- **Result:** Balanced delays (e.g., 4ns + 3ns) enabling higher frequency (200MHz → 250MHz)  
- **Key Insight:** Utilizes available slack to improve performance  


---

## Lab 6: 
#### Lab Summary: combination optimization technique lab 

The lab module focused on demonstrating Boolean logic optimization of Verilog code using the YoSys synthesis tool and the ABC synthesis tool for technology mapping with a Sky130 liberty file.


| Verilog File    | Logic                        | Boolean Simplification | Expected Result       | Yosys Result                   |
|-----------------|------------------------------|------------------------|-----------------------|--------------------------------|
| opt_check.v     | `y = a ? b : 0;`             | `y = a ⋅ b`            | Two-input AND gate    | Single 2-input AND gate        |
| opt_check2.v    | `y = a ? 1 : b;`             | `y = a + b`            | Two-input OR gate     | Single 2-input OR gate         |
| opt_check3.v    | `y = a ? (c ? b : 0) : 0;`   | `y = a ⋅ b ⋅ c`        | Three-input AND gate  | Single 3-input AND gate        |

**Screenshots:**

- **opt_check.v**  
<img src="https://github.com/user-attachments/assets/b05d7e42-4607-4c1f-a764-d64a21c154d6" width="300">

- **opt_check2.v**  
<img src="https://github.com/user-attachments/assets/862aff00-0339-48f9-a11d-e135384c04ab" width="300">

- **opt_check3.v**  
<img src="https://github.com/user-attachments/assets/f3914db4-dd95-4b70-a969-4ba3b96061e7" width="300">


**Absorption Law: a'b+b=a+b**



```
Key YoSys Commands for Optimization Flow
read liberty -lib [path/to/lib]: Loads the technology library (e.g., sky130.lib).

read verilog [file.v]: Loads the design file.

synth -top [module_name]: Sets the top-level module for synthesis.

opt_clean -purge: Performs constant propagation and unused cell removal (optimization).

abc -liberty [path/to/lib]: Links the optimized design to the standard cell library for technology mapping.

show: Displays the final schematic.
```

## Lab Exceriszes

commands used:
```

read_liberty -lib <library_path>
read_verilog <design_file>.v
synth -top <module_name>
flatten
opt_clean -purge
abc -liberty <library_path>/<library_file>.lib
show


```

**1. Optimization of opt_check4.v**
- RTL Design
```
module opt_check4 (input a, input b, input c, output y);
    assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```
- simplification
```
y = a ? (b ? (a & c) : c) : (!c)
Outer operator: y = a ? X : (!c) becomes y = (a & X) | (!a & !c)
Inner operator X = b ? (a & c) : c becomes X = (b & a & c) | (!b & c)

y = (a & [(b & a & c) | (!b & c)]) | (!a & !c)
y = (a & b & a & c) | (a & !b & c) | (!a & !c)
y = (a & b & c) | (a & !b & c) | (!a & !c)
y = (a & c) & (b | !b) | (!a & !c)
y = (a & c) | (!a & !c)
y=a XNOR c
```
- **Expected Optimized Result:**:single XNOR gate**

- **Actual yosys result :**

<img width="300" height="224" alt="Screenshot 2025-09-26 at 11 00 24 am" src="https://github.com/user-attachments/assets/eb54eb57-d82c-492e-b5b4-29d4836d91d8" />

Yosys successfully optimized the design to match the expected outcome. The schematic shows:

- A single XNOR gate with inputs a and c
- Input b is completely removed from the final logic
- Output y depends solely on the equality of a and c

 `` write_verilog -noattr opt_check4_net.v`` 
 output
<img width="371" height="200" alt="Screenshot 2025-09-26 at 11 18 55 am" src="https://github.com/user-attachments/assets/6f3b8691-0e0f-40eb-a00f-5477df3c0973" />



**2.optimization of the multiple_module_opt.v design**

 **RTL Design**
```

module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule


module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule


module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1); 


endmodule

```


**Circuit Structure:**
```
U1: AND gate (sub_module1) with inputs a and 1'b1 → output n1 = a & 1 = a
U2: XOR gate (sub_module2) with inputs n1 and 1'b0 → output n2 = n1 ^ 0 = n1 = a
U3: XOR gate (sub_module2) with inputs b and d → output n3 = b ^ d
Final output: y = c | (b & n1) = c | (b & a)
```

 **Simplification:**
```
- n1 simplifies to a (AND with constant 1)
- n2 simplifies to a (XOR with constant 0) - becomes redundant
- n3 (b ^ d) is computed but never used - becomes redundant
- Final output simplifies to y = c | (a & b)
```


 **Expected Optimized Result:**
```
- U1 becomes a simple wire carrying a
- U2 and U3 are completely eliminated (dead logic removal)
- Final circuit: Single OR gate with inputs c and (a & b)
- Equivalent to: **y = c | (a & b)**
```

- **Actual result**

<img width="300" height="200" alt="Screenshot 2025-09-26 at 12 25 45 pm" src="https://github.com/user-attachments/assets/876e051f-628f-4235-8828-6cfd20b205b9" />

```
- Circuit simplification: Reduced from 4 modules to 2 fundamental gates
- U1 optimization: a & 1 simplified to direct wire connection a
- U2 elimination: a ^ 0 redundant operation completely removed
- U3 elimination: Unused b ^ d calculation removed as dead logic
- Final output: Simplified to y = c | (a & b)
```

<img width="407" height="476" alt="Screenshot 2025-09-26 at 12 26 56 pm" src="https://github.com/user-attachments/assets/32af50e2-6f6b-40d7-b9af-f8e4a6756b46" />


- Verification:
```
Boolean function matches expected simplification: y = c | (a & b)
Gate count reduced from multiple modules to single compound gate
All redundant inputs and logic successfully optimized out
```

### 3.analysis and optimization of the multiple_module_opt2.v design**

**RTL Design:**
<img width="626" height="276" alt="Screenshot 2025-09-26 at 1 18 12 pm" src="https://github.com/user-attachments/assets/cafe7858-02d0-45db-b562-b62f1fad7c35" />

**circuit analysis:**

Four instances of sub_module (AND gates) in cascade
U1: n1 = a & 0
U2: n2 = b & c
U3: n3 = n2 & d = (b & c) & d
U4: y = n3 & n1 = [(b & c) & d] & [a & 0]

**Boolean Simplification:**

U1 output: n1 = a & 0 = 0 (constant zero)
U4 output: y = n3 & 0 = 0 (AND with constant zero)
Final output: y = 0 regardless of inputs a, b, c, d
**Expected Optimization:**

Complete elimination of all logic gates
Output y hardwired to constant 0

**Actual result:**
<img width="300" height="400" alt="Screenshot 2025-09-26 at 12 58 01 pm" src="https://github.com/user-attachments/assets/6920ef36-ce5a-4fbd-8825-a1882c6fea07" />
<img width="300" height="400" alt="Screenshot 2025-09-26 at 1 30 54 pm" src="https://github.com/user-attachments/assets/ff9c5591-ca65-4cd8-a8ba-5ab84f2d646d" />


**Optimization Achievements:**

All four AND gates removed: U1, U2, U3, U4 completely eliminated
Constant propagation success: U1's a & 0 propagated through entire logic chain
Dead logic elimination: All intermediate signals (n1, n2, n3) optimized away
Input isolation: All inputs (a, b, c, d) remain declared but disconnected from output

**Signal Mapping:**

- _4_ = 1'h0 - Constant zero driver
- y = _4_ - Output directly connected to constant zero
- Inputs a, b, c, d assigned to wires but unused in final logic
  
**Verification:**

- Boolean function confirmed: y = 0 (constant low)
- Original 4-gate cascade completely eliminated
- Optimal constant propagation achieved
- Functional equivalence maintained: output always 0 as per original design


---

## Lab 7:

### **Lab Summary: Sequential Optimization Techniques**

#### **1. Objective**
To investigate when a flip-flop with a constant input is optimized away by the synthesis tool and when it is not.

#### **2. Key Concept**
A flip-flop is only optimized if its output is **provably constant at all times**. If the output change is synchronized to a clock edge (even if the final value is constant), the flip-flop is necessary and will **not** be optimized.

#### **3. Circuits Analyzed**

**Circuit: `dff_const1.v`**
*   **Behavior:** Asynchronous reset sets `Q=0`. The `else` clause assigns `Q <= 1'b1`.
*   **Analysis:** When reset is de-asserted, `Q` must wait for the next clock edge to become `1`. The change is **synchronous**.
*   **Result:** A flip-flop is **inferred** (not optimized). Simulation shows `Q` waiting for the clock edge. Synthesis statistics show one DFF.

**Circuit: `dff_const2.v`**
*   **Behavior:** Asynchronous reset acts as a set, making `Q=1`. The `D` input is also tied to `1'b1`.
*   **Analysis:** The output `Q` is `1` immediately upon reset and remains `1` forever, independent of the clock.
*   **Result:** The flip-flop is **optimized away**. Synthesis shows zero cells; `Q` is connected directly to `1'b1`.

**Circuit: `dff_const3.v`**
*   **Behavior:** Two flip-flops in series. The first (`q1`) is a reset flop (`q1=0` on reset), and the second (`q`) is a set flop (`q=1` on reset). `q1` has `D=1'b1`, and `q` has `D=q1`.
*   **Analysis:** After reset, `q1` becomes `1` on the next clock edge. This causes `q` to become `0` on the following clock edge, creating a one-cycle low pulse. Both flops are essential for this timing behavior.
*   **Result:** **Two flip-flops are inferred.** Simulation confirms the one-cycle pulse. Synthesis statistics show two DFFs.

#### **4. Synthesis Workflow for Sequential Circuits**
The lab introduced a crucial step for synthesizing sequential logic:
1.  `read_liberty -lib <library_file>`
2.  `read_verilog <design_file.v>`
3.  `synth -top <module_name>`
4.  **`dfflibmap -liberty <library_file>`** (New command to map flip-flops to library cells)
5.  `abc -liberty <library_file>`
6.  `show` (to view the netlist)

---

#### **5. Lab Exercises**

- **`dff_const4.v`**

  **RTL Design**  
  <img width="250" alt="Screenshot 2025-09-27 at 7 38 18 am" src="https://github.com/user-attachments/assets/3e6f1b3b-2550-4d55-8c11-929032ed0f8e" />

  - Two registers: q and q1.  
  - On reset: q = 1, q1 = 1.  
  - On clock edge: q1 is always updated to 1, q is assigned q1.  

  **Expected behavior:**  
  After reset, both outputs go high. Since q1 is always 1, q also becomes 1. Both remain constant at 1.


#### Simulation:
```
iverilog dff_const4.v tb_dff_const4.v 
./a.out
gtkwave tb_dff_const4.vcd
```

<img width="400" height="145" alt="dff_cons4" src="https://github.com/user-attachments/assets/bf5b8a38-e4f6-4ee3-a00d-10daa1457869" />

#### Synthesize in Yosys
```
yosys
read_liberty -lib ../lib/sky......lib
read_verilog dff_const4.v
synth -top dff_const4
dfflibmap -liberty ../lib/sky......lib
abc -liberty ../lib/sky......lib
show
```
<img width="200" height="150" alt="Screenshot 2025-09-26 at 5 27 02 pm" src="https://github.com/user-attachments/assets/4985daef-c6cd-4315-9af6-11fb8085e719" />

  **Observations**
  -  Shows q and q1 both go high on reset and stay high afterwards.
  -  The tool optimizes both flops away since their outputs are constant 1.
  -  Even if RTL is written with sequential logic, synthesis removes redundant flops if the behavior reduces to a constant.



  **`dff_const5.v`**
  
  RTL `Design
  
<img width="250" height="250" alt="Screenshot 2025-09-27 at 7 54 03 am" src="https://github.com/user-attachments/assets/d2d0a942-6c0e-43c5-954d-06c25eea1b9d" />

  **Reset = 1 (asynchronous active)**
both q and q1 to 0.
This happens independent of clock.
**After reset deasserted (reset = 0)**
On the first rising edge of clk:
q1 <= 1'b1 (so q1 will become 1 after the clock edge).
q <= q1 (but at this edge, q1 was still 0 from reset, so q gets 0).
On the second rising edge of clk:
q1 is already 1 (from previous cycle).
q <= q1 = 1.
From then onward, both q1 and q stay at constant 1.
.


**simulation:**

<img width="350" height="100" alt="dff_cons5" src="https://github.com/user-attachments/assets/960d785d-2214-4e05-be19-0dc29885f091" />


**synthesis:**

<img width="400" height="128" alt="Screenshot 2025-09-26 at 5 28 00 pm" src="https://github.com/user-attachments/assets/2d19b49a-52c7-4d6c-89e4-294a1de0972a" />

**observation:** 
  - Two flip-flops (dfrtp_1 cells).
  - One loads constant 1 into q1.
  - The other takes q1 as input for q.
  - Reset is active high in RTL, but library uses active-low RESET_B, so two inverters (5 and 6) were inserted.
  - Unlike dff_const4.v, a flop is not optimized away just because its output eventually becomes constant.

---
