# WEEK7



OPenRoad installation
1.create the Codespace inside the vsdip/vsd-pd repo
  -Open this URL:
👉 https://github.com/vsdip/vsd-pd
- Click the green Code button
- Select Codespaces
- Click Create codespace on main

2. Run OpenROAD Flow Scripts

Once inside the Codespace terminal (or through the noVNC desktop terminal):

cd ~/Desktop
git clone https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git
cd OpenROAD-flow-scripts/flow
make

When the process finishes successfully, you’ll see a log summary similar to this:

<img width="894" height="658" alt="Screenshot 2025-11-16 at 3 19 07 pm" src="https://github.com/user-attachments/assets/52807034-39f1-483c-8138-86cfd2b10eb8" />


3. Access the GUI via noVNC

Open the PORTS tab in the Codespace.
Click the 🌐 icon next to port 6080 to open the noVNC desktop.

<img width="599" height="168" alt="Screenshot 2025-11-16 at 3 20 50 pm" src="https://github.com/user-attachments/assets/d07bbc4e-7bd2-48f8-9332-832bcd905570" />

You’ll see the browser-based desktop environment: 

<img width="798" height="274" alt="Screenshot 2025-11-16 at 3 21 13 pm" src="https://github.com/user-attachments/assets/29fba69a-9890-481a-9f14-b36b1b2c3c42" />



Automated RTL2GDS Flow for VSDBabySoC:

Initial Steps:

We need to create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/sky130hd
Now create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/src and include all the verilog files here.
Now copy the folders gds, include, lef and lib from the VSDBabySoC folder in your system into this directory.
The gds folder would contain the files avsddac.gds and avsdpll.gds
The include folder would contain the files sandpiper.vh, sandpiper_gen.vh, sp_default.vh and sp_verilog.vh
The gds folder would contain the files avsddac.lef and avsdpll.lef
The lib folder would contain the files avsddac.lib and avsdpll.lib
Now copy the constraints file(vsdbabysoc_synthesis.sdc) from the VSDBabySoC folder in your system into this directory.
Now copy the files(macro.cfg and pin_order.cfg) from the VSDBabySoC folder in your system into this directory.
Now, create a config.mk file whose contents are shown below:

```
# -----------------------------
# VSDBabySoC OpenROAD Flow Config
# -----------------------------

# Absolute path to your designs folder
export DESIGN_HOME = /home/vscode/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd

# Design names
export DESIGN_NICKNAME = vsdbabysoc
export DESIGN_NAME     = vsdbabysoc
export PLATFORM        = sky130hd

# Verilog files
export VERILOG_FILES = $(DESIGN_HOME)/vsdbabysoc/src/vsdbabysoc.v \
                       $(DESIGN_HOME)/vsdbabysoc/src/rvmyth.v \
                       $(DESIGN_HOME)/vsdbabysoc/src/clk_gate.v

# Constraints
export SDC_FILE = $(DESIGN_HOME)/vsdbabysoc/vsdbabysoc_synthesis.sdc

# Include directories for Verilog headers
export VERILOG_INCLUDE_DIRS = $(DESIGN_HOME)/vsdbabysoc/include
# Include directories for Verilog headers
export VERILOG_INCLUDE_DIRS = \
    $(DESIGN_HOME)/vsdbabysoc/include \
    $(DESIGN_HOME)/vsdbabysoc/src/module


# Additional files
export ADDITIONAL_GDS  = $(DESIGN_HOME)/vsdbabysoc/gds/*.gds
export ADDITIONAL_LEFS = $(wildcard $(DESIGN_HOME)/$(DESIGN_NICKNAME)/src/lef/*.lef)
export ADDITIONAL_LIBS = $(wildcard $(DESIGN_HOME)/$(DESIGN_NICKNAME)/lib/*.lib)


# Clock configuration
export CLOCK_PORT = CLK
export CLOCK_NET  = $(CLOCK_PORT)

# Floorplanning
export FP_PIN_ORDER_CFG    = $(DESIGN_HOME)/vsdbabysoc/pin_order.cfg
export MACRO_PLACEMENT_CFG = $(DESIGN_HOME)/vsdbabysoc/macro.cfg

export DIE_AREA   = 0 0 1600 1600
export CORE_AREA  = 20 20 1590 1590
export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600: -exclude right:* -exclude top:* -exclude bottom:*

# Routing / CTS
export TNS_END_PERCENT     = 100
export REMOVE_ABC_BUFFERS  = 1
export CTS_BUF_DISTANCE    = 600
export SKIP_GATE_CLONING   = 1

# Magic settings
export MAGIC_ZEROIZE_ORIGIN = 0
export MAGIC_EXT_USE_GDS   = 1


```

## Run synthesis

Now go to terminal and run the following commands:



```
cd OpenROAD-flow-scripts
source env.sh
cd flow
```

Commands for synthesis:

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
```

<img width="693" height="222" alt="Screenshot 2025-11-16 at 10 12 01 pm" src="https://github.com/user-attachments/assets/ecafb8d4-0d2e-4cef-86db-da43a2cfc0c9" />


Synthesis netlist:

<img width="1283" height="801" alt="Screenshot 2025-11-16 at 10 18 00 pm" src="https://github.com/user-attachments/assets/52803397-6475-440c-ab10-e6f5dccb9b4f" />

Synthesis log:

<img width="886" height="650" alt="Screenshot 2025-11-16 at 10 20 22 pm" src="https://github.com/user-attachments/assets/9b140821-fd08-4393-b262-5e25c168e520" />


Synthesis Check:

<img width="872" height="339" alt="Screenshot 2025-11-16 at 10 20 06 pm" src="https://github.com/user-attachments/assets/e211dcc9-3fa9-41fb-88d6-ef5fad3b6231" />

Synthesis Stats:

<img width="951" height="653" alt="Screenshot 2025-11-16 at 10 19 57 pm" src="https://github.com/user-attachments/assets/cfd6fed6-b71a-4c83-b784-c3e3e4e669fd" />

## Run Flooplan
Commands for floorplan:

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

```
Note : Replace // comments inside .lib files with block comments /* ... */:

<img width="800" height="206" alt="Screenshot 2025-11-16 at 10 33 35 pm" src="https://github.com/user-attachments/assets/d1ba3e9f-4306-4aff-b661-7f0ff9defe8e" />

<img width="804" height="211" alt="Screenshot 2025-11-16 at 10 36 18 pm" src="https://github.com/user-attachments/assets/1a7d01da-8c1a-42bd-ab05-948b7ddeaeeb" />

<img width="778" height="180" alt="Screenshot 2025-11-16 at 11 27 27 pm" src="https://github.com/user-attachments/assets/64ab098c-9dcd-4e12-b96d-b65bb0a1cd87" />


Floorplan Result (GUI)

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
```

<img width="1271" height="667" alt="Screenshot 2025-11-16 at 11 44 00 pm" src="https://github.com/user-attachments/assets/cd8387f0-11af-4eef-8897-943d2033a6c7" />


## Run placement


 
```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

```
<img width="983" height="498" alt="Screenshot 2025-11-17 at 7 49 00 am" src="https://github.com/user-attachments/assets/3d563f61-0abe-4b24-9cda-32c67bddcd54" />


<img width="976" height="503" alt="Screenshot 2025-11-17 at 7 50 07 am" src="https://github.com/user-attachments/assets/7af350c7-fa33-4720-8adb-cdbd48a715b8" />


##### Placement Result (GUI)

```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

```

<img width="488" height="364" alt="Screenshot 2025-11-17 at 8 56 59 am" src="https://github.com/user-attachments/assets/a65b9e69-bdcf-4fac-ba5a-9f21c7ddfa5d" />

<img width="1275" height="792" alt="Screenshot 2025-11-17 at 12 38 18 pm" src="https://github.com/user-attachments/assets/21aef21e-cb49-4ebb-b576-5d7ac029cf47" />

<img width="1285" height="803" alt="Screenshot 2025-11-17 at 12 52 33 pm" src="https://github.com/user-attachments/assets/a250b654-f928-4bdf-89ca-6ced335608d9" />


## RUn CTS


```

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
```


<img width="1007" height="729" alt="Screenshot 2025-11-17 at 12 53 46 pm" src="https://github.com/user-attachments/assets/638e0cdd-87e0-475f-a9a1-063bb9cf9d0e" />

<img width="1014" height="562" alt="Screenshot 2025-11-17 at 12 54 01 pm" src="https://github.com/user-attachments/assets/715eb554-4c6a-4383-946d-dee50ef959d0" />




#### CTS Result (GUI)



```
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts
```

<img width="1254" height="660" alt="Screenshot 2025-11-17 at 1 55 58 pm" src="https://github.com/user-attachments/assets/8e2837b5-63bb-42d9-b4c7-168e18c83c9f" />


<img width="1281" height="659" alt="Screenshot 2025-11-17 at 1 53 46 pm" src="https://github.com/user-attachments/assets/8d129d50-b1da-4884-adbf-dbd0c9e492ee" />

Setup Timing Report

<img width="1280" height="794" alt="Screenshot 2025-11-17 at 2 04 33 pm" src="https://github.com/user-attachments/assets/c93bae80-a568-48d4-92ea-bc7dacb82b52" />



Hold Timing Report
<img width="1253" height="772" alt="Screenshot 2025-11-17 at 2 05 38 pm" src="https://github.com/user-attachments/assets/90418525-f745-492c-98ac-c32a5f3826bc" />


CTS Reports:

```

==========================================================================
cts final report_tns
--------------------------------------------------------------------------
tns max 0.00

==========================================================================
cts final report_wns
--------------------------------------------------------------------------
wns max 0.00

==========================================================================
cts final report_worst_slack
--------------------------------------------------------------------------
worst slack max 5.82

==========================================================================
cts final report_clock_min_period
--------------------------------------------------------------------------
clk period_min = 5.18 fmax = 193.08

==========================================================================
cts final report_clock_skew
--------------------------------------------------------------------------
Clock clk
   0.97 source latency core.CPU_src2_value_a4[26]$_DFF_P_/CLK ^
  -0.85 target latency core.CPU_Dmem_value_a5[15][26]$_SDFFE_PP0P_/CLK ^
   0.00 CRPR
--------------
   0.12 setup skew


==========================================================================
cts final report_checks -path_delay min
--------------------------------------------------------------------------
Startpoint: core.CPU_reset_a2$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_reset_a3$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.21    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01    0.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01    0.33 ^ clkbuf_3_6__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.24    0.25    0.36    0.70 ^ clkbuf_3_6__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_6__leaf_CLK (net)
                  0.25    0.01    0.71 ^ clkbuf_leaf_3_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    13    0.05    0.07    0.22    0.93 ^ clkbuf_leaf_3_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_3_CLK (net)
                  0.07    0.00    0.93 ^ core.CPU_reset_a2$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     1    0.00    0.02    0.30    1.22 v core.CPU_reset_a2$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_reset_a2 (net)
                  0.02    0.00    1.22 v core.CPU_reset_a3$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_4)
                                  1.22   data arrival time

                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.21    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01    0.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01    0.33 ^ clkbuf_3_7__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    15    0.28    0.29    0.38    0.72 ^ clkbuf_3_7__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_7__leaf_CLK (net)
                  0.29    0.02    0.73 ^ clkbuf_leaf_4_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    18    0.06    0.08    0.23    0.96 ^ clkbuf_leaf_4_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_4_CLK (net)
                  0.08    0.00    0.97 ^ core.CPU_reset_a3$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_4)
                          0.00    0.97   clock reconvergence pessimism
                         -0.03    0.94   library hold time
                                  0.94   data required time
-----------------------------------------------------------------------------
                                  0.94   data required time
                                 -1.22   data arrival time
-----------------------------------------------------------------------------
                                  0.29   slack (MET)



==========================================================================
cts final report_checks -path_delay max
--------------------------------------------------------------------------
Startpoint: core.CPU_src1_value_a3[8]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[15]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.21    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01    0.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01    0.33 ^ clkbuf_3_4__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     9    0.24    0.25    0.36    0.70 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_4__leaf_CLK (net)
                  0.25    0.00    0.70 ^ clkbuf_leaf_40_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.06    0.08    0.23    0.93 ^ clkbuf_leaf_40_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_40_CLK (net)
                  0.08    0.00    0.93 ^ core.CPU_src1_value_a3[8]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     8    0.09    0.83    0.87    1.80 ^ core.CPU_src1_value_a3[8]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_src1_value_a3[8] (net)
                  0.83    0.00    1.80 ^ _10779_/A (sky130_fd_sc_hd__ha_1)
     9    0.04    0.37    0.50    2.30 ^ _10779_/SUM (sky130_fd_sc_hd__ha_1)
                                         _00076_ (net)
                  0.37    0.00    2.30 ^ _07485_/A (sky130_fd_sc_hd__inv_1)
     4    0.01    0.12    0.15    2.45 v _07485_/Y (sky130_fd_sc_hd__inv_1)
                                         _02597_ (net)
                  0.12    0.00    2.45 v _07543_/A2 (sky130_fd_sc_hd__a21oi_1)
     3    0.01    0.32    0.32    2.78 ^ _07543_/Y (sky130_fd_sc_hd__a21oi_1)
                                         _02654_ (net)
                  0.32    0.00    2.78 ^ _07549_/A3 (sky130_fd_sc_hd__o311ai_0)
     1    0.00    0.14    0.17    2.95 v _07549_/Y (sky130_fd_sc_hd__o311ai_0)
                                         _02660_ (net)
                  0.14    0.00    2.95 v place389/A (sky130_fd_sc_hd__buf_4)
     3    0.01    0.04    0.18    3.13 v place389/X (sky130_fd_sc_hd__buf_4)
                                         net388 (net)
                  0.04    0.00    3.13 v _07675_/A1 (sky130_fd_sc_hd__o21ai_0)
     1    0.04    1.05    0.82    3.96 ^ _07675_/Y (sky130_fd_sc_hd__o21ai_0)
                                         _02784_ (net)
                  1.05    0.00    3.96 ^ place351/A (sky130_fd_sc_hd__buf_4)
     2    0.01    0.06    0.25    4.21 ^ place351/X (sky130_fd_sc_hd__buf_4)
                                         net350 (net)
                  0.06    0.00    4.21 ^ _07676_/B (sky130_fd_sc_hd__nand2_1)
     3    0.01    0.11    0.11    4.32 v _07676_/Y (sky130_fd_sc_hd__nand2_1)
                                         _02785_ (net)
                  0.11    0.00    4.33 v _07759_/A1 (sky130_fd_sc_hd__a21boi_0)
     1    0.01    0.20    0.22    4.55 ^ _07759_/Y (sky130_fd_sc_hd__a21boi_0)
                                         _02866_ (net)
                  0.20    0.00    4.55 ^ _07760_/B (sky130_fd_sc_hd__xnor2_1)
     1    0.01    0.11    0.11    4.66 v _07760_/Y (sky130_fd_sc_hd__xnor2_1)
                                         _02867_ (net)
                  0.11    0.00    4.66 v _07762_/B1 (sky130_fd_sc_hd__o32ai_1)
     1    0.00    0.23    0.14    4.81 ^ _07762_/Y (sky130_fd_sc_hd__o32ai_1)
                                         _02869_ (net)
                  0.23    0.00    4.81 ^ place313/A (sky130_fd_sc_hd__buf_4)
     3    0.02    0.07    0.18    4.99 ^ place313/X (sky130_fd_sc_hd__buf_4)
                                         net312 (net)
                  0.07    0.00    4.99 ^ _10482_/A (sky130_fd_sc_hd__nor2_1)
     1    0.00    0.06    0.04    5.03 v _10482_/Y (sky130_fd_sc_hd__nor2_1)
                                         _05139_ (net)
                  0.06    0.00    5.03 v _10490_/A1 (sky130_fd_sc_hd__o21ai_0)
     1    0.04    1.04    0.82    5.86 ^ _10490_/Y (sky130_fd_sc_hd__o21ai_0)
                                         core.CPU_src2_value_a2[15] (net)
                  1.04    0.00    5.86 ^ core.CPU_src2_value_a3[15]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
                                  5.86   data arrival time

                         11.00   11.00   clock clk (rise edge)
                          0.00   11.00   clock source latency
     1    0.21    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01   11.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31   11.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01   11.33 ^ clkbuf_3_3__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    13    0.20    0.21    0.34   11.67 ^ clkbuf_3_3__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_3__leaf_CLK (net)
                  0.21    0.00   11.67 ^ clkbuf_leaf_47_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.05    0.07    0.20   11.88 ^ clkbuf_leaf_47_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_47_CLK (net)
                  0.07    0.00   11.88 ^ core.CPU_src2_value_a3[15]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
                          0.00   11.88   clock reconvergence pessimism
                         -0.20   11.68   library setup time
                                 11.68   data required time
-----------------------------------------------------------------------------
                                 11.68   data required time
                                 -5.86   data arrival time
-----------------------------------------------------------------------------
                                  5.82   slack (MET)



==========================================================================
cts final report_checks -unconstrained
--------------------------------------------------------------------------
Startpoint: core.CPU_src1_value_a3[8]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[15]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.21    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01    0.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01    0.33 ^ clkbuf_3_4__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     9    0.24    0.25    0.36    0.70 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_4__leaf_CLK (net)
                  0.25    0.00    0.70 ^ clkbuf_leaf_40_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.06    0.08    0.23    0.93 ^ clkbuf_leaf_40_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_40_CLK (net)
                  0.08    0.00    0.93 ^ core.CPU_src1_value_a3[8]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     8    0.09    0.83    0.87    1.80 ^ core.CPU_src1_value_a3[8]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_src1_value_a3[8] (net)
                  0.83    0.00    1.80 ^ _10779_/A (sky130_fd_sc_hd__ha_1)
     9    0.04    0.37    0.50    2.30 ^ _10779_/SUM (sky130_fd_sc_hd__ha_1)
                                         _00076_ (net)
                  0.37    0.00    2.30 ^ _07485_/A (sky130_fd_sc_hd__inv_1)
     4    0.01    0.12    0.15    2.45 v _07485_/Y (sky130_fd_sc_hd__inv_1)
                                         _02597_ (net)
                  0.12    0.00    2.45 v _07543_/A2 (sky130_fd_sc_hd__a21oi_1)
     3    0.01    0.32    0.32    2.78 ^ _07543_/Y (sky130_fd_sc_hd__a21oi_1)
                                         _02654_ (net)
                  0.32    0.00    2.78 ^ _07549_/A3 (sky130_fd_sc_hd__o311ai_0)
     1    0.00    0.14    0.17    2.95 v _07549_/Y (sky130_fd_sc_hd__o311ai_0)
                                         _02660_ (net)
                  0.14    0.00    2.95 v place389/A (sky130_fd_sc_hd__buf_4)
     3    0.01    0.04    0.18    3.13 v place389/X (sky130_fd_sc_hd__buf_4)
                                         net388 (net)
                  0.04    0.00    3.13 v _07675_/A1 (sky130_fd_sc_hd__o21ai_0)
     1    0.04    1.05    0.82    3.96 ^ _07675_/Y (sky130_fd_sc_hd__o21ai_0)
                                         _02784_ (net)
                  1.05    0.00    3.96 ^ place351/A (sky130_fd_sc_hd__buf_4)
     2    0.01    0.06    0.25    4.21 ^ place351/X (sky130_fd_sc_hd__buf_4)
                                         net350 (net)
                  0.06    0.00    4.21 ^ _07676_/B (sky130_fd_sc_hd__nand2_1)
     3    0.01    0.11    0.11    4.32 v _07676_/Y (sky130_fd_sc_hd__nand2_1)
                                         _02785_ (net)
                  0.11    0.00    4.33 v _07759_/A1 (sky130_fd_sc_hd__a21boi_0)
     1    0.01    0.20    0.22    4.55 ^ _07759_/Y (sky130_fd_sc_hd__a21boi_0)
                                         _02866_ (net)
                  0.20    0.00    4.55 ^ _07760_/B (sky130_fd_sc_hd__xnor2_1)
     1    0.01    0.11    0.11    4.66 v _07760_/Y (sky130_fd_sc_hd__xnor2_1)
                                         _02867_ (net)
                  0.11    0.00    4.66 v _07762_/B1 (sky130_fd_sc_hd__o32ai_1)
     1    0.00    0.23    0.14    4.81 ^ _07762_/Y (sky130_fd_sc_hd__o32ai_1)
                                         _02869_ (net)
                  0.23    0.00    4.81 ^ place313/A (sky130_fd_sc_hd__buf_4)
     3    0.02    0.07    0.18    4.99 ^ place313/X (sky130_fd_sc_hd__buf_4)
                                         net312 (net)
                  0.07    0.00    4.99 ^ _10482_/A (sky130_fd_sc_hd__nor2_1)
     1    0.00    0.06    0.04    5.03 v _10482_/Y (sky130_fd_sc_hd__nor2_1)
                                         _05139_ (net)
                  0.06    0.00    5.03 v _10490_/A1 (sky130_fd_sc_hd__o21ai_0)
     1    0.04    1.04    0.82    5.86 ^ _10490_/Y (sky130_fd_sc_hd__o21ai_0)
                                         core.CPU_src2_value_a2[15] (net)
                  1.04    0.00    5.86 ^ core.CPU_src2_value_a3[15]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
                                  5.86   data arrival time

                         11.00   11.00   clock clk (rise edge)
                          0.00   11.00   clock source latency
     1    0.21    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.02    0.01   11.01 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.30    0.31    0.31   11.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.31    0.01   11.33 ^ clkbuf_3_3__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    13    0.20    0.21    0.34   11.67 ^ clkbuf_3_3__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_3__leaf_CLK (net)
                  0.21    0.00   11.67 ^ clkbuf_leaf_47_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.05    0.07    0.20   11.88 ^ clkbuf_leaf_47_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_47_CLK (net)
                  0.07    0.00   11.88 ^ core.CPU_src2_value_a3[15]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
                          0.00   11.88   clock reconvergence pessimism
                         -0.20   11.68   library setup time
                                 11.68   data required time
-----------------------------------------------------------------------------
                                 11.68   data required time
                                 -5.86   data arrival time
-----------------------------------------------------------------------------
                                  5.82   slack (MET)



==========================================================================
cts final report_check_types -max_slew -max_cap -max_fanout -violators
--------------------------------------------------------------------------

==========================================================================
cts final max_slew_check_slack
--------------------------------------------------------------------------
0.0548100471496582

==========================================================================
cts final max_slew_check_limit
--------------------------------------------------------------------------
1.5

==========================================================================
cts final max_slew_check_slack_limit
--------------------------------------------------------------------------
0.0365

==========================================================================
cts final max_fanout_check_slack
--------------------------------------------------------------------------
1.0000000150474662e+30

==========================================================================
cts final max_fanout_check_limit
--------------------------------------------------------------------------
1.0000000150474662e+30

==========================================================================
cts final max_capacitance_check_slack
--------------------------------------------------------------------------
0.005915610585361719

==========================================================================
cts final max_capacitance_check_limit
--------------------------------------------------------------------------
0.1620579957962036

==========================================================================
cts final max_capacitance_check_slack_limit
--------------------------------------------------------------------------
0.0365

==========================================================================
cts final max_slew_violation_count
--------------------------------------------------------------------------
max slew violation count 0

==========================================================================
cts final max_fanout_violation_count
--------------------------------------------------------------------------
max fanout violation count 0

==========================================================================
cts final max_cap_violation_count
--------------------------------------------------------------------------
max cap violation count 0

==========================================================================
cts final setup_violation_count
--------------------------------------------------------------------------
setup violation count 0

==========================================================================
cts final hold_violation_count
--------------------------------------------------------------------------
hold violation count 0

==========================================================================
cts final report_checks -path_delay max reg to reg
--------------------------------------------------------------------------
Startpoint: core.CPU_src1_value_a3[8]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[15]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.32    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.37    0.70 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.23    0.93 ^ clkbuf_leaf_40_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.93 ^ core.CPU_src1_value_a3[8]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.87    1.80 ^ core.CPU_src1_value_a3[8]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
   0.50    2.30 ^ _10779_/SUM (sky130_fd_sc_hd__ha_1)
   0.15    2.45 v _07485_/Y (sky130_fd_sc_hd__inv_1)
   0.32    2.78 ^ _07543_/Y (sky130_fd_sc_hd__a21oi_1)
   0.17    2.95 v _07549_/Y (sky130_fd_sc_hd__o311ai_0)
   0.18    3.13 v place389/X (sky130_fd_sc_hd__buf_4)
   0.82    3.96 ^ _07675_/Y (sky130_fd_sc_hd__o21ai_0)
   0.26    4.21 ^ place351/X (sky130_fd_sc_hd__buf_4)
   0.11    4.32 v _07676_/Y (sky130_fd_sc_hd__nand2_1)
   0.22    4.55 ^ _07759_/Y (sky130_fd_sc_hd__a21boi_0)
   0.11    4.66 v _07760_/Y (sky130_fd_sc_hd__xnor2_1)
   0.14    4.81 ^ _07762_/Y (sky130_fd_sc_hd__o32ai_1)
   0.18    4.99 ^ place313/X (sky130_fd_sc_hd__buf_4)
   0.04    5.03 v _10482_/Y (sky130_fd_sc_hd__nor2_1)
   0.82    5.86 ^ _10490_/Y (sky130_fd_sc_hd__o21ai_0)
   0.00    5.86 ^ core.CPU_src2_value_a3[15]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
           5.86   data arrival time

  11.00   11.00   clock clk (rise edge)
   0.00   11.00   clock source latency
   0.00   11.00 ^ pll/CLK (avsdpll)
   0.32   11.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.34   11.67 ^ clkbuf_3_3__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.21   11.88 ^ clkbuf_leaf_47_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00   11.88 ^ core.CPU_src2_value_a3[15]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.00   11.88   clock reconvergence pessimism
  -0.20   11.68   library setup time
          11.68   data required time
---------------------------------------------------------
          11.68   data required time
          -5.86   data arrival time
---------------------------------------------------------
           5.82   slack (MET)



==========================================================================
cts final report_checks -path_delay min reg to reg
--------------------------------------------------------------------------
Startpoint: core.CPU_reset_a2$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_reset_a3$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.32    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.37    0.70 ^ clkbuf_3_6__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.23    0.93 ^ clkbuf_leaf_3_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.93 ^ core.CPU_reset_a2$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.30    1.22 v core.CPU_reset_a2$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
   0.00    1.22 v core.CPU_reset_a3$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_4)
           1.22   data arrival time

   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.32    0.32 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.39    0.72 ^ clkbuf_3_7__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.25    0.96 ^ clkbuf_leaf_4_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.97 ^ core.CPU_reset_a3$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_4)
   0.00    0.97   clock reconvergence pessimism
  -0.03    0.94   library hold time
           0.94   data required time
---------------------------------------------------------
           0.94   data required time
          -1.22   data arrival time
---------------------------------------------------------
           0.29   slack (MET)



==========================================================================
cts final critical path target clock latency max path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path target clock latency min path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path source clock latency min path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path delay
--------------------------------------------------------------------------
5.8593

==========================================================================
cts final critical path slack
--------------------------------------------------------------------------
5.8209

==========================================================================
cts final slack div critical path delay
--------------------------------------------------------------------------
99.344632

==========================================================================
cts final report_power
--------------------------------------------------------------------------
Group                  Internal  Switching    Leakage      Total
                          Power      Power      Power      Power (Watts)
----------------------------------------------------------------
Sequential             4.38e-03   4.03e-04   9.27e-09   4.78e-03  37.4%
Combinational          1.05e-03   2.98e-03   1.04e-08   4.03e-03  31.5%
Clock                  2.01e-03   1.99e-03   1.67e-09   3.99e-03  31.2%
Macro                  0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
Pad                    0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
----------------------------------------------------------------
Total                  7.43e-03   5.37e-03   2.13e-08   1.28e-02 100.0%
                          58.0%      42.0%       0.0%

```

### Run Routing

<img width="921" height="308" alt="Screenshot 2025-11-26 at 2 18 41 pm" src="https://github.com/user-attachments/assets/5ec46088-fb8f-4641-84f3-628053043c7f" />



