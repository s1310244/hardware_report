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

