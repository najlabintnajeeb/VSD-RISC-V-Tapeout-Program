

### **Gate Level Simulation (GLS)**

#### **1. What is GLS?**
*   **GLS** stands for **Gate Level Simulation**.
*   It is the process of running a simulation where the **netlist** (the output of synthesis) replaces the RTL code as the **Design Under Test (DUT)**.
*   The same testbench that was used for RTL verification is used, but it now stimulates the netlist.
*   The netlist is **logically equivalent** to the RTL code, so it should work with the existing testbench.

#### **2. Why Run GLS?**
There are two main reasons:
*   **1. Verify Logical Correctness After Synthesis:** To confirm that the synthesis process did not introduce logical errors. The transcript hints that "synthesis simulation mismatches" are a key reason why this is necessary.
*   **2. Timing Verification (Conceptual):** RTL simulation has no timing information. GLS can be used for timing validation if the gate-level models include **delay annotation**. This would check for real-world timing issues like setup and hold time violations.
    *   **Note:** The speaker clarifies that the current discussion will focus on **basic GLS without timing annotation**.

#### **3. The GLS Flow (using iVerilog)**
The flow is identical to RTL simulation with one crucial addition:
*   **Inputs:** The simulator (e.g., iVerilog) needs three things:
    1.  The **Design Netlist** (gate-level Verilog).
    2.  The **Testbench**.
    3.  **Gate-Level Verilog Models** (a library that defines the functionality of the standard cells like AND gates and Flip-Flops used in the netlist).
*   **Output:** The simulator generates a VCD file, which can be viewed in a wave viewer like GTKWave, just like in RTL simulation.

#### **4.  Synthesis-Simulation Mismatch**
*   This is a very important topic mentioned as a primary reason for running GLS.
*   It refers to situations where the behavior of the synthesized netlist does not match the behavior of the original RTL code during simulation. The details of why this happens are promised for "subsequent slides."

 Here is a clean, concise summary of the transcript for notes or a portfolio.

---

### **Gate Level Simulation (GLS) & Synthesis Mismatches**

#### **1. Purpose of GLS: Why it's Necessary**
*   **Synthesis-Simulation Mismatches:** The synthesized netlist can behave differently from the original RTL simulation due to coding errors.
*   **GLS catches these mismatches** by simulating the netlist with the original testbench.
*   **Goal:** Verify that the gate-level circuit (netlist) is logically equivalent to the RTL design.

#### **2. Common Causes of Synthesis Mismatches**

**A. Missing Sensitivity List**
*   **Problem:** An `always` block is only sensitive to some inputs, not all.
*   **Example:** A multiplexer (`always @(sel)`) only updates when `sel` changes, not when data inputs (`i0`, `i1`) change. Simulation behaves like a latch; synthesis produces a correct mux.
*   **Fix:** Use `always @(*)` to make the block sensitive to all relevant signals.

**B. Blocking vs. Non-Blocking Assignments**
*   **Blocking (`=`):** Executes sequentially, like software. Order matters.
*   **Non-Blocking (`<=`):** Executes in parallel. All right-hand sides are evaluated first, then assignments happen simultaneously.
*   **Critical Rule:** **Use non-blocking assignments (`<=`) for sequential logic (flip-flops).**
*   **Example (Shift Register):**
    *   Using blocking assignments (`=`) and writing `q0 = d` before `q = q0` results in only one flip-flop (simulation mismatch).
    *   Using non-blocking (`<=`) creates two flip-flops correctly, regardless of statement order.

**C. Order of Blocking Assignments**
*   **Problem:** The evaluation order of blocking assignments can create a "simulated latch" that doesn't exist in hardware.
*   **Example:** If `y = q0 & c` is evaluated *before* `q0 = a | b` in an `always @(*)` block, `y` uses the *old* value of `q0`. This makes it seem like `q0` is registered. The synthesized circuit is purely combinational.
*   **Result:** Simulation and synthesis behavior differ.

#### **summary**
*   GLS is essential for verifying that the physical netlist matches the RTL design's intended behavior.
*   Proper coding styles (complete sensitivity lists, non-blocking assignments for sequential logic) are critical to avoid mismatches.


  Lab:
  This session covered gls and demonstrated a common synthesis-simulation mismatch scenario using a multiplexer example.
  Concepts

Ternary Operator in Verilog

? : is called the ternary operator
Syntax: condition ? true_expression : false_expression
Used for simple 2:1 MUX implementation: assign y = sel ? i1 : i0

### Lab Exercises

Exercise 1: Basic GLS Flow with Ternary MUX

RTL Simulation:
```
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

<img width="622" height="137" alt="Screenshot 2025-09-27 at 9 20 40 am" src="https://github.com/user-attachments/assets/7ae81afe-da6f-4364-8537-c8805143c6bf" />

Synthesis:

```
cd yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog ternary_operator_mux_net.v

```
<img width="300" height="250" alt="Screenshot 2025-09-27 at 6 09 12 pm" src="https://github.com/user-attachments/assets/b280e446-7d13-4f1d-9705-93e318e13ebe" />


<img width="300" height="250" alt="Screenshot 2025-09-27 at 5 48 54 pm" src="https://github.com/user-attachments/assets/cfcf6657-5cef-481d-813a-0ec03245a493" />


GLS

```
iverilog ./my_lib/verilog_models/primitives.v \
         ./my_lib/verilog_models/sky130_fd_sc_hd.v \
         ternary_operator_mux_net.v \
         tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
<img width="400" height="228" alt="Screenshot 2025-09-27 at 6 05 52 pm" src="https://github.com/user-attachments/assets/f93b991b-745b-4ee5-b0e5-70028247cc77" />



Exercise 2: Synthesis-Simulation Mismatch Example

Problematic Code (bad_mux.v):

RTL design Results

<img width="400" height="159" alt="Screenshot 2025-09-27 at 6 12 13 pm" src="https://github.com/user-attachments/assets/ac2289ba-e5ff-4642-8fc6-f302a25717a7" />

<img width="400" height="256" alt="Screenshot 2025-09-27 at 6 15 46 pm" src="https://github.com/user-attachments/assets/9f5f8a1a-e0c9-4930-84e0-b82f6522ce0d" />

Issues:

Incomplete sensitivity list: Only triggers on sel changes
RTL simulation: Behaves like a latch/flop (only updates when sel changes)
Synthesis: Infers combinational MUX (ignores sensitivity list)

Synthesis & GLS Results:

<img width="400" height="151" alt="Screenshot 2025-09-27 at 6 17 22 pm" src="https://github.com/user-attachments/assets/00939228-9101-4d1f-b6ad-11fe1668a825" />

The bad_mux example shows why GLS is mandatory - RTL simulation can show correct behavior while the actual synthesized circuit behaves differently due to incomplete sensitivity lists.

### Lab :Synthesis-Simulation Mismatch due to Blocking Statements
 This session demonstrated a critical synthesis-simulation mismatch caused by improper use of blocking statements in procedural assignments.
 
- RTL design
  
 ,,,

module blocking_caveat (input a , input b , input  c, output reg d); 
reg x;
always @ (*)
begin
	d = x & c;
	x = a | b;
end
endmodule
,,,


**Issue with Blocking Statements**

   - Execution order: Statements execute sequentially in simulation
   - Evaluation timing: D = X & C uses previous value of X (before update)
   - Simulation behavior: Creates false memory effect (appears like a flop)
   - Synthesis result: Pure combinational logic (no memory elements)

  **RTL Simulation**
  
```
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd

```

<img width="400" height="187" alt="Screenshot 2025-09-27 at 6 40 10 pm" src="https://github.com/user-attachments/assets/2bf7485d-02c2-422e-9ab1-12d8f925619b" />

Observations:

Output D appears to have memory effect
At point where A=0, B=0: D should be 0 but shows previous value
When A=1, C=1: D should be 1 but shows incorrect value
Behavior: Looks like X is flopped before AND operation

**Synthesis:**

<img width="400" height="200" alt="Screenshot 2025-09-27 at 6 43 57 pm" src="https://github.com/user-attachments/assets/77f6948d-0a16-4414-bcde-d8f291ff2915" />

Observation:

Pure combinational logic: OR gate + AND gate
No latches/flops inferred
Direct implementation: D = (A | B) & C

**GLS:**


<img width="400" height="145" alt="Screenshot 2025-09-27 at 6 45 38 pm" src="https://github.com/user-attachments/assets/31f9832f-8bda-46b5-a435-7216907f3826" />



Observation:

Correct combinational behavior
Output D responds immediately to input changes
No memory effect - matches intended logic (A+B)·C





Summary :


Blocking Statement Pitfall: Order matters in simulation but not in synthesis
GLS Importance: Essential for catching such subtle mismatches

Coding Best Practice:

Use non-blocking assignments for sequential logic
For combinational logic, ensure proper ordering or use non-blocking
Alternative: Use continuous assignments for pure combinational logic
Verification Strategy:

Always run GLS to validate RTL simulation results
Compare waveforms side-by-side for critical paths
Pay special attention to intermediate variable handling








