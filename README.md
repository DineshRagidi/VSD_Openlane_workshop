# VSD_Openlane_workshop
# OpenLane Sky130 Workshop – VSDIAT

## Introduction

This repository contains my work, notes, screenshots, and assignments completed as part of the **VSDIAT OpenLane Sky130 Workshop**. The workshop provides a comprehensive understanding of the complete RTL-to-GDSII ASIC design flow using the open-source **OpenLane** toolchain and the **SkyWater SKY130 Process Design Kit (PDK)**.

Throughout this workshop, I explored the various stages of physical design, including:

- Design synthesis
- Floorplanning
- Standard cell characterization
- Placement
- Clock Tree Synthesis (CTS)
- Routing
- Static Timing Analysis (STA)
- Power Distribution Network (PDN) generation
- Design Rule Check (DRC) and Layout Versus Schematic (LVS) verification

### Workshop Progress

- **Day 1:** Inception of Open-Source EDA, OpenLANE and Sky130 PDK
- **Day 2:** Good Floorplan vs Bad Floorplan and Introduction to Library Cells
- **Day 3:** Design Library Cell using Magic Layout and Characterization
- **Day 4:** Pre-layout Timing Analysis and Importance of Good Clock Tree
- **Day 5:** Final Steps for RTL2GDS using TritonRoute and Sign-off Checks

The objective of this workshop is to gain hands-on experience with open-source ASIC design methodologies and understand the complete chip implementation flow from RTL design to manufacturable layout.

---

**Technology:** SKY130 PDK  
**EDA Flow:** OpenLane  
**Platform:** Linux (Ubuntu)  
**Workshop:** VSDIAT OpenLane Sky130 Workshop


Day 1 — Inception of Open-Source EDA, OpenLANE & Sky130 PDK
Understanding the Chip Package

When we look at any embedded board and point to what we call the "chip," we're actually looking at the package — a protective casing around the actual silicon die. The real chip sits in the centre of this package and communicates with the outside world via wire bonding — tiny wires that connect the chip's pads to the package pins.
Inside the Chip: Core, Pads, and Die

Zooming into the chip itself, all signals between the chip and the external world pass through pads placed around the periphery. The region enclosed by the pads is called the core — this is where all the actual digital logic lives. Together, the core and the pads form the die, which is the fundamental unit of chip manufacturing.

    Foundry — the place where chips are physically manufactured
    Foundry IPs — IP blocks that require specialized process knowledge to implement (e.g., PLLs, SRAMs)
    Macros — reusable, purely digital logic blocks

From Software to Silicon — The ISA Bridge

A C program running on a chip goes through a multi-layer transformation:

    The C code is compiled into RISC-V assembly (or another ISA)
    The assembler converts it to binary machine code (0s and 1s)
    This binary pattern needs an RTL implementation of the ISA
    The RTL gets synthesized and goes through the full PnR (Place and Route) flow to become a physical layout

The system software stack (OS → Compiler → Assembler) acts as the bridge between what the programmer writes and what the hardware executes.
Why Open-Source EDA Matters

For a fully open-source ASIC design flow, three things are needed:

    RTL Designs (e.g., from opencores.org)
    EDA Tools (synthesis, P&R, verification)
    PDK Data (process-specific design rules, standard cell libraries)

Historically, PDKs were proprietary and distributed only under NDAs, making chip design inaccessible to most people. This changed in June 2020, when Google collaborated with SkyWater Technology to release the Sky130 PDK as the world's first open-source process design kit — a massive milestone for the VLSI community.
OpenLANE and the Automated RTL to GDSII Flow

OpenLANE is an open-source flow built on top of multiple EDA tools that automates the journey from an RTL netlist all the way to the final GDSII layout file. It uses:
Stage 	Tool(s) Used
Synthesis 	Yosys, ABC
Floorplan & PDN 	OpenROAD
Placement 	OpenROAD
CTS 	TritonCTS
Routing 	FastRoute, TritonRoute
SPEF Extraction 	OpenRCX
GDS Streaming 	Magic, KLayout
Timing Analysis 	OpenSTA
DRC & LVS 	Magic, Netgen
Lab — Running OpenLANE for picorv32a
Setting Up and Invoking OpenLANE

The very first step is to navigate to the OpenLANE working directory and launch the tool in interactive mode, which lets us run each stage step-by-step.

cd /home/vscode/Desktop/OpenLane
make mount
./flow.tcl -interactive
package require openlane 1.0.2

![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/terminal_open_magic.png?raw=true)
![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/openlane_setup_dine.png?raw=true)

Running Synthesis
![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/synthesis_statistics_1.png?raw=true)
![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/synthesis_statistics_2.png?raw=true)
![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/synythesis_success.png?raw=true)
After synthesis completes, we can calculate the flop ratio — a useful sanity check:
Flop Ratio = (No. of D Flip-Flops) / (Total No. of Cells)
           = 1613 / 15762
           ≈ 0.1023  →  ~10.23%


           Day 2 — Floorplanning and Introduction to Library Cells
Chip Floorplanning — Core Area and Utilisation

Floorplanning is about deciding where everything goes on the chip. Two key parameters drive this:

    Utilisation Factor = (Area occupied by Netlist) / (Total Core Area)
        A utilisation of 0.5–0.6 is typical — you want room for buffers, routing, etc.
    Aspect Ratio = Height / Width of the core
        A ratio of 1 means a square; anything else is a rectangle.

Pre-Placed Cells and Decoupling Capacitors

Pre-placed cells (like memories, PLLs, and complex IP blocks) are fixed in position before automated placement runs. Their location is determined manually based on connectivity and power intent.

Decoupling capacitors are placed around pre-placed cells to act as local charge reservoirs — they compensate for voltage drops caused by switching activity and ensure these blocks see clean power.
Power Planning — Mesh vs Ring

A good power grid uses both power rings around the core and a power mesh across the chip. Multiple VDD and VSS rails are distributed in both metal layers so that every standard cell has a nearby power tap, minimising IR drop and electromigration risk.
Pin Placement and Logical Cell Blockage

Input and output pins are placed along the chip boundary. The relative placement of pins is guided by connectivity — a pin that drives logic deep in the core should be closer to that logic. The area between the core and the die boundary (I/O ring area) is blocked from automated cell placement to reserve it for pin buffers and ESD cells.
Lab — Floorplan and Placement
Running Floorplan
![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/floor_plan_run.png?raw=true)

cd results/floorplan/
less picorv32a.def
Viewing the Floorplan in Magic
magic -T /home/vsduser/Desktop/OpenLane/designs/picorv32a/sky130A/libs.tech/magic/sky130A.tech \
      lef read ../../tmp/merged.nom.lef \
      def read picorv32a.def &
      ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/floor_plan_complete.png?raw=true)
      ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/floor_plan_complete.png?raw=true)

      Running Placement
      run_placement
      ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/processing_before_placement.png?raw=true)
      ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/placement_in_magic.png?raw=true)
       ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/placement_run_complete.png?raw=true)


       Day 3 — Design and Characterisation of Library Cells using Magic & ngspice
      CMOS Inverter — SPICE Deck

      To characterise a standard cell, we write a SPICE netlist describing the PMOS and NMOS transistors along with their W/L ratios, supply voltage, input stimulus, and load       capacitance.

      Key parameters we extract from simulation:

    Rise time — 20% to 80% of output rising edge
    Fall time — 80% to 20% of output falling edge
    Propagation delay — 50% input to 50% output

    16-Mask CMOS Fabrication Process (Brief Overview)

The chip fabrication follows a sequence of about 16 mask steps:

   1. Substrate selection (p-type, high resistivity)
   2. Active region creation (field oxidation + Si3N4 mask)
   3.N-well and P-well formation (ion implantation)
   4. Gate oxide growth
   5. Polysilicon gate deposition
   6. Source/Drain implantation (LDD + halo)
   7.Contacts and metal layers
   8. Final passivation
# Lab — Cloning and Characterising a Custom Inverter Cell
Cloning the Standard Cell Repository

git clone https://github.com/nickson-jose/vsdstdcelldesign.git
magic -T sky130A.tech sky130_inv.mag &
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/tkcon_lefcreation.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/mask_inverter.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/inverter_pmos_in_tcl.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Inverter_layout_in_magic.png?raw=true)
   ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Grid_formation_using_track.png?raw=true)
 #  Extracting SPICE Netlist from Magic
Inside the tkcon console:
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
   ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/created_spice_file.png?raw=true)
   Editing the spice model file for analysis through simulation.
   Final edited spice file ready for ngspice simulation 
   ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Ngspice_transient.png?raw=true)

 #  Running ngspice Simulation
 ngspice sky130_inv.spice
 plot y vs time a
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/transient_plot_values.png?raw=true)
  Screenshot of generated plot 
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Transient_response.png?raw=true)
From the waveform, measure rise time, fall time, and propagation delay values. Rise transition time calculation

Rise transition time = Time taken for output to rise to 80% - Time taken for output to rise to 20%

20% of output = 660 mV

80% of output = 2.64 V Fall transition time calculation

Fall transition time = Time taken for output to fall to 20% - Time taken for output to fall to 80%

20% of output = 660 mV

80% of output = 2.64 V
 ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/trans_verifying.png?raw=true)
# Day 4 — Pre-Layout Timing Analysis and Clock Tree Synthesis
LEF Files and Guidelines for Standard Cell Ports

Before a custom cell can be used inside OpenLANE, it needs a proper LEF file describing its physical boundary, pin locations, and metal layer information. Port definitions must follow two important rules:

    All input and output ports must lie on the intersection of horizontal and vertical routing tracks
    The cell width must be an odd multiple of the track pitch, and height must be an odd multiple of the vertical track pitch

Static Timing Analysis (STA) Concepts

Setup slack = Data Required Time − Data Arrival Time (must be ≥ 0)

Key sources of uncertainty accounted for in STA:

    OCV (On-Chip Variation) — process/voltage/temperature variation modelled using derate factors
    Clock Uncertainty — jitter and skew margins added to timing paths
    CRPR (Clock Reconvergence Pessimism Removal) — removes artificial pessimism when launch and capture paths share clock buffers

Clock Tree Synthesis (CTS)

CTS builds a balanced tree of clock buffers to distribute the clock signal across the chip with minimal skew. After CTS:

    Hold timing must be re-checked (CTS inserts buffers that add real delay)
    Setup timing should be re-verified post-CTS as clock paths have changed

Lab — Custom Cell Integration and STA with OpenSTA
 ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/defined_ports.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/merging_LEF_files.png?raw=true)
  Get syntax for grid command
help grid
Set grid values accordingly
grid 0.46um 0.34um 0.23um 0.17um
 ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/diagonal_equidistant_tapcells.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/equidistant_ports.png?raw=true)
  Screenshot of placement def in magic 
   ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/floor_plan_complete.png?raw=true)
    ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/placement_in_magic.png?raw=true)
    Command to run OpenROAD tool

openroad

Reading lef file: read_lef /OpenLane/designs/picorv32a/runs/24-03_10-03/tmp/merged.nom.lef

Reading def file: read_def /OpenLane/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.def

Creating an OpenROAD database to work with: write_db pico_cts.db

Loading the created database in OpenROAD: read_db pico_cts.db

Read netlist post CTS: read_verilog /OpenLane/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.v

Read library for design: read_liberty $::env(LIB_SYNTH_COMPLETE)

Link design and library: link_design picorv32a

Read in the custom sdc we created: read_sdc /OpenLane/designs/picorv32a/src/my_base.sdc

Setting all cloks as propagated clocks: set_propagated_clock [all_clocks]

Check syntax of 'report_checks' command: help report_checks

Generating custom timing report: report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
 ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/placement_run_complete.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/processing_before_placement.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Read_lef_file.pngraw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/Routing_done.png?raw=true)
  Exit to OpenLANE flow exit
# Day 5 — Final RTL to GDSII using TritonRoute & OpenSTA
Routing — Global vs Detailed

Routing happens in two stages:

    Global Routing (FastRoute) — divides the chip into routing regions and finds approximate paths for each net, respecting layer and congestion constraints
    Detailed Routing (TritonRoute) — takes the global routing guides and assigns exact wire segments, vias, and metal tracks while adhering to DRC rules

SPEF and Post-Route STA
After routing, parasitics (resistance and capacitance of actual wires) are extracted into a SPEF (Standard Parasitic Exchange Format) file. These parasitics are then back-annotated into the netlist and STA is re-run for final sign-off timing.
 Lab — Power Distribution, Routing
 Generating Power Distribution Network
 gen_pdn
 Running Routing
 run_routing
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/arrangement_of_diffcells.png?raw=true)
  ![](https://github.com/DineshRagidi/VSD_Openlane_workshop/blob/e48f4b4dd9a6846be0652084d3f910e713c1b56b/floor_plan_complete.png?raw=true)
  Common violations to look out for:

.    Min spacing violations – two wires too close on the same layer
 .   Antenna violations – long metal segments accumulating charge during etch (can damage gate oxide)
        Fix: insert antenna diodes or use jumper vias to a higher layer

Tools & Environment
Tool 	Purpose
OpenLANE 	RTL-to-GDSII automation flow
Yosys 	RTL synthesis
OpenROAD 	Floorplan, Placement, CTS, Routing
Magic 	Layout editor, DRC, LVS
OpenSTA 	Static Timing Analysis
ngspice 	SPICE simulation
TritonRoute 	Detailed routing
Netgen 	LVS (Layout vs Schematic)
Sky130 PDK 	SkyWater 130nm open-source PDK

 # Key Learnings

    Understood how a chip moves from an idea (RTL) to a manufacturable file (GDSII) using a fully open-source toolchain
    Got hands-on with floorplanning, placement, CTS, and routing for the picorv32a RISC-V core
    Learned how to characterise custom standard cells and integrate them into an existing flow
    Gained practical experience with STA concepts — setup/hold slack, OCV, CRPR — using OpenSTA
    Understood how parasitics from post-route SPEF extraction affect timing sign-off

  # Acknowledgements

A huge thank you to Kunal Ghosh (Co-founder, VSD Corp. Pvt. Ltd.) and Nickson P Jose (Physical Design Engineer, Intel) for putting together such a well-structured and genuinely practical workshop. Running a real CPU from RTL to GDSII using nothing but open-source tools is something I didn’t expect to be possible — and yet here we are.

    Kunal Ghosh — Co-founder, VSD (VLSI System Design)
    Nickson Jose — for the vsdstdcelldesign repository used in Day 3 labs
    NASSCOM — for facilitating this workshop program




   
  

