# **EXP5: Design and Simulation of RAM, ROM, and FIFO Memory with Read and Write Operations Using Verilog HDL**

---

## **Aim**
To design, simulate, and verify the functionality of **RAM**, **ROM**, and **FIFO memory** modules with **read and write operations** using Verilog HDL.

---

## **Apparatus Required**
- System with **Vivado Design Suite**
---

## **Theory**

### **1. Random Access Memory (RAM)**
RAM is a volatile memory used to store data temporarily during program execution. It supports both **read** and **write** operations. Data can be accessed randomly using the address lines.  
In Verilog, RAM can be modeled using a `reg` array with write operation controlled by a **write enable (WE)** signal.

### **2. Read Only Memory (ROM)**
ROM is a non-volatile memory where data is permanently stored and can only be **read**, not written. The contents are initialized during declaration using an `initial` block or `$readmemb`/`$readmemh` commands.

### **3. First In First Out (FIFO) Memory**
FIFO is a sequential buffer that stores data such that the **first data written is the first data read**. It uses two pointers — **write pointer** and **read pointer** — to manage data flow. FIFO finds application in data buffering and inter-module communication.

---

## **Program**

### **1. RAM Module**
```verilog
`timescale 1ns/1ps

module ram (
    input clk,
    input we,
    input [11:0] addr,
    input [7:0] din,
    output reg [7:0] dout
);

    reg [7:0] mem [0:4095];

    always @(posedge clk) begin
        if (we)
            mem[addr] <= din;
        dout <= mem[addr];
    end

endmodule
```
### Testbench for RAM
```
module tb_ram;

    reg clk;
    reg we;
    reg [11:0] addr;
    reg [7:0] din;
    wire [7:0] dout;

    integer i;

   
    ram dut (
        .clk(clk),
        .we(we),
        .addr(addr),
        .din(din),
        .dout(dout)
    );

  
    initial begin
        clk = 0;
    end

    always #5 clk = ~clk; 

  
    initial begin
        we = 0;
        addr = 0;
        din = 0;
        #10;

        for (i = 0; i < 20; i = i + 1) begin
            @(posedge clk);
            addr = $random % 4096; 
            din  = $random % 256;   
            we   = 1;
            @(posedge clk);
            we   = 0;
        end

        $finish;
    end

endmodule
```
### Simulation Output for RAM
<img width="1470" height="923" alt="513131939-55b2c5be-6eab-4dc0-9876-ec9e350165a6" src="https://github.com/user-attachments/assets/8d2689a9-dbca-4e3d-9b78-a97e48e7f720" />

### 2. ROM Module
```
timescale 1ns/1ps

module rom_1 (
    input clk,
    input [11:0] addr,
    output reg [7:0] dout
);


    reg [7:0] mem [0:4095];
    
    integer i;

    initial begin
     
        for (i = 0; i < 4096; i = i + 1) begin
       
            mem[i] = $urandom; 
        end
    
        mem[12'h000] = 8'h77;
        mem[12'h001] = 8'h24;
        mem[12'h002] = 8'h69;
     

        $display("ROM contents initialized.");
    end

  
    always @(posedge clk) begin
    
        dout <= mem[addr];
    end

endmodule

```
### Testbench for ROM
```
module tb_rom_1;


    reg clk;
    reg [11:0] addr;
    
 
    wire [7:0] dout;

    integer i;

  
    rom_1 dut (
        .clk(clk),
        .addr(addr),
        .dout(dout)
    );

   
    initial begin
        clk = 0;
    end

    always #5 clk = ~clk; 

  
    initial begin
     
        addr = 12'h000;
        
       
        #10; 

   
        for (i = 0; i < 20; i = i + 1) begin
           
            @(posedge clk);
           
            addr = i; 
           
        end
        
     
        @(posedge clk);
        addr = 20;
        
        #50;

        $finish;
    end

endmodule
  
```
### Simulation Output for ROM
<img width="1470" height="923" alt="513132261-4eb941c0-424a-4259-b01b-70d948b65a3e" src="https://github.com/user-attachments/assets/f0de97d7-043d-4112-a5d8-e3c759d9920b" />



### 3. FIFO Memory Module
```
module fifo (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count);
    input clk;
    input rst;
    input wr_en;
    input [7:0] data_in;
    input rd_en;
    output reg full;
    output reg [7:0] data_out;
    output reg empty;
    output reg [4:0] count;

    reg [7:0] mem [0:15];
    reg [3:0] wr_ptr;
    reg [3:0] rd_ptr;
    always @(posedge clk) 
      begin
         if (rst) 
            begin
              wr_ptr   <= 0;
              rd_ptr   <= 0;
              count    <= 0;
              data_out <= 0;
              full     <= 0;
              empty    <= 1;
            end 
          else 
             begin
               full  <= (count == 16);
               empty <= (count == 0);
          if (wr_en && !full) 
               begin
                 mem[wr_ptr] <= data_in;
                 wr_ptr <= wr_ptr + 1'b1;
               end

          if (rd_en && !empty) 
               begin
                  data_out <= mem[rd_ptr];
                  rd_ptr <= rd_ptr + 1'b1;
               end

        case ({wr_en && !full, rd_en && !empty})
                2'b10: count <= count + 1'b1;
                2'b01: count <= count - 1'b1;
                default: count <= count;
        endcase
         full  <= (count == 16);
            empty <= (count == 0);
        end
    end
endmodule

```
### Testbench for FIFO
```
timescale 1ns/1ps

module fifo_tb;
    reg clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire full;
    wire empty;
    wire [4:0] count;

    
    fifo uut (
        clk,
        rst,
        wr_en,
        rd_en,
        data_in,
        full,
        data_out,
        empty,
        count
    );

    
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    
    initial begin
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h00;
        #10;
        
        rst = 0; #10;
        rst = 1; #10;
        rst = 0;

     
        repeat (5) begin
            @(posedge clk);
            wr_en = 1;
            data_in = data_in + 1;
        end
        @(posedge clk);
        wr_en = 0;

        
        repeat (3) begin
            @(posedge clk);
            rd_en = 1;
        end
        @(posedge clk);
        rd_en = 0;

        #20;
        $finish;
    end

endmodule
```
### Simulation Output for FIFO
<img width="1470" height="923" alt="513132608-c94f547a-b603-4f0e-8eef-116ad4767d3f" src="https://github.com/user-attachments/assets/1925a92e-6ee2-4b01-947e-c6f3c6c4bce3" />

### Result
