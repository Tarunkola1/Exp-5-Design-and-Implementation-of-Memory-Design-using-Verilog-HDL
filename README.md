# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
#Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# Code
# RAM
// Verilog code
    
    module RAM
    #(
    parameter ADDR_WIDTH = 4,      
    parameter DATA_WIDTH = 8,      
    parameter DEPTH = 16         
    )
    (
    input clk,                  
    input cs,                 
    input we,                   
    input oe,                    
    input [ADDR_WIDTH-1:0] addr,   
    inout [DATA_WIDTH-1:0] data   
    );

    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];  
    reg [DATA_WIDTH-1:0] tmp_data;         

    always @(posedge clk) begin
        if (cs && we)
            mem[addr] <= data;
    end
    always @(posedge clk) begin
        if (cs && !we)
            tmp_data <= mem[addr];
    end
    assign data = (cs && oe && !we) ? tmp_data : {DATA_WIDTH{1'bz}};
    endmodule

// Test bench

    module tb_ram;
    reg clk, cs, we, oe;
    reg [3:0] addr;
    wire [7:0] data;
    reg [7:0] data_in;
    reg drive_data;
    assign data = drive_data ? data_in : 8'bz;

    RAM uut (
        .clk(clk),
        .cs(cs),
        .we(we),
        .oe(oe),
        .addr(addr),
        .data(data)
    );
    always #5 clk = ~clk;
    initial begin
        clk = 0; cs = 1; oe = 0; we = 0; addr = 0; data_in = 0; drive_data = 0;
        #10;
        we = 1; drive_data = 1;
        addr = 4'd2; data_in = 8'hAA; #10;
        addr = 4'd3; data_in = 8'h55; #10;
        we = 0; drive_data = 0;
        oe = 1;
        addr = 4'd2; #10;
        addr = 4'd3; #10;
        oe = 0;

        $finish;
    end
    endmodule


// output Waveform
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/719b7d27-1622-4376-974b-787f83177e6d" />


# ROM
 // write verilog code for ROM using $random
 
    module ROM
    #(
    parameter ADDR_WIDTH = 4,         
    parameter DATA_WIDTH = 8,          
    parameter DEPTH = 16              
    )
    (
    input clk,                         
    input cs,                       
    input oe,                        
    input [ADDR_WIDTH-1:0] addr,       
    output reg [DATA_WIDTH-1:0] data   
    );

    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];  

    integer i;
    initial begin

        for (i = 0; i < DEPTH; i = i + 1) begin
            mem[i] = $random; 
        end
    end
    always @(posedge clk) begin
        if (cs && oe)
            data <= mem[addr];
        else
            data <= {DATA_WIDTH{1'b0}}; 
    end
    endmodule
 // Test bench
 
    `timescale 1ns/1ps
    module tb_random_rom;
    parameter ADDR_WIDTH = 4;
    parameter DATA_WIDTH = 8;
    reg clk = 0;
    reg cs, oe;
    reg [ADDR_WIDTH-1:0] addr;
    wire [DATA_WIDTH-1:0] data;
    ROM #(.ADDR_WIDTH(ADDR_WIDTH), .DATA_WIDTH(DATA_WIDTH)) uut (
        .clk(clk),
        .cs(cs),
        .oe(oe),
        .addr(addr),
        .data(data)
    );
    always #5 clk = ~clk;
    initial begin
        cs = 1; oe = 0; addr = 0;
        #10;
        oe = 1;  
        repeat (10) begin
            #10 addr = addr + 1;
        end
        #20;
        $finish;
    end
    initial begin
        $display("Time\tAddr\tData");
        $monitor("%0t\t%0d\t%0h", $time, addr, data);
    end

endmodule

// output Waveform
<img width="1036" height="553" alt="Screenshot 2025-11-20 193002" src="https://github.com/user-attachments/assets/5bcd552a-d190-48e7-a0f5-39432e7e0a14" />



 # FIFO
 // write verilog code for FIFO

    module RAM
    #(
    parameter DATA_WIDTH = 8,      
    parameter ADDR_WIDTH = 4      
    )
    (
    input  wire                   clk,     
    input  wire                   rst,      
    input  wire                   wr_en,   
    input  wire                   rd_en,   
    input  wire [DATA_WIDTH-1:0]  data_in,  
    output reg  [DATA_WIDTH-1:0]  data_out, 
    output wire                   empty,  
    output wire                   full,    
    output reg  [ADDR_WIDTH:0]    count     
    );
    localparam DEPTH = (1 << ADDR_WIDTH);
    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];    
    reg [ADDR_WIDTH-1:0] wr_ptr, rd_ptr;     
    assign empty = (count == 0);
    assign full  = (count == DEPTH);
    integer i;
    initial begin
        for (i = 0; i < DEPTH; i = i + 1)
            mem[i] = {DATA_WIDTH{1'b0}};
        count = 0;
        wr_ptr = 0;
        rd_ptr = 0;
        data_out = 0;
    end
    always @(posedge clk) begin
        if (rst) begin
            count <= 0;
            wr_ptr <= 0;
            rd_ptr <= 0;
            data_out <= {DATA_WIDTH{1'b0}};
        end else begin
            if (wr_en && !full) begin
                mem[wr_ptr] <= data_in;
                wr_ptr <= wr_ptr + 1;
                count <= count + 1;
            end
            if (rd_en && !empty) begin
                data_out <= mem[rd_ptr];
                rd_ptr <= rd_ptr + 1;
                count <= count - 1;
            end
        end
    end
    endmodule

 // Test bench

    `timescale 1ns/1ps
    module tb_fifo;
    parameter DATA_WIDTH = 8;
    parameter ADDR_WIDTH = 3; 
    reg clk = 0;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [DATA_WIDTH-1:0] data_in;
    wire [DATA_WIDTH-1:0] data_out;
    wire empty, full;
    wire [ADDR_WIDTH:0] count;
    RAM #(.DATA_WIDTH(DATA_WIDTH), .ADDR_WIDTH(ADDR_WIDTH)) uut (
        .clk(clk),
        .rst(rst),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .data_in(data_in),
        .data_out(data_out),
        .empty(empty),
        .full(full),
        .count(count)
    );
    always #5 clk = ~clk;
    initial begin
        rst = 1; wr_en = 0; rd_en = 0; data_in = 0; #12;
        rst = 0;
        wr_en = 1; rd_en = 0;
        data_in = 8'h11; #10;
        data_in = 8'h22; #10;
        data_in = 8'h33; #10;
        wr_en = 0; #10;
        rd_en = 1; #10;
        #10;
        #10;
        rd_en = 0; #10;
        wr_en = 1; rd_en = 1; data_in = 8'h66; #10;
        wr_en = 1; rd_en = 1; data_in = 8'h77; #10;
        wr_en = 0; rd_en = 1; #30; 
        rd_en = 0; #20;
        $finish;
    end
    endmodule
// output Waveform
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/42abb518-c108-4a8d-85e6-b0a45219df5f" />



# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

