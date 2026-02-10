## OpenSTA for timing analysis
## Static Timing Analysis with OpenSTA

This section covers the setup and execution of Static Timing Analysis (STA) using OpenSTA to verify timing closure for the VSDBabySoC design across multiple Process-Voltage-Temperature (PVT) corners.

### Installation of OpenSTA

#### Step 1: Clone the Repository
To download and enter the OpenSTA source directory, run:
``` bash
git clone https://github.com/parallaxsw/OpenSTA.git
cd OpenSTA

```
#### Step 2: Build the Docker Image
Build the OpenSTA Docker image using the provided Ubuntu 22.04 Dockerfile:

``` bash
docker build --file Dockerfile.ubuntu22.04 --tag opensta
```
This command creates a Docker image named **opensta**, installing all required dependencies during the build process.


#### Step 3: Run the OpenSTA Container
To run OpenSTA inside a Docker container, use the -v option to mount your local directory (so OpenSTA can access your files) and -i to run it interactively:

``` bash
docker run -i -v $(pwd):/data opensta
```

Here:
- -v $(pwd):/data mounts your current working directory to /data inside the container.
- -i enables interactive mode, allowing you to execute OpenSTA commands directly.

  After a successful setup, you’ll see the % prompt — this indicates that the OpenSTA interactive shell is active and ready for use.
  
 ### Performing Timing Analysis Using Inline Commands
Once inside the OpenSTA shell (``%'' prompt), you can perform a basic Static Timing Analysis (STA) using the following inline commands:

``` tcl
# Load Liberty files (min/max) for standard cells and analog IPs
read_liberty -min /data/examples/timing_libs/timing/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max /data/examples/timing_libs/timing/sky130_fd_sc_hd__tt_025C_1v80.lib

read_liberty -min /data/examples/timing_libs/timing/avsdpll.lib
read_liberty -max /data/examples/timing_libs/timing/avsdpll.lib

read_liberty -min /data/examples/timing_libs/timing/avsddac.lib
read_liberty -max /data/examples/timing_libs/timing/avsddac.lib

# Read synthesized Verilog netlist
read_verilog /data/examples/BabySoC/vsdbabysoc.synth.v

# Link the design and set it as current
link_design vsdbabysoc

# Apply timing constraints
read_sdc /data/examples/BabySoC/vsdbabysoc_synthesis.sdc

# Generate basic timing report
report_checks


```

<img width="575" height="556" alt="Screenshot 2025-10-10 at 6 01 19 pm" src="https://github.com/user-attachments/assets/156c94c7-ca76-4fde-ac2c-7e17a240977d" />

<img width="574" height="440" alt="Screenshot 2025-10-10 at 6 01 34 pm" src="https://github.com/user-attachments/assets/29e16ae1-58f6-4ddf-9cf6-8565f1104206" />



### Prepare Required Files for OpenSTA
Before running Static Timing Analysis (STA) on the VSDBabySoC design, organize the required input files.

``` bash
root@ubuntu:/home/najla/VSD/VSDBabySoC/OpenSTA# mkdir -p examples/timing_libs/
# Next, copy the required library files from your previous PDK or simulation library directories into this folder

root@ubuntu:/home/najla/VSD/VSDBabySoC/OpenSTA/examples# ls timing_libs/
avsddac.lib  avsdpll.lib  sky130_fd_sc_hd__tt_025C_1v80.lib

#Create a folder to store your synthesized Verilog netlist and SDC constraint file:
root@ubuntu:/home/najla/VSD/VSDBabySoC/OpenSTA# mkdir -p examples/BabySoC


#copy the synthesized netlist and SDC files from your synthesis output folder into this directory:
#Verify that the files have been copied successfully:

root@ubuntu:/home/najla/VSD/VSDBabySoC/OpenSTA/examples# ls BabySoC/
vsdbabysoc.synth.v  vsdbabysoc_synthesis.sdc


```

### VSDBabySoC PVT Corner Analysis (Post-Synthesis Timing)

Static Timing Analysis (STA) is performed across all PVT corners to verify that the design meets its timing specifications.

- Setup-critical paths (max delay) usually occur under slow corners, e.g., ss_LowTemp_LowVolt, ss_HighTemp_LowVolt.

- Hold-critical paths (min delay) typically occur under fast corners, e.g., ff_LowTemp_HighVolt, ff_HighTemp_HighVolt.

The timing libraries can be downloaded from:

https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing

The following TCL script can be executed to perform STA for the available PVT corners using the Sky130 timing libraries.

<details><summary>TCL Script: </summary>
  
``` tcl
# ============================================================
#   OpenSTA – Multi-Corner STA Execution Script (BabySoC)
# ============================================================

#  Create output directory
file mkdir /data/examples/BabySoC/STA_OUTPUT

# ------------------------------------------------------------
#  Define Liberty (.lib) files for each PVT corner
# ------------------------------------------------------------
set list_of_lib_files(1)  "sky130_fd_sc_hd__tt_025C_1v80.lib"
set list_of_lib_files(2)  "sky130_fd_sc_hd__ff_100C_1v65.lib"
set list_of_lib_files(3)  "sky130_fd_sc_hd__ff_100C_1v95.lib"
set list_of_lib_files(4)  "sky130_fd_sc_hd__ff_n40C_1v56.lib"
set list_of_lib_files(5)  "sky130_fd_sc_hd__ff_n40C_1v65.lib"
set list_of_lib_files(6)  "sky130_fd_sc_hd__ff_n40C_1v76.lib"
set list_of_lib_files(7)  "sky130_fd_sc_hd__ss_100C_1v40.lib"
set list_of_lib_files(8)  "sky130_fd_sc_hd__ss_100C_1v60.lib"
set list_of_lib_files(9)  "sky130_fd_sc_hd__ss_n40C_1v28.lib"
set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

# ------------------------------------------------------------
# Read common library files (PLL, DAC, etc.)
# ------------------------------------------------------------
read_liberty /data/examples/timing_libs/timing/avsdpll.lib
read_liberty /data/examples/timing_libs/timing/avsddac.lib

# ------------------------------------------------------------
# Run STA for each Liberty file (PVT corner)
# ------------------------------------------------------------
for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {

    puts "\n=== Running STA for $list_of_lib_files($i) ===\n"

    # Load Liberty file
    read_liberty /data/examples/timing_libs/timing/$list_of_lib_files($i)

    # Read synthesized Verilog netlist
    read_verilog /data/examples/BabySoC/vsdbabysoc.synth.v

    # Link and set current design
    link_design vsdbabysoc
    current_design

    # Apply timing constraints
    read_sdc /data/examples/BabySoC/vsdbabysoc_synthesis.sdc

    # Run setup and hold checks
    check_setup -verbose

    # --------------------------------------------------------
    # Generate timing reports
    # --------------------------------------------------------
    report_checks -path_delay min_max \
        -fields {nets cap slew input_pins fanout} \
        -digits {4} > /data/examples/BabySoC/STA_OUTPUT/min_max_$list_of_lib_files($i).txt

    # Worst Setup Slack (Max)
    exec echo "$list_of_lib_files($i)" >> /data/examples/BabySoC/STA_OUTPUT/sta_worst_max_slack.txt
    report_worst_slack -max -digits {4} >> /data/examples/BabySoC/STA_OUTPUT/sta_worst_max_slack.txt

    # Worst Hold Slack (Min)
    exec echo "$list_of_lib_files($i)" >> /data/examples/BabySoC/STA_OUTPUT/sta_worst_min_slack.txt
    report_worst_slack -min -digits {4} >> /data/examples/BabySoC/STA_OUTPUT/sta_worst_min_slack.txt

    # Total Negative Slack (TNS)
    exec echo "$list_of_lib_files($i)" >> /data/examples/BabySoC/STA_OUTPUT/sta_tns.txt
    report_tns -digits {4} >> /data/examples/BabySoC/STA_OUTPUT/sta_tns.txt

    # Worst Negative Slack (WNS)
    exec echo "$list_of_lib_files($i)" >> /data/examples/BabySoC/STA_OUTPUT/sta_wns.txt
    report_wns -digits {4} >> /data/examples/BabySoC/STA_OUTPUT/sta_wns.txt
}



```



</details>


**Running OpenSTA with the TCL Script**

To execute OpenSTA along with your TCL script:

```
docker run -i -v $(pwd):/data opensta /data/sta_across_pvt.tcl


```
- The local folder **/home/..../VSDBabySoC/OpenSTA** is mounted to **/data** inside the container.
- The TCL script **sta_across_pvt.tcl** is located in my OpenSTA folder, so it can be accessed from **/data** within the container.

<details><summary>Alternate method</summary>

You can ran inside Docker:

`` docker run -i -v $(pwd):/data opensta ``

and then inside OpenSTA:

`` source /data/sta_across_pvt.tcl``

</details>

After executing the above script, you can find the generated timing reports in the STA_OUTPUT directory:

``/data/examples/BabySoC/STA_OUTPUT/``

<img width="400" height="200" alt="Screenshot 2025-10-10 at 3 57 13 pm" src="https://github.com/user-attachments/assets/451dab18-dd30-4d37-8f3c-8e33b1b38157" />


### Timing Summary Across PVT Corners (Post-Synthesis STA Results)

The table below presents the timing summary obtained by performing STA across 13 PVT corners using OpenSTA.
Key metrics, including Worst Hold Slack, Worst Setup Slack, WNS, and TNS, were extracted from the STA reports to assess the design’s timing performance.

<img width="932" height="340" alt="image" src="https://github.com/user-attachments/assets/356d83ec-18a5-4ef0-a190-178471ff2c5c" />

### Timing Plots Across PVT Corners

Plots of the timing results across PVT corners help visualize setup and hold margins across varying conditions.

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/14822984-7966-40ed-ae2e-7b4dbad7015b" />


<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/443f0320-97a1-44d8-ba56-6658ff8d7617" />



<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/c4814e17-4b1f-4adf-9fc4-661c2b54969e" />


<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/0388bbba-d1e6-434b-b092-9f1673655ebe" />
