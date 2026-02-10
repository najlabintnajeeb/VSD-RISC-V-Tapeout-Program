# Gate-Level Simulation (GLS) of BabySoC

## 📋 Table of Contents
- [Introduction](#introduction)
- [Synthesis Flow](#synthesis-flow)
- [Post-Synthesis Simulation](#post-synthesis-simulation)
- [Results](#results)
- [References](#references)

### Introduction

Gate-Level Simulation (GLS) is a crucial step performed after the synthesis process, bridging the gap between RTL functional verification and physical implementation. The goal is to verify the design using the actual netlist—a representation built entirely from standard library cells (like the sky130 standard cells).

### Purpose
GLS is performed to ensure:
- **Functional Equivalence**: The logic synthesized to gates still performs the exact same function as the original RTL code
- **Timing Validation**: When using Standard Delay Format (SDF) files, GLS verifies the design's behavior under real-world timing constraints

### Synthesis Flow (Yosys)

The gate-level netlist (`vsdbabysoc.synth.v`) was generated using the Yosys Open Synthesis Suite, mapping the design to the sky130 standard cell library.




#### Step 5: Map D Flip-Flops to Liberty Cells

command:

`` yosys> dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib ``

 Result: ✅ D flip-flops successfully mapped. All sequential elements were replaced with specific library cells 
 
<img width="400" height="200" alt="Screenshot 2025-10-01 at 12 49 39 pm" src="https://github.com/user-attachments/assets/8507ecf5-5ed4-41c3-accc-15691f41021a" />

#### Step 6: Technology Mapping with ABC

Command:

`` yosys> abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib ``

Result: ✅ Technology mapping completed. Combinational logic was mapped to the target standard cells using the ABC tool.

#### Step 7: Clean Up the Design

Command: ``yosys> opt_clean -purge ``

Result: ✅ Unused cells and wires removed successfully. The final netlist is optimized and ready for output.

<img width="400" height="200" alt="Screenshot 2025-10-01 at 12 55 20 pm" src="https://github.com/user-attachments/assets/1ad53b93-6cec-420b-96e2-22f2d49c28a5" />


#### Step 8: Generate Netlist

```bash
# Ensure the path matches the agreed-upon folder structure (e.g., /synthesized)
yosys> write_verilog -noattr ../VSDBabySoC/output/synthesized/vsdbabysoc.synth.v
```


✅ Netlist saved. The final gate-level netlist is available at ../VSDBabySoC/output/synthesized/vsdbabysoc.synth.v for physical design and post-synthesis simulation.

<img width="400" height="200" alt="Screenshot 2025-10-01 at 1 05 27 pm" src="https://github.com/user-attachments/assets/96d2375b-c1f0-4861-bab4-b069792a67f6" />

---

---

### 8: Post-Synthesis Simulation
The Post-Synthesis Simulation validates that the gate-level netlist generated from synthesis behaves functionally like the original RTL design. This is done using a testbench with the synthesized netlist.




#### step 1: Compile Gate-Level Netlist with Icarus Verilog
```bash
iverilog -o /home/najla/VSD/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/najla/VSD/VSDBabySoC/src/include -I /home/najla/VSD/VSDBabySoC/src/module /home/najla/VSD/VSDBabySoC/src/module/testbench.v

````

#### Step 3: Run Simulation

```base
cd output/post_synth_sim/
./post_synth_sim.out
```

<img width="400" height="100" alt="Screenshot 2025-10-03 at 12 04 02 pm" src="https://github.com/user-attachments/assets/f2cd51e3-f668-4538-99ff-105460d64d66" />

#### Step 4: View Waveforms in GTKWave

gtkwave post_synth_sim.vcd

<img width="1369" height="557" alt="Screenshot 2025-10-03 at 11 20 33 am" src="https://github.com/user-attachments/assets/b1ad224a-cb23-40c2-9155-c1a0bdbd18af" />

---

### 9. Simulation Comparison

The pre-synthesis (RTL) and post-synthesis (gate-level) simulations of VSDBabySoC exhibit matching behavior. CPU outputs, DAC responses, and clock signals observed in the RTL waveforms are preserved in the gate-level simulation, confirming that synthesis maintained functional correctness.

Below are the waveforms for visual comparison:

<img width="500" height="505" alt="Screenshot 2025-10-03 at 12 31 11 pm" src="https://github.com/user-attachments/assets/80b36710-8e25-4d10-8904-94ce7fa9cfa4" />



<img width="500" height="551" alt="Screenshot 2025-10-03 at 11 21 29 am" src="https://github.com/user-attachments/assets/dcd37d78-1c0d-4298-aa53-dd9c4f000903" />

#### Step 3: Finalization and Output

| Command | Purpose | Screenshot |
|---------|---------|------------|
| `flatten`<br>`setundef -zero`<br>`clean -purge`<br>`rename -enumerate` | Clean up, resolve undefined nets, and finalize the netlist structure | [Screenshot 8] |
| `stat` | Generate final cell and resource usage statistics | [Screenshot 9] |
| `write_verilog -noattr ...vsdbabysoc.synth.v` | Write the completed gate-level netlist to a file | [Screenshot 10] |

### 🔍 Post-Synthesis Simulation and Waveforms

The final netlist (`vsdbabysoc.synth.v`) is compiled and simulated with the existing testbench, using specific flags to activate the gate-level view.

#### Step 1: Compilation and Execution

```bash
# Compile the design for unit-delay functional GLS
iverilog -o .../post_synth_sim.out \
  -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
  -I .../src/include \
  -I .../src/module \
  .../src/module/testbench.v
```

```bash
# Navigate to the output directory and run the simulation
cd output/post_synth_sim/
./post_synth_sim.out
```

#### Step 2: Waveform Analysis (GTKWave)

```bash
gtkwave post_synth_sim.vcd
```

| Result | Screenshot |
|--------|------------|
| Waveforms showing correct functional behavior of the synthesized design | [Screenshot 11] |

### Results

#### Verification Outcomes

- ✅ **Functional equivalence** confirmed between RTL and gate-level netlist
- ✅ **All key modules** (RISC-V core, PLL, DAC) operating correctly
- ✅ **Reset sequences** and clock distribution validated
- ✅ **No critical timing violations** in unit-delay simulation

#### Key Statistics
- **Technology**: SkyWater 130nm
- **Synthesis Tool**: Yosys
- **Simulation Tool**: Icarus Verilog
- **Waveform Viewer**: GTKWave
- **Status**: GLS completed successfully

### Screenshots

Screenshots 1:


<img width="801" height="236" alt="Screenshot 2025-10-05 at 1 13 36 pm" src="https://github.com/user-attachments/assets/47eb0a14-c658-42a4-a37a-264cc71ac306" />

screeshot 2:
<img width="810" height="509" alt="2" src="https://github.com/user-attachments/assets/37e1a66f-2c4b-46fb-9dbf-d8ec4544e7c3" />

scrreensheet 3:
<img width="923" height="271" alt="3" src="https://github.com/user-attachments/assets/ec959bec-922e-476d-89e5-51c3641f9ed6" />


screenshot 4:

<img width="813" height="405" alt="4" src="https://github.com/user-attachments/assets/275f1a7a-8d0f-4117-9bd0-3196dc546f00" />

<img width="819" height="136" alt="4 1" src="https://github.com/user-attachments/assets/5c0e05ea-f2d9-49ab-b037-d016addd5345" />
<img width="824" height="210" alt="4 2" src="https://github.com/user-attachments/assets/7b0c011d-68ed-4db4-8473-97895e6cc1fc" />
<img width="816" height="448" alt="4 3" src="https://github.com/user-attachments/assets/70bec66f-5279-4051-94a4-509191aa4991" />
<img width="816" height="258" alt="4 4" src="https://github.com/user-attachments/assets/730ce75a-53d4-403c-bbeb-a2e3d57c8c4f" />

<img width="695" height="498" alt="4 6" src="https://github.com/user-attachments/assets/8850d5b3-30ad-420a-91e6-1db9ff79967e" />

screenshot 5:

<img width="926" height="649" alt="5" src="https://github.com/user-attachments/assets/005ec0f4-5449-4dd3-b8fb-6a5b67659bf1" />

screenshot 6 
<img width="640" height="695" alt="6" src="https://github.com/user-attachments/assets/febe04c2-37d4-40ea-9101-ac2b384700d3" />

screenshot 7

<img width="831" height="327" alt="7" src="https://github.com/user-attachments/assets/cc29cb9a-8a34-4223-8067-9b5424e328e0" />


<img width="806" height="143" alt="7 1" src="https://github.com/user-attachments/assets/5c3f0187-eb0f-49c6-b489-1bebd866c2a0" />

screeshot 8

<img width="731" height="322" alt="8" src="https://github.com/user-attachments/assets/31b5db0e-fb4f-4a00-843a-72e1e800d5c3" />

screenshot 9
<img width="620" height="619" alt="9" src="https://github.com/user-attachments/assets/32b8de60-6eb0-45a0-9a23-de5f06d16d68" />
<img width="566" height="543" alt="9 1" src="https://github.com/user-attachments/assets/baec13bd-49f1-4e42-be7e-0d2c82981a72" />

screenshot 10

<img width="630" height="134" alt="10" src="https://github.com/user-attachments/assets/d750f80d-c1cb-4ff1-824f-69f3a4bbd1f7" />

screenshot 11

<img width="843" height="238" alt="11" src="https://github.com/user-attachments/assets/ad2e83eb-fd7b-4890-ba8b-341a2d95f488" />




### References

- [Yosys Manual](https://yosyshq.readthedocs.io/)
- [Icarus Verilog Documentation](http://iverilog.icarus.com/)
- [GTKWave Documentation](http://gtkwave.sourceforge.net/)
- [SkyWater PDK](https://skywater-pdk.readthedocs.io/)

---
