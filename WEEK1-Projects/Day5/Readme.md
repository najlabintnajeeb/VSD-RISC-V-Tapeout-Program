## Day 5:

  This session covers the proper usage and common pitfalls of if-else statements , case statements ,for generate loos in Verilog, focusing on hardware inference and synthesis implications.

### If-Else Statements

```
if (condition1) begin
    // Code block 1
end else if (condition2) begin
    // Code block 2
end else begin
    // Default code
end
```

### Case Statements

```
case (select)
    2'b00: begin
        // Code for select=00
    end
    2'b01: begin
        // Code for select=01
    end
    default: begin
        // Default case
    end
endcase

```

1. Incomplete If-Else Statements

```
always @(*) begin
    if (condition1) y = a;
    else if (condition2) y = b;
    // Missing else case!
end

```

Issue: Creates inferred latches

When no conditions match, output y must retain previous value
Synthesis tool inserts latch to store previous state

```
// Sequential circuit - intended behavior
always @(posedge clk) begin
    if (reset) count <= 0;
    else if (enable) count <= count + 1;
    // No else: count retains value (correct for counter)
end

```
Rule: Incomplete if-else is acceptable for sequential logic but dangerous for combinational logic


2. Incomplete Case Statements

Problem Code:
```
case (select)
    2'b00: y = a;
    2'b01: y = b;
    // Missing cases for 2'b10, 2'b11!
endcase

```
Solution: Always use default case

```
case (select)
    2'b00: y = a;
    2'b01: y = b;
    default: y = c;  // Prevents inferred latches
endcase

```

3. Partial Assignments in Case Statements

Problem Code:

```
reg x, y;
case (select)
    2'b00: begin x = a; y = b; end
    2'b01: begin x = c; end  // y not assigned!
    default: begin x = d; y = b; end
endcase

```

Issue: Creates inferred latch for signal y in case 2'b01

Solution: Assign all outputs in all case segments


```
case (select)
    2'b00: begin x = a; y = b; end
    2'b01: begin x = c; y = b; end  // y explicitly assigned
    default: begin x = d; y = b; end
endcase

```

4. Overlapping Case Statements

Problem Code:

```
case (select)
    2'b00: y = a;
    2'b01: y = b;
    2'b1?: y = c;  // Overlaps with 2'b10 and 2'b11
endcase

```

summary:

For If-Else Statements:

Complete all branches for combinational logic
Order conditions by priority (highest priority first)
Use for priority-encoded logic intentionally
For Case Statements:

Always include default case
Assign all outputs in every case segment
Avoid overlapping cases unless intentional
Use for parallel selection (non-priority mux)

## Lab Exercises 
1) If-Else Priority Verification
2) incomplete if
3) Incomplete Case 
4)  Partial Assignment Test
5)  Overlapping Case Analysis

##### Lab Procedures & Tools

Standard Workflow for Each Example:

- RTL Simulation: iverilog design.v tb_design.v + GTKWave
- Synthesis: Yosys + ABC with Sky130 library
- GLS: iverilog netlist.v tb_design.v + primitives
- Comparison: Side-by-side waveform analysis

##### Verification Strategy:

- Always run GLS to catch mismatches
- Check synthesis reports for latch inferences
- Compare RTL vs GLS waveforms systematically
- Test corner cases where conditions don't match

## Looping Constructs

#### 1. For Loop (inside always)

- Used for evaluation of expressions, not for replicating hardware.
- Syntax is similar to C-style loops.
- Handy for writing large multiplexers/demultiplexers (e.g., 32:1 MUX, 256:1 MUX).
- 
Example:
MUX: Loop checks which input matches select and assigns to y.
DEMUX: Initialize outputs to 0, then assign input to one selected output bit.

**Executes sequentially inside always with blocking statements.**

#### 2. Generate with For (generate block)

- Used outside always.
- Purpose: replicate hardware instances (e.g., multiple gates, full adders).
- Requires genvar declaration.

Example:
Instantiating 8 AND gates with one loop instead of writing 8 instances manually.
Ripple Carry Adder: replicate full adders for N-bit addition.

#### 3. If-Generate
- Conditional hardware generation.
- If condition met → instantiate hardware; else → skip.
- Also used outside always.

#### 2) Example
**Multiplexer:**

Writing large case statements becomes unreadable for big multiplexers

using for loop:

```
always @* begin
  integer i;
  for (i = 0; i < N; i = i+1) begin
    if (i == select)
      y = in[i];
  end
end

```
Simple and scalable: changing N adjusts the mux size.

**Demultiplexer**

1-to-8 DMUX using for loop:

```
always @* begin
  out_bus = 8'b0;  // initialize all outputs to 0
  integer i;
  for (i = 0; i < 8; i = i+1) begin
    if (i == select)
      out_bus[i] = in;  // one input routed to selected output
  end
end

```

Initialization ensures unused outputs remain 0.
Blocking statements make logic clean and predictable.


---

### Lab : mux_generate.v.

- simulation

<img width="400" height="214" alt="Screenshot 2025-09-27 at 8 34 21 pm" src="https://github.com/user-attachments/assets/964edb5f-b2aa-4f8a-a2c2-9b2fa1e4c770" />


-synthesis

<img width="400" height="200" alt="Screenshot 2025-09-27 at 8 39 18 pm" src="https://github.com/user-attachments/assets/d2db884b-3a80-4182-b009-f3f8b9f2965a" />


-GLS

<img width="400" height="194" alt="Screenshot 2025-09-27 at 8 41 29 pm" src="https://github.com/user-attachments/assets/9ec0727c-585c-4444-8b43-322cf6319ec5" />



### Lab : dmux_case.v

- simulation


<img width="400" height="174" alt="Screenshot 2025-09-27 at 8 48 46 pm" src="https://github.com/user-attachments/assets/d3352c2b-588c-470e-9981-c665aa0eed24" />



-synthesis


<img width="498" height="472" alt="Screenshot 2025-09-27 at 8 50 21 pm" src="https://github.com/user-attachments/assets/1a31f177-2d8b-40a8-a5cc-c7a4b51e89c2" />



-GLS
<img width="1080" height="274" alt="Screenshot 2025-09-27 at 8 53 27 pm" src="https://github.com/user-attachments/assets/bac32e68-304c-442c-b660-5a04256d04a3" />

Observations
RTL case statement → synthesizes to a decoder + AND gates.
Each output = i AND (select matches).
No latches inferred since outputs are initialized in RTL.
The gate-level representation matches expected demux functionality.



### Lab : demux_generate.v

- simultaion
  
<img width="400" height="195" alt="Screenshot 2025-09-27 at 9 00 22 pm" src="https://github.com/user-attachments/assets/62443934-ac78-486d-9c76-be8148adbf71" />


-synthesis

<img width="400" height="472" alt="Screenshot 2025-09-27 at 9 03 41 pm" src="https://github.com/user-attachments/assets/61e2d477-50c8-49ec-815a-c8a79c45ce4b" />

- GLS

<img width="400" height="309" alt="Screenshot 2025-09-27 at 9 18 04 pm" src="https://github.com/user-attachments/assets/cc93a6e8-1611-4858-8fd6-ec3853f0f265" />

Observations: the RTL code and netlist are functionally matching perfectly for the demux_generate.v
  

 ### Lab : Ripple Carry Adder - rca.v

- simulation

 <img width="400" height="157" alt="Screenshot 2025-09-27 at 9 21 17 pm" src="https://github.com/user-attachments/assets/ccb72656-b0f2-4222-a9ec-d173601a8464" />

  

- synthesis
  
<img width="500" height="511" alt="Screenshot 2025-09-27 at 9 25 39 pm" src="https://github.com/user-attachments/assets/5f58ae8e-34b9-4996-970d-d9afd7e1432c" />


- GLS

<img width="400" height="189" alt="Screenshot 2025-09-27 at 9 29 29 pm" src="https://github.com/user-attachments/assets/06554ae1-9c48-4291-a221-d2955bc67aaa" />

- Observations:

  The RTL and GLS matches.
  Generate-for loop expands at elaboration → identical to writing 7 manual FA instantiations. 

  


