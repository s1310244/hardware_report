# 1280*8 SRAM 

## Purpose
Design of 1280*8 SRAM combining 512*8, 512*8, and 256*8 SRAMs.
  
## Step for the design
1.Generate SRAM configuration files for each size.
1.1Modify a parameter of data word size and number of words in the ```OpenRAM/myconfig.py``` file to generate 512*8 SRAM configuration file.   
```
# Data word size
word_size = 8 # modified
# Number of words in the memory
num_words = 512 # modified
```
1.2Execute the below command  
```
python3 $OPENRAM_HOME/openram.py myconfig
```
1.3Modify a parameter of data word size and number of words in the ```OpenRAM/myconfig.py``` file to generate 256*8 SRAM configuration file.  
```
# Data word size
word_size = 8 # modified
# Number of words in the memory
num_words = 256 # modified
```
1.2Execute the below command  
```
python3 $OPENRAM_HOME/openram.py myconfig
```

2.Modify the top module ```OpenRAM_test/src/ram_only.v``` file.
```
module ram_only(
  input clk,
  input rst_n,
  //input   csb0, // active low chip select
  //input  csb1, // Added by me
  //input  csb2, // Added by me
  input  web0, // active low write control
  //input  web1, // added by me
  input [7:0] din0,
  //input [7:0] din1,
  input [10:0]  addr0, // modify the number of bits 
  //input [8:0]  addr1,
  input cntrl, // Added by me. control bit of MUX
  //output [7:0] dout0, // output from R0
  //output [7:0] dout1, // Added by me. output from R1
  //output reg [7:0] dout2, // Added by me. output from R2 
  output [7:0] dout_mux, // Added by me
  output [7:0] dout4,
  output equal);

wire [7:0] dout0; // output from R0
wire [7:0] dout1; // Added by me. output from R1
wire [7:0] dout2; // Added by me. output from R2 

reg[1:0] cntrl;
reg   csb0 = 1'b1;  // active low chip select
reg  csb1 = 1'b1; // Added by me
reg  csb2 = 1'b1; // Added by me

always @ (posedge clk or negedge rst_n) begin
  if (~rst_n)
    dout4 <= 8'd0;
  else
    dout4 <= din0 + din1;
end


assign equal = (din0 == din1);

always@(*)begin
  cntrl = 2'b00;
  csb0 = 1'b1;
  csb1 = 1'b1;
  csb2 = 1'b1;
    if(addr0 < 11'd512) begin
      cntrl = 2'b00;
      csb0 = 1'b0;
      csb1 = 1'b1;
      csb2 = 1'b1;
    end
    else if(addr0 < 11'd1024) begin
      cntrl = 2'b01;
      csb0 = 1'b1;
      csb1 = 1'b0;
      csb2 = 1'b1;
    end
    else if(addr0 < 11'd1280)begin          
      cntrl = 2'b10;
      csb0 = 1'b1;
      csb1 = 1'b1;
      csb2 = 1'b0;
    end
end

//SRAM Instance keeping din0 value
  sram_sp_w8_b512_freepdk45
  R0 (
   .clk0  (clk )   ,
   .csb0  (csb0 )   ,
   .web0  (web0 )   ,
   .addr0 (addr0)   ,
   .din0  (din0 )   ,
   .dout0 (dout0)     );

//Added
//SRAM Instance keeping din1 value 
  sram_sp_w8_b512_freepdk45
  R1 (
   .clk0  (clk )    ,
   .csb0  (csb1 )   ,
   .web0  (web0 )   ,
   .addr0 (addr0)   ,
   .din0  (din0 - 11'd512)   ,
   .dout0 (dout1)     );

   sram_sp_w8_b256_freepdk45
   R2 (
    .clk0  (clk )   ,
    .csb0  (csb2 )  ,
    .web0  (web0 )  ,
    .addr0 (addr0 - 11'd1024)  ,
    .din0  (din0 )  ,
    .dout0 (dout2)  );

   // The multiplexer selects one of two inputs. 
   assign dout_mux = (cntrl == 2'b00)? dout0 : (cntrl == 2'b01)? dout1 : dout2;

endmodule
```
