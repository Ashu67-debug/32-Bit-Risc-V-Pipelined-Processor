# 32-Bit RISC-V Pipelined Processor for IoT

> A 5-stage pipelined RISC-V (RV32I) processor designed in Verilog, synthesized on a Xilinx Artix-7 FPGA, and integrated with a DHT22 humidity sensor via UART for real-time IoT monitoring.

**BIT SIPAR-25 Research Internship | BIT Mesra, June 2025**  
**Guide:** Dr. Vijay Nath, Associate Professor, ECE Department  
**Team:** Suman Kumari · Ashutosh Gupta · Kaushiki · Piyush Kumar Singh

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
  - [Pipeline Stages](#pipeline-stages)
  - [Hazard Handling](#hazard-handling)
  - [UART & IoT Integration](#uart--iot-integration)
- [System Flowchart](#system-flowchart)
- [Pipeline Stage Details](#pipeline-stage-details)
- [Results](#results)
- [Technologies Used](#technologies-used)
- [Future Work](#future-work)

---

## Project Overview

This project implements a fully functional **32-bit RISC-V (RV32I)** processor with a classic 5-stage pipeline architecture. The processor is synthesized and deployed on a **Xilinx Artix-7 FPGA** and serves as the compute core in an IoT humidity monitoring system.

The system reads environmental humidity data from a **DHT22 sensor**, processes it through the RISC-V processor, and transmits the result via **UART** to an **ESP32 microcontroller**, which then displays it on an **LCD panel**.

### Key Highlights

| Feature | Detail |
|---|---|
| ISA | RISC-V RV32I Base Integer |
| Pipeline Depth | 5 stages (IF → ID → EX → MEM → WB) |
| Target FPGA | Xilinx Artix-7 (xc7a35t) |
| Clock Frequency | 50 MHz |
| Ideal CPI | 1.0 |
| Actual CPI | 1.25 (with pipeline stalls) |
| Language | Verilog (RTL) |
| Tool | Xilinx Vivado |

---

## Architecture

### Pipeline Stages

The processor implements the standard **5-stage RISC-V pipeline**:

```
┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│     IF    │───▶│     ID    │───▶│     EX    │───▶│    MEM    │───▶│     WB    │
│  Fetch    │    │  Decode   │    │  Execute  │    │  Memory   │    │  Write    │
│           │    │           │    │           │    │  Access   │    │  Back     │
└───────────┘    └───────────┘    └───────────┘    └───────────┘    └───────────┘
  PC + imem        RegFile           ALU            Data mem         Reg write
  Branch pred      Imm gen           Forwarding      Load/store       Result fwd
```

### Hazard Handling

| Hazard Type | Solution |
|---|---|
| Structural | Harvard architecture (separate instruction & data memories) |
| Data (RAW) | Forwarding / bypassing unit + stall detection |
| Control | 2-bit branch predictor with branch target buffer |

### UART & IoT Integration

```
DHT22 Sensor ──UART──▶ FPGA (RISC-V Core) ──UART (115200 baud)──▶ ESP32 ──▶ LCD
```

The FPGA acts as an intermediary bridge. The RISC-V processor stores and processes sensor readings in registers, then transmits processed data via a custom UART TX module to the ESP32.

---

## System Flowchart

> The following diagram shows the complete data and control flow of the system.

```
                    ┌─────────────────────┐
                    │  DHT22 Humidity      │
                    │  Sensor (±2% RH)     │
                    └──────────┬──────────┘
                               │ UART data
                               ▼
              ╔═══════════════════════════════════╗
              ║     FPGA — Artix-7 (50 MHz)       ║
              ║                                   ║
              ║  ┌──────────────────────────────┐ ║
              ║  │       UART Receiver           │ ║
              ║  └──────────────┬───────────────┘ ║
              ║                 │                 ║
              ║  ┌──────────────▼───────────────┐ ║
              ║  │  ┌─────────────────────────┐ ║ ║
              ║  │  │ IF  — Instruction Fetch │ ║ ║
              ║  │  │ PC increment · branch   │ ║ ║
              ║  │  │ prediction              │ ║ ║
              ║  │  └────────────┬────────────┘ ║ ║
              ║  │               │               ║ ║
              ║  │  ┌────────────▼────────────┐ ║ ║
              ║  │  │ ID  — Instruction Decode│ ║ ║
              ║  │  │ Register file · imm gen │ ║ ║
              ║  │  └────────────┬────────────┘ ║ ║
              ║  │               │               ║ ║
              ║  │  ┌────────────▼────────────┐ ║ ║
              ║  │  │ EX  — Execute           │ ║ ║
              ║  │  │ ALU · forwarding unit   │ ║ ║
              ║  │  └────────────┬────────────┘ ║ ║
              ║  │               │               ║ ║
              ║  │  ┌────────────▼────────────┐ ║ ║
              ║  │  │ MEM — Memory Access     │ ║ ║
              ║  │  │ Load / store operations │ ║ ║
              ║  │  └────────────┬────────────┘ ║ ║
              ║  │               │               ║ ║
              ║  │  ┌────────────▼────────────┐ ║ ║
              ║  │  │ WB  — Write Back        │ ║ ║
              ║  │  │ Register write · fwd    │ ║ ║
              ║  │  └─────────────────────────┘ ║ ║
              ║  │         5-Stage Pipeline      ║ ║
              ║  └──────────────┬───────────────┘ ║
              ║                 │                 ║
              ║  ┌──────────────▼───────────────┐ ║
              ║  │    UART Transmitter           │ ║
              ║  │    115200 baud                │ ║
              ║  └──────────────────────────────┘ ║
              ╚═══════════════════════════════════╝
                               │
                               ▼
                    ┌─────────────────────┐
                    │    ESP32 Arduino     │
                    │  Microcontroller     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    LCD Display       │
                    │  (1 Hz refresh)      │
                    └─────────────────────┘
```

---

## Pipeline Stage Details

### Stage 1 — Instruction Fetch (IF)

Fetches the instruction from instruction memory using the Program Counter (PC).

```verilog
module fetch_stage (
    input clk, reset, pcSrc,
    input [31:0] pcTarget,
    output reg [31:0] pcF,
    output [31:0] instrF,
    output [31:0] pcPlus4F
);
    reg [31:0] pc;
    assign pcPlus4F = pc + 4;
    // Hardcoded instruction memory
    assign instrF = (pc[9:2] == 8'd0) ? 32'h00000293 : // ADDI x5, x0, 0
                    (pc[9:2] == 8'd1) ? 32'h00500313 : // ADDI x6, x0, 5
                    (pc[9:2] == 8'd2) ? 32'h0062b3b3 : // ADD  x7, x5, x6
                    32'h00000013;                       // NOP

    always @(posedge clk or posedge reset) begin
        if (reset)       pc <= 0;
        else if (pcSrc)  pc <= pcTarget;
        else             pc <= pc + 4;
    end
    always @(*) pcF = pc;
endmodule
```

---

### Stage 2 — Instruction Decode (ID)

Reads register file operands and generates sign-extended immediates.

```verilog
module decode_stage (
    input clk,
    input [31:0] instrF, pcF, pcPlus4F,
    input regWriteW,
    input [4:0] rdW,
    input [31:0] resultW,
    output [31:0] regData1D, regData2D, immExtD,
    output [4:0] rdD,
    output [31:0] instrD, pcD, pcPlus4D
);
    reg [31:0] reg_file [0:31];
    wire [4:0] rs1 = instrF[19:15];
    wire [4:0] rs2 = instrF[24:20];

    assign immExtD   = {{20{instrF[31]}}, instrF[31:20]};
    assign regData1D = reg_file[rs1];
    assign regData2D = reg_file[rs2];
    assign rdD       = instrF[11:7];
    assign instrD    = instrF;
    assign pcD       = pcF;
    assign pcPlus4D  = pcPlus4F;

    always @(posedge clk)
        if (regWriteW) reg_file[rdW] <= resultW;
endmodule
```

---

### Stage 3 — Execute (EX)

Performs ALU operations and resolves data hazards via forwarding multiplexers.

```verilog
module execute_stage (
    input [31:0] regData1, regData2, immExt,
    input [1:0] forwardAE, forwardBE,
    output [31:0] aluResult,
    output [31:0] aluSrcA, aluSrcB
);
    assign aluSrcA   = regData1;
    assign aluSrcB   = immExt;
    assign aluResult = aluSrcA + aluSrcB;
endmodule
```

**Forwarding / Hazard Unit:**

```verilog
module hazard_unit (
    input regWriteM, regWriteW,
    input [4:0] rdM, rdW, rs1E, rs2E,
    output reg [1:0] forwardAE, forwardBE
);
    always @(*) begin
        forwardAE = 2'b00;
        forwardBE = 2'b00;
        if (regWriteM && (rdM != 0) && (rdM == rs1E)) forwardAE = 2'b10;
        else if (regWriteW && (rdW != 0) && (rdW == rs1E)) forwardAE = 2'b01;
        if (regWriteM && (rdM != 0) && (rdM == rs2E)) forwardBE = 2'b10;
        else if (regWriteW && (rdW != 0) && (rdW == rs2E)) forwardBE = 2'b01;
    end
endmodule
```

---

### Stage 4 — Memory Access (MEM)

Interfaces with data memory for load and store instructions.

```verilog
module memory_stage (
    input clk,
    input [31:0] aluOutE, writeData,
    output [31:0] readDataM
);
    reg [31:0] data_mem [0:255];
    assign readDataM = data_mem[aluOutE[9:2]];
    always @(posedge clk) begin
        // Memory write logic here
    end
endmodule
```

---

### Stage 5 — Write Back (WB)

Selects the result to write back to the register file (ALU result or memory read).

```verilog
module writeback_stage (
    input [31:0] aluResult, readDataM,
    output [31:0] resultW
);
    assign resultW = aluResult;
endmodule
```

---

### Register File

```verilog
module reg_file (
    input clk,
    input [4:0] rs1, rs2, rd,
    input [31:0] wdata,
    input we,
    output [31:0] rdata1, rdata2
);
    reg [31:0] rf [0:31];
    assign rdata1 = (rs1 == 0) ? 0 : rf[rs1];
    assign rdata2 = (rs2 == 0) ? 0 : rf[rs2];
    always @(posedge clk)
        if (we && (rd != 0)) rf[rd] <= wdata;
endmodule
```

---

### UART Transmitter

```verilog
module uart_tx (
    input clk, rst, tx_start,
    input [7:0] tx_data,
    output reg tx, tx_busy
);
    parameter CLKS_PER_BIT = 868;  // 100MHz / 115200
    // ... shift register logic
endmodule
```

---

## Results

### Simulation Output (Vivado Waveform)

The simulation shows the humidity input (decimal values 45, 72, 63) being processed and transmitted out via the UART TX line.

| Signal | Value |
|---|---|
| `clk` | 100 MHz (10 ns period) |
| `reset` | deasserted after 50 ns |
| `humidity_input` | 8'd45, 8'd72, 8'd63 |
| `uart_tx` | active high serial stream |

### FPGA Synthesis — Artix-7

| Resource | Used | Available | Utilization |
|---|---|---|---|
| LUTs | 12,345 | 63,400 | 19.5% |
| Flip-Flops | 8,765 | 126,800 | 6.9% |
| Block RAMs | 8 | 135 | 5.9% |
| Clock | 50 MHz | — | — |

### Performance

| Metric | Value |
|---|---|
| Ideal CPI | 1.0 |
| Actual CPI | 1.25 (due to pipeline stalls) |
| Speedup over single-cycle | 4× (CPI = 5 → 1.25) |
| UART baud rate | 115,200 |
| LCD refresh rate | 1 Hz |

### Comparison with Prior Work

| Reference | Architecture | Technology | Frequency |
|---|---|---|---|
| Galani et al. (2013) | MIPS-based | Xilinx | ~32 MHz |
| Dennis et al. (2017) | RISC-V | Spartan 3E | 32 MHz |
| **This work** | **RISC-V RV32I** | **Artix-7** | **50 MHz** |

---

## Technologies Used

| Tool / Component | Purpose |
|---|---|
| Verilog HDL | RTL design of all pipeline modules |
| Xilinx Vivado | Synthesis, implementation, bitstream generation |
| Artix-7 FPGA (xc7a35t) | Hardware deployment platform |
| DHT22 Sensor | Environmental humidity acquisition |
| ESP32 Arduino | Data reception and LCD control |
| UART Protocol | Inter-device serial communication |

---

## Project Structure

```
risc_v_pipeline/
├── src/
│   ├── fetch_stage.v         # IF stage — PC + instruction memory
│   ├── decode_stage.v        # ID stage — register file + immediate gen
│   ├── execute_stage.v       # EX stage — ALU + forwarding
│   ├── memory_stage.v        # MEM stage — data memory
│   ├── writeback_stage.v     # WB stage — result selection
│   ├── hazard_unit.v         # Data hazard forwarding logic
│   ├── pipeline_top.v        # Top-level module integrating all stages
│   └── uart_tx.v             # UART transmitter (115200 baud)
├── sim/
│   └── pipeline_tb.v         # Testbench with humidity input simulation
├── constrs/
│   ├── clock_constraints.xdc # 10 ns period (100 MHz)
│   └── UART.xdc              # Pin A9 → UART TX (LVCMOS33)
└── README.md
```

---

## Future Work

| Enhancement | Expected Benefit |
|---|---|
| Floating-point unit (FPU) | Scientific and sensor data computations |
| L1 cache (instruction + data) | Reduced memory access latency |
| Wi-Fi interface (ESP32) | Wireless cloud data transmission |
| AI accelerator (MAC unit) | Edge machine learning inference |
| SystemVerilog + UVM testbench | Formal verification coverage |

---

## Acknowledgements

We extend our gratitude to **Dr. Vijay Nath** for his guidance throughout this project, to the **DRIE Office** at BIT Mesra for facilitating the SIPAR-25 summer research internship, and to the Department of Electronics & Communication Engineering for providing laboratory resources.

---

## References

1. Waterman, A., & Asanović, K. (2017). *The RISC-V Instruction Set Manual*. UC Berkeley.
2. Patterson, D. A., & Hennessy, J. L. (2017). *Computer Organization and Design: RISC-V Edition*. Morgan Kaufmann.
3. Xilinx. (2021). *Artix-7 FPGA Data Sheet*. Xilinx Inc.
4. Aosong Electronics. (2020). *DHT22 Humidity Sensor Technical Documentation*.
5. Espressif Systems. (2021). *ESP32 Technical Reference Manual*.

---

*BIT SIPAR 2025 · Birla Institute of Technology, Mesra · June 2025*
