# Display images on the monitor
Try to display images(jpeg/png) on the monitor.

Used FPGA :  the Basys3 card which have an ARTIX-7 FPGA chipset

displayed image(jpeg): sample.jpeg (.png) 

## A module that converts jpeg and png image files to VGA format.
### Use python code.
Preparation: 
Install Pillow library.
```
pip install Pillow
```

Python script
```
from PIL import Image
import os

input_filename = "sample.jpeg"  # Image file name to convert (PNG is also acceptable)
output_filename = "image.mem"  # Name of file to output
# adjust to FPGA VGA drawing scale
target_width = 320             # width 
target_height = 240            # height

def convert_to_mem():
    # open the image and resize the image to 320x240
    img = Image.open(input_filename)
    img = img.resize((target_width, target_height))

    # Convert the image to RGB format
    rgb_img = img.convert('RGB')

    with open(output_filename, 'w') as f:
        # Determine the reading order to match the address specification order of the pixels in the FPGA image
        # From top left to top right, top right to bottom left, bottom left to bottom right
        for y in range(target_height):
            for x in range(target_width):
                r, g, b = rgb_img.getpixel((x, y))

                # 8bit(0-255) to 4bit(0-15) (vga format is 12bit)
                r = r >> 4
                g = g >> 4
                b = b >> 4

                # 12bit (RRRRGGGGBBBB)
                pixel_val = (r << 8) | (g << 4) | b
                
                # Verilog's $readmemh (read memory hex) is designed to read hex text data, so it writes it as a 3-digit hex string followed by a newline.
                f.write(f"{pixel_val:03X}\n")

if __name__ == "__main__":
    convert_to_mem()
```
a generated file: image.mem  

## source code
### top.v
```
module top (
    input  wire        clk,         // 100 MHz clock (Basys3)
    output wire        hsync,
    output wire        vsync,
    output wire [3:0]  vga_r,
    output wire [3:0]  vga_g,
    output wire [3:0]  vga_b
);

    // --------------------------------------------------
    // Pixel clock (25 MHz)
    // --------------------------------------------------
    wire pix_clk;

    pixel_clock clk_div (
        .clk(clk),
        .pixel_clk(pix_clk)
    );

    // --------------------------------------------------
    // VGA timing
    // --------------------------------------------------
    wire [9:0] x;
    wire [9:0] y;
    wire       visible;

    vga_sync vga_inst (
        .clk(pix_clk),
        .hsync(hsync),
        .vsync(vsync),
        .x(x),
        .y(y),
        .video_on(visible)
    );

    // --------------------------------------------------
    // Scale 640x480 → 320x240
    // --------------------------------------------------
    wire [7:0] fb_x = x[9:1];   // 0-319
    wire [7:0] fb_y = y[9:1];   // 0-239
    
    wire [16:0] fb_addr = fb_y * 320 + fb_x;

    // --------------------------------------------------
    // Framebuffer memory
    // --------------------------------------------------
    wire [11:0] fb_pixel;

    memory_block framebuffer (
        .clk(pix_clk),
        .addr(fb_addr),
        .dout(fb_pixel)
    );

    // --------------------------------------------------
    // VGA output selection
    // --------------------------------------------------
    assign vga_r = visible ? fb_pixel[11:8] : 4'h0;
    assign vga_g = visible ? fb_pixel[7:4]  : 4'h0;
    assign vga_b = visible ? fb_pixel[3:0]  : 4'h0;

endmodule
```

### pixel_clock.v
```
module pixel_clock (
    input  wire clk,
    output wire pixel_clk
);
    reg [1:0] div = 0;

    always @(posedge clk) begin
        div <= div + 1;
    end

    assign pixel_clk = div[1]; // 100 MHz / 4 = 25 MHz
endmodule
```

### vga_sync.v
```
module vga_sync (
    input  wire clk,
    output reg  hsync,
    output reg  vsync,
    output reg  video_on,
    output reg [9:0] x,
    output reg [9:0] y
);

    localparam H_VISIBLE = 640;
    localparam H_FRONT   = 16;
    localparam H_SYNC    = 96;
    localparam H_BACK    = 48;
    localparam H_TOTAL   = 800;

    localparam V_VISIBLE = 480;
    localparam V_FRONT   = 10;
    localparam V_SYNC    = 2;
    localparam V_BACK    = 33;
    localparam V_TOTAL   = 525;

    reg [9:0] h_count = 0;
    reg [9:0] v_count = 0;

    always @(posedge clk) begin
        if (h_count == H_TOTAL-1) begin
            h_count <= 0;
            v_count <= (v_count == V_TOTAL-1) ? 0 : v_count + 1;
        end else begin
            h_count <= h_count + 1;
        end
    end

    always @(*) begin
        hsync = ~(h_count >= (H_VISIBLE+H_FRONT) &&
                h_count <  (H_VISIBLE+H_FRONT+H_SYNC));

        vsync = ~(v_count >= (V_VISIBLE+V_FRONT) &&
                v_count <  (V_VISIBLE+V_FRONT+V_SYNC));

        video_on = (h_count < H_VISIBLE) && (v_count < V_VISIBLE);

        x = h_count;
        y = v_count;
    end
endmodule
```

### Module for extracting images from SRAM

### memory_block.v
```
module memory_block (
    input  wire        clk,
    input  wire [16:0] addr,
    output reg  [11:0] dout
);

    // 320 x 240 = 76800 locations
    reg [11:0] mem [0:76799];

    // Read a file generated by Python
    initial begin
        $readmemh("image.mem", mem);
    end

    always @(posedge clk) begin
        dout <= mem[addr];
    end

endmodule
```



## Constraint file
```
################################################################################
## Basys 3 Rev B - Complete Constraints for VGA Framebuffer Design
################################################################################

########################################
# Clock (100 MHz)
########################################
set_property PACKAGE_PIN W5 [get_ports clk]
set_property IOSTANDARD LVCMOS33 [get_ports clk]
create_clock -period 10.000 -name sys_clk [get_ports clk]


########################################
# VGA Red (4-bit)
########################################
set_property PACKAGE_PIN G19 [get_ports {vga_r[0]}]
set_property PACKAGE_PIN H19 [get_ports {vga_r[1]}]
set_property PACKAGE_PIN J19 [get_ports {vga_r[2]}]
set_property PACKAGE_PIN N19 [get_ports {vga_r[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {vga_r[*]}]


########################################
# VGA Green (4-bit)
########################################
set_property PACKAGE_PIN J17 [get_ports {vga_g[0]}]
set_property PACKAGE_PIN H17 [get_ports {vga_g[1]}]
set_property PACKAGE_PIN G17 [get_ports {vga_g[2]}]
set_property PACKAGE_PIN D17 [get_ports {vga_g[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {vga_g[*]}]


########################################
# VGA Blue (4-bit)
########################################
set_property PACKAGE_PIN N18 [get_ports {vga_b[0]}]
set_property PACKAGE_PIN L18 [get_ports {vga_b[1]}]
set_property PACKAGE_PIN K18 [get_ports {vga_b[2]}]
set_property PACKAGE_PIN J18 [get_ports {vga_b[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {vga_b[*]}]


########################################
# VGA Sync Signals
########################################
set_property PACKAGE_PIN P19 [get_ports hsync]
set_property PACKAGE_PIN R19 [get_ports vsync]
set_property IOSTANDARD LVCMOS33 [get_ports {hsync vsync}]


########################################
# Required Configuration Properties
########################################
set_property CONFIG_VOLTAGE 3.3 [current_design]
set_property CFGBVS VCCO [current_design]


########################################
# Bitstream Configuration
########################################
set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]
set_property BITSTREAM.CONFIG.CONFIGRATE 33 [current_design]
set_property CONFIG_MODE SPIx4 [current_design]
```

## Implement
1, Put the image and python code in the same folder.
2, Excute the python script.
3, Add the converted image file.
       Add Sources -> Add or create design sources
4, Using vivado tool and souce codes which is verilog HDL file and constraints file, implement.

## Reference 
website: "FPGAボードで遊ぼう！- Basys3でVGA出力 -" https://qiita.com/Kenta11/items/34555852efdf8d8f4b0c  
https://github.com/klab-aizu/m5292015_Atharv_SHARMA/blob/main/Report/Torch2HW/VGA_model.md
