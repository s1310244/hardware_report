# [T0.3] LIF neuron
## Pre-prepare
glone Torch2HW
```
git clone -b t1-3_dev https://github.com/klab-aizu/Torch2HW.git
git submodule update --init --recursive
```
go to folder
```
cd Torch2HW/HW/SIM
```
## simulation and coverage
Source the environment variables  
```
source /home/share/cad_sh/cad.sh
source ~/ModelSim_tutorial/env.sh
```
open vsim
```
vsim
```
Create a working library by using the transcript box
```
vlib work
vmap work work
```
Compile the RTL design and testbench
```
vlog +cover ../RTL/LIF_neuron.v
vlog ../TB/TB_LIF_neuron.v
```
start simulation
```
vsim -coverage -voptargs="+acc" work.TB_LIF_neuron
add wave /*
run -all
```
the result of  
simulation  
<img width="1920" height="1050" alt="image" src="https://github.com/user-attachments/assets/f7c3363b-cd41-474d-b4eb-85e2ccae0167" />  
  
coverage
<img width="1051" height="101" alt="image" src="https://github.com/user-attachments/assets/200262dd-1523-4425-8506-57e45273ec07" />  
the coveraged source code  
<img width="1486" height="763" alt="image" src="https://github.com/user-attachments/assets/ab06c12f-ae25-40be-a0fc-175875599ecb" />  
<img width="886" height="808" alt="image" src="https://github.com/user-attachments/assets/8a21a056-718e-4cc7-8b7b-9bede538e47e" />  
<img width="656" height="257" alt="image" src="https://github.com/user-attachments/assets/d9a7711e-71c6-4697-994e-5ee80d485b9b" />  
<img width="880" height="622" alt="image" src="https://github.com/user-attachments/assets/b8d7180c-1409-4a20-9c46-ed73faa9528b" />  

The coverage report
```
Coverage Report by instance with details

================================================================================                                                                                                             =
=== Instance: /TB_LIF_neuron/LIF0
=== Design Unit: work.LIF_neuron
================================================================================                                                                                                             =
Branch Coverage:
    Enabled Coverage              Bins      Hits    Misses  Coverage
    ----------------              ----      ----    ------  --------
    Branches                        32        30         2    93.75%

================================Branch Details================================

Branch Coverage for instance /TB_LIF_neuron/LIF0

    Line         Item                      Count     Source
    ----         ----                      -----     ------
  File ../RTL/LIF_neuron.v
------------------------------------IF Branch-----------------------------------                                                                                                             -
    150                                      372     Count coming in to IF
    150             1                        359
    151             1                         13
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    156                                      347     Count coming in to IF
    156             1                         20
    157             1                        327
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    157                                      327     Count coming in to IF
    157             2                         22
    157             3                        305
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    158                                      332     Count coming in to IF
    158             1                          1
    159             1                        331
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    159                                      331     Count coming in to IF
    159             2                        318
    159             3                         13
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    160                                      331     Count coming in to IF
    160             1                    ***0***
    161             1                        331
Branch totals: 1 hit of 2 branches = 50.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    161                                      331     Count coming in to IF
    161             2                    ***0***
    161             3                        331
Branch totals: 1 hit of 2 branches = 50.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    185                                       58     Count coming in to IF
    185             1                          2
    186             1                         56
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    186                                       55     Count coming in to IF
    186             2                         31
    187             1                         24
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    187                                       26     Count coming in to IF
    187             2                          5
    187             3                         21
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    202                                       54     Count coming in to IF
    202             1                         12
    202             2                         42
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    214                                      299     Count coming in to IF
    214             1                          2
    216             1                        297
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    217                                      297     Count coming in to IF
    217             1                        172
    219             1                        109
                                              16     All False Count
Branch totals: 3 hits of 3 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    232                                       16     Count coming in to IF
    232             1                          2
    234             1                         14
Branch totals: 2 hits of 2 branches = 100.00%

------------------------------------IF Branch-----------------------------------                                                                                                             -
    249                                        7     Count coming in to IF
    249             1                          2
    251             1                          2
    258             1                          3
Branch totals: 3 hits of 3 branches = 100.00%


Condition Coverage:
    Enabled Coverage              Bins   Covered    Misses  Coverage
    ----------------              ----      ----    ------  --------
    Conditions                      16        11         5    68.75%

================================Condition Details===============================                                                                                                             =

Condition Coverage for instance /TB_LIF_neuron/LIF0 --

  File ../RTL/LIF_neuron.v
----------------Focused Condition View-------------------
Line       156 Item    1  (~w_acc_V_sum[6] && w_acc_V_sum[5])
Condition totals: 2 of 2 input terms covered = 100.00%

----------------Focused Condition View-------------------
Line       157 Item    1  (w_acc_V_sum[6] && ~w_acc_V_sum[5])
Condition totals: 2 of 2 input terms covered = 100.00%

----------------Focused Condition View-------------------
Line       160 Item    1  (~w_acc_V_leak[6] && w_acc_V_leak[5])
Condition totals: 0 of 2 input terms covered = 0.00%

       Input Term   Covered  Reason for no coverage   Hint
      -----------  --------  -----------------------  --------------
  w_acc_V_leak[6]         N  '_0' not hit             Hit '_0'
  w_acc_V_leak[5]         N  '_1' not hit             Hit '_1'

     Rows:       Hits  FEC Target            Non-masking condition(s)
 ---------  ---------  --------------------  -------------------------
  Row   1:    ***0***  w_acc_V_leak[6]_0     w_acc_V_leak[5]
  Row   2:          1  w_acc_V_leak[6]_1     -
  Row   3:          1  w_acc_V_leak[5]_0     ~w_acc_V_leak[6]
  Row   4:    ***0***  w_acc_V_leak[5]_1     ~w_acc_V_leak[6]

----------------Focused Condition View-------------------
Line       161 Item    1  (w_acc_V_leak[6] && ~w_acc_V_leak[5])
Condition totals: 0 of 2 input terms covered = 0.00%

       Input Term   Covered  Reason for no coverage   Hint
      -----------  --------  -----------------------  --------------
  w_acc_V_leak[6]         N  '_1' not hit             Hit '_1'
  w_acc_V_leak[5]         N  '_0' not hit             Hit '_0'

     Rows:       Hits  FEC Target            Non-masking condition(s)
 ---------  ---------  --------------------  -------------------------
  Row   1:          1  w_acc_V_leak[6]_0     -
  Row   2:    ***0***  w_acc_V_leak[6]_1     ~w_acc_V_leak[5]
  Row   3:    ***0***  w_acc_V_leak[5]_0     w_acc_V_leak[6]
  Row   4:          1  w_acc_V_leak[5]_1     w_acc_V_leak[6]

----------------Focused Condition View-------------------
Line       186 Item    1  (r_refrac == 0)
Condition totals: 1 of 1 input term covered = 100.00%

----------------Focused Condition View-------------------
Line       187 Item    1  ((r_refrac != 0) && (i_State == 4))
Condition totals: 1 of 2 input terms covered = 50.00%

       Input Term   Covered  Reason for no coverage   Hint
      -----------  --------  -----------------------  --------------
  (r_refrac != 0)         N  '_0' not hit             Hit '_0'
   (i_State == 4)         Y

     Rows:       Hits  FEC Target            Non-masking condition(s)
 ---------  ---------  --------------------  -------------------------
  Row   1:    ***0***  (r_refrac != 0)_0     -
  Row   2:          1  (r_refrac != 0)_1     (i_State == 4)
  Row   3:          1  (i_State == 4)_0      (r_refrac != 0)
  Row   4:          1  (i_State == 4)_1      (r_refrac != 0)

----------------Focused Condition View-------------------
Line       217 Item    1  (w_spike | (r_refrac != 0))
Condition totals: 2 of 2 input terms covered = 100.00%

----------------Focused Condition View-------------------
Line       219 Item    1  ((w_leak_en | i_svalid) | i_recc)
Condition totals: 3 of 3 input terms covered = 100.00%


Expression Coverage:
    Enabled Coverage              Bins   Covered    Misses  Coverage
    ----------------              ----      ----    ------  --------
    Expressions                      5         5         0   100.00%

================================Expression Details==============================                                                                                                             ==

Expression Coverage for instance /TB_LIF_neuron/LIF0 --

  File ../RTL/LIF_neuron.v
----------------Focused Expression View-----------------
Line       178 Item    1  ((i_State == 4) && (r_acc_V >= w_threshold))
Expression totals: 2 of 2 input terms covered = 100.00%

----------------Focused Expression View-----------------
Line       180 Item    1  (i_State == 3)
Expression totals: 1 of 1 input term covered = 100.00%

----------------Focused Expression View-----------------
Line       202 Item    1  ((i_State == 6)? i_recc: 1'b0)
Expression totals: 2 of 2 input terms covered = 100.00%


Statement Coverage:
    Enabled Coverage              Bins      Hits    Misses  Coverage
    ----------------              ----      ----    ------  --------
    Statements                      18        18         0   100.00%

================================Statement Details===============================                                                                                                             =

Statement Coverage for instance /TB_LIF_neuron/LIF0 --

    Line         Item                      Count     Source
    ----         ----                      -----     ------
  File ../RTL/LIF_neuron.v
    150             1                        374
    156             1                        348
    158             1                        334
    160             1                        332
    178             1                        150
    180             1                         50
    185             1                         60
    202             1                         55
    213             1                        299
    215             1                          2
    218             1                        172
    220             1                        109
    231             1                         16
    233             1                          2
    235             1                         14
    248             1                          7
    250             1                          2
    252             1                          2
Toggle Coverage:
    Enabled Coverage              Bins      Hits    Misses  Coverage
    ----------------              ----      ----    ------  --------
    Toggles                        176       146        30    82.95%

================================Toggle Details================================

Toggle Coverage for instance /TB_LIF_neuron/LIF0 --

                                              Node      1H->0L      0L->1H  "Cov                                                                                                             erage"
                                              ----------------------------------                                                                                                             -----
                                        i_State[3]           0           0                                                                                                                     0.00
                                        i_Thres[0]           0           1                                                                                                                    50.00
                                        i_Thres[1]           0           0                                                                                                                     0.00
                                        i_Thres[2]           0           1                                                                                                                    50.00
                                        i_Thres[3]           1           0                                                                                                                    50.00
                                        i_Thres[4]           0           1                                                                                                                    50.00
                                        i_Thres[5]           0           0                                                                                                                     0.00
                                       r_refrac[3]           0           0                                                                                                                     0.00
                                    r_threshold[0]           0           1                                                                                                                    50.00
                                    r_threshold[1]           0           0                                                                                                                     0.00
                                    r_threshold[2]           0           1                                                                                                                    50.00
                                    r_threshold[3]           1           0                                                                                                                    50.00
                                    r_threshold[4]           0           1                                                                                                                    50.00
                                    r_threshold[5]           0           0                                                                                                                     0.00
                                   w_nxt_refrac[3]           0           0                                                                                                                     0.00
                                    w_threshold[0]           0           1                                                                                                                    50.00
                                    w_threshold[1]           0           0                                                                                                                     0.00
                                    w_threshold[2]           0           1                                                                                                                    50.00
                                    w_threshold[3]           1           0                                                                                                                    50.00
                                    w_threshold[4]           0           1                                                                                                                    50.00
                                    w_threshold[5]           0           0                                                                                                                     0.00

Total Node Count     =         88
Toggled Node Count   =         67
Untoggled Node Count =         21

Toggle Coverage      =      82.95% (146 of 176 bins)


Total Coverage By Instance (filtered view): 89.09%
```
There are misses.

## perform synthesis and layout

