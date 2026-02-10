# VSDBabySoC Project Documentation


## Table of Contents
1. [Project Overview](#1-Project-Overview)
2. [Cloning Git Repository](#2-cloning-git-repositories)
3. [Project Structure & Required Files](#3-project-structure--required-files)
4. [TLV to Verilog Conversion](#4-tlv-to-verilog-conversion)
5. [Module Descriptions](#5-module-descriptions)
6. [Pre-Synthesis Simulation](#6-pre-synthesis-simulation)
7. [Synthesis](#7-synthesis)
8. [Post-Synthesis Simulation](#8-post-synthesis-simulation)
9. [Simulation Comparison](#9-simulation-comparison)
10. [Errors & Solutions](#10-errors-challenges--solutions)
11. [Troubleshooting](#11-General-Troubleshooting)
12. [References](#12-references)

---

### 1. Project Overview

**VSDBabySoC** is a compact, open-source **System-on-Chip (SoC)** based on the **RVMYTH RISC-V processor core**.  

#### Integrated Modules
- **Phase-Locked Loop (PLL):** Provides precise clock generation for synchronous operation.  
- **10-bit Digital-to-Analog Converter (DAC):** Interfaces with analog systems.  
- **Testbenches:** Supports pre-synthesis, post-synthesis, and gate-level simulations (GLS).  

#### Objective
The main objective of this project is to run **pre-synthesis**, **RTL synthesis (Yosys)**, and **post-synthesis simulations** to verify SoC functionality and validate all IP cores.

---

---

### 2. Cloning Git Repositories

-  Main Repository
The primary repository contains all main RTL files, modules, and testbenches.

```bash
git clone https://github.com/manili/VSDBabySoC.git
```
-   Supporting Repositories

Refer to the References section for links to supporting repositories.

---

### 3. Project Structure & Required Files

The expected project structure for **VSDBabySoC** is as follows:

```bash

VSDBabySoC/
├── src/
│   ├── module/
│   ├── include/
│   ├── sdc/
│   └── gls_model/
├── output/
│   ├── pre_synth_sim/
│   ├── post_synth_sim/
│   └── synthesized/
├── images/
└── README.md

```

##### Verify Project Tree

After cloning the repository, you can check the project folder structure using the `tree` command in the terminal:

```bash
tree VSDBabySoC

```

<img width="500" height="300" alt="Screenshot 2025-10-02 at 5 12 38 pm" src="https://github.com/user-attachments/assets/8d7a5711-fe40-4fe4-aaac-026b463e246a" />

⚠️ Note: The output folders will not be present initially and must be created manually.

`` mkdir -p output/pre_synth_sim output/post_synth_sim output/synthesized ``


**Required Source Files**

Make sure these files are present in your project:

**Modules (src/module):**
`` vsdbabysoc.v, rvmyth.tlv, avsddac.v, avsdpll.v``
``clk_gate.v, pseudo_rand.sv, pseudo_rand_gen.sv``

**Headers (src/include):**
``sp_verilog.vh, sp_default.vh, sandpiper.vh, sandpiper_gen.vh``

⚠️ Note: Some files may need to be copied from supporting repositories.
⚠️ Output files will only be generated after running pre-synthesis and post-synthesis simulations.

---

---

### 4. TLV to Verilog Conversion

Some modules, like `rvmyth.tlv`, require conversion into Verilog before synthesis or simulation.  This is done using **SandPiper**.


```
step 1: Install dependencies
sudo apt update
sudo apt install python3-pip python3-venv -y

#Step 2: Create a Python virtual environment

cd ~/VLSI/VSDBabySoC
python3 -m venv my_env
source my_env/bin/activate

#Step 3: Install SandPiper

pip install pyyaml click sandpiper-saas


#Step 4: Convert TLV to Verilog
cd ~/VLSI/VSDBabySoC/src/module
sandpiper-saas -i rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir .

#Step 5: Exit Virtual Environment
deactivate
```

✅ After this step, the rvmyth.v file will be generated in the src/module/ directory.
This file can now be used for pre-synthesis simulation, synthesis, and post-synthesis simulation.

---

---

### 5. Module Descriptions

1. **RVMYTH Core (rvmyth)**
   
-  Inputs: CLK, reset
-  Outputs: OUT (10-bit digital, assigned to RV_TO_DAC)
-  Behavior: Executes instructions, updates registers, generates CPU output.
-  Key points:
- - Instruction memory, register file, and data memory are included internally.
- - OUT is mapped to RVMYTH register #17 (via CPU_Xreg_value_a5[17]).
- - Output is connected to DAC input in top-level module.

2. **DAC Module (avsddac)**
   
- Inputs: D (10-bit), VREFH, VREFL
- Output: OUT (real, analog-like)
- Behavior:
- - Converts 10-bit digital D to analog-like OUT using formula:
- - OUT <= VREFL + ($itor(Dext)/1023.0)*(VREFH - VREFL)

**Reference Voltages (VREFH and VREFL):**
- **VREFH**: Maximum voltage level the DAC can output
- **VREFL**: Minimum voltage level the DAC can output  
- These set the scaling factor for mapping digital values to analog voltage

**Example Scaling:**
If VREFL = 0.0V and VREFH = 3.3V:
- Digital 0 → 0.0V
- Digital 1023 (max 10-bit) → 3.3V  
- Digital 512 → ~1.65V

4. **PLL Module (avsdpll)**
- Inputs: VCO_IN, ENb_CP, ENb_VCO, REF
- Output: CLK
- Behavior:
- - Generates clock signal for CPU (rvmyth) depending on ENb_VCO.
- - Clock period updated on each reference edge: period = refpd / 8.0.
- Observation: When ENb_VCO = 1, clock toggles; else CLK = 0.

5. **Clock Gate Module (clk_gate)**
- Inputs: free_clk, func_en, pwr_en, gating_override
- Output: gated_clk
- Behavior: Currently, gated_clk = free_clk.

---

---

### 6. Pre-Synthesis Simulation
In this phase, the RTL design is verified at a higher level using Icarus Verilog. The goal is to confirm that the SoC behaves functionally correct before synthesis.

#### 🔄 Simulation Flow Overview – VSDBabySoC

<details> <summary>Simulation Flow Details</summary>

The **Pre-Synthesis (RTL) simulation** of VSDBabySoC follows this sequence:

1. **Testbench Initialization**
   - Applies `reset = 0` initially.
   - Toggles REF, VCO, and ENb signals.
   - Sets DAC reference voltages: `VREFH = 3.3V`, `VREFL = 0.0V`.
   - Provides clock-related toggling indirectly via the PLL.

2. **PLL Module (avsdpll)**
   - Receives REF from the testbench.
   - Generates internal `CLK` signal based on REF and `ENb_VCO`.
   - **Flow:** `REF → PLL → CLK`.

3. **CPU Core (rvmyth)**
   - Receives `CLK` and `reset` from PLL/testbench.
   - Executes RISC-V instructions on every clock cycle.
   - Updates internal registers and produces `RV_TO_DAC[9:0]`.
   - **Flow:** `CLK, reset → RVMYTH → RV_TO_DAC`.

4. **DAC Module (avsddac)**
   - Receives `RV_TO_DAC[9:0]` and reference voltages (`VREFH`, `VREFL`).
   - Converts digital input into an analog-like output (`OUT`).
   - **Flow:** `RV_TO_DAC + VREFH/VREFL → DAC → OUT`.

5. **Top-Level SoC (vsdbabysoc)**
   - Wires PLL, CPU, DAC, and `clk_gate` together.
   - The SoC output (`OUT`) is the DAC’s analog result.
   - **Flow inside SoC:** `REF → PLL → CLK → CPU → RV_TO_DAC → DAC → OUT`.

6. **Testbench Monitoring**
   - Dumps simulation waveform (`pre_synth_sim.vcd` or `post_synth_sim.vcd`).
   - Observes signals: `CLK`, `reset`, `RV_TO_DAC`, `OUT`.
   - Ends simulation after ~600 cycles.

</details>

### Simulation Steps: 

<details><summary>Steps Followed for Pre-Synthesis Simulation</summary>
   
#### Step 1: Compile with Icarus Verilog

This command compiles the testbench and all associated RTL modules into a single executable.

```bash
iverilog -o output/pre_synth_sim/pre_synth_sim.out \
  -DPRE_SYNTH_SIM \
  -I src/include -I src/module \
  src/module/testbench.v
```

📌 Why RTL files aren't listed separately (vsdbabysoc.v, rvmyth.v, etc.):
The testbench contains conditional include statements that automatically pull in all RTL files when compiled with -DPRE_SYNTH_SIM.

<img width="300" height="102" alt="Screenshot 2025-10-01 at 4 25 27 pm" src="https://github.com/user-attachments/assets/433243b2-0506-42be-bdda-9567caa2f57f" />


⚠️ ⚠️ For guidance on resolving compilation issues, refer to the Errors Section (specifically  [Error Case 1: Duplicate Module Declaration](#error-case-1-duplicate-module-declaration-in-pre-synth-simulation) , [Error Case 2: Missing Include File](#error-case-2-missing-include-file) )


#### Step 2 : Run Pre-Synthesis Simulation
Execute the compiled file to run the simulation and generate the VCD file.

```bash
cd output/pre_synth_sim/
./pre_synth_sim.out

```

<img width="400" height="82" alt="Screenshot 2025-10-01 at 4 58 49 pm" src="https://github.com/user-attachments/assets/ff11e38f-be2b-4703-a45a-2075e3753f19" />


### Step 3: Generate and View Waveform

Load the generated VCD file into GTKWave for visual analysis.

```verilog
gtkwave output/pre_synth_sim.vcd
```

----
</details>

#### Key Waveform Observations

<details><summary>Observations Based on GTKWave</summary>

| Observation              | Detail |
|---------------------------|--------|
| **Clock Generation (PLL)** | The `CLK` signal is generated by the PLL (`avsdpll`) and starts toggling immediately upon power-up/initialization. This stable clock provides the timing reference necessary to synchronize the CPU's operations. |
| **System Initialization** | The `reset` signal asserts high initially, holding the `RVMYTH` core and synchronous logic inactive. Upon de-assertion (going low at ∼150 ns), the clock-driven operation of the CPU begins. |
| **Digital Output (RVMYTH)** | The 10-bit digital input to the DAC, `RV_TO_DAC[9:0]`, changes in discrete steps (e.g., `000 → 001 → 003`). This validates that the `RVMYTH` core is executing its programmed instructions and streaming data to the periphery. |
| **Analog Conversion (AVSDDAC)** | The analog output signal (`OUT`) responds to these digital steps with corresponding, monotonically increasing voltage levels. This confirms the DAC is functioning correctly by translating the digital code into the desired analog output. <img width="400" height="380" alt="Screenshot 2025-10-02 at 2 21 45 pm" src="https://github.com/user-attachments/assets/05444670-266c-46db-b629-02c30539f12c" />
 |


---

Digital Output Waveform

<img width="500" height="250" alt="Screenshot 2025-10-02 at 2 17 53 pm" src="https://github.com/user-attachments/assets/c5429ee7-a9a7-4eb8-939e-0e1b50fd40d2" />

Analog Output Waveform (DAC Output)

<img width="500" height="250" alt="Screenshot 2025-10-02 at 2 23 03 pm" src="https://github.com/user-attachments/assets/09c3fda4-d859-4c4e-8677-f9b6c8ae8608" />

---

##### Notes
- Clock toggles correctly, synchronizing RVMYTH core operations.  
- `RV_TO_DAC` represents the RVMYTH core → DAC interface; tracking it ensures correct data handoff.  
- DAC output (`OUT`) can be visualized in GTKWave using **Data Format → Analog → Step** to see analog-like behavior.  
- DAC output follows RVMYTH core output based on reference voltages.  
- Reset initializes RVMYTH core and memory as expected
- In the testbench, you can change ENb_VCO value: setting ENb_VCO = 0 will stop the clock (CLK = 0), setting ENb_VCO = 1 will start toggling. This allows verification of PLL-controlled clock generation.

</details>

✅ Conclusion:
- The simulation validates the successful interaction between all major blocks: PLL, RVMYTH core, and DAC.
- This confirms that the SoC’s RTL implementation is functionally correct and forms the baseline for synthesis verification.


---

---

### 7. Synthesis
The RTL is synthesized using Yosys to produce a gate-level netlist. This step ensures the design is mapped to Sky130 standard cells for implementation.
 
#### Step 1: Start Yosys

Launches the synthesis tool shell.
Command:
```
cd ~/VLSI/VSDBabySoC
yosys
```

Result: ✅ Yosys shell started successfully.

<img width="400" height="171" alt="Screenshot 2025-10-01 at 12 14 47 pm" src="https://github.com/user-attachments/assets/7647fbcb-cbe0-4d52-ad3c-ac504705bab5" />


#### Step 2: Read Verilog Files

Loads top-level and supporting RTL modules into Yosys for synthesis.

Command:
```
yosys> read_verilog -I ../VSDBabySoC/src/include 
             ../VSDBabySoC/src/module/vsdbabysoc.v 
             ../VSDBabySoC/src/module/rvmyth.v 
             ../VSDBabySoC/src/module/clk_gate.v

```
Result: ✅ Successfully loaded modules: vsdbabysoc.v, rvmyth.v, clk_gate.v.

<img width="400" height="400" alt="Loaded Verilog Modules" src="https://github.com/user-attachments/assets/983167e4-920e-4c79-b729-682485258ad3" />

 ⚠️ Note: modules (avsddac.v, avsdpll.v) were intentionally omitted due to unsupported datatypes. See [Error Case 3: Unsupported real Datatype](#error-case-3-unsupported-real-datatype)
- Missing include file (`sp_verilog.vh`) – see [Error Case 2: Missing Include File](#error-case-2-missing-include-file)



#### Step 3: Load Liberty Files

Commands:

```
yosys> read_liberty -lib ../VSDBabySoC/src/lib/avsddac.lib
yosys> read_liberty -lib ../VSDBabySoC/src/lib/avsdpll.lib
yosys> read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

```
Result: ✅ Imported all required Liberty files. The analog blocks are now defined as black boxes, and the standard cell library is loaded.

<img width="400" height="200" alt="Screenshot 2025-10-01 at 12 17 23 pm" src="https://github.com/user-attachments/assets/75580281-77fd-41b7-91a1-58a453da71a8" />

⚠️ Note: Loading these libraries resolves dependency issues created by skipping the Verilog files in Step 2. [Error Case 4: Module Not Part of Design](#error-case-4-module-not-part-of-design)


#### Step 4: Synthesize Top-Level Module

Command:

```
yosys> synth -top vsdbabysoc
```

Result: ✅ Synthesis executed successfully for available modules.

<img width="400" height="200" alt="Screenshot 2025-10-01 at 12 20 55 pm" src="https://github.com/user-attachments/assets/04ff0f66-f677-41c1-a514-00884ee6cb21" />

[View synthesis statistics](#Synth-Print-Statistics)



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

- ``-noattr`` option removes Yosys-specific attributes from the netlist.

✅ Netlist saved. The final gate-level netlist is available at ../VSDBabySoC/output/synthesized/vsdbabysoc.synth.v for physical design and post-synthesis simulation.

<img width="400" height="200" alt="Screenshot 2025-10-01 at 1 05 27 pm" src="https://github.com/user-attachments/assets/96d2375b-c1f0-4861-bab4-b069792a67f6" />

---

---

### 8: Post-Synthesis Simulation
The Post-Synthesis Simulation validates that the gate-level netlist generated from synthesis behaves functionally like the original RTL design. This is done using a testbench with the synthesized netlist.

#### Step 1: Prepare the Simulation Environment
1.1. Copy the Synthesized Netlist to the Module Directory

`` cp output/synth/vsdbabysoc.synth.v src/module/``


1.2. Copy Standard Cell Models for Simulation

```
cp ../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v src/module/
cp ../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v src/module/

```

- Copies the standard cell Verilog models (primitives.v and sky130_fd_sc_hd.v) into the project’s module folder.
- These models are required for simulation of the synthesized netlist, providing definitions of the library cells.
- ⚠️ Missing these files will cause simulation errors. [Error Case 5: Missing Standard Cell Models](#error-case-5-missing-standard-cell-models)

#### step 2: Compile Gate-Level Netlist with Icarus Verilog
```bash
iverilog -o /home/najla/VSD/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/najla/VSD/VSDBabySoC/src/include -I /home/najla/VSD/VSDBabySoC/src/module /home/najla/VSD/VSDBabySoC/src/module/testbench.v

````

**Notes:**
-DFUNCTIONAL activates functional models of standard cells, required to see signals in GTKWave.
-DUNIT_DELAY=#1 sets gate delay (optional for functional correctness).
Omitting FUNCTIONAL compiles successfully but results in no waveform output.

Related Errors / References:
[Error Case 6 - No Waveform](#error-case-6-post-synthesis-simulation--no-waveform)

[Error Case 7 - Syntax Error](#error-case-7-syntax-error-in-sky130_fd_sc_hd-v)

[Error Case 8 - UNIT_DELAY Warnings](#error-case-8-unit_delay-warnings)


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


### 10. Errors, Challenges & Solutions

#### Error Case 1: Duplicate Module Declaration in Pre-Synth Simulation  

**Incorrect Command:**  
```bash
iverilog -o output/pre_synth_sim/pre_synth_sim.out \
    -DPRE_SYNTH_SIM \
    -I src/include -I src/module \
    src/module/testbench.v src/module/vsdbabysoc.v
```

🛑 Error:  we get the following error as in screenshot

<img width="400" height="148" alt="Screenshot 2025-10-01 at 4 44 47 pm" src="https://github.com/user-attachments/assets/6774cd77-e3c5-453c-8d2d-0786c39be75e" />

⚠️ Cause:
File vsdbabysoc.v was explicitly passed in the command, but the testbench already included it via:
`` `include "vsdbabysoc.v" `` , So the module was compiled twice.

<img width="300" height="102" alt="Screenshot 2025-10-01 at 4 25 27 pm" src="https://github.com/user-attachments/assets/433243b2-0506-42be-bdda-9567caa2f57f" />

✅ Fix: Remove vsdbabysoc.v from the command.Run only below commands
```bash
iverilog -o output/pre_synth_sim/pre_synth_sim.out \
    -DPRE_SYNTH_SIM \
    -I src/include -I src/module \
    src/module/testbench.v
```

#### Error Case 2: Missing Include File

🛑 Error: ``ERROR: Can't open include file `sp_verilog.vh /*_\SV */'!``

<img width="400" height="200" alt="Missing Include File Error" src="https://github.com/user-attachments/assets/065eee03-37a8-4dbe-ae7b-2d7d85f47b24" />

⚠️ Cause: The file rvmyth.v depends on sp_verilog.vh header, and Yosys couldn’t locate it.

✅ Fix: Use -I ../VSDBabySoC/src/include in read_verilog.



#### Error Case 3: Unsupported real Datatype

🛑 Error: ``../VSDBabySoC/src/module/avsddac.v:14: ERROR: syntax error, unexpected TOK_REAL``

⚠️ Cause: Yosys Verilog-2005 frontend does not support real datatype.

Screenshot:

<img width="400" height="200" alt="Unsupported TOK_REAL Error" src="https://github.com/user-attachments/assets/20e6a5ac-af83-47dc-b9c9-42d0b924f921" />
✅ Fix: Skip these modules initially and use Liberty files instead.


#### Error Case 4: Module Not Part of Design 

🛑 Error: ``ERROR: Module \avsddac referenced in module \vsdbabysoc in cell \dac is not part of the design.``

⚠️ Cause: Missing module due to skipped Verilog file (avsddac.v).


Screenshot:

<img width="572" height="193" alt="Module Not Part of Design Error" src="https://github.com/user-attachments/assets/db691a15-dab3-46ab-b186-3015e0338469" />

✅ Fix: Load avsddac and avsdpll via Liberty files.

#### Error Case 5: Missing Standard Cell Models


🛑 Error: Missing standard cell models required for synthesis.

<img width="400" height="200" alt="Screenshot 2025-09-30 at 4 04 02 pm" src="https://github.com/user-attachments/assets/99e89046-58e0-4e18-8a55-b6885aeab5f7" />

⚠️ Cause: Library or model files not loaded in the tool.

✅ Fix: Ensure all required standard cell and primitive files are present in the same directory as the testbench (src/module). You can copy them using:

```
# Copy standard cell model
cp sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v src/module/

# Copy primitive models
cp sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v src/module/
```

#### Error Case 6: Post-Synthesis Simulation – No Waveform
🛑 Issue: Simulation executes without error, VCD file is created, but GTKWave shows no signal toggles.

<img width="400" height="250" alt="Screenshot 2025-10-03 at 12 12 48 pm" src="https://github.com/user-attachments/assets/1930e08b-a2de-4bac-9cfd-d5372769d262" />

<img width="400" height="250" alt="Screenshot 2025-10-03 at 11 50 44 am" src="https://github.com/user-attachments/assets/e8fa14fe-714b-4ff6-a920-032ecb997974" />

⚠️ Cause:
- Gate-level netlist uses .behavioral.pp.v models by default, leaving several signals unconnected.
- Without -DFUNCTIONAL define, GTKWave cannot show meaningful waveforms.

✅ Fix: Compile with FUNCTIONAL and UNIT_DELAY defined:

```bash
iverilog -o output/post_synth_sim/post_synth_sim.out \
-DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
-I src/include -I src/module \
src/module/testbench.v

```
#### error-case-7-syntax-error-in-sky130_fd_sc_hd-v

🛑 Error at line 74452:

<img width="1365" height="140" alt="Screenshot 2025-10-03 at 11 17 01 am" src="https://github.com/user-attachments/assets/a3612815-7e91-4fa8-afff-eb1cf329cfe4" />

✅ Fix:
- change
  `` `endif SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V ``
  
- To
  
`` `endif //SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V ``

#### Error Case 8: UNIT_DELAY Warnings
🛑 Symptom / Warning:
When running post-synthesis simulation without defining UNIT_DELAY, you see multiple warnings like:
<img width="1334" height="621" alt="Screenshot 2025-10-03 at 11 16 00 am" src="https://github.com/user-attachments/assets/c7b75cbd-78c1-4070-8936-c7c96beb601c" />

✅ Fix: Define UNIT_DELAY during compilation. For example:

```
iverilog -o output/post_synth_sim/post_synth_sim.out \
  -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
  -I src/include -I src/module \
  src/module/testbench.v

```

---


##### Synth Print Statistics

<img width="400" height="400" alt="Screenshot 2025-10-01 at 12 22 56 pm" src="https://github.com/user-attachments/assets/8f55a3f8-2c96-4518-b72f-7a8572ccb106" />


<img width="400" height="500" alt="Screenshot 2025-10-01 at 12 23 07 pm" src="https://github.com/user-attachments/assets/6e8cf524-d727-43b6-9037-45481a3c727a" />



### 11. General Troubleshooting

Here is a generalized table of common issues encountered during the setup and execution of VLSI (Verilog/Yosys) projects, along with their solutions.

| Issue | Cause/Context | Solution |
| :--- | :--- | :--- |
| **File Not Found** (`No such file or directory`) | The tool (e.g., `iverilog`, `yosys`) cannot locate a source file, testbench, or library. | **Verify Path:** Ensure the tool is run from the correct directory, or use the **full relative/absolute path** to the missing file. |
| **Source File Not Found** (e.g., `rvmyth.v`) | The required source file exists in a non-standard format (like TLV) or was expected to be generated during setup. | **Generate Source:** Convert the file from its source format (e.g., **TLV to Verilog** using SandPiper) or check the file copying process. |
| **Synthesized File Not Found** (e.g., `vsdbabysoc.synth.v`) | A post-synthesis tool (like a Gate-Level Simulation testbench) is looking for a netlist or cell view that hasn't been generated yet. | **Check Build Flow:** Ensure the synthesis step has successfully run and placed the generated netlist in the expected output directory (e.g., `output/synthesized/`). |
| **Command Not Found** (e.g., `sandpiper-saas: command not found`) | The necessary utility or tool is not in the system's PATH. | **Install/Activate Environment:** Install the tool (e.g., using `pip` or `apt`), or **activate the relevant Python virtual environment** (`source my_env/bin/activate`) where the tool was installed. |
| **No Top Level Modules/Design Unit** (Synthesis or Simulation) | The tool requires a primary entry point to start its process. | **Specify Top Module:** Use the tool's option (e.g., `yosys -s <top_module>`, `synth -top <module>`) or ensure the testbench/RTL file contains a correctly defined, non-empty module. |


### 12. References**

* [VSDBabySoC main repo](https://github.com/manili/VSDBabySoC.git)
* [RVMyth core](https://github.com/kunalg123/rvmyth)
* [RVMyth-AVSDDAC interface](https://github.com/vsdip/rvmyth_avsddac_interface.git)
* [AVS PLL](https://github.com/ireneann713/PLL.git)
* https://github.com/google/skywater-pdk/issues/310

---
