## OpenLANE Flow — `picorv32a` Design Synthesis & Analysis

---

### 1. Run Synthesis for `picorv32a` Design

### **Commands to Invoke OpenLANE Flow**

```bash
# Change directory to OpenLANE flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# Enter into Docker environment
docker

# Start OpenLANE in interactive mode
./flow.tcl -interactive

# Load the required OpenLANE package
package require openlane 0.9

# Prepare the design (creates necessary directories and configs)
prep -design picorv32a

# Run synthesis
run_synthesis

```

Screenshots of running each commands

| Step                           | Screenshot                                                                                                                    |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| Entering Docker / Running Flow | <img width="2235" alt="Screenshot 1" src="https://github.com/user-attachments/assets/76bdecb8-b09b-4ac4-b0f6-f1c5fc2d5832" /> |
| Loading Package                | <img width="2218" alt="Screenshot 2" src="https://github.com/user-attachments/assets/8f134101-63d9-4739-b5c7-e17c03418425" /> |
| Preparing Design               | <img width="2219" alt="Screenshot 3" src="https://github.com/user-attachments/assets/c7189597-d11c-42ee-ac11-379c6da56d7b" /> |
| Running Synthesis              | <img width="2225" alt="Screenshot 4" src="https://github.com/user-attachments/assets/20424b4d-8770-46af-a079-aa9a5a2958c8" /> |


<img width="2235" height="1373" alt="Screenshot 2025-11-11 155633" src="https://github.com/user-attachments/assets/76bdecb8-b09b-4ac4-b0f6-f1c5fc2d5832" />

<img width="2218" height="1223" alt="Screenshot 2025-11-11 155652" src="https://github.com/user-attachments/assets/8f134101-63d9-4739-b5c7-e17c03418425" />

<img width="2219" height="836" alt="Screenshot 2025-11-11 155739" src="https://github.com/user-attachments/assets/c7189597-d11c-42ee-ac11-379c6da56d7b" />

<img width="2225" height="1365" alt="Screenshot 2025-11-11 155812" src="https://github.com/user-attachments/assets/20424b4d-8770-46af-a079-aa9a5a2958c8" />

Calculate the flop ratio.

<img width="3603" height="1626" alt="Screenshot 2025-11-11 160744" src="https://github.com/user-attachments/assets/21d16103-a378-49b9-aaca-d68282deaf8d" />

Screenshots of synthesis statistics report file with required values 


<img width="3562" height="1845" alt="Screenshot 2025-11-11 160830" src="https://github.com/user-attachments/assets/5247e807-a0d7-4d88-85c6-a8a8d71a3f72" />



# Calculation of Flop Ratio and DFF Percentage from Synthesis Statistics Report

## Formula

Flop Ratio = (Number of DFFs) / (Total Number of Cells)

DFF Percentage = Flop Ratio × 100

---

## Given Data

| Parameter | Description | Value |
|------------|--------------|--------|
| Number of DFFs | Total count of D-type flip-flops | 1613 |
| Total Number of Cells | All synthesized cells in design | 14876 |

---

## Step-by-Step Calculation

1. Calculate the Flop Ratio:

   Flop Ratio = 1613 / 14876  
   Flop Ratio = 0.108429685

2. Calculate the DFF Percentage:

   DFF Percentage = 0.108429685 × 100  
   DFF Percentage = 10.84296854 %

---

## Final Results

| Metric | Value |
|---------|--------|
| **Flop Ratio** | 0.1084 |
| **DFF Percentage** | 10.84 % |

---


### 2. Good floorplan vs bad floorplan and introduction to library cells

1. Run 'picorv32a' design floorplan using OpenLANE flow and generate necessary outputs.
Commands to invoke the OpenLANE flow and perform floorplan

```
# Change directory to openlane flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'
# Since we have aliased the long command to 'docker' we can invoke the OpenLANE flow docker sub-system by just running this command
docker
# Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command
./flow.tcl -interactive

# Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
package require openlane 0.9

# Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
prep -design picorv32a

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Now we can run floorplan
run_floorplan
```

Screenshot of floorplan run

<img width="600" height="600" alt="Screenshot 2025-11-11 163419" src="https://github.com/user-attachments/assets/59d31d54-04fa-40b3-8e96-6b4536b7fee6" />

<img width="600" height="600" alt="Screenshot 2025-11-11 163500" src="https://github.com/user-attachments/assets/a6dacfe5-8f41-4298-8fda-56bce2964371" />
2. #### Floorplan DEF — Die Area Calculation


Screenshot of contents of floorplan def

<img width="600" height="600" alt="image" src="https://github.com/user-attachments/assets/2d8a4dc5-21f6-4e0a-ab20-9aad7d2ae8a4" />

- Calculate the die area in microns from the values in floorplan def.
Unit Distance = 1000  
(1000 units = 1 micron)

Die width in unit distance = 660685 − 0 = 660685  
Die height in unit distance = 671405 − 0 = 671405  

---

###### Conversion from Unit Distance to Microns

Distance in microns = (Value in Unit Distance) / 1000

Die width in microns = 660685 / 1000 = 660.685 microns  
Die height in microns = 671405 / 1000 = 671.405 microns  

---

## Die Area Calculation

Area of die in square microns = Die width × Die height  
= 660.685 × 671.405  
= 443587.212425 square microns  

---

##### Final Results

| Parameter | Value (in microns) |
|------------|--------------------|
| Die Width  | 660.685 µm |
| Die Height | 671.405 µm |
| **Die Area** | **443587.21 µm²** |

---

#### Load generated floorplan def in magic tool and explore the floorplan.

Commands to load floorplan def in magic in another terminal
```
# Change directory to path containing generated floorplan def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/floorplan/

# Command to load the floorplan def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

```

Screenshots of floorplan def in magic

<img width="2197" height="400" alt="Screenshot 2025-11-11 165318" src="https://github.com/user-attachments/assets/08c93777-48e3-46cb-8b62-f272be4c9bea" />

<img width="3642" height="1820" alt="Screenshot 2025-11-11 165246" src="https://github.com/user-attachments/assets/4b23aa5a-0b12-4772-8915-98bba6750c56" />



<img width="3633" height="1906" alt="Screenshot 2025-11-11 165633" src="https://github.com/user-attachments/assets/785b4cf9-952f-46b3-85b7-767ad56d88d9" />

<img width="2880" height="1560" alt="Screenshot 2025-11-11 165901" src="https://github.com/user-attachments/assets/7f3c1724-6941-45bd-b128-83d1e0c94436" />

<img width="2443" height="1540" alt="Screenshot 2025-11-11 170657" src="https://github.com/user-attachments/assets/8f25ad0c-c9a3-4834-b907-ff5afe11c191" />

<img width="2463" height="1704" alt="Screenshot 2025-11-11 170801" src="https://github.com/user-attachments/assets/87071c26-9c2b-4724-91c4-f4b36642b657" />



### 3.Run Placememt using openlane
Run 'picorv32a' design congestion aware placement using OpenLANE flow and generate necessary outputs.

Command to run placement
```
# Congestion aware placement by default
run_placement
```
Screenshots of placement run

<img width="1000" height="1000" alt="Screenshot 2025-11-12 122732" src="https://github.com/user-attachments/assets/448fa42f-6012-4336-8a9d-782664d31a9b" />


Load generated placement def in magic tool and explore the placement.
Commands to load placement def in magic in another terminal
```
# Change directory to path containing generated placement def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/placement/

# Command to load the placement def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```



<img width="2852" height="1737" alt="Screenshot 2025-11-12 123242" src="https://github.com/user-attachments/assets/f0f81eeb-9b09-48e3-8cc1-c3f9e53b0a8f" />


Screenshots of floorplan def in magic


<img width="2872" height="1570" alt="Screenshot 2025-11-12 123308" src="https://github.com/user-attachments/assets/8dd9a162-b3c4-4490-aa2c-aaad53c32a20" />

Standard cells legally placed
<img width="2875" height="1557" alt="Screenshot 2025-11-12 123453" src="https://github.com/user-attachments/assets/f8875285-672a-42bb-902a-2c94003cf264" />

Viewing placement PNG output
 

<img width="3010" height="1849" alt="Screenshot 2025-11-12 131447" src="https://github.com/user-attachments/assets/df183a75-08d8-478e-b721-8b15c10a347c" />

### 4. Run cts

```
command
run_cts

```
<img width="2998" height="1560" alt="Screenshot 2025-11-12 144806" src="https://github.com/user-attachments/assets/d9d1daa4-84a2-4881-b4d2-94828fd1df4a" />


### 5. run routing

```

command
run_routing
```
<img width="3001" height="403" alt="Screenshot 2025-11-12 145105" src="https://github.com/user-attachments/assets/d34d72f8-4346-4709-bfd8-c1539181e23e" />

<img width="3622" height="1861" alt="Screenshot 2025-11-12 145321" src="https://github.com/user-attachments/assets/9ac553c3-edda-4d7a-9321-31740dbb27d8" />




### DAY 3- Design library cell using Magic Layout and ngspice characterization


3.1. Clone custom inverter standard cell design from github repository

```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign

# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &

```

<img width="2529" height="1497" alt="Screenshot 2025-11-10 175816" src="https://github.com/user-attachments/assets/59cdd6a4-c694-4b92-8f6b-8ccaeb7e5821" />


Screenshot of commands run

3.2 Load the custom inverter layout in magic and explore.
Screenshot of custom inverter layout in magic

<img width="2529" height="1497" alt="Screenshot 2025-11-10 175816" src="https://github.com/user-attachments/assets/be8f0a61-c025-4219-9be5-22ecba94b65e" />




NMOS and PMOS identified

<img width="3107" height="1558" alt="Screenshot 2025-11-12 150617" src="https://github.com/user-attachments/assets/5f9bb8a4-cad7-48d9-ba30-5a8e9b46e5c7" />

<img width="3096" height="1567" alt="Screenshot 2025-11-12 150643" src="https://github.com/user-attachments/assets/d75cb98c-e8f8-43ad-a20c-60ce16a0351c" />




PMOS source connectivity to VDD (here VPWR) verified






Post-layout ngspice simulations
<img width="1990" height="1840" alt="Screenshot 2025-11-11 125857" src="https://github.com/user-attachments/assets/062af042-da09-4381-800b-4f1ab709c815" />

Commands for ngspice simulation

#### Command to directly load spice file for simulation to ngspice
ngspice sky130_inv.spice

#### Now that we have entered ngspice with the simulation spice file loaded we just have to load the plot
plot y vs time a
Screenshots of ngspice run and  generated plot::

<img width="2207" height="1893" alt="Screenshot 2025-11-11 130018" src="https://github.com/user-attachments/assets/81207dc2-c615-4837-b004-de0a3827aefa" />


<img width="3151" height="1709" alt="Screenshot 2025-11-11 130106" src="https://github.com/user-attachments/assets/a73f85e0-d173-4c14-8033-b5b8bf44fbb9" />


#### Rise Transition Time Calculation

Formula

Rise transition time = Time taken for output to rise to 80% − Time taken for output to rise to 20%
Rise Time = T80% − T20%

Reference Values (for VDD = 3.3V)

20% of output voltage:
0.20 × 3.3V = 660 mV

80% of output voltage:
0.80 × 3.3V = 2.64 V

The rise transition time is the difference in time between the output reaching 2.64 V and 660 mV during the rising edge of the signal.

output rising to 20%:

<img width="3285" height="1770" alt="Screenshot 2025-11-11 130835" src="https://github.com/user-attachments/assets/29041f1a-3252-47c8-874a-a9be630b7b8b" />




output rising to 80%:
<img width="3330" height="1776" alt="Screenshot 2025-11-11 130732" src="https://github.com/user-attachments/assets/d1a1e252-8b11-418a-988e-557f4fd3ff3b" />






terminal values:


<img width="1039" height="168" alt="Screenshot 2025-11-11 133238" src="https://github.com/user-attachments/assets/0ea34592-93dd-4096-84a2-e91fb988ff67" />



Rise Transition Time Calculation from ngspice

Time at 20% of Vout (0.66 V): t_20 = 2.1813 ns
Time at 80% of Vout (2.64 V): t_80 = 2.2388 ns
Formula:
Rise Time = t_80 − t_20
=  2.2388 ns − 2.1813ns
= 0.0575 ns
= 57.5 ps
✅ Rise Transition Time = 57.5 ps


##### Fall Transition Time Calculation

Fall transition time = Time taken for output to fall to 20% − Time taken for output to fall to 80%
Fall Time = T20% − T80%

Reference Values (for VDD = 3.3V)

20% of output voltage:
0.20 × 3.3V = 660 mV

80% of output voltage:
0.80 × 3.3V = 2.64 V

The fall transition time is the difference in time between the output falling from 2.64 V to 660 mV during the falling edge of the signal.

output falling to 20%:


<img width="1706" height="1649" alt="Screenshot 2025-11-11 141600" src="https://github.com/user-attachments/assets/0219c4f1-43e9-4cb8-beea-6cabc6633bbe" />




output falling to 80%:


<img width="1699" height="1794" alt="Screenshot 2025-11-11 142526" src="https://github.com/user-attachments/assets/d2c41860-3fb4-4a4e-97a1-f6c27d4cca5b" />



terminal values:



Fall Transition Time Calculation from ngspice

Time at 80% of Vout (2.64 V): t_80 = 4.0505 ns
Time at 20% of Vout (0.66 V): t_20 = 4.0932 ns
Formula:
Fall Time = t_20 − t_80
= 4.0932 ns − 4.0505 ns
= 0.0427 ns
= 42.7 ps
✅ Fall Transition Time = 42.7 ps


#### Rise Cell Delay Calculation

Rise Cell Delay is the time it takes for the output to reach 50% of VDD after the input begins transitioning.

Formula:
Rise Cell Delay = Time(output rises to 50%) − Time(input falls to 50%)

For VDD = 3.3V,

50% of VDD = 1.65V

output rising to 50% and input falling to 50%:

<img width="1740" height="1188" alt="Screenshot 2025-11-11 143452" src="https://github.com/user-attachments/assets/0e5db37a-e513-447d-a583-33bdc10c8eac" />

Rise Cell Delay = Time(output @ 50%) − Time(input @ 50%)
= 2.21177e-09 − 2.14989e-09
= 0.0619 s
= 61.9 ps

✅ Rise Cell Delay = 61.9 ps

#### Fall Cell Delay Calculation

Fall Cell Delay is the time it takes for the output to fall to 50% of VDD after the input begins transitioning.

Formula:
Fall Cell Delay = Time(output falls to 50%) − Time(input rises to 50%)

For VDD = 3.3V,

50% of VDD = 1.65V

output falling to 50% and input rising to 50%:

<img width="1746" height="1305" alt="Screenshot 2025-11-11 144123" src="https://github.com/user-attachments/assets/a6cd2456-5bec-41e9-90dd-fcb4f05d7827" />




Fall Cell Delay = Time(output @ 50%) − Time(input @ 50%)
= 4.07594e-09 − 4.05e-09
= 0.02594e-09 s
= 25.94 ps
✅ Fall Cell Delay = 25.94 ps




Find problem in the DRC section of the old magic tech file for the skywater process and fix them.
Link to Sky130 Periphery rules: https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html

Commands to download and view the corrupted skywater process magic tech file and associated files to perform drc corrections

```bash 
# Change to home directory
cd

# Command to download the lab files
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz

# Since lab file is compressed command to extract it
tar xfz drc_tests.tgz

# Change directory into the lab folder
cd drc_tests

# List all files and directories present in the current directory
ls -al

# Command to view .magicrc file
gvim .magicrc

# Command to open magic tool in better graphics
magic -d XR &
```
