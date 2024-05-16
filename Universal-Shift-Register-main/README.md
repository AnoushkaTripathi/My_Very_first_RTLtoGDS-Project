# n-Bit Universal Shift Register
This project give overview of RTL to GDSII of universal shift register using OpenLane and Skywater130 PDK. OpenLane is an automated open-source EDA tool which gives RTL to GDSII flow. 

# Contents
- [1 Desing and Test bench](#1-Desing-and-Test-bench)
- [2 OpenLane Flow](#2-OpenLane-Flow)
- [3 OpenLane VLSI Design Execution](#3-OpenLane-VLSI-Design-Execution)
  * [3.1 Non-Interactive Mode](#3.1-Non-Interactive-Mode)
  * [3.2 Interactive Mode](#3.2-Interactive-Mode)
  * [3.3 Synthesis](#3.3-Synthesis)
  * [3.4 Floorplan](#3.4-Floorplan)
  * [3.5 Placement](#3.5-Placement)
  * [3.6 Clock Tree Synthesis](#3.6-Clock-Tree-Synthesis)
  * [3.7 Routing](#3.7-Routing)
  * [3.8 Design Checks](#3.8-Design-Checks)
  * [3.9 GDS II Generation](#3.9-GDS-II-Generation)
- [4 Physical Design Issues](#4-Physical-Design-Issues)
- [5 Conclusion](#5-Conclusion)
- [6 References](#6-References)
- [7 Acknowledgment](#7-Acknowledgment)

  

# 1 Desing and Test bench

This is 1-bit Universal Shift Register, written using Flip-Flop with case statement (universal_shift_register_1bit).
![usr_1bit](https://user-images.githubusercontent.com/63975346/142893103-e208d580-2bd2-4e43-b369-9cc11f55f1f2.png)

 
| Select Line | Function |
|     :---:   |     :---:   |
| 00 | Memory State |
| 01 | LeftIn |
| 10 | RightIn |
| 00 | Parallel Input |



This is n-bit Universal Shift Register.
![usr_nbit_0](https://user-images.githubusercontent.com/63975346/142893301-5a0a5bc6-86e7-42ac-aba9-e2eae45a7769.png)

The `usr_nbit.v` contain 2 module `universal_shift_register_1bit` and `usr_nbit`. `usr_nbit` is the instantiated module of `universal_shift_register_1bit`, with variable size. The parameter size can be changed as per requirements. In this example I used `size=4` which means 4 bit universal shift register.

`usr_nbit_tb.v` is test bench for `usr_nbit.v` with following values for corresponding time.

Test bench in Tabular format.
Here, `clk period = 2ns`, `LeftIn = 1`, `RightIn = 1` 

| Time (ns) | Reset |Select Line | ParalleIn | ParallelOut | 
| :---: | :---: |:---: | :---: | :---: |
| 0 | x | xx | xxxx | xxxx |
| 1 | 1 | 00 | 0000 | xxxx |
| 2 | 1 | 00 | 0000 | 0000 |
| 3 | 1 | 00 | 0000 | 0000 |
| 4 | 1 | 00 | 0000 | 0000 |
| 5 | 0 | 00 | 0000 | 0000 |
| 6 | 0 | 00 | 0000 | 0000 |
| 7 | 0 | 00 | 0000 | 0000 |
| 8 | 0 | 00 | 0000 | 0000 |
| 9 | 0 | 11 | 0110 | 0000 |
| 10 | 0 | 11 | 0110 | 0110 |
| 11 | 0 | 11 | 0110 | 0110 |
| 12 | 0 | 11 | 0110 | 0110 |
| 13 | 0 | 00 | 1010 | 0110 |
| 14 | 0 | 00 | 1010 | 0110 |
| 15 | 0 | 00 | 1010 | 0110 |
| 16 | 0 | 00 | 1010 | 0110 |
| 17 | 0 | 01 | 1010 | 0110 |
| 18 | 0 | 01 | 1010 | 1011 |
| 19 | 0 | 01 | 1010 | 1011 |
| 20 | 0 | 01 | 1010 | 1011 |
| 21 | 0 | 10 | 1010 | 1011 |
| 22 | 0 | 10 | 1010 | 0111 |
| 23 | 0 | 10 | 1010 | 0111 |
| 24 | 0 | 10 | 1010 | 0111 |
| 25 | 0 | 00 | 0001 | 0111 |
| 26 | 0 | 00 | 0001 | 0111 |
| 27 | 0 | 00 | 0001 | 0111 |
| 28 | 0 | 00 | 0001 | 0111 |
| 29 | 0 | 01 | 1010 | 0111 |
| 30 | 0 | 01 | 1010 | 1011 |
| 31 | 0 | 01 | 1010 | 1011 |


*The below figure is waveform from `GTKWAVE` generated by `iverilog`.*
![tb_wave](https://user-images.githubusercontent.com/63975346/142893792-d2aba94b-d1d5-41e2-96e0-f513baf5fac9.PNG)


I modified the netlist generated after synthesis (`usr_nbit.synthesis.v`) and verified using iverilog. Below figure is waveform from `GTKWAVE` generated by `iverilog`.

![tb_wave_after_synth](https://user-images.githubusercontent.com/63975346/142900601-8cf55e84-14fe-4403-8acb-b41f88e2e04c.PNG)

# 2 OpenLane Flow

| Index  | Input | Function | Output |
|     :---:      |     :---:      |     :---:      |     :---:      |
| Prep  | `sky130`(PDK)\, `sky130_fd_sc_hd`(Std cell Library)\, `LEF's` | Prepare LEF files\, Extract metal layers from `.tlef.`\, Merging LEF's\, Trimming Liberty | Preparation complete  |
| Index1  | `src/usr_nbit.v`  | Optimize the design and remove unused part of design, Find no. of gates required and map corresponding gates fron PDK. Generate Graphviz representation of design\, Generate synthesis stats.   | `usr_nbit.synthesis.v` \, `hierarchy.dot` |
| Index2  | `merged_unpadded.lef`\, `usr_nbit.synthesis.v`\, `sky130_fd_sc_hd__ss_100C_1v60.lib`\, ` sky130_fd_sc_hd__ff_n40C_1v95.lib` | Set input and output delay to 2.00\, Static Timing Analysis (STA)\, Generate STA report.   | `2-opensta.min_max.rpt`  |
| Index3  | `usr_nbit.synthesis.v`\, `merged_unpadded.lef`\, Synthesis Report | Generate Floorplan report(core area,die area).  | `3-verilog2def_openroad.def`  |
| Index4  | `3-verilog2def_openroad.def`\, `merged.lef` | Create pins & components-terminal with default 5u boundaries offset  | `4-ioPlacer.def`  |
| Index5  | `4-ioPlacer.def`\, `merged_unpadded.lef` | Insert Tap/Decap cells  | `4-ioPlacer.def`  |
| Index4 | `merged.lef`\, `3-verilog2def_openroad.def` | IO Placement\, Random pin placement with default 5u boundaries offset | `4-ioPlacer.def` |
| Index5 | `merged_unpadded.lef`\, `4-ioPlacer.def` | Tap and Decap cell insertion | `usr_nbit.floorplan.def` |
| Index6 | `sky130A.lyt`\, `usr_nbit.floorplan.def` | Screenshot using `sky130.lyt` and layout. | `usr_nbit.floorplan.def.png` |
| Index7 | `merged_unpadded.lef`\, `usr_nbit.floorplan.def`\, `common_pdn.tcl` |Power planning\, met1 used as Stdcell Rails\, met4 and met5 used as Straps for PDN | `7-pdn.def` |
| Index8 | `merged_unpadded.lef`\, `7-pdn.def`|Global placement using `7-pdn.def`\, Running Resizer Design Optimizations\, Design Stats and Placement Analysis. | `8-replace.def `\, `8-resizer.def` |
| Index9 | `merged_unpadded.lef`\, `8-resizer.def`| Writing Verilog using `8-resizer.def`. |  `usr_nbit.synthesis_optimized.v` |
| Index10 | `merged_unpadded.lef`\, `usr_nbit.synthesis_optimized.v`\, ` sky130_fd_sc_hd__ss_100C_1v60.lib`\, ` sky130_fd_sc_hd__ff_n40C_1v95.lib` | Static Timing Analysis (STA)\, Generate STA Report. | `10-opensta_post_resizer.min_max.rpt` |
| Index11 | `merged_unpadded.lef`\, `8-resizer.def` | Detailed Placement. Design Stats and Placement Analysis. | `usr_nbit.placement.def` | 
| Index12 | `sky130A.lyt`\, `usr_nbit.placement.def` | Screenshot using `sky130.lyt` and layout. | `usr_nbit.placement.def.png` |
| Index13 | `merged_unpadded.lef`\, `usr_nbit.placement.def` | Create Characterization and compile LUT, build clock tree using **H-Tree** Topology. Repairing long wires on clock nets. Design Stats and Placement Analysis. | `usr_nbit.cts.def` |
| Index14 | `merged_unpadded.lef`\, `usr_nbit.cts.def` | Writing Verilog using `usr_nbit.cts.def`. | `usr_nbit.synthesis_cts.v` |
| Index15 | `sky130A.lyt`\, `usr_nbit.cts.def` | Screenshot using `sky130.lyt` and layout. Running Resizer Timing Optimization\, Design Stats and Placement Analysis. | `usr_nbit.cts.def.png`\, ` 15-resizer_timing.def` |
| Index16 | `merged_unpadded.lef`\, `15-resizer_timing.def ` | Writing Verilog using `15-resizer_timing.def`. | `usr_nbit.synthesis_optimized.v` |
| Index17 | `merged_unpadded.lef`\, `usr_nbit.synthesis_optimized.v`\, ` sky130_fd_sc_hd__ss_100C_1v60.lib`\, ` sky130_fd_sc_hd__ff_n40C_1v95.lib` | Static Timing Analysis (STA)\, Generate STA Report. | `17-opensta_post_resizer.min_max.rpt` |
| Index18 | `merged_unpadded.lef`\, `15-resizer_timing.def ` | Global Routing using `15-resizer_timing.def` with Maze Routing Topology\, Generate Routing report | `18-fastroute.def`\, `18-fastroute.guide` |
| Index19 | `merged_unpadded.lef`\, `18-fastroute.def` | Fill Instance Insertion | `19-addspacers.def` |
| Index20 | `merged_unpadded.lef`\, `19-addspacers.def` | Writing Verilog using `19-addspacers.def`. | `usr_nbit.synthesis_preroute.v` |
| Index21 | `merged_unpadded.lef`\, `19-addspacers.def `\, `18-fastroute.guide` | Running Detailed Routing\, Optimize detailed routing\, Generate report. | `usr_nbit.def` |
| Index22 | `sky130A.lyt`\, ` usr_nbit.def` | Screenshot using `sky130.lyt` and layout. | `usr_nbit.def.png` |
| Index23 | `usr_nbit.def` | Parsing LEF and DEF files\, SPEF extraction. | `usr_nbit.spef` |
| Index24 | `merged_unpadded.lef`\, `usr_nbit.synthesis_preroute.v`\, `sky130_fd_sc_hd__ss_100C_1v60.lib`\, ` sky130_fd_sc_hd__ff_n40C_1v95.lib` | Static Timing Analysis (STA)\, Generate STA Report. | `24-opensta_post_resizer.min_max.rpt` |
| Index25 | `merged.lef`\, `usr_nbit.def` | Writing Powered Verilog using `usr_nbit.def`. | `25-add_sub2.powered.def` |
| Index26 | `merged_unpadded.lef`\, `25-usr_nbit.powered.def` | Writing Verilog using `25-usr_nbit.powered.def`. | `usr_nbit.lvs.powered.v` |
| Index27 | `.magicrc`\, `mag_gds.tcl`\, `sky130_fd_sc_hd.tlef`\, `usr_nbit.def` | Streaming out GDS II using Magic | `usr_nbit.gds` |
| Index28 | `sky130A.lyt`\, `usr_nbit.gds` | Screenshot using `sky130.lyt` and magic layout. | `usr_nbit.gds.png` |
| Index29 | `.magicrc`\, `gds_pointers.tcl` | Report Properties of GDS II. | `magic_gds_ptrs.mag` |
| Index30 | `.magicrc`\, `lef.tcl`\, `sky130_fd_sc_hd.tlef` | Writing abstract LEF. | `usr_nbit.lef` |
| Index31 | `.magicrc`\, ` usr_nbit.lef`\, ` maglef.tcl` | Generating MAGLEF using `usr_nbit.lef`. | ` usr_nbit.lef .mag` |
| Index27 | `.magicrc`\, `mag_gds.tcl`\, `sky130_fd_sc_hd.tlef`\, `usr_nbit.def` | Streaming out GDS II using Klayout | `usr_nbit.gds` |
| Index33 | `sky130A.lyt`\, `usr_nbit.gds` | Screenshot using `sky130.lyt` and klayout. | `usr_nbit.gds.png` |
| Index34 | `magic/usr_nbit.gds`\, `klayout/usr_nbit.gds` | Running XOR on layout using Klayout. | `usr_nbit.xor.gds` |
| Index35 | `sky130A.lyt`\, `usr_nbit.xor.gds` | Screenshot using `sky130.lyt` and layout. | `usr_nbit.xor.gds.png` |
| Index36 | `magic/usr_nbit.gds`\, `klayout/usr_nbit.gds` | Running XOR on layout using Klayout and generate xml file. | `usr_nbit.xor.xml` |
| Index37 | `.magicrc`\, `magic_spice.tcl`\, `sky130_fd_sc_hd.tlef`/, `usr_nbit.def` | Running Magic Spice Export from LEF. | `usr_nbit.spice` |
| Index38 | `usr_nbit.spice`\, `usr_nbit.lvs.powered.v`\, `sky130A_setup.tcl` | Running LEF LVS\, Generate LVS log. | `usr_nbit.lvs.lef.log` |
| Index39 | `.magicrc`\, `drc.tcl` | Running Magic DRC | `39-magic.drc`\, `usr_nbit.drc.mag` |
| Index40 | `merged_unpadded.lef`\, `usr_nbit.def` | Running Antenna Checks using OpenRoad Antenna rule checker | `41-antenna-rpt` |
| Index41 | `merged_unpadded.lef`\, `usr_nbit.def` | Circuit Validation Check | `cvc_usr_nbit.log`\, `cvc_usr_nbit.error.gz`\, `cvc_usr_nbit.debug.gz` |
| Index42 | `cvcrc.sky130A`\, `usr_nbit.cdl`\, `cvc_usr_nbit.log` | Generating Final Summary Report | `final_summary_report.csv` |


# 3 OpenLane VLSI Design Execution
`flow.tcl -design <design> -src <verilog file path> -init_design_config`. This code prepare design for execution.

## 3.1 Non-Interactive Mode
`flow.tcl -design <design> -tag <tag>`. This is automated execution of Design from RTL to GDS II. Index1-Index42 run in automated mode.

## 3.2 Interactive Mode
`flow.tcl -design <design> -tag <tag> -interactive`. This is user friendly, manual execution of tools from RTL to GDS II.

| Code | Corresponding Index |
 |     :---:      |     :---:      |
| `run_synthesis` | Index1 – Index2 |
| `run_floorplan` | Index3 – Index7 |
| `run_placement` | Index8 – Index12 |
| `run_cts` | Index13 – Index15 (Screenshot) |
| `run_resizer_timing` | Index15(Resizer) – Index17 |
| `run_routing`| Index18 – Index24 |
| `write_powered_verilog` | Index25 – Index26 |
| `set_netlist $::env(lvs_result_file_tag).powered.v` | set variable to write Verilog. |
| `run_magic` | Index27 – Index31 |
| `run_klayout` | Index32 – Index33 |
| `run_klayout_gds_xor` | Index34 – Index36 |
| `run_magic_spice_export` | Index37 |
| `run_lvs` | Index38 |
| `run_magic_drc` | Index39 |
| `run_klayout_drc` | |
| `run_antenna_check` | Index40 |
| `run_lef_cvc` | Index41 – Index42 |
| `calc_total_runtime` | |
| `generate_final_summary_report` | |

 ## 3.3 Synthesis
`Yosys`, is a Verilog RTL synthesis framework that generates gate level netlist from verilog code and `abc` performs technology mapping. The resulting netlist is used by the `OpenSTA`  tool for static timing analysis, generating timing reports. OpenLane EDA tool comes with different synthesis scripts, Synthesis Exploration helps in picking the best synthesis strategy which will provide a netlist functionally same as input design.
 
*Graphviz representation of design.*
![#1 usr_hs](https://user-images.githubusercontent.com/63975346/142900991-93ffb20e-5036-44e0-aa2e-be6fdd5a7d5d.png)

```
=== usr_nbit ===

   Number of wires:                 26
   Number of wire bits:             33
   Number of public wires:           7
   Number of public wire bits:      14
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                 23
     sky130_fd_sc_hd__a32o_2         4
     sky130_fd_sc_hd__buf_1          2
     sky130_fd_sc_hd__dfxtp_2        4
     sky130_fd_sc_hd__inv_2          3
     sky130_fd_sc_hd__mux2_1         4
     sky130_fd_sc_hd__nor2_2         1
     sky130_fd_sc_hd__o221a_2        4
     sky130_fd_sc_hd__or2_2          1

   Chip area for module '\usr_nbit': 251.491200
```

Cells Description
```
sky130_fd_sc_hd__a32o_2  --> X = (A1 | A2) & (B1 | B2) & C1       --> 4.14 x 2.72 um
sky130_fd_sc_hd__buf_1   --> X = A                                --> 1.38 x 2.72 um
sky130_fd_sc_hd__dfxtp_2 --> X <= A                               --> 7.82 x 2.72 um
sky130_fd_sc_hd__inv_2   --> X = ~A                               --> 1.38 x 2.72 um
sky130_fd_sc_hd__mux2_1  --> X = (S & A0) | (S & A1)              --> 4.14 x 2.72 um
sky130_fd_sc_hd__nor2_2  --> Y = ~(A | B)                         --> 2.30 x 2.72 um
sky130_fd_sc_hd__o221a_2 --> X = (A1 & A2 & A3) | (B1 & B2)       --> 4.14 x 2.72 um
sky130_fd_sc_hd__or2_2   --> X = A | B                            --> 2.30 x 2.72 um
```
   

## 3.4 Floorplan
For Good placement, the most important value would be `FP_CORE_UTIL` (area occupied by the standard cells, macros, and other cells), `FP_ASPECT_RATIO` (This ratio is determined by the horizontal routing resources to vertical routing resources (or) height/width). Before we proceed with floor planning, we need to ensure that all inputs we need for the floorplan are prepared properly. Because it deals with placement of I/O pads, macros, power, and ground structure. In addition to the macros core area, its rows (which is essential during placement) and its tracks (which is essential during routing) are defined by `init_fp` Macro inputs and outputs ports are placed by `ioplacer` and the `pdn` tool generates the power distribution network. `tapcell` tool used to insert `welltap` and `decap` cells in the floorplan.
 
 ```
 Floorplanned on a die area of 0.0 0.0 39.995 50.715 (microns). 
 Floorplanned on a core area of 5.52 10.88 34.04 38.08 (microns).
 Core area width: 28.52
 Core area height: 27.199999999999996
 ```
 `usr_nbit` Floorplan The visible structure is tapcells and decap cells.
 At bottom there are Stdcells which are need to be placed.
 
 ![#2 fp](https://user-images.githubusercontent.com/63975346/142964537-f43b7af7-e801-47a2-897a-151d36964ba2.PNG)
 
 This is io Placement viewed using Klayout.
 
 ![#3 io](https://user-images.githubusercontent.com/63975346/142964860-d0f4584f-bf22-4469-bd4e-e6117260480c.png)

 Power Network Distribution.
 
 ![#4 pdn](https://user-images.githubusercontent.com/63975346/142965040-ad9944ff-05b2-4cb1-88cc-fdc1ff9b44a1.PNG)
 ```
 Stdcell Rails
      Layer: met1 -  width: 0.480  pitch: 2.720  offset: 0.000 
    Straps
      Layer: met4 -  width: 1.600  pitch: 9.507  offset: 4.753 
      Layer: met5 -  width: 1.600  pitch: 9.067  offset: 4.533 
    Connect: {met4 met5} {met1 met4}
 ```


## 3.5 Placement
Core area and rows of macros were determined in Floorplanning. The role of placement is to determine the locations of standard cells present in the netlist by placing these standard cells inside the core area of IC. A logical representation of the cells is in the Netlist. A tool places cells at the desired location based on the physical location of cells in LEF. Placement of cells is most important and challenging, because good placement minimises area and also ensures good routing. Lib file contains a number of the same kind of cells, so the tool picks the cell considering logic present in netlist and input constraints.
The most important configure parameter for placement is `PL_TARGET_DESNSITY` which describes how close or how far cells are from each other and is easy to set. `PL_TARGET_DESNSITY` ranges from 0.0 to 1.0. It is the measure of how widely the cells would spread throughout the core area. The `RePLace` tool performs global placement. Resizer tool performs optional optimizations on the design. The `OpenPhySyn` tool performs timing optimizations on the design. `OpenDP` this tool legalizes global components by performing detailed placement.
 
 Global Placement
 
 ![#6 gb placement](https://user-images.githubusercontent.com/63975346/142964947-4fbd4f26-38e2-408c-adb8-715a7c496f14.PNG)

 Detailed Placement
 
 ![#7 placement](https://user-images.githubusercontent.com/63975346/142964968-3ace609a-0155-422e-b7fa-a63b073ab20a.PNG)

 

## 3.6 Clock Tree Synthesis
Clock Tree Synthesis ensures that all of the clock signals in a design are distributed evenly to all sequential circuits in a design. In CTS, skew and latency are minimized. The clock tree constraints and the placement data will be given as input to CTS. Output of CTS are Latency and skew report. CTS def, timing report and clock structure report is also generated after CTS. Clock tree building and clock tree balancing are done by CTS. `TritonCTS` tool is used to synthesize the clock distribution network (the clock tree). `TritonCTS` uses H-Tree Toplogy.

## 3.7 Routing 
After CTS, routing is the stage at which the necessary interconnections are determined by finding the exact paths for each network. By the end of CTS, the tool will know the locations of cells, pins, IO ports, and pads. The logical connectivity and design rules are defined by netlist and technology files respectively and are available to the tool. Routing is the process after CTS in which interconnection of the macro pins, the standard cells, the pins of the block boundary, the pads of the chip boundary using technology files. All connections defined by the netlist are electrically connected using metal and vias in the routing stage. So, basically, routing is allocating a set of metal layers (wires) in the routing space that makes interconnections between all the nets in the netlist by ensuring certain design rules for the metals and vias. `FastRoute` tool performs global routing to generate a guide file for the detailed router. `CU-GR` is another option for performing global routing. The `TritonRoute` tool performs detailed routing. `SPEF-Extractor`, performs SPEF extraction.

After Routing

![usr_nbit def](https://user-images.githubusercontent.com/63975346/142967039-514c2a31-2637-42f4-8669-1fea74a2d9d5.png)

```
complete detail routing
total wire length = 934 um
total wire length on LAYER li1 = 0 um
total wire length on LAYER met1 = 414 um
total wire length on LAYER met2 = 454 um
total wire length on LAYER met3 = 65 um
total wire length on LAYER met4 = 0 um
total wire length on LAYER met5 = 0 um
total number of vias = 266
up-via summary (total 266):

----------------------
 FR_MASTERSLICE      0
            li1    124
           met1    131
           met2     11
           met3      0
           met4      0
----------------------
                   266
```

### Static Timming Analysis after Routing.
**Hold Time Analysis:-** The minimum amount of time after the active edge of the clock during which the data must be stable.


```
Startpoint: _39_ (rising edge-triggered flip-flop clocked by clk)
Endpoint: _39_ (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                  0.00    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                  0.00    0.00    0.00 ^ _39_/CLK (sky130_fd_sc_hd__dfxtp_1)
                  0.03    0.18    0.18 ^ _39_/Q (sky130_fd_sc_hd__dfxtp_1)
     3    0.00                           net10 (net)
                  0.03    0.00    0.18 ^ _23_/B1 (sky130_fd_sc_hd__o221a_1)
                  0.03    0.08    0.26 ^ _23_/X (sky130_fd_sc_hd__o221a_1)
     1    0.00                           _06_ (net)
                  0.03    0.00    0.26 ^ _39_/D (sky130_fd_sc_hd__dfxtp_1)
                                  0.26   data arrival time

                  0.00    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                          0.00    0.00   clock reconvergence pessimism
                                  0.00 ^ _39_/CLK (sky130_fd_sc_hd__dfxtp_1)
                         -0.02   -0.02   library hold time
                                 -0.02   data required time
-----------------------------------------------------------------------------
                                 -0.02   data required time
                                 -0.26   data arrival time
-----------------------------------------------------------------------------
                                  0.28   slack (MET)

```
![STA_hold](https://user-images.githubusercontent.com/63975346/143566522-ddae62ee-5ca4-4642-85a8-da6f7d93afe4.png)

T<sub>c2q</sub> + T<sub>comb</sub>  \<  T<sub>Hold</sub> + T<sub>skew</sub>

<br />

T<sub>skew</sub> = T<sub>capture</sub> - T<sub>launch</sub>

<br />

Arrival Time = T<sub>c2q</sub> + T<sub>comb</sub> + T<sub>launch</sub>

   = T<sub>c2q</sub> + T<sub>o221a_1</sub> + T<sub>launch</sub>

   = 0.18 + 0.08 + 0.00

   = 0.26

<br />

Required Time = T<sub>Hold</sub> + T<sub>capture</sub>

   = 0.02 + 0.00

   = 0.02

<br />

Slack = Required Time – Arrival Time

   = 0.02 – (-0.26)

   = 0.28

<br />

Slack met


<br />

**Setup Time Analysis:-** The minimum amount of time before the clock’s active edge that the data must be stable for it to be latched correctly.


```
Startpoint: select[0] (input port clocked by clk)
Endpoint: _39_ (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                  0.00    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                          2.00    2.00 v input external delay
                  0.01    0.00    2.00 v select[0] (in)
     1    0.00                           select[0] (net)
                  0.01    0.00    2.00 v input8/A (sky130_fd_sc_hd__clkbuf_1)
                  0.07    0.15    2.15 v input8/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.00                           net8 (net)
                  0.07    0.00    2.15 v _25_/A (sky130_fd_sc_hd__clkbuf_2)
                  0.04    0.18    2.33 v _25_/X (sky130_fd_sc_hd__clkbuf_2)
     5    0.00                           _12_ (net)
                  0.04    0.00    2.33 v _26_/A (sky130_fd_sc_hd__inv_2)
                  0.17    0.15    2.48 ^ _26_/Y (sky130_fd_sc_hd__inv_2)
     5    0.02                           _00_ (net)
                  0.17    0.00    2.48 ^ _28_/A (sky130_fd_sc_hd__nor2_2)
                  0.06    0.12    2.59 v _28_/Y (sky130_fd_sc_hd__nor2_2)
     4    0.01                           _14_ (net)
                  0.06    0.00    2.59 v _29_/B2 (sky130_fd_sc_hd__a32o_1)
                  0.07    0.37    2.96 v _29_/X (sky130_fd_sc_hd__a32o_1)
     1    0.00                           _01_ (net)
                  0.07    0.00    2.96 v _34_/A0 (sky130_fd_sc_hd__mux2_1)
                  0.12    0.64    3.60 v _34_/X (sky130_fd_sc_hd__mux2_1)
     1    0.00                           _17_ (net)
                  0.12    0.00    3.60 v _23_/A1 (sky130_fd_sc_hd__o221a_1)
                  0.09    0.49    4.08 v _23_/X (sky130_fd_sc_hd__o221a_1)
     1    0.00                           _06_ (net)
                  0.09    0.00    4.08 v _39_/D (sky130_fd_sc_hd__dfxtp_1)
                                  4.08   data arrival time

                  0.00   10.00   10.00   clock clk (rise edge)
                          0.00   10.00   clock network delay (ideal)
                          0.00   10.00   clock reconvergence pessimism
                                 10.00 ^ _39_/CLK (sky130_fd_sc_hd__dfxtp_1)
                         -0.30    9.70   library setup time
                                  9.70   data required time
-----------------------------------------------------------------------------
                                  9.70   data required time
                                 -4.08   data arrival time
-----------------------------------------------------------------------------
                                  5.62   slack (MET)


```

![STA_setup](https://user-images.githubusercontent.com/63975346/143566591-a9e2d7a6-9bbc-40b7-af4b-56aeaecc74af.png)


T<sub>c2q</sub> + T<sub>comb</sub>  \<  T<sub>Period</sub> - T<sub>Setup</sub> + T<sub>skew</sub>

<br />

T<sub>skew</sub> = T<sub>capture</sub> - T<sub>launch</sub>

<br />

Arrival Time = T<sub>c2q</sub> + T<sub>comb</sub> + T<sub>launch</sub>

   = T<sub>input external delay</sub> + T<sub>clkbuf_1</sub> + T<sub>clkbuf_2</sub> + T<sub>inv_2</sub> + T<sub>nor2_2</sub> + T<sub>a32o_1</sub> + T<sub>mux2_1</sub> + T<sub>o221a_1</sub>

   = 2.00 + 0.15 + 0.18 + 0.15 + 0.12 + 0.37 + 0.64 + 0.49

   = 4.1

<br />

Required Time = T<sub>Period</sub> + T<sub>Setup</sub> + T<sub>capture</sub>

   = T<sub>Period</sub> + T<sub>Setup</sub> + T<sub>capture</sub>

   = 10.00 – 0.30

   = 9.7

<br />

Slack = Required Time – Arrival Time

   = 9.7 – 4.1

   = 5.6

<br />

Slack met

<br />

## 3.8 Design Checks
DRC checks are performed using `Magic` and `Klayout`. LVS Checks are performed using `Netgen`. Antenna Checks are performed by `Magic` and `CVC` performs Circuit Validity Checks.

## 3.9 GDS II Generation
After OpenLane execution the main outputs of the flow, are mainly GDSII and LEF views, which can be used in bigger designs and by foundry. `Magic` tool is used to stream out the final GDSII layout file from the routed def. `Klayout` tool is used to stream out the final GDSII layout file from the routed def as a back-up.
![usr_nbit gds](https://user-images.githubusercontent.com/63975346/142967009-a7a8531b-32fd-4f8f-9411-37c0aa6b234d.png)

# 4 Physical Design Issues
The process of converting the gate-level netlist to layout is termed as physical design. In physical design, there are various stages of design, various mandatory checks in each stage and involved various analysis and verifications. There are multiple challenges as we move to advanced nodes in physical design. Here is the detailed explanation of [Physical Design Issues](https://github.com/harshithsn/Physical-Design-Issues)

# 5 Conclusion
We overviewed the key components of OpenLane, the only open-source EDA tool which is automated for manufacturing IC’s. We overviewed how OpenLane combines logic synthesis, placement and routing, as well as physical verification, with a manufacturing-ready open process development kit (PDK), we saw how EDA Tool practically performs IC design flow. OpenLane provides us to configure parameters by which optimal config for design can be found. It also has an option for regression run and various parameters can be compared. We executed RTL to GDSII Flow of 4-bit universal shift register. The final goal of our overall work is to understand VLSI design flow with a practical approach using the EDA tool. We can achieve 0 negative slack by varying various configuration parameters and changing clock period. Currently, OpenLane is the only open-source flow developed by Google and SkyWater which can be readily used to almost fully automate chip integration for the open PDK. This tool is useful for students who need practical experience in chip design.

# 6 References
[1] `“OpenLANE”` https://github.com/The-OpenROAD-Project/OpenLane

[2] `“SkyWater SKY130 PDK,”` https://skywater-pdk.readthedocs.io

[3] Ahmed Alaa Ghazy, and Mohamed Shalan Efabless Corporation, San Jose, USA, “OpenLANE: The Open-Source Digital ASIC Implementation Flow”

[4] `“LEF Technology file,”` https://youtu.be/OXrLdlz_4CM

[5] `“DEF Technology file,”` https://youtu.be/Ze9Mc6j8nFY

[6] `“LIB Technology file,”` https://youtu.be/0rFoqV8L0GM

[7] `“Yosys,”` https://github.com/YosysHQ/yosys

[8] `“OpenSTA,”` https://github.com/The-OpenROAD-Project/OpenSTA

[9] `“Fault,”` https://github.com/Cloud-V/Fault

[10] `“Design Flow,”` https://www.vlsiguide.com/search/label/Physical%20Design%20Flow

[11] `“TritonRoute,”` https://github.com/The-OpenROAD-Project/TritonRoute

[12] `“SPEF_EXTRACTOR,”` https://github.com/HanyMoussa/SPEF_EXTRACTOR

[13] `“Magic VLSI,”` https://github.com/RTimothyEdwards/magic

[14] `“KLayout,”` https://github.com/KLayout/klayout

[15] `OpenLane Overview` https://youtu.be/d0hPdkYg5QI

[16] `“OpenLANE Output,”` https://openlane.readthedocs.io/en/latest/#openlane-output

[17] `“Config Parameters,”` https://openlane.readthedocs.io/en/develop/configuration/README.html


# 7 Acknowledgment
I am extremely thankful to Dr. Ramesh Kini M Associate Professor NITK, Surathkal for sharing expertise, valuable guidance and encouragement.