## Explore the library
I checked several parameters in typical.lib, such as gate area cost, dimensions, pin names, pin locations, power/energy, and latency.  
part of typical.lib  
<img width="545" height="993" alt="image" src="https://github.com/user-attachments/assets/1309ae2e-8445-4783-890d-86e884196938" />  

Units of time, power, area, etc...: see lines 50-56  
<img width="508" height="159" alt="image" src="https://github.com/user-attachments/assets/5098f109-d2af-45ec-a92b-3fd0e5179092" />  

Condition of operation such as temperature or voltage: see lines 58-61  
<img width="472" height="97" alt="image" src="https://github.com/user-attachments/assets/f5b7a625-5903-42de-bed9-1c2bdc3be4cf" />  

the wire's parameter  
<img width="341" height="985" alt="image" src="https://github.com/user-attachments/assets/6ee37464-a785-436c-bfa3-f2606333863a" />  

And cells.v and cells.lef checked too.  
part of cells.v  
<img width="1013" height="839" alt="image" src="https://github.com/user-attachments/assets/4a248d0e-9b48-4232-843d-6956474dc9d4" />

part of cells.lef  
<img width="463" height="779" alt="image" src="https://github.com/user-attachments/assets/b81e05d6-c703-41ca-9cc3-fa6a70d415c0" />

## Post-synthesis simulation  
Execute the simulation used the below three file with Modelsim.   
```
/home/lib/cells.v  
post_syn_sim/verilog_files/detector110_tester.v  
post_syn_sim/verilog_files/detector110_net.v
```
<img width="1920" height="1050" alt="image" src="https://github.com/user-attachments/assets/d598d9b3-ca4d-414e-848e-ea9ab793556c" />

## Post-synthesis simulation with timing
### The synthesis script (in the synthesis folder, file decoder110.tcl) modified by adding two lines at the end  
```
######################################################
# Script for Cadence RTL Compiler synthesis
# Erik Brunvand, 2008
# Use with syn-rtl -f rtl-script
# Replace items inside <> with your own information
######################################################

# Set the search paths to the libraries and the HDL files
# Remember that "." means your current directory. Add more directories
# after the . if you like.
set_attribute hdl_search_path {../rtl/}
set_attribute lib_search_path {/home/lib/}
set_attribute library [list typical.lib]
set_attribute information_level 6

set myFiles [list detector110.v]         ;# All HDL files
set basename detector110                 ;# name of top level module
set myClk clk                    ;# clock name
set myPeriod_ps 2400             ;# Clock period in ps
set myInDelay_ns 0.1             ;# delay from clock to inputs valid
set myOutDelay_ns 0.1            ;# delay from clock to output valid
set runname net                  ;# name appended to output files

#*********************************************************
#*   below here shouldn't need to be changed...          *
#*********************************************************

# Analyze and Elaborate the HDL files
# set_attribute hdl_language vhdl
read_hdl ${myFiles}
elaborate ${basename}

# Apply Constraints and generate clocks
set clock [define_clock -period ${myPeriod_ps} -name ${myClk} [clock_port]]
external_delay -input $myInDelay_ns -clock ${myClk} [find / -port ports_in/*]
external_delay -output $myOutDelay_ns -clock ${myClk} [find / -port ports_out/*]

# Sets transition to default values for Synopsys SDC format,
# fall/rise 400ps
dc::set_clock_transition .4 $myClk

# check that the design is OK so far
check_design -unresolved
report timing -lint

# Synthesize the design to the target library
synthesize -to_mapped

# Write out the reports
report timing > ./reports/${basename}_${runname}_timing.rep
report gates  > ./reports/${basename}_${runname}_cell.rep
report power  > ./reports/${basename}_${runname}_power.rep

# Write out the structural Verilog and sdc files
write_hdl -mapped >  ./output_files/${basename}_${runname}.v
write_sdc >  ./output_files/${basename}_${runname}.sdc
write_sdf >  ./output_files/${basename}_${runname}.sdf
write_spef >  ./output_files/${basename}_${runname}.spef
```
### New simulation script
part of the previous simulation script in post_syn_sim folder.
```
vsim -voptargs="+acc" work.detector110_tester
```
part of a new simulation script in post_syn_sim_tim folder.
```
# simulate
vsim -t 1ps -voptargs="+acc" -sdfmax /UUT=./verilog_files/detector110_net.sdf -sdfnoerror -L work +no_neg_tcheck  detector110_tester
```
Now we know what the options mean.

### Compare waveform 
the previous waveform without timing  
<img width="1138" height="643" alt="image" src="https://github.com/user-attachments/assets/f42131ae-be58-4418-a1b0-cdec23315abb" />

the waveform with timing  
<img width="1145" height="656" alt="image" src="https://github.com/user-attachments/assets/8ae045ae-96ad-4570-9833-9ace52aba1f8" />  
After considering timing, we confirmed that there was a delay of around 15,000ps between the rising edge of the clock and the change in the detected signal.  

(Note that the timing verification is not complete because the layout design is not yet complete. It is sufficient for the current stage.)

## Post place and route simulation
the result of simulation  
<img width="1907" height="785" alt="image" src="https://github.com/user-attachments/assets/6b0b323d-6c2d-46b7-b57d-e4c0877deddc" />

## Macro
Macro: the tools will re-use the files(lib, lef) for synthesizing or place and route in black box.

TSV.lib | for synthesis
   .lef | for place-and-route(layout)  

TSV.lib  
<img width="517" height="858" alt="Screenshot 2025-12-10 at 13 07 18" src="https://github.com/user-attachments/assets/140c9fdf-67f0-4436-98ed-7258821ddc64" />  

TSV.lef  
<img width="369" height="858" alt="Screenshot 2025-12-10 at 13 08 46" src="https://github.com/user-attachments/assets/56e7a14c-5ac2-40fa-a4c6-92162a1b2ca4" />  

### How can I get a macro file
### Synthesize with macro cell
```
  genvar g_i;
  generate
  for (g_i = 0; g_i < 8;  g_i=g_i+1) begin: TSV_GEN
    TSV TSV0 (
       .data_in     (r_temp2[g_i]),
       .data_out    (output1[g_i])
    );
  end
  endgenerate
```
the below state to instantiate 8 TSVs in top.v.

This code on the line 3 in synthesis script for top.v
```
set_attribute library [list typical.lib TSV_lib/TSV.lib]
```
This code is to specify the library file.  
  
netlist file after synthesis
```
// Generated by Cadence Genus(TM) Synthesis Solution 18.13-s027_1
// Generated on: Nov 20 2023 10:34:16 JST (Nov 20 2023 01:34:16 UTC)

// Verification Directory fv/top

module top(clk, rst_n, input1, input2, output1);
  input clk, rst_n;
  input [7:0] input1, input2;
  output [7:0] output1;
  wire clk, rst_n;
  wire [7:0] input1, input2;
  wire [7:0] output1;
  wire [7:0] r_temp2;
  wire UNCONNECTED, UNCONNECTED0, UNCONNECTED1, UNCONNECTED2,
       UNCONNECTED3, UNCONNECTED4, UNCONNECTED5, UNCONNECTED6;
  wire n_0, n_1, n_2, n_3, n_4, n_5, n_6, n_7;
  wire n_8, n_9, n_10, n_11, n_12, n_13, n_14, n_15;
  TSV \TSV_GEN[0].TSV0 (.data_in (r_temp2[0]), .data_out (output1[0]));
  TSV \TSV_GEN[1].TSV0 (.data_in (r_temp2[1]), .data_out (output1[1]));
  TSV \TSV_GEN[2].TSV0 (.data_in (r_temp2[2]), .data_out (output1[2]));
  TSV \TSV_GEN[3].TSV0 (.data_in (r_temp2[3]), .data_out (output1[3]));
  TSV \TSV_GEN[4].TSV0 (.data_in (r_temp2[4]), .data_out (output1[4]));
  TSV \TSV_GEN[5].TSV0 (.data_in (r_temp2[5]), .data_out (output1[5]));
  TSV \TSV_GEN[6].TSV0 (.data_in (r_temp2[6]), .data_out (output1[6]));
  TSV \TSV_GEN[7].TSV0 (.data_in (r_temp2[7]), .data_out (output1[7]));
  DFFR_X1 \r_temp2_reg[7] (.RN (rst_n), .CK (clk), .D (n_15), .Q
       (r_temp2[7]), .QN (UNCONNECTED));
  XNOR2_X1 g650(.A (n_13), .B (n_0), .ZN (n_15));
  DFFR_X1 \r_temp2_reg[6] (.RN (rst_n), .CK (clk), .D (n_14), .Q
       (r_temp2[6]), .QN (UNCONNECTED0));
  FA_X1 g652(.A (input2[6]), .B (input1[6]), .CI (n_11), .CO (n_13), .S
       (n_14));
  DFFR_X1 \r_temp2_reg[5] (.RN (rst_n), .CK (clk), .D (n_12), .Q
       (r_temp2[5]), .QN (UNCONNECTED1));
  FA_X1 g654(.A (input2[5]), .B (input1[5]), .CI (n_9), .CO (n_11), .S
       (n_12));
  DFFR_X1 \r_temp2_reg[4] (.RN (rst_n), .CK (clk), .D (n_10), .Q
       (r_temp2[4]), .QN (UNCONNECTED2));
  FA_X1 g656(.A (input2[4]), .B (input1[4]), .CI (n_7), .CO (n_9), .S
       (n_10));
  DFFR_X1 \r_temp2_reg[3] (.RN (rst_n), .CK (clk), .D (n_8), .Q
       (r_temp2[3]), .QN (UNCONNECTED3));
  FA_X1 g658(.A (input2[3]), .B (input1[3]), .CI (n_5), .CO (n_7), .S
       (n_8));
  DFFR_X1 \r_temp2_reg[2] (.RN (rst_n), .CK (clk), .D (n_6), .Q
       (r_temp2[2]), .QN (UNCONNECTED4));
  FA_X1 g660(.A (input2[2]), .B (input1[2]), .CI (n_3), .CO (n_5), .S
       (n_6));
  DFFR_X1 \r_temp2_reg[1] (.RN (rst_n), .CK (clk), .D (n_4), .Q
       (r_temp2[1]), .QN (UNCONNECTED5));
  FA_X1 g662(.A (input2[1]), .B (input1[1]), .CI (n_1), .CO (n_3), .S
       (n_4));
  DFFR_X1 \r_temp2_reg[0] (.RN (rst_n), .CK (clk), .D (n_2), .Q
       (r_temp2[0]), .QN (UNCONNECTED6));
  HA_X1 g664(.A (input2[0]), .B (input1[0]), .CO (n_1), .S (n_2));
  XNOR2_X1 g665(.A (input2[7]), .B (input1[7]), .ZN (n_0));
endmodule
```
  
### Place-and-Route
a layout of module top shown in tutorial.  
<img width="1516" height="857" alt="image" src="https://github.com/user-attachments/assets/c69fcd5d-87af-4b41-a09e-e7e17ea5561a" />  
A white "x" symbol on the layout indicates a violation.  
open the violation brower.  
<img width="983" height="841" alt="image" src="https://github.com/user-attachments/assets/ae7b96b3-78bd-41ca-a4a5-eb4d6b044aa2" />
This violation is due to this wire being open (not connected to anything).  
<img width="1064" height="824" alt="image" src="https://github.com/user-attachments/assets/f5b4d2f9-7c62-4ec4-8fcf-7b26aed25322" />  

```
createObstruct 15 15 25 35

placeInstance \TSV_GEN[0].TSV0 15 15
placeInstance \TSV_GEN[1].TSV0 15 20
placeInstance \TSV_GEN[2].TSV0 15 25
placeInstance \TSV_GEN[3].TSV0 15 30
placeInstance \TSV_GEN[4].TSV0 20 15
placeInstance \TSV_GEN[5].TSV0 20 20
placeInstance \TSV_GEN[6].TSV0 20 25
placeInstance \TSV_GEN[7].TSV0 20 30
```
Layout when the above code is added to line 23 of top_basic.tcl and executed again  
<img width="1066" height="821" alt="image" src="https://github.com/user-attachments/assets/aee813c8-46eb-460e-a8b7-d56c77d30b89" />  

   "By placing the macro first, we can avoid routing the power rails (VDD and VSS nets)."

## Power estimation
Important Concepts  
Static power: also called leakage power. This kind of power consumption is due to the leakage current in the CMOS transistor. From 45nm downward, this type of static power has become more dominant than dynamic power.  
Dynamic power: this power caused due to switching activity. Some CAD tools even separate into internal and dynamic power.  

the descripStatic power

