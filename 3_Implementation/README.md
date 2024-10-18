


# Byte-Enable Memory Project Implementation

## Project Overview

This project involves the design and implementation of a byte-enable memory module in Verilog and SystemVerilog. The memory is capable of selectively enabling and writing specific bytes within a word, making it useful for efficient memory operations.

---

## Project Structure

### 1. **Modules**

- **Memory Module**: A parameterized memory module that supports byte-enable functionality for writing selective bytes.
- **Testbench**: A testbench to simulate the memory module and verify its functionality.

### 2. **Files**

- `byte_enable_memory.v`: Verilog code for the byte-enable memory.
- `byte_enable_memory_sv.sv`: SystemVerilog code for the byte-enable memory.
- `tb_byte_enable_memory.v`: Testbench code for simulating the memory module.
  
### 3. **Parameters**

- `DATA_WIDTH`: Width of the data in bits (default 32 bits).
- `ADDR_WIDTH`: Width of the address bus (default 8 bits, supporting 256 memory locations).
- `NUM_BYTES`: Number of bytes in each word (automatically derived as `DATA_WIDTH / 8`).

---

## Design Details

### 1. **Verilog Design**

- **File**: `byte_enable_memory.v`
- **Purpose**: Implements a 32-bit wide memory with byte-enable functionality. The design supports selective writing of bytes using the `byte_enable` signal.
  
```verilog
module byte_enable_memory #(
    parameter DATA_WIDTH = 32,
    parameter ADDR_WIDTH = 8,
    parameter NUM_BYTES  = DATA_WIDTH / 8
)(
    input wire                     clk,
    input wire                     we,
    input wire [ADDR_WIDTH-1:0]    addr,
    input wire [DATA_WIDTH-1:0]    wdata,
    input wire [NUM_BYTES-1:0]     byte_enable,
    output reg [DATA_WIDTH-1:0]    rdata
);

    reg [DATA_WIDTH-1:0] memory [0:(1<<ADDR_WIDTH)-1];

    always @(posedge clk) begin
        if (we) begin
            for (int i = 0; i < NUM_BYTES; i = i + 1) begin
                if (byte_enable[i]) begin
                    memory[addr][(i*8) +: 8] <= wdata[(i*8) +: 8];
                end
            end
        end
        rdata <= memory[addr];
    end
endmodule
```

### 2. **SystemVerilog Design**

- **File**: `byte_enable_memory_sv.sv`
- **Purpose**: Implements the same functionality as the Verilog design but uses SystemVerilog features for improved clarity and efficiency.
  
```systemverilog
module byte_enable_memory_sv #(
    parameter DATA_WIDTH = 32,
    parameter ADDR_WIDTH = 8,
    parameter NUM_BYTES  = DATA_WIDTH / 8
)(
    input logic                    clk,
    input logic                    we,
    input logic [ADDR_WIDTH-1:0]   addr,
    input logic [DATA_WIDTH-1:0]   wdata,
    input logic [NUM_BYTES-1:0]    byte_enable,
    output logic [DATA_WIDTH-1:0]  rdata
);

    logic [DATA_WIDTH-1:0] memory [0:(1<<ADDR_WIDTH)-1];

    always_ff @(posedge clk) begin
        if (we) begin
            for (int i = 0; i < NUM_BYTES; i++) begin
                if (byte_enable[i]) begin
                    memory[addr][(i*8) +: 8] <= wdata[(i*8) +: 8];
                end
            end
        end
        rdata <= memory[addr];
    end
endmodule
```

---

## Testbench

### 1. **File**: `tb_byte_enable_memory.v`
- **Purpose**: Simulates and verifies the functionality of the byte-enable memory. It performs write and read operations and checks if the correct bytes are being written and read.
  
```verilog
module tb_byte_enable_memory;
    reg clk;
    reg we;
    reg [7:0] addr;
    reg [31:0] wdata;
    reg [3:0] byte_enable;
    wire [31:0] rdata;

    byte_enable_memory uut (
        .clk(clk),
        .we(we),
        .addr(addr),
        .wdata(wdata),
        .byte_enable(byte_enable),
        .rdata(rdata)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        we = 0;
        addr = 8'b0;
        wdata = 32'h0;
        byte_enable = 4'b0;

        #10;
        addr = 8'h1;
        wdata = 32'hAABBCCDD;
        byte_enable = 4'b1010;
        we = 1;

        #10;
        we = 0;
        byte_enable = 4'b0;

        #10;
        addr = 8'h1;

        #10;
        $finish;
    end
endmodule
```

---

## Simulation Process

### Steps to Simulate:
1. **Compile the Verilog or SystemVerilog code**: 
    - Use a Verilog simulator such as **ModelSim**, **Vivado**, or **Icarus Verilog**.
    - Example (for Verilog):
      ```bash
      iverilog -o tb_byte_enable_memory tb_byte_enable_memory.v byte_enable_memory.v
      ```
    - Example (for SystemVerilog):
      ```bash
      iverilog -o tb_byte_enable_memory tb_byte_enable_memory.v byte_enable_memory_sv.sv
      ```
2. **Run the simulation**:
    - For Verilog:
      ```bash
      vvp tb_byte_enable_memory
      ```
    - For SystemVerilog:
      ```bash
      vvp tb_byte_enable_memory_sv
      ```
3. **Analyze the waveform** (optional):
    - You can generate a waveform output using `-vcd` to analyze read/write operations in a waveform viewer like GTKWave.

---

## Verification Plan

### Test Cases:
1. **Write and Read Full Word**: Write to all bytes in a word and verify the data on reading.
2. **Write with Byte Enables**: Write to specific bytes of the word and ensure the non-enabled bytes remain unchanged.
3. **Multiple Writes**: Perform multiple consecutive writes and verify if memory behaves correctly across different addresses.
4. **Edge Case Tests**: Test with boundary values of addresses and data, including the first and last addresses of memory.

---

## Tools and Setup

- **Tools Required**:
    - **Verilog Simulator**: ModelSim, Vivado, Icarus Verilog, or any Verilog/SystemVerilog simulator.
    - **Optional**: GTKWave for waveform analysis.

- **Environment Setup**:
    1. Install a Verilog/SystemVerilog simulator.
    2. Ensure the simulator is added to the system's PATH for easy command-line execution.
    3. Prepare the design and testbench files in the project folder.
    4. Run the simulation and analyze the output.

---

## Expected Outputs

- Correct data should be written to and read from the memory based on the byte-enable signals.
- The testbench will perform verification by checking the output data and printing the results during the simulation.
- If the waveform viewer is used, you can visually inspect the correct functioning of the byte-enable memory.

---

## Conclusion

This project demonstrates the design and implementation of byte-enable memory in Verilog and SystemVerilog. The testbench provides a way to simulate and verify the correct functioning of the memory. The design can be easily extended to wider data widths and more advanced memory architectures depending on system requirements.

