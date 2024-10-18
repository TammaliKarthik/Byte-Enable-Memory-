# Byte-Enable Memory Design in Verilog and SystemVerilog

Designing byte-enable memory involves creating memory where specific bytes within a word can be enabled or disabled for writing. This optimizes memory writes by allowing selective byte updates instead of writing an entire word.

## Verilog Design

Here’s a simple Verilog design for a byte-enable memory module.

### Verilog Code

```verilog
module byte_enable_memory #(
    parameter DATA_WIDTH = 32,   // 32-bit data width
    parameter ADDR_WIDTH = 8,    // 8-bit address width
    parameter NUM_BYTES  = DATA_WIDTH / 8 // Number of bytes in each word
)(
    input wire                     clk,          // Clock signal
    input wire                     we,           // Write enable
    input wire [ADDR_WIDTH-1:0]    addr,         // Address for read/write
    input wire [DATA_WIDTH-1:0]    wdata,        // Write data
    input wire [NUM_BYTES-1:0]     byte_enable,  // Byte enable signals (one for each byte)
    output reg [DATA_WIDTH-1:0]    rdata         // Read data
);

    // Memory array
    reg [DATA_WIDTH-1:0] memory [0:(1<<ADDR_WIDTH)-1];

    // Writing to memory with byte enable
    always @(posedge clk) begin
        if (we) begin
            for (int i = 0; i < NUM_BYTES; i = i + 1) begin
                if (byte_enable[i]) begin
                    memory[addr][(i*8) +: 8] <= wdata[(i*8) +: 8];  // Write to the enabled byte
                end
            end
        end
        rdata <= memory[addr];  // Read memory content
    end

endmodule
```

### Key Features of this Design:
1. **Memory Array**: A 2D array represents the memory. Each entry has a width defined by `DATA_WIDTH` (e.g., 32 bits), and the depth is determined by the `ADDR_WIDTH`.
2. **Byte Enable**: A `byte_enable` signal (4 bits for 32-bit memory) controls which bytes of the memory are written. Only the enabled bytes of `wdata` are written to memory.
3. **Reading**: The value at the specified address is always available on `rdata`.

---

## SystemVerilog Design

SystemVerilog provides enhanced constructs for memory design. Below is the equivalent design using SystemVerilog.

### SystemVerilog Code

```systemverilog
module byte_enable_memory_sv #(
    parameter DATA_WIDTH = 32,   // 32-bit data width
    parameter ADDR_WIDTH = 8,    // 8-bit address width
    parameter NUM_BYTES  = DATA_WIDTH / 8 // Number of bytes in each word
)(
    input logic                    clk,          // Clock signal
    input logic                    we,           // Write enable
    input logic [ADDR_WIDTH-1:0]   addr,         // Address for read/write
    input logic [DATA_WIDTH-1:0]   wdata,        // Write data
    input logic [NUM_BYTES-1:0]    byte_enable,  // Byte enable signals (one for each byte)
    output logic [DATA_WIDTH-1:0]  rdata         // Read data
);

    // Memory array
    logic [DATA_WIDTH-1:0] memory [0:(1<<ADDR_WIDTH)-1];

    // Writing to memory with byte enable
    always_ff @(posedge clk) begin
        if (we) begin
            for (int i = 0; i < NUM_BYTES; i++) begin
                if (byte_enable[i]) begin
                    memory[addr][(i*8) +: 8] <= wdata[(i*8) +: 8];  // Write to the enabled byte
                end
            end
        end
        rdata <= memory[addr];  // Read memory content
    end

endmodule
```

### Key Improvements in SystemVerilog:
1. **Data Types**: `logic` is used instead of `wire` or `reg` for improved clarity and functionality.
2. **Enhanced Control Flow**: SystemVerilog introduces constructs like `always_ff` to clearly indicate a flip-flop-based process.
3. **Slicing and Arrays**: More advanced handling of array slicing and dynamic arrays is available, making the code more efficient.

---

## Testbench for Byte-Enable Memory

Here is a simple testbench to simulate and verify the byte-enable memory design.

### Testbench Code

```verilog
module tb_byte_enable_memory;
    reg clk;
    reg we;
    reg [7:0] addr;
    reg [31:0] wdata;
    reg [3:0] byte_enable;
    wire [31:0] rdata;

    // Instantiate memory
    byte_enable_memory uut (
        .clk(clk),
        .we(we),
        .addr(addr),
        .wdata(wdata),
        .byte_enable(byte_enable),
        .rdata(rdata)
    );

    // Clock generation
    always #5 clk = ~clk;

    initial begin
        // Initialize inputs
        clk = 0;
        we = 0;
        addr = 8'b0;
        wdata = 32'h0;
        byte_enable = 4'b0;

        // Write to byte 1 and byte 3
        #10;
        addr = 8'h1;
        wdata = 32'hAABBCCDD;
        byte_enable = 4'b1010;  // Enable byte 1 and 3
        we = 1;

        #10;
        we = 0;  // Disable write
        byte_enable = 4'b0;

        #10;
        // Read back data
        addr = 8'h1;

        #10;
        $finish;
    end
endmodule
```

### Key Points of Testbench:
1. **Clock Generation**: A clock with a period of 10 time units is generated.
2. **Write and Read Operations**: A write is performed with selective bytes enabled, followed by a read to verify the operation.

This design demonstrates byte-enable functionality for memory using both Verilog and SystemVerilog. The design can be extended to larger data widths or more advanced memory systems depending on your requirements.

---
